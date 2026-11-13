# Règles — visibilité entre fichiers et `uses:`

Référence de [`SKILL.md`](../../SKILL.md). Entête d'un fichier (`uses`, `tags`) : [`syntaxe/fichier.md`](../syntaxe/fichier.md).

## Visibilité entre fichiers

- Domaines : visibles partout, sans `uses:`. Converters : ajoutés d'office comme dépendance de tout fichier qui a des mappers.
- Classes, décorateurs, endpoints, annotations non globales : `uses:` obligatoire, sinon « introuvable » — `TMD0002` (classe), `TMD0005` (décorateur), `TMD0006` (endpoint), `TMD2001` (annotation). Les propriétés d'un décorateur sont recopiées dans la cible : une propriété de même nom → `TMD0001`.
- Effet de bord : un fichier de domaines est dépendance implicite de tous ; une classe ou une annotation qu'on y place devient donc visible partout.
- **Tags et références** : si un objet en référence un autre qui n'a aucun tag commun avec la configuration du générateur, la **référence est omise** du code généré (composition, association, mapper), sans erreur. `referencedTags` dans la configuration du générateur permet de référencer des tags d'autres configurations.

## `uses:` circulaires

TopModel les **accepte** (depuis la 3.2.0) : chaque cycle de fichiers est regroupé et résolu comme un seul fichier. Aucun code d'erreur.

| Situation | Statut |
| --- | --- |
| `withReverse` ou `multiple` entre deux fichiers | **obligatoire** : `uses:` retour sur le fichier de la classe cible, sinon `TMD9008` |
| autre cycle | toléré ; la doc conseille de les réduire au minimum |

Conséquences d'un cycle :

- un seul espace de noms pour le groupe (`TMD3001` si deux classes du groupe ont le même nom — contrôle qui vaut aussi, hors cycle, entre un fichier et ses dépendances) ;
- tout le groupe est re-résolu à chaque modification d'un de ses fichiers ;
- le code généré peut ne pas le supporter si les fichiers partent vers des projets différents (références circulaires entre projets ou modules compilés séparément) : limite de l'architecture, pas de TopModel ;
- un fichier ne contenant que des domaines évite les cycles involontaires (il est dépendance implicite de tous).

Le projet peut interdire les cycles par choix de lisibilité : c'est alors une convention du projet, avec l'exception obligatoire ci-dessus.
