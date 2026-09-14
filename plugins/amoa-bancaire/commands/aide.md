---
description: Mode d'emploi du plugin AMOA Bancaire
---

Affiche le mode d'emploi du plugin, sans rien inventer au-delà de ce qui suit.

## Commandes

| Commande | Usage |
|---|---|
| `/contexte` | À lancer en premier sur une nouvelle mission. Produit le fichier de contexte à déposer dans le Projet. |
| `/spec` | Spécification fonctionnelle |
| `/recette` | Plan de recette |
| `/atelier` | Compte rendu d'atelier |
| `/bpmn` | Modélisation BPMN |
| `/aide` | Ce mode d'emploi |

## Skills automatiques

`amoa-bancaire` se déclenche seul sur toute demande de livrable AMOA bancaire. `monetique` sur toute question de cartes, TPE, GAB, ISO 8583, EMV, 3DS, chargeback, interchange ou réconciliation monétique. `sql-donnees` sur toute requête SQL ou PL/SQL, modèle de données, indexation, performance ou extraction. Les trois se combinent.

## Sub-agents — Cowork uniquement

| Agent | Usage |
|---|---|
| `@amoa-bancaire:preparateur-atelier` | Préparation d'atelier **avant** la séance : questions bloquantes, hypothèses implicites, collisions avec ce qui est tranché |
| `@amoa-bancaire:verificateur-tracabilite` | Contrôle de couverture besoin → exigence → règle → test |
| `@amoa-bancaire:relecteur-conformite` | Relecture réglementaire, écarts notés Bloquant / Majeur / Mineur |
| `@amoa-bancaire:expert-monetique` | Relecture ou diagnostic monétique |
| `@amoa-bancaire:expert-sql` | Relecture SQL / PL/SQL : exactitude, sécurité, performance |

En chat, ces agents apparaissent grisés. Les skills et les commandes fonctionnent normalement.

## Enchaînement recommandé

1. `/contexte` une fois par mission, résultat déposé dans le Projet
2. `preparateur-atelier` avant chaque séance métier ; `/atelier` après, pour le compte rendu
3. `/spec` ou `/recette` pour produire
4. Dans Cowork, les deux ou trois relecteurs avant envoi

Rappelle ensuite à l'utilisateur qu'il peut demander n'importe quel livrable en langage naturel : les commandes sont des raccourcis, pas une obligation.
