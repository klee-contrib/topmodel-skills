# Règles invisibles

Référence de [`SKILL.md`](../../SKILL.md). Pièges mal ou pas signalés par les erreurs.

1. La clé discriminante d'une propriété (`name`, `alias`, `association`, `composition`) doit être **en premier** dans la map YAML. Le schéma ne le vérifie pas ; le chargement échoue avec « Le premier attribut de la propriété doit être … ».
2. Les `tags` d'un fichier décident quels générateurs le traitent ; ceux posés sur une **propriété** la restreignent (`tags: []` = jamais générée, sauf paramètre de route, toujours conservé). Aucune erreur. Une référence vers un objet sans tag commun est omise : [`fichiers-uses.md`](fichiers-uses.md) § Visibilité entre fichiers.
