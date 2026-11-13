# Effets du générateur C#

Référence de [`SKILL.md`](../../SKILL.md) : ce qui, dans le code généré, dépend du générateur C#. Index : [`generateurs.md`](../generateurs.md). Les chemins et options exacts viennent toujours de la configuration (`topmodel.config`) : plusieurs effets ci-dessous dépendent d'une option, dont la valeur par défaut est indiquée.

- **`readonly: true` sur un DTO** : propriétés en `init`. Les classes sont des `record` quand l'option `useRecords` est active (défaut), `readonly` ou non.
- **Liste à `values:` dont la PK est auto-générée** : une constante par valeur, nommée `{NomDeValeur}{NomDeLaPk}` (ex. `ActifId`).
- **`enum: class`** : la PK est typée par une enum générée, nommée d'après la PK au pluriel (`Codes` pour une PK `Code`).
- **`enum: true`** : avec le générateur SQL pour Postgres, un type enum natif est créé (`create type … as enum`), nommé d'après le **nom SQL** de la classe (`sqlName` : au singulier, ex. `statut_commande`, sauf avec `pluralizeTableNames`). L'option C# `usePostgresEnums: true` mappe la colonne sur ce type côté EF (`MapEnum`) ; sinon, colonne texte.
- **`reference: true`** :
  - si `referencesModelPath` est renseigné, la classe y est générée (souvent un projet commun visible de **toutes** les couches) au lieu de `persistentModelPath` / `nonPersistentModelPath` : le flag change alors son projet et son namespace. Sans cette option (défaut `null`), rien ne bouge ;
  - si `referenceAccessorsInterfacePath` est renseigné, des accesseurs de référence (listes chargées en entier, mises en cache) sont générés pour les classes `reference` **sauf `enum: true`** ;
  - les FK (sans `useClass`) qui pointent vers une classe de référence reçoivent `[ReferencedType(typeof(Classe))]` ;
  - une enum (`enum: true`) persistée utilisée par des DTO non persistés doit porter le flag si `referencesModelPath` est défini : sinon elle reste dans le projet du modèle persisté, que les DTO de commande ou d'API ne voient généralement pas (erreur de compilation).
- **Mappers** :
  - `from:` produit une méthode statique `Create{Classe}(…)` dans la classe de mappers du module ;
  - `to:` produit une méthode d'extension `To{Cible}()` (ou le `name` du mapper) ; les propriétés **requises** de la cible sans correspondance deviennent des paramètres supplémentaires de cette méthode ;
  - avec `requiredNonNullable`, les propriétés requises portent le mot-clé `required` : un `from` qui n'en initialise pas une ne compile pas ;
  - liste d'objets : le code généré appelle le collecteur du `genericType` du domaine (`To` + nom du type générique sans le `I` initial ni les chevrons, ex. `ToCollection()` pour `ICollection<{T}>`), surchargeable par `collector:` dans l'implémentation `csharp:` du domaine. Le projet doit fournir cette méthode si elle n'existe pas.
- **Domaine** : `length` génère aussi `[StringLength(n)]` quand le type C# est `string`.
- **Annotations** : implémentation sous `csharp:` dans le document `annotation` (`text` sans les `[ ]`, ajoutés par le générateur ; `imports` = namespaces).
- **Contrôleurs partiellement générés** (en-tête « PARTIELLEMENT GENERE AUTOMATIQUEMENT ») : n'y toucher que les zones non générées ; des `using` peuvent rester à compléter à la main.
