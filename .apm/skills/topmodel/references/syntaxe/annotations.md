# Syntaxe — annotations

Référence de [`SKILL.md`](../../SKILL.md). Entête de fichier et types de documents : [`fichier.md`](fichier.md).

## Annotations (`annotation`) — ex. validation regex sur un domaine

⚠️ **En v4, une annotation ne s'écrit jamais dans l'implémentation de langage** (`csharp:`, `ts:`…) d'un domaine : `csharp: { type, annotations: [{ text, imports }] }` est une syntaxe obsolète, rejetée par le schéma (dump `oneOf` illisible). Il faut **deux documents** :

1. un document `annotation` qui porte le code par langage ;
2. une référence **par nom** dans `annotations:` au niveau racine du domaine (ou d'une classe, propriété, endpoint, décorateur).

Exemple pour le générateur C# (la clé d'implémentation suit le générateur : `csharp:`, `java:`… ; détails : [`generateurs.md`](../generateurs.md)) :

```yaml
---
annotation:
  name: Numerique
  description: La valeur ne doit contenir que des chiffres.
  target: # optionnel : class, endpoint, client-endpoint, server-endpoint, property, association-property, composition-property, regular-property
    - property
  csharp: # LISTE d'implémentations
    - text: RegularExpression("^[0-9]*$") # sans le [ ] : le générateur l'ajoute
      imports:
        - System.ComponentModel.DataAnnotations
      # when: [...]  optionnel, toutes les conditions requises : class-property, endpoint-param, non-persisted, persisted, primary-key (depuis 4.8.7, primary-key exclut les associations)
---
domain:
  name: DO_CODE_POSTAL
  label: Code postal
  length: 5 # → taille de colonne (+ validation de longueur selon le générateur)
  annotations: # au niveau du domaine, PAS sous csharp:
    - Numerique
  ts: # une clé par générateur (ts:, csharp:, java:…)
    type: string
  csharp:
    type: string
```

Visibilité, accolades dans `text`, effet front et back : [`regles/annotations-decorateurs.md`](../regles/annotations-decorateurs.md) § Annotations.
