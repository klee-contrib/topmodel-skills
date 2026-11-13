# Syntaxe — propriétés

Référence de [`SKILL.md`](../../SKILL.md). Entête de fichier et types de documents : [`fichier.md`](fichier.md) ; clés d'une classe : [`classe.md`](classe.md).

## Propriétés — 4 variantes exclusives

Chaque entrée de `properties:` doit correspondre à **exactement une** des 4 formes. La clé discriminante (`name`, `association`, `alias`, `composition`) doit être la **première clé de la map** ([`regles/invisibles.md`](../regles/invisibles.md)) ; `domain` peut être n'importe où.

### Propriété simple — requis : `name`, `domain`, `comment`

```
name, label, primaryKey, required, readonly, domain, trigram, comment,
defaultValue, paramLocation, annotations, excludedAnnotations,
customProperties, tags
```

### `association` — requis : `association`, `comment`

```
association, label, role, multiple, as, useClass, required, primaryKey,
readonly, defaultValue, paramLocation, withReverse, comment, trigram,
className, property, annotations, excludedAnnotations, customProperties, tags
```

⚠️ **Il n'y a pas de clé `name`.** Pour nommer une association (notamment deux associations vers la même classe cible), utiliser `role:` → génère `ClasseCibleIdRole` (ou `ClasseCibleRole` si `useClass` est effectif, [`regles/associations-alias.md`](../regles/associations-alias.md) § Associations).

- `property:` cible une autre propriété que la clé primaire (utile vers une classe non persistée).
- `useClass` : `false` génère la FK sans la navigation de classe ; défaut et conditions : [`regles/associations-alias.md`](../regles/associations-alias.md) § Associations.
- `withReverse: true` ou `withReverse: { className: Lignes }` (aussi `comment`, `label`, `annotations`, `excludedAnnotations`) crée la relation réciproque.
- `multiple: true` déclare l'association comme collection (le domaine devient l'`asDomains.{as}` de la PK cible, `as` valant `list` par défaut).
- `primaryKey: true` sur deux associations = table de jointure N-N.

Noms générés, contraintes de `withReverse` / `multiple` : [`regles/associations-alias.md`](../regles/associations-alias.md) § Associations.

### `alias` — requis : `alias`

```
alias (class|endpoint|decorator + include|exclude|property), prefix, name, trigram, suffix, domain,
composition, label, required, primaryKey, preservePrimaryKey, preserveTrigram,
discardAssociations, readonly, comment, as, useClass, defaultValue,
paramLocation, annotations, excludedAnnotations, customProperties, tags
```

Recopie des définitions de propriétés d'une autre classe (ou des paramètres d'un endpoint, des propriétés d'un décorateur). Seules vont **dans** le bloc `alias:` : **une** des clés `class` / `endpoint` / `decorator`, et au plus **une** des clés `include` / `exclude` / `property` (`property` = une seule propriété) ; toutes les autres clés (`prefix`, `suffix`, `as`, `name`, `required`…) sont ses **sœurs**, au niveau de la propriété — une clé mal placée donne un dump `oneOf` ([`outillage/modgen.md`](../outillage/modgen.md) § Décoder les erreurs) :

```yaml
- alias:
    class: Adresse
    exclude: [Id]
  prefix: Livraison # sœur de alias:, pas dedans
```

`include` / `exclude` / `property` désignent les propriétés **par leur nom généré dans la classe source**, sinon `TMD0004`. Pour une association, ce nom dépend de `useClass` ([`regles/associations-alias.md`](../regles/associations-alias.md) § Associations) : `CiviliteCode`, `UtilisateurIdClient` (avec `role: Client`) par défaut ; `Civilite`, `UtilisateurClient` si `useClass` est effectif ou si la cible est `enum: true`. Ex. du modèle de test : `property: TableId` sur l'association `Table` déclarée `useClass: false`.

`prefix: true` / `suffix: true` utilisent le nom de la classe (endpoint, décorateur) source. `as: list` transforme en liste. Clé primaire, tags et surcharges : [`regles/associations-alias.md`](../regles/associations-alias.md) § Alias.

### `composition` — requis : `composition`, `name`, `comment`

```
composition, name, label, domain, comment, readonly, required, trigram,
paramLocation, annotations, excludedAnnotations, customProperties, tags
```

Référence la **classe entière** (pas de colonne FK) : DTO imbriqué, JSON.

### association vs composition vs alias

|  | Génère | Usage |
| --- | --- | --- |
| `association` | une colonne FK vers la PK de la cible | relation persistée |
| `composition` | un objet imbriqué, aucune FK | DTO, structure JSON |
| `alias` | des propriétés locales recopiées | DTO qui reprend des champs d'une entité |

Groupe de champs répété (deux rôles d'un même concept) : [`regles/associations-alias.md`](../regles/associations-alias.md) § Composition plutôt qu'alias préfixés.
