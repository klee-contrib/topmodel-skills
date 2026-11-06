# Exemples : associations

Extraits du modèle de test officiel ([`tests/model/`](https://github.com/klee-contrib/topmodel/tree/develop/tests/model)), parfois **tronqués**. Attention : la config de ce modèle pose `defaultAssociationUseClass: true`, ce qui change les noms générés des associations ([`regles/associations-alias.md`](../regles/associations-alias.md) § Associations). Index : [`exemples.md`](../exemples.md).

## Réciproque, rôle, propriété cible

```yaml
- association: Restaurant
  required: true
  withReverse: true
  comment: Restaurant proposant ce menu

- association: Commande
  required: true
  withReverse:
    className: Lignes
  comment: Commande à laquelle appartient la ligne

- association: Table
  useClass: false
  comment: Table associée à la commande

- association: TypeFacture
  property: Code
  comment: type de facture
```

Deux associations vers la **même** classe cible — c'est `role`, jamais `name` (extrait de [`properties.md`](https://github.com/klee-contrib/topmodel/blob/develop/docs/docs/model/properties.md)) :

```yaml
- association: ClasseCible
  required: true
  comment: C'est une FK obligatoire
  role: Exemple
```

→ propriété générée `ClasseCibleIdExemple` (PK `Id`, `useClass` non effectif) ; `ClasseCibleExemple` si `useClass` est effectif.
