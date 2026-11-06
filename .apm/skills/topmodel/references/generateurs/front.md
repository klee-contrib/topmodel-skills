# Effets du générateur front (TypeScript)

Référence de [`SKILL.md`](../../SKILL.md) : ce qui, dans le code généré, dépend du générateur front. Index : [`generateurs.md`](../generateurs.md). Les chemins et options exacts viennent toujours de la configuration (`topmodel.config`).

- Les fichiers générés côtoient souvent des fichiers écrits à la main : se fier à `generatedFiles:` du lock.
- `readonly: true` sur un DTO : sans effet.
- `enum: true` : liste `…List` + union de chaînes.
- `reference: true` : une définition de liste est générée, prête à enregistrer dans un store de référence :
  - `{list, …}` (valeurs copiées, aucun appel) pour une classe `readonly` — ce que force `enum: true` ;
  - sinon `{type, …}`, chargée du serveur au premier accès, **même si la classe a des `values:`** ;
  - `labelKey` vient du `defaultProperty`, déduit d'une propriété `Label` ou `Libelle` : sans l'un ni l'autre, il est **vide** et la liste s'affiche sans libellé.
- Annotations : non générées.
