# Modèle de données, indexation et performance

## Modélisation : ce qui se décide une fois et se paie longtemps

| Décision | Bon choix | Conséquence d'un mauvais choix |
|---|---|---|
| Clé primaire | Clé technique (séquence) + clé fonctionnelle en contrainte unique | Une clé métier en PK se propage partout et devient immodifiable |
| Montants | `NUMBER(p,s)` avec précision explicite | Écarts de centimes en réconciliation, irrécupérables |
| Devise | Colonne obligatoire à côté de chaque montant | Un montant sans devise est ininterprétable |
| Dates | Distinguer date d'opération, date de valeur, date comptable, date d'intégration | Toute l'arithmétique de réconciliation devient fausse |
| Historisation | Bornes ouvertes à droite : `date_debut <= d < date_fin` | Doublons à la jonction des versions |
| Codes de référence | Table de référence, pas de littéral | Impossible de faire évoluer un code sans toucher au code source |
| Suppression | Marquage logique sur les données réglementées | La conservation légale interdit le `DELETE` physique |

**Les quatre dates** sont le point le plus sous-estimé. Une transaction bancaire porte au minimum : la date à laquelle l'opération a eu lieu, la date à laquelle elle produit ses effets sur le solde, la date d'arrêté comptable de rattachement, et la date d'intégration dans le système. Le modèle doit les porter toutes, et la spécification doit dire laquelle fait foi pour chaque usage.

## Normalisation

Normalise jusqu'à la 3NF pour les données transactionnelles et les référentiels. Dénormalise seulement sur preuve : un plan d'exécution qui montre le problème, une volumétrie mesurée, un besoin de restitution avéré. Une dénormalisation décidée « pour la performance » sans mesure crée une incohérence de données garantie.

Les tables de restitution et les agrégats précalculés sont une exception légitime, à condition que leur mode de rafraîchissement et leur fraîcheur acceptable soient spécifiés.

## Indexation

### Ce qui mérite un index

| Cible | Raison |
|---|---|
| Clés étrangères | Sans index, un `DELETE` sur la table parente pose un verrou de table sur la fille |
| Colonnes de filtre fréquentes et sélectives | Le cas nominal |
| Colonnes de jointure | Évite le balayage complet |
| Colonnes de tri quand le tri est coûteux | L'index fournit l'ordre |

### Ce qui n'en mérite pas

Une colonne à faible cardinalité (deux ou trois valeurs distinctes), une table petite et entièrement en cache, une colonne jamais utilisée en filtre. Chaque index ralentit les écritures et occupe de l'espace : un index inutile est un coût permanent pour un gain nul.

### L'ordre des colonnes d'un index composite

C'est la règle la plus mal comprise. Un index sur `(A, B, C)` sert les filtres sur `A`, sur `(A,B)` et sur `(A,B,C)` — mais pas un filtre sur `B` seul.

Ordre recommandé : **d'abord les colonnes en égalité, ensuite les colonnes en intervalle**. Pour une requête filtrant `numero_compte = :x AND date_operation BETWEEN :d1 AND :d2`, l'index utile est `(numero_compte, date_operation)`, pas l'inverse.

### Index fonctionnel

Un filtre sur `UPPER(nom)` ou `TRUNC(date_operation)` n'utilise pas l'index de la colonne brute. Deux solutions : créer un index fonctionnel sur l'expression, ou réécrire le filtre pour qu'il porte sur la colonne nue.

```sql
-- Ne pas faire : la fonction empêche l'usage de l'index
WHERE TRUNC(date_operation) = TRUNC(:date_ref)

-- Faire : la colonne reste nue, l'index sert
WHERE date_operation >= TRUNC(:date_ref)
AND   date_operation <  TRUNC(:date_ref) + 1
```

### Le tueur silencieux : la conversion implicite

Comparer une colonne `VARCHAR2` à une valeur numérique force le moteur à convertir la colonne ligne par ligne. L'index devient inutilisable et la requête passe en balayage complet, sans aucun message d'erreur. C'est la première chose à vérifier quand une requête indexée est lente.

## Lire un plan d'exécution

```sql
EXPLAIN PLAN FOR
SELECT ... ;

SELECT * FROM TABLE(DBMS_XPLAN.DISPLAY(FORMAT => 'ALLSTATS LAST +PREDICATE'));
```

Pour le plan réellement exécuté, avec les volumes constatés :

```sql
SELECT * FROM TABLE(DBMS_XPLAN.DISPLAY_CURSOR(NULL, NULL, 'ALLSTATS LAST'));
```

| Ce que tu regardes | Signification |
|---|---|
| Écart entre `E-Rows` et `A-Rows` | Estimation fausse — statistiques obsolètes ou prédicat mal estimé. C'est la cause racine la plus fréquente d'un mauvais plan. |
| `TABLE ACCESS FULL` sur une grosse table | Attendu pour un agrégat global, suspect pour une recherche ciblée |
| `NESTED LOOPS` sur un gros volume | Efficace sur peu de lignes, catastrophique sur des millions |
| `HASH JOIN` | Bon choix sur gros volumes, à condition que la mémoire suffise |
| Opérations `TEMP` | Le tri ou le hachage déborde sur disque |
| `FILTER` avec sous-requête corrélée | Exécutée pour chaque ligne — souvent réécrivable en jointure |

Avant d'optimiser quoi que ce soit : vérifier que les statistiques sont à jour. Un plan dégradé après une montée de charge vient neuf fois sur dix de statistiques périmées, pas d'un index manquant.

## Partitionnement

Sur les tables de transactions, le partitionnement par intervalle de date est presque toujours le bon choix :

```sql
PARTITION BY RANGE (date_operation)
INTERVAL (NUMTOYMINTERVAL(1,'MONTH'))
( PARTITION p_initiale VALUES LESS THAN (DATE '2024-01-01') );
```

Trois bénéfices : élagage automatique des partitions sur les requêtes bornées par date, purge d'un mois entier par `DROP PARTITION` au lieu d'un `DELETE` massif, et statistiques par partition donc plus justes.

**Condition** : les requêtes doivent filtrer sur la clé de partitionnement. Un partitionnement par date que personne n'exploite en filtre ajoute de la complexité sans gain.

## Migrations et reprise de données

Une migration se livre avec quatre scripts, pas un :

| Script | Rôle |
|---|---|
| `up` | Applique le changement |
| `down` | Revient en arrière — obligatoire, même si on espère ne jamais l'utiliser |
| `verify` | Contrôle que l'état final est conforme : comptages, sommes, contraintes |
| `estimate` | Durée et volume attendus, mesurés sur une copie de production |

Règles de sûreté :

- Chaque script est **idempotent** : rejoué, il ne casse rien.
- Un ajout de colonne se fait `NULL` d'abord, puis alimentation par lots, puis passage en `NOT NULL`. Ajouter directement une colonne `NOT NULL` avec valeur par défaut peut verrouiller la table pendant des minutes sur certaines versions.
- Un `ALTER TABLE` sur une table de production nécessite une fenêtre validée et une estimation de durée.
- Les contraintes désactivées pendant la reprise doivent être **réactivées avec validation** ensuite, et le résultat de la validation contrôlé. Une contrainte réactivée en `NOVALIDATE` laisse passer des données incohérentes.

Toute reprise de données se livre avec son **contrôle de bouclage** : nombre de lignes source, nombre de lignes cible, somme des montants des deux côtés, liste des rejets avec leur cause. Sans ces quatre chiffres, la reprise n'est pas recettable.

## Sécurité

| Risque | Parade |
|---|---|
| Injection SQL | Variables bind systématiques. Jamais de concaténation de paramètre dans du SQL dynamique |
| Données sensibles en environnement de test | Anonymisation à la source, avant extraction |
| Droits trop larges | Accès en lecture seule par défaut ; les droits d'écriture sur production sont nominatifs et tracés |
| Absence de traçabilité | Journalisation des accès aux données sensibles — souvent une obligation réglementaire, pas une option |
| Extraction non bornée | Toute extraction de production est bornée par une période et un volume maximal |
