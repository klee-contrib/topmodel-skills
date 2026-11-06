# Outillage TopModel

Référence de [`SKILL.md`](../SKILL.md) : les trois outils en ligne de commande et ce qu'ils ont en commun. La boucle de travail obligatoire reste dans [`SKILL.md`](../SKILL.md).

Un fichier par outil : n'ouvrir que celui dont on a besoin.

| Outil                                              | Rôle                                                  | Fichier                                  |
| -------------------------------------------------- | ----------------------------------------------------- | ---------------------------------------- |
| `modgen` (`TopModel.Generator`)                    | génère le code à partir des `.tmd`                    | [`outillage/modgen.md`](outillage/modgen.md) |
| `tmdgen` (`TopModel.ModelGenerator`)               | génère des `.tmd` à partir d'OpenAPI ou d'une base    | [`outillage/tmdgen.md`](outillage/tmdgen.md) |
| `modls` (`TopModel.LanguageServer`)                | language server : diagnostics, navigation, survol     | [`outillage/modls.md`](outillage/modls.md)   |

| Besoin                                   | Où                                                                            |
| ---------------------------------------- | ----------------------------------------------------------------------------- |
| itérer vite sur le modèle                | [`modgen.md`](outillage/modgen.md) § Mode watch                               |
| `FileNotFoundException`, version du lock | [`modgen.md`](outillage/modgen.md) § Version de `modgen`                      |
| quel outil voit quelle erreur            | [`modls.md`](outillage/modls.md) § Qui attrape quoi                           |
| qui référence une classe                 | [`modls.md`](outillage/modls.md) § Explorer le modèle existant                |
| dump `oneOf`, codes `TMDxxxx`            | [`modgen.md`](outillage/modgen.md) § Décoder les erreurs                      |
| options spécifiques de `modgen`          | [`modgen.md`](outillage/modgen.md) § Options                                  |
| code généré faux, modèle valide          | [`modgen.md`](outillage/modgen.md) § Bug du générateur                        |
| produire des `.tmd` depuis OpenAPI / SQL | [`tmdgen.md`](outillage/tmdgen.md)                                            |

## Socle commun

Les trois outils reposent sur la même ligne de commande et le même chargement de configuration.

### Options communes

| Option           | Effet                                                                                                                                  |
| ---------------- | -------------------------------------------------------------------------------------------------------------------------------------- |
| `-f, --file`     | fichier de config (répétable) ; sans cette option, les configs sont cherchées depuis le dossier courant                                |
| `-w, --watch`    | traite en continu : chaque modification relance, la config est rechargée à chaud                                                       |
| `-c, --check`    | vérifie la conformité de ce qui est écrit ; sort en 1 si des fichiers ont été créés, modifiés ou supprimés. **N'est pas en lecture seule** pour `modgen` ([`SKILL.md`](../SKILL.md)) |
| `-p, --parallel` | traite les configurations données en parallèle                                                                                         |
| `--help`, `--version` | aide et numéro de version                                                                                                         |

`modls` force lui-même `-w` et `-p`. Options propres à un outil : dans son fichier (`-e`, `-u` pour `modgen` ; `-s` pour `modgen` et `tmdgen`).

### Fichiers de configuration

Détection par motif quand `-f` est absent : `topmodel*.config` (`modgen`, `modls`) ou `tmdgen*.config` (`tmdgen`). Chaque config a son lock (`topmodel.lock`, `tmdgen.lock`) qui liste les fichiers générés.

### Versions

- Chaque outil est un dotnet tool avec son propre package NuGet et affiche sa version au démarrage, avec un avertissement si une version plus récente est publiée.
- Mettre à jour un outil est une action sur le poste de l'humain : la lui demander. Sans `--version`, `dotnet tool update -g` installe la dernière version, pas celle attendue par le lock.
- Détail pour `modgen` (version du lock) et `modls` (même version majeure) : leurs fichiers.

### Sortie console

La sortie ne fixe pas l'encodage : elle suit la page de code de la console (souvent CP850 sur un Windows français). Accents illisibles → la relire avec `iconv -f CP850 -t UTF-8`.
