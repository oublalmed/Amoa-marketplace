# Patterns de requêtes bancaires

Syntaxe Oracle par défaut. Les écarts PostgreSQL et SQL Server sont signalés.

## 1. Anti-jointure — présent d'un côté, absent de l'autre

Le pattern le plus utile en réconciliation. Répond à « quelles transactions autorisées n'ont jamais été compensées ».

```sql
-- Transactions autorisées sans ligne de compensation correspondante
SELECT  a.reference_transaction,
        a.date_operation,
        a.montant,
        a.devise,
        a.identifiant_terminal
FROM    autorisation a
WHERE   a.date_operation >= TRUNC(:date_debut)
AND     a.date_operation <  TRUNC(:date_fin) + 1
AND     a.code_reponse = '00'
AND NOT EXISTS (
          SELECT 1
          FROM   compensation c
          WHERE  c.reference_transaction = a.reference_transaction
          AND    c.date_compensation >= TRUNC(:date_debut)
        );
```

**Pourquoi `NOT EXISTS` et pas `NOT IN`** : si la sous-requête renvoie un seul `NULL`, `NOT IN` retourne zéro ligne sans aucune erreur. C'est la première cause de réconciliation qui « ne trouve rien » alors que les écarts existent.

**Pourquoi borner la sous-requête aussi** : sans la borne de date sur `compensation`, le moteur balaye toute l'historique à chaque ligne.

## 2. Réconciliation bilatérale — les écarts des deux côtés en une passe

```sql
-- Écarts entre deux référentiels, dans les deux sens, avec qualification
SELECT  COALESCE(a.reference_transaction, c.reference_transaction) AS reference,
        a.montant        AS montant_autorisation,
        c.montant        AS montant_compensation,
        CASE
          WHEN c.reference_transaction IS NULL THEN 'AUTORISE_NON_COMPENSE'
          WHEN a.reference_transaction IS NULL THEN 'COMPENSE_NON_AUTORISE'
          WHEN a.montant <> c.montant          THEN 'ECART_MONTANT'
          WHEN a.devise  <> c.devise           THEN 'ECART_DEVISE'
        END AS type_ecart
FROM        autorisation a
FULL OUTER JOIN compensation c
        ON  c.reference_transaction = a.reference_transaction
WHERE   COALESCE(a.date_operation, c.date_compensation) >= TRUNC(:date_debut)
AND     COALESCE(a.date_operation, c.date_compensation) <  TRUNC(:date_fin) + 1
AND    (c.reference_transaction IS NULL
     OR a.reference_transaction IS NULL
     OR a.montant <> c.montant
     OR a.devise  <> c.devise);
```

`FULL OUTER JOIN` évite d'écrire deux requêtes et de les unir. Le `CASE` qualifie l'écart, ce qui rend le résultat directement exploitable par le back office : chaque type de suspens a un traitement différent.

**SQL Server** : identique. **PostgreSQL** : remplacer `TRUNC(d)` par `date_trunc('day', d)`.

## 3. Écart de montant avec tolérance

Indispensable dès qu'un ajustement légitime existe (pourboire, carburant, hôtellerie).

```sql
SELECT  a.reference_transaction,
        a.montant AS montant_autorise,
        c.montant AS montant_capture,
        c.montant - a.montant                                      AS ecart_absolu,
        ROUND((c.montant - a.montant) / NULLIF(a.montant,0) * 100, 2) AS ecart_pct
FROM    autorisation a
JOIN    capture      c ON c.reference_transaction = a.reference_transaction
WHERE   a.date_operation >= TRUNC(:date_debut)
AND     a.date_operation <  TRUNC(:date_fin) + 1
AND    (ABS(c.montant - a.montant) > :tolerance_absolue
    OR  ABS(c.montant - a.montant) / NULLIF(a.montant,0) > :tolerance_pct);
```

`NULLIF(a.montant,0)` évite la division par zéro sur une transaction à montant nul, qui existe plus souvent qu'on ne croit (annulation, contrôle de carte).

**Règle de conception** : la tolérance se définit toujours en valeur absolue **et** en pourcentage, avec un `OR`. Un seuil en pourcentage seul laisse passer les gros montants ; un seuil absolu seul bloque les petits.

## 4. Détection de doublons

```sql
-- Doublons sur la clé fonctionnelle, avec le détail des lignes concernées
SELECT  reference_transaction,
        date_operation,
        montant,
        COUNT(*)                    AS nb_occurrences,
        MIN(identifiant_technique)  AS premier_id,
        MAX(identifiant_technique)  AS dernier_id
FROM    compensation
WHERE   date_compensation >= TRUNC(:date_debut)
AND     date_compensation <  TRUNC(:date_fin) + 1
GROUP BY reference_transaction, date_operation, montant
HAVING  COUNT(*) > 1
ORDER BY nb_occurrences DESC, montant DESC;
```

Pour ne garder qu'une occurrence et identifier les autres :

```sql
SELECT  identifiant_technique,
        reference_transaction,
        rang
FROM   (SELECT  identifiant_technique,
                reference_transaction,
                ROW_NUMBER() OVER (
                  PARTITION BY reference_transaction, montant
                  ORDER BY     date_integration ASC
                ) AS rang
        FROM    compensation
        WHERE   date_compensation >= TRUNC(:date_debut))
WHERE   rang > 1;
```

`ROW_NUMBER()` avec un `ORDER BY` déterministe désigne sans ambiguïté la ligne à conserver. Sans `ORDER BY` stable, deux exécutions peuvent désigner des lignes différentes — défaut classique d'un script de dédoublonnage.

## 5. Solde progressif et contrôle de continuité

```sql
-- Solde recalculé à partir des mouvements, comparé au solde stocké
SELECT  m.numero_compte,
        m.date_valeur,
        m.montant,
        SUM(m.montant) OVER (
          PARTITION BY m.numero_compte
          ORDER BY     m.date_valeur, m.identifiant_technique
          ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
        ) AS solde_recalcule,
        m.solde_apres_operation AS solde_stocke
FROM    mouvement m
WHERE   m.numero_compte = :compte
AND     m.date_valeur  >= TRUNC(:date_debut)
ORDER BY m.date_valeur, m.identifiant_technique;
```

Le `ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW` est explicite volontairement : le comportement par défaut d'une fenêtre avec `ORDER BY` est `RANGE`, qui agrège toutes les lignes de même valeur de tri. Sur des mouvements de même date, cela fausse le solde progressif.

## 6. Détection de trous dans une séquence

Utile pour repérer un fichier de compensation manquant ou un numéro de remise absent.

```sql
SELECT  numero_remise                                        AS remise_precedente,
        LEAD(numero_remise) OVER (ORDER BY numero_remise)    AS remise_suivante,
        LEAD(numero_remise) OVER (ORDER BY numero_remise)
          - numero_remise - 1                                AS nb_manquants
FROM    remise
WHERE   date_reception >= TRUNC(:date_debut)
QUALIFY LEAD(numero_remise) OVER (ORDER BY numero_remise) - numero_remise > 1;
```

`QUALIFY` n'existe pas sur Oracle avant la version 23. Version portable : envelopper dans une sous-requête et filtrer dans le `WHERE` extérieur.

## 7. Historisation — l'état à une date donnée

Sur une table historisée en SCD type 2 (avec `date_debut_validite` / `date_fin_validite`) :

```sql
SELECT  identifiant_client,
        segment,
        statut_kyc
FROM    client_historique
WHERE   :date_observation >= date_debut_validite
AND     :date_observation <  NVL(date_fin_validite, DATE '9999-12-31');
```

**Le piège** : utiliser `<=` sur `date_fin_validite` au lieu de `<`. Si la fin d'une version et le début de la suivante portent la même date, la requête retourne deux lignes pour un même client. Fixer la convention (bornes ouvertes à droite) dès la spécification du modèle.

## 8. Jeu de recette — échantillon représentatif

```sql
-- Un cas par combinaison de critères discriminants, pour un plan de recette
SELECT  identifiant_technique, canal, type_carte, pays_accepteur, code_reponse
FROM   (SELECT  t.*,
                ROW_NUMBER() OVER (
                  PARTITION BY canal, type_carte, pays_accepteur, code_reponse
                  ORDER BY     DBMS_RANDOM.VALUE
                ) AS rang
        FROM    transaction t
        WHERE   t.date_operation >= TRUNC(SYSDATE) - 30)
WHERE   rang <= :nb_cas_par_combinaison;
```

Un échantillon aléatoire pur rate les cas rares — et ce sont précisément ceux qui cassent en recette. Le `PARTITION BY` sur les critères discriminants garantit au moins un cas par combinaison.

**Avant toute extraction destinée à un jeu de recette** : masquer les données sensibles à la source.

```sql
SELECT  SUBSTR(pan,1,6) || '******' || SUBSTR(pan,-4) AS pan_masque,
        NULL                                          AS cvv,
        'CLIENT_' || identifiant_technique            AS nom_anonymise
FROM    porteur;
```

## 9. Contrôle de bouclage

À joindre à toute extraction livrée. Sans lui, aucun destinataire ne peut vérifier que le chiffre est complet.

```sql
SELECT  COUNT(*)                                   AS nb_lignes,
        SUM(montant)                               AS total_montant,
        MIN(date_operation)                        AS premiere_operation,
        MAX(date_operation)                        AS derniere_operation,
        COUNT(DISTINCT devise)                     AS nb_devises,
        SUM(CASE WHEN montant IS NULL THEN 1 ELSE 0 END) AS nb_montants_nuls
FROM    transaction
WHERE   date_operation >= TRUNC(:date_debut)
AND     date_operation <  TRUNC(:date_fin) + 1;
```

Le comptage des montants `NULL` révèle les lignes qui disparaîtront silencieusement de toute somme.
