---
description: Modéliser un processus en BPMN avec swimlanes et points de contrôle
---

Modélise le processus en BPMN en appliquant le skill `amoa-bancaire`.

Lis `${CLAUDE_PLUGIN_ROOT}/skills/amoa-bancaire/references/formats-livrables.md` (section 3).

Processus : $ARGUMENTS

Exigences :
1. Diagramme Mermaid avec swimlanes par acteur.
2. Distingue tâche manuelle, tâche automatique et tâche de contrôle.
3. Modélise les sorties de flux nominal (rejets, exceptions, timeouts), pas seulement le chemin heureux.
4. Termine par « Optimisations proposées » : automatisation, contrôles redondants, boucles de retour évitables.

Si les acteurs, le déclencheur ou l'événement de fin sont inconnus, demande-les avant de modéliser.
