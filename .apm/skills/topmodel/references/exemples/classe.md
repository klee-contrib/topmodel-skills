# Exemples : classes

Extraits du modèle Utilisateur / Profil du starter kit, sauf mention contraire. Ce sont de bons exemples de constructions, pas des conventions métier : noms à adapter. Index : [`exemples.md`](../exemples.md). Règles : [`regles.md`](../regles.md), [`syntaxe.md`](../syntaxe.md).

## Entité persistée avec décorateur, `values` et `mappers`

```yaml
---
module: Model
tags:
  - back
uses:
  - decorators
  - References

---
class:
  name: Profil
  trigram: PRO
  label: Profil
  comment: Profil des utilisateurs
  decorators:
    - EntityListeners # (starter) colonnes de dates seulement, jamais renseignées : voir decorateur.md

  properties:
    - name: Id
      label: Id technique du profil
      primaryKey: true
      domain: DO_ID
      comment: Id technique

    - name: Libelle
      label: Libellé du profil
      domain: DO_LIBELLE
      required: true
      comment: Libellé du profil.

  mappers:
    to:
      - class: Profil
        mappings:
          Id: false
          DateCreation: false
          DateModification: false
  values:
    Profil1: { Id: 1, Libelle: "Profil 1", DateCreation: init }
```

`DateCreation: init` : surcharge `init` déclarée dans les implémentations par langage du domaine `DO_DATE_HEURE` (`csharp: { values: { overrides: { init: … } } }`, idem `sql:`…). `init` / `now` sont des clés libres du projet, pas des mots-clés TopModel.

Décorateur : [`decorateur.md`](decorateur.md).

## Table de jointure N-N

Même fichier : deux associations en clé primaire.

```yaml
class:
  name: DroitProfil
  trigram: DPR
  comment: N-N droits - profils

  properties:
    - association: Droit
      primaryKey: true
      comment: Droit.

    - association: Profil
      primaryKey: true
      comment: Profil.
  values:
    DroitProfil1: { DroitCode: "CREATE", ProfilId: "1" }
```

`DroitCode`, `ProfilId` : noms des FK générés sans `useClass` effectif ([`regles/associations-alias.md`](../regles/associations-alias.md) § Associations).

## Liste de référence

Extrait de [`References.tmd`](https://github.com/klee-contrib/topmodel/blob/develop/tests/model/Restaurant/References.tmd) (choix du flag : [`regles/classes.md`](../regles/classes.md)) :

```yaml
class:
  name: TypeUtilisateur
  trigram: TUT
  reference: true
  comment: Type d'utilisateur

  properties:
    - name: Code
      label: Type d'utilisateur
      domain: DO_CODE
      primaryKey: true
      comment: Code du type d'utilisateur

    - name: Libelle
      domain: DO_LIBELLE
      required: true
      comment: Libellé du type d'utilisateur

  values:
    Admin: { Code: ADMIN, Libelle: Administrateur }
```

## Contrainte d'unicité

```yaml
unique:
  - [Email]
```

## Héritage

Modèle de test officiel ; extraits parfois **tronqués** (`# …`) : les clés requises omises (`comment`, `properties`…) restent obligatoires.

`single-table`, avec une **association** comme discriminant :

```yaml
class:
  name: Plat
  inheritanceStrategy: single-table
  discriminatorProperty: CategoriePlat # nom généré de l'association CategoriePlat (useClass effectif)
  discriminatorValue: AUTRE
  # … comment, properties (dont l'association CategoriePlat)
---
class:
  name: PlatBoisson
  extends: Plat
  discriminatorValue: BOISSON
  # … comment, properties
```

`distinct-tables` avec classe mère abstraite (`Vaisselle` + `Assiette`/`Couvert`/`Verre`). Stratégies et associations vers une classe mère : [`syntaxe/classe.md`](../syntaxe/classe.md), [`regles/classes.md`](../regles/classes.md).
