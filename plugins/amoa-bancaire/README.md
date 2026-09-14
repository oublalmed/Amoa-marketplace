# AMOA Bancaire — plugin Claude

Livrables AMOA / Business Analyst pour projets bancaires, au standard cabinet de conseil, avec expertise monétique.

## Démarrage en 3 étapes

1. Installer le plugin (voir ci-dessous).
2. Lancer `/contexte` une fois, en décrivant votre mission. Déposer le fichier produit dans la base de connaissances de votre Projet Claude.
3. Demander vos livrables, en langage naturel ou via les commandes.

`/aide` affiche le mode d'emploi à tout moment.

## Contenu

| Composant | Nom | Rôle |
|---|---|---|
| Skill | `amoa-bancaire` | Posture, arbitrages, méthode, conditions d'arrêt, 11 formats de livrables |
| Skill | `monetique` | Flux cartes, ISO 8583, EMV, 3DS, chargeback, réconciliation, commissions |
| Skill | `sql-donnees` | SQL, PL/SQL, modèle de données, indexation, performance, migrations |
| Commande | `/contexte` | Capture du contexte de mission, une fois par mission |
| Commande | `/spec` | Spécification fonctionnelle |
| Commande | `/recette` | Plan de recette |
| Commande | `/atelier` | Compte rendu d'atelier métier |
| Commande | `/bpmn` | Modélisation BPMN avec swimlanes |
| Commande | `/aide` | Mode d'emploi |
| Sub-agent | `relecteur-conformite` | Relecture réglementaire, écarts notés |
| Sub-agent | `verificateur-tracabilite` | Couverture besoin → exigence → règle → test |
| Sub-agent | `expert-monetique` | Relecture et diagnostic monétique |
| Sub-agent | `expert-sql` | Relecture SQL / PL/SQL : exactitude, sécurité, performance |

Les deux skills se déclenchent seuls et se combinent. Les commandes sont des raccourcis, jamais une obligation.

## Installation

Customize (barre latérale gauche) > onglet Plugins > téléverser le zip.
Dans Cowork, ouvrir d'abord l'onglet Cowork, puis Customize.

**Avant d'installer une nouvelle version, désinstallez l'ancienne**, ainsi que le skill `amoa-bancaire` s'il a été installé seul. Sinon le skill existe en double.

## Limites connues

| Limite | Détail |
|---|---|
| Sub-agents | Ne tournent que dans Cowork. En chat ils apparaissent grisés ; skills et commandes fonctionnent normalement. |
| Connecteurs | Aucun n'est déclaré. Dans Cowork, les connecteurs passent par le cloud d'Anthropic : un Jira ou SharePoint on-premise derrière le pare-feu de la banque ne sera pas joignable sans exposition dédiée. |
| Valeurs réglementaires | Les skills refusent délibérément de citer de mémoire un taux d'interchange, un code motif de chargeback ou un délai de scheme. Ces valeurs sont révisées périodiquement : le skill renvoie au document source. |
| Données sensibles | Aucun PAN valide, CVV, donnée de piste ou clé n'est produit, y compris dans les jeux de test. |

## Ajouter un connecteur

Créer un fichier `.mcp.json` à la racine du plugin :

```json
{
  "mcpServers": {
    "mon-outil": {
      "type": "http",
      "url": "https://exemple.com/mcp"
    }
  }
}
```

## Usage type

```
/contexte Banque X, core banking Y, projet de réconciliation monétique acquéreur
/spec Contrôle du mandat avant présentation
```

Puis dans Cowork, avant envoi :

```
@amoa-bancaire:verificateur-tracabilite vérifie la couverture
@amoa-bancaire:expert-monetique relis sous l'angle monétique
```

## Versions

| Version | Contenu |
|---|---|
| 1.0.0 | Skill AMOA, 4 commandes, 2 sub-agents |
| 1.1.0 | Ajout du skill monétique et du sub-agent expert-monetique |
| 1.2.0 | Chemins de référence fiabilisés, commandes `/contexte` et `/aide`, modes de défaillance, limites documentées |
| 1.3.0 | Ajout du skill `sql-donnees` et du sub-agent `expert-sql` |
