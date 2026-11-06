# Règles — annotations et décorateurs

Référence de [`SKILL.md`](../../SKILL.md). Syntaxe d'un document `annotation` : [`syntaxe/annotations.md`](../syntaxe/annotations.md) ; clés d'un `decorator` : [`syntaxe/fichier.md`](../syntaxe/fichier.md) (tableau des types de documents).

## Annotations

- **Visibilité** : contrairement aux domaines, une annotation n'est **pas** visible partout. La définir dans le même fichier que ce qui l'utilise (pour une annotation de domaine : en tête du fichier des domaines), ou l'importer par `uses:`. Exception : `global: true` (posée sur tous les objets ciblés, sans paramètres).
- **Template** : `text` est templaté, les `{...}` sont interprétés comme des variables (`{label}`, paramètres…). Un quantificateur de regex `{6}` déclenche l'avertissement `TMD0011` mais est généré tel quel (vérifié en 4.8.5) : garder la regex et commenter l'avertissement dans le `.tmd`.
- Les annotations ne sont générées que par les générateurs back qui les implémentent ([`generateurs.md`](../generateurs.md)).
- Paramètres (`parameters:`), exclusions (`excludedAnnotations:`), `propertyAnnotations:` : [`annotations.md`](https://github.com/klee-contrib/topmodel/blob/develop/docs/docs/model/annotations.md) du dépôt topmodel.

## Décorateurs

- Un décorateur ne fait que recopier des propriétés (avec leur éventuel `defaultValue`) et poser `extends` / `implements` / `imports` : il ne met à jour aucune valeur à l'enregistrement. Des dates de modification sont tenues par la couche de persistance.
- Un décorateur **sans propriété** peut aussi ne poser qu'une interface, commune à plusieurs classes générées qui en déclarent déjà les propriétés, pour les traiter ensemble dans le code écrit à la main.
