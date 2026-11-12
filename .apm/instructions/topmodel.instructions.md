---
description: Règles TopModel pour les fichiers de modèle (.tmd) et la configuration
applyTo: "**/*.tmd,**/topmodel.config,**/topmodel*.config"
---

# TopModel

- Avant de créer ou modifier un fichier `.tmd` (entité, liste de référence, enum, association, composition, commande, requête, endpoint), **invoque le skill `topmodel`**.
- Le `.tmd` est la source de vérité : ne modifie jamais le code généré (C#, Java, TypeScript…). Corrige le `.tmd` puis relance `modgen`.
- Après modification du modèle, commence par vérifier les diagnostics du serveur LSP `modls`. S'il n'y a pas d'erreur, régénère avec `modgen`.
