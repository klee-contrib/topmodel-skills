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

Avant de commencer, noter le `git status` du modèle et du code généré : la génération écrit dans le dépôt, un arbre propre permet d'isoler ses effets et de les annuler (`git restore` / `git clean`). Si des fichiers générés ont déjà des modifications en cours, le signaler à l'humain avant de générer.

1. Éditer le `.tmd` **uniquement avec l'outil d'édition de fichiers de l'agent**, jamais via le terminal (heredoc, `sed`, `cp`…) : un fichier écrit hors de cet outil n'est pas signalé au language server `modls`, qui ne renvoie alors aucun diagnostic
2. Si des diagnostics `modls` (codes `TMDxxxx`) sont remontés après l'écriture, les corriger avant de générer
3. **Générer** : `modgen -f <config>`, lancé et annoncé par l'agent (inutile en mode watch)
4. **Vérifier que le résultat est conforme à l'attendu** (ci-dessous) ; sinon corriger le `.tmd` et relancer
5. Traiter la glu (ci-dessous), sinon le build casse

- **Mode watch** : à proposer si l'arbre est propre, jamais sans accord de l'humain ([`references/outillage/modgen.md`](references/outillage/modgen.md)).
- **« Ce modèle a été généré pour la dernière fois avec… »** (avertissement), éventuellement suivi d'une `FileNotFoundException` ou d'une erreur « Le module … référence une version … » : version de `modgen` différente du lock, pas une erreur de modèle ([`references/outillage/modgen.md`](references/outillage/modgen.md) § Version de `modgen`).

### Lire le résultat de la génération

- erreurs `TMDxxxx` ou dump `oneOf` → modèle invalide, aucun code généré n'est écrit : corriger le `.tmd` et relancer ([`references/outillage/modgen.md`](references/outillage/modgen.md) § Décoder les erreurs) ;
- `Mise à jour terminée avec succès.` → relire les `Créé:` / `Modifié:` / `Supprimé:` : chaque fichier doit découler du `.tmd` modifié (une classe → son entité ou DTO, ses mappers ; un endpoint → son contrôleur ou client ; des libellés → les ressources). Cette liste suffit : n'ouvrir un fichier généré que s'il est inattendu (classe non touchée modifiée, suppression), avec `git diff` sur ce fichier ;
- `Modifié: …topmodel.lock` : attendu quand la liste des fichiers générés change ; un changement de `version:` est à annuler ou signaler.

**LSP silencieux ne veut pas dire valide** : `modls` ne publie aucun diagnostic pour les erreurs de schéma ([`references/outillage/modls.md`](references/outillage/modls.md) § Qui attrape quoi), c'est `modgen` qui fait autorité. Le serveur démarre au premier `.tmd` écrit de la session et met ~5 s à répondre : le premier diagnostic peut arriver avec un léger décalage.

### `modgen --check` est réservé à la CI

Il vérifie que le code généré est à jour par rapport au modèle (pipelines CI/CD) : ce n'est **pas** un outil de validation d'un modèle en cours d'édition. Il n'est pas en lecture seule : il réécrit le lock et son cache `.modgen/`, puis régénère tout le code et sort en 1 si un fichier a changé. Après une modification de `.tmd`, il échoue forcément : utiliser `modgen -f <config>`.

## Respecter l'existant

- **Un DTO par usage** : chaque usage (création, mise à jour, liste, détail…) a son propre DTO, même si un DTO proche existe déjà ailleurs dans le modèle. Si plusieurs DTO partagent réellement un socle et que le contexte le justifie, factoriser par héritage (`extends`, [`references/syntaxe/classe.md`](references/syntaxe/classe.md)), jamais en réutilisant le DTO d'un autre usage.
- **Nommer selon la convention du projet** : la relever sur les classes et fichiers existants de même rôle, et l'appliquer à l'identique — même racine (le nom **exact** de la classe source, sans le passer au singulier ni au pluriel : `Profils` → `ProfilsCreation`, jamais `ProfilCreation`), mêmes suffixes de rôle, même langue, même nommage des fichiers `.tmd`. Si le nom obtenu est déjà pris dans le module, changer le **suffixe** (de préférence un suffixe déjà employé dans le projet), jamais la racine, et **sans demander** : deux classes de même nom dans un module ne sont pas forcément signalées, et la seconde écrase le code généré de la première.
- **Ne pas déplacer, renommer, supprimer ni modifier une classe ou un fichier existant** pour l'adapter à la tâche : créer à côté. Ne demander à l'humain que si la tâche ne peut pas être faite sans modifier l'existant ; sinon, choisir l'option qui ne le touche pas et poursuivre.
- **Ne pas modifier une classe persistée existante** (propriétés, `required`, domaines…) si la tâche ne le demande pas : migration de base, API cassante.

### Erreurs les plus fréquentes

- Paramètre de route d'un alias de clé primaire : `{<trigramme, sinon classe en camelCase><Propriété>}` — `Profils.Id` sans trigramme → `{profilsId}`, pas `{profilId}` (sinon `TMD7005`).
- Toute classe citée (alias, composition, association, `returns`) doit être visible via `uses:` (sinon `TMD0002`) ; un `uses:` inutilisé déclenche `TMD1003`.

## Quand lire les références

**Le projet fait référence.** Si un `.tmd` existant du projet contient déjà la même construction (entité, DTO, endpoint, mapper, annotation…), la reproduire en l'adaptant, **sans ouvrir `references/`**. Ne pas inventer de clé absente des `.tmd` du projet : les variantes de propriétés ont des listes de clés **fermées**.

N'ouvrir une référence que dans deux cas :

- la construction n'existe nulle part dans le projet ;
- une erreur `TMDxxxx` ou un dump `oneOf` que tu ne sais pas corriger : ouvrir alors le fichier qui traite de cette erreur, pas les autres.

**Budget : au plus 2 fichiers de `references/` par tâche, un seul par question.** Écrire d'abord, générer, et laisser les erreurs dire quoi lire. Ne pas lire `references/generateurs/` pour vérifier une génération réussie : § Lire le résultat de la génération suffit.

| Si tu en as besoin (cas ci-dessus) | Lire un seul fichier |
| --- | --- |
| une construction absente du projet | [`references/exemples.md`](references/exemples.md) (index), puis l'exemple correspondant |
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
- la génération a été relancée en dernier, sans erreur, et ses `Créé:` / `Modifié:` / `Supprimé:` correspondent à l'attendu.

## Red flags — arrêter et reprendre

- Tu t'apprêtes à éditer un fichier listé dans le lock → édite le `.tmd` (ou un point d'extension)
- Tu as écrit du `.tmd` sans générer → `modgen -f <config>` et lire le résultat
- Tu t'apprêtes à déplacer, modifier ou supprimer une classe existante pour la réutiliser → un DTO dédié à l'usage, nommé selon la convention du projet (héritage si le contexte le justifie)
- Tu t'arrêtes pour demander alors qu'une option ne touchant pas l'existant permet de finir (autre nom, DTO dédié) → la prendre et poursuivre
- Tu modifies une classe persistée existante alors que la tâche ne le demande pas → arrêter et demander à l'humain
- Tu t'apprêtes à écrire un `.tmd` depuis le terminal (heredoc, `sed`, `cat >`) → outil d'édition de fichiers de l'agent, sinon aucun diagnostic `modls`
- Tu t'apprêtes à lancer `modgen --check` pour valider un modèle en cours d'édition, ou à générer dans une copie temporaire → `modgen -f <config>` dans le dépôt, `--check` est réservé à la CI
- Tu répètes les mêmes alias avec `prefix: A` puis `prefix: B` → une classe DTO et deux `composition`
- Tu poses `reference: true` sur une entité volumineuse ou importée → lecture par des queries, pas liste de référence
- Tu écris `annotations:` sous l'implémentation de langage d'un domaine (`csharp:`, `java:`…) → syntaxe obsolète ([`references/syntaxe/annotations.md`](references/syntaxe/annotations.md))
- Tu écris une association de mémoire → la clé `name` n'existe pas sur une association (`role:`)
- Tu écris une propriété avec une autre clé en premier que `name` / `association` / `alias` / `composition` → la clé discriminante doit venir en premier, sans message d'erreur clair ([`references/regles/invisibles.md`](references/regles/invisibles.md))
- Tu cites une association par son nom (`values:`, `unique:`, `exclude:`, `discriminatorProperty`) → nom généré, qui dépend de `useClass` et de `defaultAssociationUseClass` dans la config ([`references/regles/associations-alias.md`](references/regles/associations-alias.md) § Associations)
- Tu poses `primaryKey: true` dans un DTO → il devient persisté et génère une table
- Une composition, une association ou un mapper manque dans le code généré sans erreur → tags du fichier référencé ([`references/regles/fichiers-uses.md`](references/regles/fichiers-uses.md) § Visibilité entre fichiers)
- Le code généré te paraît faux → [`references/outillage/modgen.md`](references/outillage/modgen.md) § Bug du générateur, pas de retouche silencieuse
