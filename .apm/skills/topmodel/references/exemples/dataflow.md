# Exemples : `dataFlow` (BETA)

Modèle de test officiel. Index : [`exemples.md`](../exemples.md).

```yaml
dataFlow:
  name: ExportRestaurants
  target: External
  class: RestaurantRead
  type: insert
  sources:
    - class: Restaurant
      mode: queryAll
      source: Primary
  dependsOn:
    - ExportMenus
  hooks:
    - afterSource
    - map
    - beforeTarget
```
