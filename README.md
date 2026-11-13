# topmodel

Package APM fournissant un skill agent pour écrire un modèle [TopModel](https://github.com/klee-contrib/topmodel) : fichiers `.tmd`, génération avec `modgen`, diagnostics via le serveur LSP `modls`.

## Installation

De manière ensembliste (recommandé) :

```bash
apm install klee-contrib/topmodel-skills --global
```

Au sein d'un projet :

```bash
apm install klee-contrib/topmodel-skills
```

## Mise à jour

De manière ensembliste (recommandé) :

```bash
apm update klee-contrib/topmodel-skills --global
```

Au sein d'un projet :

```bash
apm update klee-contrib/topmodel-skills
```

## Contenu

- `.apm/skills/topmodel/` : skill TopModel (syntaxe, règles, générateurs, outillage, exemples)
- Serveur LSP `modls` déclaré dans `apm.yml` pour les fichiers `.tmd`

## Prérequis

- `modls` disponible dans le `PATH`

## Licence

MIT
