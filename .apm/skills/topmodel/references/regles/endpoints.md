# Règles — endpoints

Référence de [`SKILL.md`](../../SKILL.md). Clés d'un endpoint : [`syntaxe/fichier.md`](../syntaxe/fichier.md) (tableau des types de documents) ; propriétés et paramètres : [`syntaxe/proprietes.md`](../syntaxe/proprietes.md) ; noms de routes et de fichiers selon le langage : [`generateurs.md`](../generateurs.md).

## Endpoints

- `paramLocation` déduit : `json-body` pour une composition (`form-data` s'il existe un autre paramètre `form-data`, ou si la classe composée a une propriété `form-data`), `query` pour le reste ; un paramètre présent dans la route est un paramètre `route`. Réglable aussi sur le domaine.
- Un seul corps par endpoint : une propriété `json-body`, ou plusieurs `form-data` (sinon `TMD7007`).
- Chaque `{param}` de la route doit correspondre à un paramètre (`TMD7005`). L'inverse n'est contrôlé que pour un paramètre explicitement `paramLocation: route` (`TMD7010`) : un paramètre absent de la route devient silencieusement `query`.
- Alias de PK en paramètre : nom préfixé par le trigramme (de la propriété, sinon de sa classe), sinon par le nom de la classe (`{difId}`) ; `trigram:` le surcharge, `trigram: ""` le retire.
- `returns` en composition exige `name` (inutilisé) et `comment`, comme toute composition.
- Un fichier client ou serveur généré par fichier de modèle contenant des endpoints (`options.endpoints.fileName` / `prefix`) ; noms d'endpoints uniques par fichier cible. Des fichiers du même module au même nom sont fusionnés et doivent avoir le même `prefix`.
