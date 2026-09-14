# Marketplace AMOA Bancaire

Marketplace locale contenant le plugin `amoa-bancaire`.

## Installation dans Claude Code

```bash
cd /chemin/vers/amoa-marketplace
claude
```

Puis, dans Claude Code :

```
/plugin marketplace add .
/plugin install amoa-bancaire@amoa-bancaire-marketplace
```

Choisir la portée à l'invite : `user` (toutes les sessions), `project` (partagé via le dépôt), `local` (ce dépôt, pour vous seul).

## Vérification

```
/plugin
```

Le plugin doit apparaître comme activé. Taper `/` doit faire apparaître `/spec`, `/recette`, `/atelier`, `/bpmn`, `/contexte`, `/aide`.

## Si le plugin apparaît installé mais inactif

Un défaut connu de Claude Code n'active pas toujours les plugins issus d'une marketplace de type répertoire. Vérifier `~/.claude/settings.json` et ajouter si nécessaire :

```json
"enabledPlugins": {
  "amoa-bancaire@amoa-bancaire-marketplace": true
}
```

Puis relancer Claude Code.

## Mise à jour

Pour une marketplace de type répertoire, le cache n'est pas toujours rafraîchi. Le plus fiable :

```
/plugin uninstall amoa-bancaire@amoa-bancaire-marketplace
/plugin marketplace remove amoa-bancaire-marketplace
/plugin marketplace add .
/plugin install amoa-bancaire@amoa-bancaire-marketplace
```

## Distribution à l'équipe

Pousser ce dossier sur un dépôt Git, puis partager :

```
/plugin marketplace add <url-du-depot>
/plugin install amoa-bancaire@amoa-bancaire-marketplace
```

Les marketplaces de type Git se mettent à jour correctement avec `/plugin marketplace update`.
