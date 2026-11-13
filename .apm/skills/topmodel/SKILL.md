---
name: topmodel
description: Use when modelling data or APIs with TopModel - writing or editing .tmd files, adding an entity, reference list, enum, association, composition, command, query or endpoint, running modgen, when generating .tmd files from an OpenAPI specification or an existing database (reverse-engineering, tmdgen), or when generated files (C#, Java, TypeScript…) are stale, missing, fail to compile, or seem wrong because of a generator bug.
---

# TopModel

Le modèle (`*.tmd`, en YAML) est la source de vérité. `modgen` en génère le code (C#, TypeScript…) selon les générateurs déclarés dans la configuration (`topmodel.config`), qui fait foi pour les tags et les chemins de sortie.

**Ne jamais modifier le code généré. Corriger uniquement le `.tmd` et régénèrer** Deux exceptions encadrées : les points d'extension prévus par le générateur et le protocole « bug du générateur » ([`references/outillage/modgen.md`](references/outillage/modgen.md)).

## Liens utiles

- [Documentation TopModel](https://klee-contrib.github.io/topmodel)
- Starters kit :
  - [Dotnet - React](https://github.com/klee-contrib/dfta-starter-kit)
  - [Spring - Angular](https://github.com/klee-contrib/asta-starter-kit)

## Boucle de travail obligatoire

1. Éditer le `.tmd`
2. **Vérification qui fait autorité** — sur copie temporaire (recette ci-dessous)
3. Corriger, relancer, jusqu'à ce que ce soit propre
4. Génération réelle `modgen -f <config>`, lancée et annoncée par l'agent — **seulement une fois les étapes 2 et 3 propres** (inutile en mode watch)
5. Traiter la glu (ci-dessous), sinon le build casse

- **Mode watch** : écrit dans le dépôt ; à proposer si l'arbre est propre, jamais sans accord de l'humain ([`references/outillage/modgen.md`](references/outillage/modgen.md)).
- **« Ce modèle a été généré pour la dernière fois avec… »** (avertissement), éventuellement suivi d'une `FileNotFoundException` ou d'une erreur « Le module … référence une version … » : version de `modgen` différente du lock, pas une erreur de modèle ([`references/outillage/modgen.md`](references/outillage/modgen.md) § Version de `modgen`).

### ⚠️ `modgen --check` n'est PAS en lecture seule

Il sert à vérifier, en CI/CD, que le code généré est à jour par rapport au modèle. Concrètement :

1. avant tout, il réécrit le lock (`version:` = celle de l'outil installé) et son cache `.modgen/` ;
2. il charge et valide le modèle — invalide : exit 1, **aucun code généré n'est écrit** ;
3. valide : **il régénère tout le code** (et supprime les fichiers générés qui ne le sont plus), puis sort en 1 si un fichier a été créé, modifié ou supprimé — lock compris —, 0 sinon.

Ne jamais le lancer dans le dépôt pour « juste valider » : utiliser la copie temporaire.
Ne lancer que pour vérifier que le code généré n'a pas été modifié, seulement à la fin d'un travail hors fichiers tmd. Le relancer après un changement de `.tmd` est inutile : le code généré est forcément modifié, et le lock aussi.

### Recette de validation sans écrire dans le dépôt

Copier la config, le lock, le modèle **et les dossiers de sortie** (`outputDirectory` de chaque générateur de la config, relatif à la config, souvent en `../…`), en conservant leurs positions relatives : sans le code généré existant, tout sort « Créé » et le code de retour ne dit plus rien.

```bash
TMP=$(mktemp -d)   # ou un dossier ignoré du dépôt si l'environnement interdit d'écrire ailleurs
cp -r <dossier du modèle> <dossiers de sortie de la config> "$TMP/"   # ex. model sources env
cd "$TMP/<dossier du modèle>" && modgen -f <config> --check; echo "exit=$?"
```

Lecture du résultat :

- erreurs `TMDxxxx` ou dump `oneOf`, exit 1 → modèle invalide, aucun code généré n'est écrit ;
- `Mise à jour terminée avec succès.` puis exit 1 → modèle valide, des fichiers changent (attendu après une modification du `.tmd`) : relire les `Créé:` / `Modifié:` / `Supprimé:` pour vérifier que seuls les fichiers attendus bougent (un `Modifié: …topmodel.lock` seul = simple changement de version de l'outil) ;
- exit 0 → modèle valide et code généré déjà à jour.

Diagnostics complets (schéma **et** sémantique), avec position ; le code généré atterrit dans `$TMP`. **LSP silencieux ne veut pas dire valide** : il ne publie aucun diagnostic pour les erreurs de schéma ([`references/outillage/modls.md`](references/outillage/modls.md) § Qui attrape quoi).

## Lire la référence AVANT d'écrire

Ne pas écrire de `.tmd` de mémoire : les variantes de propriétés ont des listes de clés **fermées**.

| Ce que tu écris ou cherches | À lire d'abord |
| --- | --- |
| n'importe quoi de non trivial | les `.tmd` existants du projet (prioritaires : ils portent les conventions locales), puis [`references/exemples.md`](references/exemples.md) |
| une classe, une propriété (`association`, `alias`, `composition`), un domaine, un endpoint, une annotation | [`references/syntaxe.md`](references/syntaxe.md) (index) → un seul fichier : [fichier et types de documents](references/syntaxe/fichier.md), [classe](references/syntaxe/classe.md), [propriétés](references/syntaxe/proprietes.md) ou [annotations](references/syntaxe/annotations.md) |
| ce que le générateur fait de ces clés : persistée ou DTO, `reference: true`, noms générés, mappers, paramètres d'endpoint, `uses:` et cycles | [`references/regles.md`](references/regles.md) (index) → un seul fichier : [classes](references/regles/classes.md), [associations et alias](references/regles/associations-alias.md), [mappers](references/regles/mappers.md), [endpoints](references/regles/endpoints.md), [annotations et décorateurs](references/regles/annotations-decorateurs.md), [`uses:` et tags](references/regles/fichiers-uses.md) |
| ce qu'un générateur donné (C#, Java, front) produit concrètement : noms, emplacements, types | [`references/generateurs.md`](references/generateurs.md) (index) → un seul fichier : [C#](references/generateurs/csharp.md), [Java](references/generateurs/java.md) ou [front](references/generateurs/front.md), selon le générateur de la configuration |
| un groupe de champs répété (deux rôles d'un même concept) | [`references/regles/associations-alias.md`](references/regles/associations-alias.md) § Composition plutôt qu'alias préfixés |
| un message d'erreur (dump `oneOf`, `TMDxxxx`), une option de `modgen` | [`references/outillage/modgen.md`](references/outillage/modgen.md) § Décoder les erreurs |
| qui référence une classe, avant de la renommer ou de la supprimer | [`references/outillage/modls.md`](references/outillage/modls.md) § Explorer le modèle existant |
| les trois outils en ligne de commande (`modgen`, `tmdgen`, `modls`), options communes, produire des `.tmd` depuis OpenAPI ou une base | [`references/outillage.md`](references/outillage.md) (index) → un seul fichier par outil |
| un besoin sans syntaxe TopModel (index full text, colonne calculée) | point d'extension de la couche de persistance (hors TopModel) |

## Glu (côté TopModel)

| Tu as ajouté ou modifié | Tu dois aussi |
| --- | --- |
| renommé ou supprimé une classe ou un fichier | vérifier les `Supprimé:` de la génération (`modgen` supprime lui-même les fichiers sortis de `generatedFiles:` du lock — sauf génération avec `-e`, qui ne met pas le lock à jour) et corriger les usages dans le code écrit à la main. Ne jamais supprimer à la main un fichier à en-tête « GENERE AUTOMATIQUEMENT » sur ce seul critère : le lock lui-même en porte un, et certains fichiers générés n'en ont pas |
| une classe persistée, des `values:`, une classe `enum: true`, un renommage persisté, un domaine, un endpoint, une liste de référence | traiter la glu côté stack (migration, enum des domaines, contrats, store de référence) |

## Quels fichiers sont générés

- La section `generatedFiles:` du lock fait autorité : les fichiers générés vivent dans des dossiers `generated/` ou côtoient des fichiers écrits à la main (fréquent côté front).

## Relecture

À passer sur le diff dès que le modèle ou du code généré a changé :

- aucun fichier généré édité à la main (hors contournement décidé par l'humain) ;
- la glu (ci-dessus) est traitée ;
- les `uses:` circulaires respectent les règles du projet ([`references/regles/fichiers-uses.md`](references/regles/fichiers-uses.md) § `uses:` circulaires) ;
- `reference: true` justifié pour chaque classe qui le porte ; aucun groupe d'alias préfixés répété ;
- les fichiers générés ne changent que par l'effet du `.tmd` modifié ; un changement de version dans le lock est annulé ou signalé ;
- toute modification d'un objet existant (renommage, domaine, suppression) est listée à l'humain avec ses impacts : migration, API cassante, front ;
- la validation sur copie temporaire a été relancée en dernier et est propre.

## Red flags — arrêter et reprendre

- Tu t'apprêtes à éditer un fichier listé dans le lock → édite le `.tmd` (ou un point d'extension)
- Tu as écrit du `.tmd` sans le valider → `modgen` **sur copie temporaire**
- Tu t'apprêtes à lancer `modgen` (même `--check`) dans le dépôt pour valider → il écrira le code généré
- Tu répètes les mêmes alias avec `prefix: A` puis `prefix: B` → une classe DTO et deux `composition`
- Tu poses `reference: true` sur une entité volumineuse ou importée → lecture par des queries, pas liste de référence
- Tu écris `annotations:` sous l'implémentation de langage d'un domaine (`csharp:`, `java:`…) → syntaxe obsolète ([`references/syntaxe/annotations.md`](references/syntaxe/annotations.md))
- Tu écris une association de mémoire → la clé `name` n'existe pas sur une association (`role:`)
- Tu écris une propriété avec une autre clé en premier que `name` / `association` / `alias` / `composition` → la clé discriminante doit venir en premier, sans message d'erreur clair ([`references/regles/invisibles.md`](references/regles/invisibles.md))
- Tu cites une association par son nom (`values:`, `unique:`, `exclude:`, `discriminatorProperty`) → nom généré, qui dépend de `useClass` et de `defaultAssociationUseClass` dans la config ([`references/regles/associations-alias.md`](references/regles/associations-alias.md) § Associations)
- Tu poses `primaryKey: true` dans un DTO → il devient persisté et génère une table
- Une composition, une association ou un mapper manque dans le code généré sans erreur → tags du fichier référencé ([`references/regles/fichiers-uses.md`](references/regles/fichiers-uses.md) § Visibilité entre fichiers)
- Le code généré te paraît faux → [`references/outillage/modgen.md`](references/outillage/modgen.md) § Bug du générateur, pas de retouche silencieuse
