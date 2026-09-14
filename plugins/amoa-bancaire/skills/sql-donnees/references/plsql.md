# PL/SQL

## Quand écrire du PL/SQL, et quand ne pas en écrire

Le SQL ensembliste bat presque toujours une boucle PL/SQL. Une boucle qui traite 500 000 lignes une par une fait 500 000 allers-retours entre le moteur PL/SQL et le moteur SQL. La même opération en un `MERGE` unique s'exécute souvent cent fois plus vite.

PL/SQL se justifie quand il y a : de la logique conditionnelle non exprimable en SQL, un besoin de reprise ligne par ligne avec journalisation des rejets, un appel externe, ou un traitement par lots avec points de reprise.

**Règle** : écris d'abord le SQL ensembliste. Ne passe en PL/SQL que si tu peux dire précisément pourquoi il est nécessaire.

## Structure d'un package

Le package est l'unité de livraison. Une procédure isolée dans le schéma est un défaut de conception : pas de regroupement logique, pas d'état partagé, invalidation en cascade au moindre changement.

```sql
CREATE OR REPLACE PACKAGE pkg_reconciliation AS

  -- Constantes de statut, jamais de littéral en dur dans le corps
  c_statut_rapproche  CONSTANT VARCHAR2(20) := 'RAPPROCHE';
  c_statut_suspens    CONSTANT VARCHAR2(20) := 'SUSPENS';

  -- Exception métier nommée, avec son code
  e_periode_non_clos  EXCEPTION;
  PRAGMA EXCEPTION_INIT(e_periode_non_clos, -20101);

  PROCEDURE rapprocher_periode (
    p_date_debut  IN  DATE,
    p_date_fin    IN  DATE,
    p_nb_traites  OUT NUMBER,
    p_nb_rejetes  OUT NUMBER
  );

END pkg_reconciliation;
/
```

Exposer `p_nb_traites` et `p_nb_rejetes` en sortie n'est pas cosmétique : sans compteurs, l'ordonnanceur ne peut pas distinguer un batch qui a tout traité d'un batch qui n'a rien trouvé.

## Traitement de masse : `BULK COLLECT` et `FORALL`

```sql
PROCEDURE rapprocher_periode (
  p_date_debut IN DATE, p_date_fin IN DATE,
  p_nb_traites OUT NUMBER, p_nb_rejetes OUT NUMBER
) IS
  c_taille_lot CONSTANT PLS_INTEGER := 5000;

  CURSOR cur_ecarts IS
    SELECT identifiant_technique, reference_transaction, montant
    FROM   suspens
    WHERE  date_operation >= TRUNC(p_date_debut)
    AND    date_operation <  TRUNC(p_date_fin) + 1
    AND    statut = pkg_reconciliation.c_statut_suspens;

  TYPE t_lot IS TABLE OF cur_ecarts%ROWTYPE;
  l_lot t_lot;
BEGIN
  p_nb_traites := 0;
  p_nb_rejetes := 0;

  OPEN cur_ecarts;
  LOOP
    FETCH cur_ecarts BULK COLLECT INTO l_lot LIMIT c_taille_lot;
    EXIT WHEN l_lot.COUNT = 0;

    BEGIN
      FORALL i IN 1 .. l_lot.COUNT SAVE EXCEPTIONS
        UPDATE suspens
        SET    statut       = pkg_reconciliation.c_statut_rapproche,
               date_maj     = SYSDATE
        WHERE  identifiant_technique = l_lot(i).identifiant_technique;

      p_nb_traites := p_nb_traites + l_lot.COUNT;

    EXCEPTION
      WHEN OTHERS THEN
        -- SAVE EXCEPTIONS : les lignes valides sont passées, on journalise les autres
        FOR j IN 1 .. SQL%BULK_EXCEPTIONS.COUNT LOOP
          pkg_journal.tracer_rejet(
            p_identifiant => l_lot(SQL%BULK_EXCEPTIONS(j).ERROR_INDEX).identifiant_technique,
            p_message     => SQLERRM(-SQL%BULK_EXCEPTIONS(j).ERROR_CODE)
          );
        END LOOP;
        p_nb_rejetes := p_nb_rejetes + SQL%BULK_EXCEPTIONS.COUNT;
        p_nb_traites := p_nb_traites + l_lot.COUNT - SQL%BULK_EXCEPTIONS.COUNT;
    END;

    COMMIT;   -- point de reprise à chaque lot
  END LOOP;
  CLOSE cur_ecarts;

EXCEPTION
  WHEN OTHERS THEN
    IF cur_ecarts%ISOPEN THEN CLOSE cur_ecarts; END IF;
    pkg_journal.tracer_erreur(SQLCODE, SQLERRM, DBMS_UTILITY.FORMAT_ERROR_BACKTRACE);
    RAISE;
END rapprocher_periode;
```

Trois points structurants dans ce code :

- **`LIMIT`** borne la mémoire consommée. Un `BULK COLLECT` sans `LIMIT` sur une table de plusieurs millions de lignes sature la PGA et fait tomber la session.
- **`SAVE EXCEPTIONS`** laisse passer les lignes valides et collecte les rejets. Sans lui, une seule ligne en erreur annule le lot entier — inacceptable sur un batch de nuit.
- **Le `COMMIT` par lot** crée des points de reprise. Il implique que le traitement doit être **rejouable** : un batch relancé après incident ne doit ni doubler ni sauter de lignes. C'est une exigence à écrire dans la spécification, pas une propriété qui s'obtient toute seule.

## Gestion des erreurs

```sql
EXCEPTION
  WHEN NO_DATA_FOUND THEN
    -- Cas métier connu : traité, jamais silencieux
    pkg_journal.tracer_information('Aucune ligne pour la période');
  WHEN OTHERS THEN
    pkg_journal.tracer_erreur(SQLCODE, SQLERRM, DBMS_UTILITY.FORMAT_ERROR_BACKTRACE);
    RAISE;   -- toujours relancer : avaler une exception masque l'incident
```

| À bannir | Pourquoi |
|---|---|
| `WHEN OTHERS THEN NULL;` | Le batch se termine « en succès » alors qu'il a échoué. Défaut le plus grave du PL/SQL. |
| `WHEN OTHERS` sans `RAISE` | Même effet, l'ordonnanceur ne voit rien |
| `SQLERRM` sans `FORMAT_ERROR_BACKTRACE` | On sait quelle erreur, pas à quelle ligne |
| Message d'erreur sans identifiant métier | Impossible de retrouver la ligne concernée |

`RAISE_APPLICATION_ERROR` s'utilise pour les erreurs métier, avec un code dans la plage `-20000` à `-20999` et un message exploitable par l'exploitant, pas par le développeur.

## Journalisation en transaction autonome

Une trace écrite dans la même transaction disparaît au `ROLLBACK` — c'est-à-dire exactement quand elle est utile.

```sql
PROCEDURE tracer_erreur (p_code IN NUMBER, p_message IN VARCHAR2, p_pile IN VARCHAR2) IS
  PRAGMA AUTONOMOUS_TRANSACTION;
BEGIN
  INSERT INTO journal_batch (date_evenement, code_erreur, message, pile_appel)
  VALUES (SYSTIMESTAMP, p_code, SUBSTR(p_message,1,4000), SUBSTR(p_pile,1,4000));
  COMMIT;   -- obligatoire dans une transaction autonome
END;
```

## Typage et conventions

| Sujet | Règle |
|---|---|
| Types de colonnes | `%TYPE` et `%ROWTYPE` systématiquement — le code suit les évolutions du modèle |
| Montants | `NUMBER(p,s)` avec précision explicite. Jamais `BINARY_DOUBLE` ni `FLOAT` |
| Compteurs de boucle | `PLS_INTEGER`, plus rapide que `NUMBER` |
| Variables | Préfixes explicites : `p_` paramètre, `l_` locale, `c_` constante, `g_` globale package |
| Littéraux | Aucun en dur dans le corps : constantes de package ou table de paramétrage |
| SQL dynamique | `EXECUTE IMMEDIATE` avec `USING` et variables bind, jamais par concaténation — risque d'injection et saturation du cache de curseurs |

## Ce qu'un livrable AMOA doit exiger d'un batch PL/SQL

Quand tu spécifies un traitement batch, ces six points sont des exigences, pas des détails d'implémentation :

1. **Rejouabilité** — relancé après incident, le batch ne double ni ne saute.
2. **Points de reprise** — taille de lot et fréquence de commit définies.
3. **Compteurs en sortie** — lignes lues, traitées, rejetées.
4. **Journal des rejets** — chaque ligne rejetée avec son identifiant métier et la cause.
5. **Fenêtre d'exécution** — durée maximale acceptable et comportement en cas de dépassement.
6. **Ordre des dépendances** — ce qui doit être terminé avant, ce qui attend derrière.
