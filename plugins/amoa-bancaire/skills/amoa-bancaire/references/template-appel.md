# Template de collecte du contexte

À utiliser quand l'utilisateur formule une demande sans fournir le contexte de mission. Ne pas dérouler le template entier systématiquement : ne demander que les lignes **bloquantes** pour le livrable en cours, 5 questions maximum.

## Template complet

```
TASK
Livrable attendu : …
Périmètre inclus : …
Hors périmètre : …
Destinataire (métier / MOE / COPIL / conformité / testeurs) : …

CONTEXTE
Banque / client : …
Core banking en place : …
Processus concerné : …
Volumétrie : …
Méthodologie projet (Agile / Cycle en V) : …
Échéance : …
Contraintes réglementaires : …
Décisions déjà actées : …
Risques perçus (top 3) : …
Ce qui a déjà été livré : …

STOP
Le livrable est terminé quand : …
```

## Quelles lignes sont bloquantes selon le livrable

| Livrable | Information bloquante |
|---|---|
| Spécification fonctionnelle | Processus concerné, core banking, règles métier existantes, interfaces amont/aval |
| Plan de recette | Spécification de référence, environnement de test, jeux de données disponibles |
| Analyse de processus | AS-IS observé (qui fait quoi, avec quel outil), volumétrie, points de douleur |
| BPMN | Acteurs, déclencheur, événement de fin, points de décision |
| Expression de besoin | Objectif métier, périmètre, contrainte réglementaire éventuelle |
| Compte rendu d'atelier | Date, participants, notes brutes ou ordre du jour |
| User stories | Acteur, bénéfice métier attendu, règles de gestion connues |
| Support de présentation | Audience, durée, décision attendue en sortie |

## Formulation des questions

Poser les questions groupées et numérotées, avec l'impact de chacune sur le livrable :

> Avant de rédiger, trois points bloquants :
> 1. **Core banking cible** — conditionne la faisabilité des contrôles temps réel (T24 et Amplitude ne se comportent pas pareil sur ce point).
> 2. **Périmètre devises** — détermine si les règles de conversion entrent dans le scope.
> 3. **Virements instantanés inclus ou non** — change les SLA et les contrôles AML applicables.
>
> Points secondaires que je traite par hypothèse si tu n'as pas l'info : volumétrie, environnement de recette.
