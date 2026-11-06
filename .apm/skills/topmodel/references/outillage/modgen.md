# `modgen` — générateur de code

Référence de [`SKILL.md`](../../SKILL.md) : mode watch, version de l'outil, décodage des erreurs, options, protocole « bug du générateur ». Ce qui est commun aux trois outils (options `-f -c -w -p`, détection des configs, version affichée) est dans [`outillage.md`](../outillage.md) § Socle commun. La boucle de travail obligatoire reste dans [`SKILL.md`](../../SKILL.md).

## Sommaire

| Besoin                                   | Section                                                   |
| ---------------------------------------- | --------------------------------------------------------- |
| itérer vite sur le modèle                | [Mode watch](#mode-watch-sur-accord-explicite-de-lhumain) |
| `FileNotFoundException`, version du lock | [Version de `modgen`](#version-de-modgen)                 |
| dump `oneOf`, codes `TMDxxxx`            | [Décoder les erreurs](#décoder-les-erreurs)               |
| options spécifiques de `modgen`          | [Options](#options)                                       |
| code généré faux, modèle valide          | [Bug du générateur](#bug-du-générateur)                   |

Quel outil voit quelle erreur (`modgen` ou `modls`) : [`modls.md`](modls.md) § Qui attrape quoi.

## Mode watch (sur accord explicite de l'humain)

Boucle plus rapide que la copie temporaire : le modèle n'est chargé qu'une fois, chaque enregistrement de `.tmd` régénère en ~1-2 s. **Contrepartie : chaque version valide régénère le code et le lock dans le dépôt** — n'utiliser ce mode qu'avec l'accord de l'humain pour la session (l'agent peut le proposer, ci-dessous).

**En début de travail sur le modèle, choisir le mode avec l'humain** : si `git status` ne montre aucune modification en cours dans le modèle ni dans le code généré, **lui proposer** le mode watch dans le dépôt (le code généré reste lisible dans l'IDE et réversible par `git restore` / `git clean`). Sinon, rester sur la copie temporaire.

- Un seul watcher à la fois : vérifier qu'aucun `modgen` ne tourne déjà (Windows : `tasklist | grep -i modgen`, ligne de commande via `Get-CimInstance Win32_Process -Filter "Name='modgen.exe'"`) ; sinon, demander à l'humain de l'arrêter.
- Le lancer en tâche de fond, sortie redirigée vers un fichier temporaire : `modgen -f <config> -w > <tmp>/modgen-watch.log 2>&1`
- Après chaque édition, lire **uniquement les nouvelles lignes** du log (noter `wc -l` avant), attendre `Mise à jour terminée` (succès) ou un code `{TMDxxxx}` / un dump `oneOf` (modèle invalide, aucun code généré n'est écrit). En watch, les `.tmd` modifiés apparaissent aussi en `Créé:` / `Modifié:` / `Supprimé:` / `Renommé:` : ne pas les confondre avec des fichiers générés.
- Une modification de la configuration est prise en compte à chaud (le worker redémarre et recharge tout).
- Les diagnostics IDE remontés après un `Edit` sont un signal gratuit mais parfois décalés : alerte, pas preuve.
- **Avant de rendre la main**, la validation sur copie temporaire (étape 2 de la boucle de [`SKILL.md`](../../SKILL.md)) reste obligatoire.

## Version de `modgen`

- La version installée doit être **celle du lock** (`version:` de `topmodel.lock`). Sinon, avertissement `Ce modèle a été généré pour la dernière fois avec TopModel.Generator vX, qui n'est pas la version actuellement installée (vY)`, et `modgen` **réécrit la `version:` du lock** avec la sienne. Selon l'écart :
  - `modgen` plus récent : pas d'erreur, seul le lock change (à annuler ou signaler, [`SKILL.md`](../../SKILL.md) § Relecture) ;
  - `modgen` plus ancien que ce qu'exigent les modules du lock : erreur `Le module '…' référence une version plus récente de TopModel…` (ou « … pas la bonne version majeure… »), ou, pour un écart de version patch seulement, `FileNotFoundException: … TopModel.Generator.Core, Version=X` au chargement du module — **ce n'est pas une erreur de modèle**.
- Alignement : `dotnet tool update -g TopModel.Generator --version <version du lock>`, action sur le poste de l'humain : la lui demander.
  - Sans `--version`, la commande installe la dernière version, qui réécrirait la version du lock à la génération suivante.
  - Modifier le lock d'une copie ne contourne pas le problème.
- Montée de version de `modgen` (nouvelle version dans le lock) : décision de l'humain, faite dans un changement à part.
- Les **modules** de génération (`csharp`, `jpa`, `javascript`…) ont leur propre version, sous `modules:` du lock ; `modgen -u <module|all>` les met à jour.
- Version du language server : [`modls.md`](modls.md) § Version.

## Décoder les erreurs

Accents illisibles dans la sortie : [`outillage.md`](../outillage.md) § Sortie console.

### Dump `oneOf` (validation de schéma)

```cmd
Erreur dans le fichier ./model/Data/X.tmd :/oneOf/0:[required, Required properties ["module","tags"] are not present]/oneOf/1:… /oneOf/5/properties/class/properties/properties/items/$ref/oneOf/1/additionalProperties:[, All values fail against the false schema]…
```

Après `Erreur dans le fichier ./… :`, le validateur liste une ligne `{chemin}:{erreurs}` par échec, pour les 8 branches du document. **Ignorer les branches des autres types de documents** : 0 = entête, 1 = `annotation`, 2 = `domain`, 3 = `converter`, 4 = `decorator`, 5 = `class`, 6 = `endpoint`, 7 = `dataFlow`. Le vrai message est le chemin **le plus profond** sous la branche du document fautif, ex. pour une propriété de classe `oneOf/5/properties/class/…/items/$ref/oneOf/N` (`oneOf/4` pour un décorateur, `oneOf/6` pour les `params` / `returns` d'un endpoint) :

| Fragment                                                         | Sens                                                                               |
| ---------------------------------------------------------------- | ---------------------------------------------------------------------------------- |
| `additionalProperties: All values fail against the false schema` | une **clé interdite** dans cette variante de propriété — c'est presque toujours ça |
| `required, Required properties ["x"] are not present`            | clé obligatoire manquante dans la variante visée                                   |

Le `N` final indique la variante : 0 = propriété simple, 1 = association, 2 = alias, 3 = composition.

### Codes `TMDxxxx`

Message lisible, avec fichier et position : `./model/…/X.tmd[15,26] - {TMD0004} La propriété 'Type' est introuvable sur la classe 'Partie' (…)`. Les codes `TMD1xxx` sont des avertissements (désactivables par `noWarn:` dans la config).

`{TMD0011} La variable '6' est introuvable` sur un texte d'annotation contenant des accolades (quantificateur de regex `{6}`) : TopModel les lit comme une variable de template. Simple avertissement, le texte est généré tel quel ; le commenter dans le `.tmd` plutôt que réécrire la regex.

## Options

Options communes (`-f`, `-c`, `-w`, `-p`) : [`outillage.md`](../outillage.md) § Socle commun. Spécifiques à `modgen` :

| Option                       | Effet                                                                                                                                |
| ---------------------------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| `-e, --exclude <tag>`        | exclut des tags (répétable) ; **désactive la mise à jour du lock**, donc la suppression des fichiers générés devenus orphelins       |
| `-u, --update <module\|all>` | met à jour les modules de génération (dernière version publiée)                                                                      |
| `-s, --schema`               | régénère le schéma JSON de la **configuration** (`<config>.schema.json`) et le référence en tête de la config ; pas celui des `.tmd` |

`-c` régénère quand même le code si le modèle est valide : [`SKILL.md`](../../SKILL.md) § `modgen --check` n'est PAS en lecture seule.

## Bug du générateur

Le code généré est faux alors que le modèle est valide (ne compile pas, comportement erroné). Écarter d'abord les causes connues : [`regles/endpoints.md`](../regles/endpoints.md) § Code généré et [`regles/mappers.md`](../regles/mappers.md).

1. Vérifier que les versions installées correspondent au lock, et qu'une version plus récente du module ne corrige pas déjà (`modgen -u <module>`).
2. Le signaler à l'humain avec le symptôme et le fichier généré en cause ; c'est lui qui remonte le bug à l'éditeur ou décide d'attendre.
3. **Contournement seulement sur décision de l'humain** : retouche manuelle du fichier généré, annoncée comme temporaire (commit dédié qui le dit). Elle sera écrasée à la prochaine génération : la noter pour la réappliquer ou la retirer.
4. Correctif publié : `modgen -u all` (ou le module), régénérer, vérifier que la retouche a disparu du diff et que le lock porte la nouvelle version du module.
