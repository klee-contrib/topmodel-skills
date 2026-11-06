# `modls` — language server

Référence de [`SKILL.md`](../../SKILL.md) : ce que le language server voit et ne voit pas, ses requêtes spécifiques, l'exploration du modèle. Options communes et détection des configs : [`outillage.md`](../outillage.md) § Socle commun.

## Sommaire

| Besoin                          | Section                                                     |
| ------------------------------- | ----------------------------------------------------------- |
| quel outil voit quelle erreur   | [Qui attrape quoi](#qui-attrape-quoi)                       |
| version à installer             | [Version](#version)                                         |
| diagramme Mermaid, schéma       | [Requêtes spécifiques](#requêtes-spécifiques-de-modls)      |
| qui référence une classe        | [Explorer le modèle existant](#explorer-le-modèle-existant) |

## Ce qu'il apporte

`modls` (dotnet tool `TopModel.LanguageServer`) donne diagnostics sémantiques, définition, références, symboles et survol. Il faut seulement que le binaire `modls` soit installé et dans le `PATH`, sinon **proposer à l'humain de l'installer**. ~19 s au premier appel, ~170 ms ensuite.

Il tourne toujours en mode watch et en parallèle (il ajoute lui-même `-w` et `-p`) et lit les configs `topmodel*.config`. Il **n'écrit rien** : pas de génération de code, pas de lock.

Dans VS Code, l'extension `jabx.topmodel` apporte en plus la **validation de schéma** (via `redhat.vscode-yaml`) ; ses diagnostics peuvent être décalés d'un état.

## Version

Le language server a son **propre** numéro de version, absent du lock (ex. modgen 4.8.7, modls 4.8.6) : même version **majeure** que `modgen` (c'est ce que vérifie l'extension VS Code), dernière version patch publiée. Version de `modgen` et lock : [`modgen.md`](modgen.md) § Version de `modgen`.

## Qui attrape quoi

| Classe d'erreur                                                                                                             | LSP (`modls`)                                                                                              | `modgen` sur copie                  |
| --------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------- | ----------------------------------- |
| Schéma : clé interdite, document mal formé                                                                                  | ❌ **aucun diagnostic** (dump seulement dans le journal du serveur ; il garde l'état précédent du fichier) | ✅ dump `oneOf`                     |
| Sémantique : `TMD0002` classe introuvable (`uses:` manquant), `TMD0003` domaine, `TMD0004` propriété, `TMD3005` rôle requis | ✅                                                                                                         | ✅                                  |
| Import inutilisé (`TMD1003`)                                                                                                | ✅ (warning)                                                                                               | ✅                                  |
| Clé discriminante pas en premier ([`regles/invisibles.md`](../regles/invisibles.md))                                          | ❌ aucun diagnostic                                                                                        | ✅ message sans code                |
| Écrit sur le disque                                                                                                         | non                                                                                                        | oui (code généré, lock, `.modgen/`) |

**LSP silencieux ne veut pas dire valide.** La validation `modgen` sur copie reste l'autorité. Décodage des messages : [`modgen.md`](modgen.md) § Décoder les erreurs.

## Requêtes spécifiques de `modls`

En plus du protocole LSP standard, `modls` expose deux requêtes JSON-RPC personnalisées. Elles ne sont pas des opérations LSP standard : il faut un client qui sait envoyer une requête arbitraire (c'est le cas de l'extension VS Code), a priori pas l'outil LSP natif de l'agent.

| Méthode   | Paramètres                                                                     | Réponse                                                                 | Usage                                                                                                       |
| --------- | ------------------------------------------------------------------------------ | ----------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------- |
| `mermaid` | `uri` : URI du fichier `.tmd` ; `scope` : `File` (défaut), `Module` ou `Model` | `{ diagram, module, fileName, app, scope }` ; `null` si fichier inconnu | Diagramme Mermaid (classes et associations) du fichier, du module ou de tout le modèle auquel il appartient |
| `schema`  | aucun                                                                          | `{ content }` (JSON Schema des fichiers `.tmd`) ; `null` si absent      | Schéma des `.tmd` **de la version de TopModel réellement utilisée**, à la place d'une copie figée côté client |

- `mermaid` attend la fin du chargement du modèle ; `app` désigne l'application du fichier quand un même `modls` en sert plusieurs.
- `schema` ne dépend pas du modèle chargé : il répond dès le démarrage.

## Explorer le modèle existant

Outil LSP (symboles, définition, références) s'il est branché ; sinon, recherche textuelle dans les `.tmd`. Utile pour savoir **qui référence** une classe avant de la renommer.
