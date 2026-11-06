# Effets du générateur Java

Référence de [`SKILL.md`](../../SKILL.md) : ce qui, dans le code généré, dépend du générateur Java. Index : [`generateurs.md`](../generateurs.md). Les chemins et options exacts viennent toujours de la configuration (`topmodel.config`).

- **`readonly: true` sur un DTO** : record sans setter **seulement** si l'option `useRecords` est active dans la configuration JPA (classe non persistée, `readonly`, `regular`) ; sinon les setters sont générés.
- **Mappers** : `from:` produit une méthode statique `create{Classe}` (et `map{Classe}` vers une cible existante) dans la classe de mappers ; `to:` produit `to{Cible}`, ou `create{Cible}` si la cible est un record.
- **Annotations** et **converters** : implémentation sous `java:`.
- **DAO** : les DAO ne sont générés qu'une seule fois, afin d'être modifiables.
