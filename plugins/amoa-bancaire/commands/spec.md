---
description: Rédiger une spécification fonctionnelle bancaire complète
---

Rédige une spécification fonctionnelle en appliquant le skill `amoa-bancaire`.

Lis d'abord `${CLAUDE_PLUGIN_ROOT}/skills/amoa-bancaire/references/formats-livrables.md` (section 2) et `${CLAUDE_PLUGIN_ROOT}/skills/amoa-bancaire/references/checklist-qualite.md`.
Si le sujet touche aux cartes, aux TPE, aux GAB, à l'e-commerce ou au switch, applique aussi le skill `monetique`.

Sujet : $ARGUMENTS

Avant de rédiger :
1. Vérifie que tu disposes du processus concerné, du core banking cible, des règles métier existantes et des interfaces amont/aval. Si l'un manque, pose la question et arrête-toi.
2. Numérote les exigences `EF-xxx` et les règles de gestion `RG-xxx`.
3. Termine par « Prochaines étapes » et « À confirmer avec le métier ».

Si l'utilisateur veut un fichier diffusable, produis un `.docx`.
