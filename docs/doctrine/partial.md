# Sélectionner certains champs d'une table

Si vous voulez sélectionner seulement certains champs d'une table parmis les champs mappés, vous pouvez utiliser le mot-clé `PARTIAL` :

```php
/** @Entity */
class Foo
{
    /** @Column(type="integer") */
    protected ?int $id = null;

    /** @Column(length=140) */
    protected ?string $name = null;
}
```

```php
use Doctrine\ORM\EntityRepository;

class FooRepository extends EntityRepository
{
    public function bar()
    {
        $query = $this
            ->createQueryBuilder('foo')
            ->select('PARTIAL foo.{id}')
            ->getQuery();

        return $query->getResult();
    }
}
```

Le SQL généré :
```sql
SELECT c0_.id as id_0 FROM foo
```

# Ne pas récupérer les liaisons (oneToMany etc)

Si on ajoute une liaison quelconque à l'entité `Foo`, même avec `PARTIAL` dans la requête cette liaison sera ajoutée dans la requête :
```php
/** @Entity */
class Foo
{
    /** @Column(type="integer") */
    protected ?int $id = null;

    /** @Column(length=140) */
    protected ?string $name = null;
    
    /** @OneToMany(targetEntity="Comments", mappedBy="foo") */
    protected Collection $comments;
    
    public function __construct()
    {
        $this->comments = new ArrayCollection();
    }
}
```

En appelant `FooRepository::bar()` de l'exemple précédent on obtient cette requête SQL :
```sql
SELECT c0_.id as id_0, c0_.comments as comments_1 FROM foo
```

Pour supprimer `c0_.comments as comments_1` il faut ajouter le hint `Query::HINT_FORCE_PARTIAL_LOAD` suivant à votre requête :
```php
use Doctrine\ORM\Query;
use Doctrine\ORM\EntityRepository;

class FooRepository extends EntityRepository
{
    public function bar()
    {
        $query = $this
            ->createQueryBuilder('foo')
            ->select('PARTIAL foo.{id}')
            ->getQuery();
        $query->setHint(Query::HINT_FORCE_PARTIAL_LOAD, true);

        return $query->getResult();
    }
}
```

La requête SQL générée :
```sql
SELECT c0_.id as id_0 FROM foo
```

# Références

[Documentation du mot-clé PARTIAL](https://www.doctrine-project.org/projects/doctrine-orm/en/2.8/reference/partial-objects.html)

[Documentation des hints](http://docs.doctrine-project.org/projects/doctrine-orm/en/latest/reference/dql-doctrine-query-language.html#query-hints)

[SqlWalker qui gère le hint HINT_FORCE_PARTIAL_LOAD](https://github.com/doctrine/orm/blob/v2.5.4/lib/Doctrine/ORM/Query/SqlWalker.php#L705)
