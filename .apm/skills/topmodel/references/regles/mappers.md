# Règles — mappers

Référence de [`SKILL.md`](../../SKILL.md). Clés autorisées (`from`, `to`, `mappings`) : [`syntaxe/classe.md`](../syntaxe/classe.md) § `mappers` ; noms par langage : [`generateurs.md`](../generateurs.md).

## Mappers

- Correspondances déduites par alias, puis par nom + domaine identique **ou convertible** (un `converter` existe). `mappings:` existe pour `from` comme pour `to` (renommage possible) ; la propriété de gauche est **toujours** celle de la classe courante ; `MaPropriete: false` désactive une correspondance.
- Les paramètres de classe d'un `from` sont obligatoires par défaut (`required: false` possible) ; un paramètre de type propriété suit son `required` / `defaultValue` et accepte `target`. Un paramètre obligatoire ne peut pas suivre un facultatif (`TMD8012`).
- Une propriété n'est initialisée qu'une fois : deux sources possibles → erreur (`TMD8002` explicite, `TMD8003` automatique), à lever par `mappings` ou `Prop: false`.
- `name` devient obligatoire si plusieurs `to` visent la même classe (ou si un `from` prend deux fois la même classe) : doublon de nom → `TMD0001`.
- Domaines différents entre source et cible : déclarer un `converter` (un seul par couple de domaines, sinon `TMD6002`).
- Liste d'objets : mappée seulement si son domaine est `collection: true`, si la correspondance existe (alias, même nom ou `mappings`) **et** si les classes d'éléments sont identiques ou reliées par un mapper. Le code généré peut s'appuyer sur un collecteur fourni par le projet ([`generateurs.md`](../generateurs.md)).
- Propriété requise de la cible laissée sans correspondance : l'effet dépend du générateur (C# : paramètre supplémentaire du `to`, ou erreur de compilation avec `requiredNonNullable`, [`generateurs.md`](../generateurs.md)) : vérifier le code généré.
