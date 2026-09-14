# Checklist qualité — à passer avant de rendre

## 1. Traçabilité

Chaque exigence porte un identifiant et se relie en chaîne :

```
BES-01 → EF-003 → RG-007 → §4.2 spéc → CT-014, CT-015
```

Si un maillon manque, le signaler dans le livrable plutôt que le laisser implicite.

## 2. Couverture de test

Pour chaque règle de gestion, trois cas minimum :

| Type | Question à se poser |
|---|---|
| Nominal | Que se passe-t-il quand tout est conforme ? |
| Limite | Montant nul, plafond, date de fin de mois, jour non ouvré, devise exotique, champ à la longueur max |
| Erreur | Rejet, timeout, interface indisponible, double soumission, rejeu après incident |

Pour les processus réglementés, ajouter un cas **piste d'audit** : l'opération est-elle tracée de façon opposable ?

## 3. Risques

| Risque | Criticité | Probabilité | Impact | Propriétaire | Mitigation |
|---|---|---|---|---|---|

Un risque sans propriétaire nommé n'est pas un risque géré. Un risque sans mitigation est un constat, pas un risque.

## 4. Impacts sur 4 axes

| Axe | Ce qu'il faut couvrir |
|---|---|
| **SI** | Modules impactés, interfaces, batchs, paramétrage vs développement spécifique |
| **Données** | Nouveaux champs, reprise de l'existant, qualité, rétention, RGPD |
| **Organisation** | Rôles modifiés, charge, formation, procédures à réécrire |
| **Réglementaire** | Texte applicable, contrôle exigé, reporting, auditabilité |

## 5. KPI

Tout KPI proposé doit être renseigné sur 4 colonnes :

| KPI | Baseline | Cible | Mode de mesure | Fréquence |
|---|---|---|---|---|

Un KPI sans baseline est invérifiable. Si la baseline n'est pas connue, l'indiquer comme action préalable, pas comme détail.

## 6. Chasse aux formules creuses

Ces tournures signalent une exigence non spécifiée. Les remplacer systématiquement.

| À bannir | À écrire à la place |
|---|---|
| « Il faudra veiller à la performance » | « Temps de réponse < 2 s au 95e centile, pour 10 000 opérations/jour » |
| « Une attention particulière sera portée à la sécurité » | « Double validation obligatoire au-delà de 50 000 EUR (RG-012) » |
| « Le système devra être robuste » | « Rejeu automatique de 3 tentatives à 5 min d'intervalle, puis alerte N2 » |
| « Les utilisateurs seront formés » | « 2 sessions de 3 h, 40 agents back office, semaine 12, support rédigé par la MOA » |
| « Prévoir une reprise de données » | « Reprise de 180 000 lignes de l'historique 2024-2026, contrôle de cohérence par échantillon de 5 % » |

## 7. Distinction constat / décision / hypothèse

| Nature | Formulation | Où ça va |
|---|---|---|
| Constat | Indicatif présent, factuel, sourcé | AS-IS |
| Décision | Passé, sans conditionnel, avec date et instance | Décisions |
| Hypothèse | Marquée `[HYPOTHÈSE — à confirmer]` | À confirmer avec le métier |

Ne jamais présenter une hypothèse au même niveau typographique qu'un fait établi.

## 8. Clôture obligatoire

Tout livrable se termine par ces deux sections, sans exception :

**Prochaines étapes**

| # | Étape | Responsable | Échéance |
|---|---|---|---|

**À confirmer avec le métier**

| # | Hypothèse | Impact si fausse | Interlocuteur | Échéance |
|---|---|---|---|---|
