# Syntaxe TopModel — clés autorisées

Index de [`SKILL.md`](../SKILL.md). Distillé du schéma JSON faisant autorité, livré avec l'outil : [`schema.json`](https://github.com/klee-contrib/topmodel/blob/develop/TopModel.Core/schema.json) du package `topmodel.generator` (relu en 4.8.7 ; schéma inchangé depuis 4.8.0). Si la version du lock du projet s'en écarte nettement, revérifier ces fichiers dans le package installé (`modgen -s` ne régénère que le schéma de la **configuration**, pas celui des `.tmd`).

Les listes de clés sont **fermées** : une clé non listée fait échouer la validation avec un message illisible. **Lire un seul fichier selon ce que tu écris :**

| Ce que tu écris | Fichier |
| --- | --- |
| entête (`module`, `tags`, `uses`, `options`), `domain`, `converter`, `decorator`, `endpoint`, `dataFlow` : clés racines | [`syntaxe/fichier.md`](syntaxe/fichier.md) |
| une classe : clés, `reference`, `enum`, héritage, `mappers` | [`syntaxe/classe.md`](syntaxe/classe.md) |
| une propriété : simple, `association`, `alias`, `composition` | [`syntaxe/proprietes.md`](syntaxe/proprietes.md) |
| une annotation (ex. regex sur un domaine) | [`syntaxe/annotations.md`](syntaxe/annotations.md) |

Ce que le générateur fait de ces clés, et les pièges : [`regles.md`](regles.md) ; effets propres à un langage : [`generateurs.md`](generateurs.md).

## Sources

- Schéma : [`schema.json`](https://github.com/klee-contrib/topmodel/blob/develop/TopModel.Core/schema.json) (source) ; copie dans le package `topmodel.generator` installé (`~/.dotnet/tools/.store/topmodel.generator/<version>/…/tools/net8.0/any/`)
- Doc : <https://klee-contrib.github.io/topmodel/> — sources markdown dans [`docs/docs/model/`](https://github.com/klee-contrib/topmodel/tree/develop/docs/docs/model) du dépôt
- Dépôt : <https://github.com/klee-contrib/topmodel>
