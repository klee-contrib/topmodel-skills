# Règles — associations, alias et compositions

Référence de [`SKILL.md`](../../SKILL.md). Clés autorisées : [`syntaxe/proprietes.md`](../syntaxe/proprietes.md) ; classes persistées ou DTO : [`classes.md`](classes.md).

## Associations

- **Nom généré** d'une association. C'est ce nom qu'on utilise partout ailleurs : `values:`, `unique:`, `include` / `exclude` / `property` d'un alias, `discriminatorProperty`, code écrit à la main.
  - Par défaut : `{Cible}{PkCible}{Role}` (ex. `TypeUtilisateurCode`, `ProfilId`, `UtilisateurIdClient`).
  - `useClass` effectif : `{Cible}{Role}`, typé par la classe. Il est effectif quand la classe source **et** la cible sont persistées et que `useClass` vaut `true`. Le défaut de `useClass` est l'option de configuration `defaultAssociationUseClass`, elle-même `false` par défaut : **regarder la config du projet**. Sur un DTO, `useClass` est sans effet.
  - Cible `enum: true` : toujours `{Cible}{Role}` (`useClass` forcé).
  - `multiple: true` : `{CiblePluriel}{Role}`, sans la PK.
  - `className:` remplace `{Cible}`.
- `withReverse` entre deux fichiers **impose** un `uses:` circulaire : le fichier de la classe cible doit importer celui de la source (sinon `TMD9008`, [`fichiers-uses.md`](fichiers-uses.md) § `uses:` circulaires). `multiple: true` ajoute la réciproque d'office : même contrainte.
- Domaines de liste requis (`{as}` vaut `list` par défaut) :
  - `multiple: true` exige un `asDomains.{as}` sur le domaine de la PK **cible**, sinon `TMD9003` ;
  - `withReverse` sur une association ni `multiple` ni `unique` exige un `asDomains.{as}` sur le domaine de la PK de la classe **source** (la réciproque est une liste de ses identifiants), sinon `TMD9004`.

## Alias

- `primaryKey` n'est **pas** recopiée : `preservePrimaryKey: true` (sans l'auto-génération : ni séquence ni identité) ou `primaryKey: true`. Dans les deux cas, la classe devient persistée ([`classes.md`](classes.md)).
- Les `tags` de propriété ne sont pas hérités par les alias.
- Une surcharge (`required`, `label`, `defaultValue`…) s'applique à **toutes** les propriétés incluses : des surcharges différentes demandent des alias séparés.
- L'alias d'une classe qui a des associations réciproques recopie aussi ces propriétés réciproques : les `exclude` si le DTO n'en veut pas. Sur une classe persistée, un alias d'une propriété `multiple` est refusé (`TMD9012`).

## Composition plutôt qu'alias préfixés

Un même groupe de champs apparaît plusieurs fois (deux rôles d'un même concept : expéditeur / destinataire, demandeur / défendeur) → **une classe DTO pour le groupe, et une `composition` par rôle**.

Jamais N copies des mêmes alias avec `prefix: Expediteur` / `prefix: Destinataire` : le code consommateur se duplique à chaque couche (une méthode par préfixe côté back, des noms de champs construits par gabarit côté front).
