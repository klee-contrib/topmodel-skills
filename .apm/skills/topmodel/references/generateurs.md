# Effets propres à chaque générateur

Référence de [`SKILL.md`](../SKILL.md) : ce qui, dans le code généré, dépend du langage cible. Les règles indépendantes du langage sont dans [`regles.md`](regles.md) ; les clés autorisées dans [`syntaxe.md`](syntaxe.md). Les chemins et options exacts viennent toujours de la configuration (`topmodel.config`).

Un fichier par générateur : n'ouvrir que celui du générateur déclaré dans la configuration.

| Générateur         | Fichier                                          |
| ------------------ | ------------------------------------------------ |
| C#                 | [`generateurs/csharp.md`](generateurs/csharp.md) |
| Java               | [`generateurs/java.md`](generateurs/java.md)     |
| TypeScript / front | [`generateurs/front.md`](generateurs/front.md)   |
