# Exemples : endpoints

Index : [`exemples.md`](../exemples.md). Préfixe de route dans l'entête (`options`) et règles des paramètres : [`regles/endpoints.md`](../regles/endpoints.md).

## Endpoint avec paramètre de route et corps

Modèle de test officiel :

```yaml
endpoint:
  name: UpdateClient
  method: PUT
  route: /clients/{perId}
  description: Met à jour un client
  params:
    - alias:
        class: Personne
        property: Id
    - composition: ClientWrite
      comment: Client à mettre à jour
      name: client
  returns:
    composition: ClientRead
    name: client
    comment: Client mis à jour
```

## Endpoints d'envoi / de téléchargement de fichier

Starter kit. Domaines `DO_FILE_UPLOAD` / `DO_FILE_DOWNLOAD`.

```yaml
endpoint:
  name: AddUtilisateurPhoto
  method: PUT
  route: api/utilisateurs/{utiId}/photo
  description: Ajoute une nouvelle photo pour un utilisateur.
  params:
    - alias:
        class: Utilisateur
        property: Id
    - name: Photo
      domain: DO_FILE_UPLOAD
      required: true
      comment: Photo.
---
endpoint:
  name: GetUtilisateurPhoto
  method: GET
  route: api/utilisateurs/{utiId}/photo
  description: Charge la photo d'un utilisateur (si elle existe).
  params:
    - alias:
        class: Utilisateur
        property: Id
  returns:
    name: Photo
    domain: DO_FILE_DOWNLOAD
    comment: Photo.
```

## Endpoint de recherche paginée (critères + `DO_QUERY_INPUT` / `DO_QUERY_OUTPUT`)

Starter kit :

```yaml
class:
  name: UtilisateurCriteria
  comment: Critère de recherche pour les utilisateurs.
  decorators:
    - Criteria # décorateur du starter : Query, SearchFields, SourceFields
  properties:
    - alias:
        class: Utilisateur
        exclude:
          - Id
          - DateCreation
          - DateModification
      defaultValue: null
      required: false # tous les critères sont facultatifs
---
endpoint:
  name: SearchUtilisateur
  method: POST
  route: api/utilisateurs/search
  description: Recherche des utilisateurs
  params:
    - composition: UtilisateurCriteria
      name: queryInput
      domain: DO_QUERY_INPUT
      comment: Critères de recherche
  returns:
    composition: UtilisateurItem
    name: output
    domain: DO_QUERY_OUTPUT
    comment: Utilisateurs matchant les critères
```
