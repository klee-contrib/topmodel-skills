# `tmdgen` — générateur de modèle

Référence de [`SKILL.md`](../../SKILL.md) : `tmdgen` (dotnet tool `TopModel.ModelGenerator`) fait l'inverse de `modgen` : il **génère des `.tmd`** à partir d'une source externe (spécification OpenAPI ou base de données). Options communes et détection des configs : [`outillage.md`](../outillage.md) § Socle commun. Vérifié sur le code source de TopModel (`TopModel.ModelGenerator`).

## Fonctionnement

- Configuration : fichier `tmdgen*.config` (YAML), à ne pas confondre avec `topmodel.config` ; schéma `tmdgen.config.schema.json`.
- Deux familles de sources, cumulables dans une même config : `openApi:` (liste) et `database:` (liste). Chaque entrée est un générateur numéroté.
- Il écrit des `.tmd` sous `modelRoot` (défaut `./`), dans le `outputDirectory` de chaque entrée, et tient son propre lock **`tmdgen.lock`**, distinct de `topmodel.lock`.
- Les `.tmd` produits sont **générés** : ne pas les éditer à la main, sous peine d'écrasement à la prochaine exécution ; en cas de besoin de retouche, en parler à l'humain.
- Après génération, les `.tmd` obtenus sont ensuite l'entrée de `modgen` comme n'importe quel modèle.

### Source OpenAPI (`openApi:`)

| Clé                         | Rôle                                                                 |
| --------------------------- | -------------------------------------------------------------------- |
| `source` (obligatoire)      | spécification OpenAPI (ex. `./petstore.json`)                        |
| `module`                    | module TopModel des classes et endpoints produits (défaut `OpenApi`) |
| `outputDirectory`           | dossier de sortie (défaut = `module` ; `{module}` y est remplacé)    |
| `domains`                   | correspondance type OpenAPI → domaine TopModel (`type:` / `domain:`) |
| `include`                   | restreint les éléments repris                                        |
| `modelTags`, `endpointTags` | tags des fichiers de classes et d'endpoints                          |
| `modelFileName`             | nom du fichier de classes (défaut `Model`)                           |
| `preservePropertyCasing`    | conserve la casse des propriétés (défaut `true`)                     |
| `classPrefix`               | préfixe des noms de classes                                          |
| `useEnumClasses`            | produit des classes `enum` pour les énumérations                     |

### Source base de données (`database:`)

| Clé                  | Rôle                                                                                                 |
| -------------------- | ---------------------------------------------------------------------------------------------------- |
| `source`             | connexion : `dbType` (Oracle, PostgreSQL, MySQL, MSSQL), hôte, port, base, utilisateur, mot de passe |
| `outputDirectory`    | dossier de sortie (défaut `./`)                                                                      |
| `domains`            | correspondance type SQL → domaine TopModel                                                           |
| `tags`               | tags des fichiers produits                                                                           |
| `exclude`            | tables à exclure                                                                                     |
| `extractValues`      | tables dont le contenu est extrait en `values:`                                                      |
| `classNameOverrides` | nom de classe imposé pour une table                                                                  |
| `modules`            | regroupement des tables en modules (`name`, `classes`, `tags`)                                       |

Ne pas écrire de mot de passe réel dans un fichier versionné ni dans une commande rapportée à l'humain.

## Options

Seules les options communes existent, plus :

| Option         | Effet                                                                                                         |
| -------------- | ------------------------------------------------------------------------------------------------------------- |
| `-s, --schema` | régénère le schéma JSON de la **configuration** (`<config>.schema.json`) et le référence en tête de la config |

Pas de `-e` ni de `-u` (propres à `modgen`, [`modgen.md`](modgen.md) § Options).

## Version

Package `TopModel.ModelGenerator`. Au démarrage, l'outil signale si une version plus récente est publiée (comme `modgen`) : ne pas mettre à jour sans l'humain, `dotnet tool update -g TopModel.ModelGenerator --version <version>` se fait sur son poste.
