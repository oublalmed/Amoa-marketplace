---
description: Construire un plan de recette à partir d'une spécification
---

Construis un plan de recette en appliquant le skill `amoa-bancaire`.

Lis `${CLAUDE_PLUGIN_ROOT}/skills/amoa-bancaire/references/formats-livrables.md` (section 4) et `${CLAUDE_PLUGIN_ROOT}/skills/amoa-bancaire/references/checklist-qualite.md` (sections 2 et 6).
Si le périmètre est monétique, applique aussi le skill `monetique` et couvre les scénarios de sa référence `incidents-et-chargeback.md`.

Périmètre à tester : $ARGUMENTS

Exigences :
1. Un tableau `ID | Cas de test | Prérequis | Étapes | Résultat attendu | Priorité | Statut`.
2. Pour chaque règle de gestion : un cas nominal, un cas limite, un cas d'erreur. Aucune exception.
3. Une section dédiée aux cas réglementaires (KYC/AML, seuils, piste d'audit).
4. Les jeux de données nécessaires et leur mode de constitution. Jamais de donnée réelle ni de PAN valide.
5. Signale toute règle de gestion non couverte comme une anomalie de couverture.

Sortie en `.xlsx` si l'utilisateur prévoit de l'exploiter en recette.
