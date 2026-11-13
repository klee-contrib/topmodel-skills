# Exemples TopModel

Trois sources, toutes authentiques :

- **Le projet** — les `.tmd` existants, à imiter en priorité car ils portent les conventions locales.
- **Le starter kit** (modèle Utilisateur / Profil) — pour les cas récurrents d'une application de gestion.
- **Le modèle de test officiel** — <https://github.com/klee-contrib/topmodel/tree/develop/tests/model>, pour les constructions plus rares.

Ces fichiers ne contiennent que du code illustratif : les règles qu'ils illustrent sont dans [`regles.md`](regles.md) et [`syntaxe.md`](syntaxe.md). Un fichier par type de construction : n'ouvrir que celui dont on a besoin.

| Construction | Fichier |
| --- | --- |
| entité persistée, `values`, `mappers to`, table de jointure N-N, liste de référence, unicité, héritage | [`exemples/classe.md`](exemples/classe.md) |
| associations : réciproque, rôle, propriété cible | [`exemples/associations.md`](exemples/associations.md) |
| DTO de lecture / d'écriture, `alias`, `as: list`, `composition`, `mappers from` | [`exemples/dto.md`](exemples/dto.md) |
| décorateur | [`exemples/decorateur.md`](exemples/decorateur.md) |
| endpoints : route + corps, fichiers, recherche paginée | [`exemples/endpoint.md`](exemples/endpoint.md) |
| domaine à séquence, converter | [`exemples/domaine.md`](exemples/domaine.md) |
| `dataFlow` | [`exemples/dataflow.md`](exemples/dataflow.md) |

## Où trouver plus d'exemples officiels

| Besoin | Chemin dans `klee-contrib/topmodel` |
| --- | --- |
| modèle de test complet | [`tests/model/`](https://github.com/klee-contrib/topmodel/tree/develop/tests/model) |
| domaines, décorateurs, annotations | [`tests/model/meta/`](https://github.com/klee-contrib/topmodel/tree/develop/tests/model/meta) |
| entités, DTOs, endpoints, flows | [`tests/model/Restaurant/`](https://github.com/klee-contrib/topmodel/tree/develop/tests/model/Restaurant) |
| doc par construction | [`classes`](https://github.com/klee-contrib/topmodel/blob/develop/docs/docs/model/classes.md), [`properties`](https://github.com/klee-contrib/topmodel/blob/develop/docs/docs/model/properties.md), [`domains`](https://github.com/klee-contrib/topmodel/blob/develop/docs/docs/model/domains.md), [`decorators`](https://github.com/klee-contrib/topmodel/blob/develop/docs/docs/model/decorators.md), [`endpoints`](https://github.com/klee-contrib/topmodel/blob/develop/docs/docs/model/endpoints.md), [`mappers`](https://github.com/klee-contrib/topmodel/blob/develop/docs/docs/model/mappers.md), [`dataFlows`](https://github.com/klee-contrib/topmodel/blob/develop/docs/docs/model/dataFlows.md) |

Contenu brut : `https://raw.githubusercontent.com/klee-contrib/topmodel/develop/<chemin>`
