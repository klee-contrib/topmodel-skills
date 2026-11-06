# Règles invisibles

Référence de [`SKILL.md`](../../SKILL.md). Pièges mal ou pas signalés par les erreurs.

1. La clé discriminante d'une propriété (`name`, `alias`, `association`, `composition`) doit être **en premier** dans la map YAML. Le schéma ne le vérifie pas ; le chargement échoue sans code TMD, avec « Le premier attribut de la propriété doit être … », ou un message trompeur (« Propriété association inconnue… ») si `name` précède `association`.
2. `discriminatorProperty` doit pointer une propriété déclarée, par son **nom généré** (sinon `TMD0004`) ; pour une association, voir [`associations-alias.md`](associations-alias.md) § Associations (ex. `TypeClient` vers une classe `enum: true`, `TypeClientCode` vers une classe ordinaire sans `useClass`).
3. Les `tags` d'un fichier décident quels générateurs le traitent ; ceux posés sur une **propriété** la restreignent (`tags: []` = jamais générée, sauf paramètre de route, toujours conservé). Aucune erreur. Une référence vers un objet sans tag commun est omise : [`fichiers-uses.md`](fichiers-uses.md) § Visibilité entre fichiers.
