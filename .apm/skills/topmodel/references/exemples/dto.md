# Exemples : DTO et mappers

Extraits du modèle Utilisateur / Profil du starter kit, sauf mention contraire. Index : [`exemples.md`](../exemples.md). Règles : [`regles.md`](../regles.md), [`syntaxe.md`](../syntaxe.md).

## DTO de lecture : `alias`, `as: list`, `prefix`/`suffix`, `composition`

Le cas le plus riche :

```yaml
---
module: Model
tags:
  - queries
  - front
uses:
  - Data/Profil
  - Queries/Utilisateur
  - References

---
class:
  name: ProfilRead
  comment: Détail d'un profil en lecture.

  properties:
    - alias:
        class: Profil
        exclude:
          - DateCreation
          - DateModification

    - alias:
        class: Profil
        include:
          - DateCreation
          - DateModification
      defaultValue: null
      domain: DO_DATE_HEURE

    - alias:
        class: Droit
        property: Code
      as: list
      prefix: true
      suffix: s

    - composition: UtilisateurItem
      name: Utilisateurs
      domain: DO_LISTE
      comment: Utilisateurs ayant ce profil.
```

## DTO d'écriture et chaîne Data → Commands → Api

Tag `commands` :

```yaml
class:
  name: UtilisateurCommand
  comment: Détail d'un utilisateur en écriture.
  properties:
    - alias:
        class: Utilisateur
        exclude:
          - Id
          - DateCreation
          - DateModification

  mappers:
    to:
      - class: Utilisateur
```

puis un fichier taggé `api-commands`, `front` réalise le même aliasing sur `UtilisateurCommand` et mappe vers lui.

## `mappers from` multi-paramètres

Modèle de test officiel :

```yaml
mappers:
  from:
    - params:
        - class: Restaurant
          name: restaurant
          required: true
        - property:
            name: NombreCommandes
            domain: DO_QUANTITE
            comment: Nombre de commandes
```
