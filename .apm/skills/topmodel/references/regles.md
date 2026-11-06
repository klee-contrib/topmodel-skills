# Règles et comportements de TopModel

Index de [`SKILL.md`](../SKILL.md) : ce que le générateur fait des clés, et les pièges qui en découlent, indépendamment du langage cible. Les clés autorisées sont dans [`syntaxe.md`](syntaxe.md) ; les effets propres à un langage dans [`generateurs.md`](generateurs.md) ; le décodage des erreurs dans [`outillage/modgen.md`](outillage/modgen.md) § Décoder les erreurs. Vérifié sur le code source de TopModel (résolveurs de `TopModel.Core`, générateurs) en 4.8.7.

**Lire un seul fichier selon le sujet :**

| Sujet                                                                                                                                                        | Fichier                                                                  |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------ |
| classe persistée ou DTO, `reference: true`, `enum`, `values`                                                                                                 | [`regles/classes.md`](regles/classes.md)                                 |
| noms générés, `useClass`, `withReverse`, `multiple` ; clé primaire et surcharges d'un alias ; groupe de champs répété (composition plutôt qu'alias préfixés) | [`regles/associations-alias.md`](regles/associations-alias.md)           |
| correspondances, listes, converters                                                                                                                          | [`regles/mappers.md`](regles/mappers.md)                                 |
| emplacement des paramètres, corps, fichiers générés, imports à compléter                                                                                     | [`regles/endpoints.md`](regles/endpoints.md)                             |
| visibilité d'une annotation, template ; décorateurs                                                                                                          | [`regles/annotations-decorateurs.md`](regles/annotations-decorateurs.md) |
| `uses:`, tags et références omises, `uses:` circulaires                                                                                                      | [`regles/fichiers-uses.md`](regles/fichiers-uses.md)                     |
| pièges sans message d'erreur clair (ordre des clés, tags)                                                                                                    | [`regles/invisibles.md`](regles/invisibles.md)                           |
