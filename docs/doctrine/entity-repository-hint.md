# EntityRepository et query hints par défaut

Vous pouvez définir des query hints par défaut qui sont censés être pris en compte dans toutes vos requêtes exécutées via Doctrine.

Plus précisément, et c'est là le soucis : ces query hints ne sont interprêtés que lors d'une requête `DQL`.

Si vous ne surchargez pas le `Repository` par défaut pour votre entité, ou que votre surcharge étend de `Doctrine\ORM\EntityRepository`, alors les méthodes `find()`, `findAll()`, `findBy()` et `findOneBy()` ne prendront pas en compte votre query hints parcequ'elles ne passent pas par du `DQL` mais exécutent directement du `SQL`.

Attention donc à ces 4 méthodes qui peuvent avoir des comportements qui ne sont pas ceux attendus !

# Références

[Documentation sur le DQL](https://www.doctrine-project.org/projects/doctrine-orm/en/2.8/reference/dql-doctrine-query-language.html)

[Documentation sur les Query Hints](https://www.doctrine-project.org/projects/doctrine-orm/en/2.8/reference/dql-doctrine-query-language.html#query-hints)

[Issue #6751 sur doctrine/doctrine2](https://github.com/doctrine/doctrine2/issues/6751)

[Exemple de code](https://github.com/steevanb/doctrine-repository-hint-bug)
