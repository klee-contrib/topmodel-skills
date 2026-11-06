# Exemples : décorateur

Index : [`exemples.md`](../exemples.md).

```yaml
decorator:
  name: TimestampedEntity
  description: Date création
  # Dates renseignées à l'enregistrement par TimestampsInterceptor (Clients.Db), via cette interface.
  csharp:
    implements:
      - ITimestampedEntity
    imports:
      - MonApp.Clients.Db.Timestamps # namespace anonymisé
  properties:
    - name: DateCreation
      label: Date de création
      required: true
      defaultValue: now
      domain: DO_DATE_HEURE
      comment: Date de création de l'enregistrement.

    - name: DateModification
      label: Date de modification
      domain: DO_DATE_HEURE
      comment: Date de modification de l'enregistrement.
```

Ce que le décorateur ne fait pas (renseigner les dates), décorateur sans propriété : [`regles/annotations-decorateurs.md`](../regles/annotations-decorateurs.md) § Décorateurs. Le décorateur du starter (`EntityListeners`, sans `csharp:`) n'ajoutait que les colonnes.
