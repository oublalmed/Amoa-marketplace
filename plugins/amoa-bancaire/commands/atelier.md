---
description: Transformer des notes brutes en compte rendu d'atelier métier
---

Rédige un compte rendu d'atelier en appliquant le skill `amoa-bancaire`.

Lis `${CLAUDE_PLUGIN_ROOT}/skills/amoa-bancaire/references/formats-livrables.md` (section 5).

Notes brutes / ordre du jour : $ARGUMENTS

Règles strictes :
1. Une décision se formule au passé, sans conditionnel. Ce qui n'est pas tranché va dans « Questions ouvertes », jamais dans « Décisions ».
2. Le tableau d'actions porte un responsable nommé et une échéance datée. Signale explicitement tout « à définir ».
3. Si les participants ou la date manquent, demande-les avant de rédiger.
