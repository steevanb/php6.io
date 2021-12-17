# Présentation

Doctrine 2 permet de lier ses entités via 2 types de liaisons : `manyToOne` et `oneToOne`.

La liaison `manyToMany` entre `Foo` et `Bar` n'est qu'un raccourci d'une entité `Foo` vers une entité "invisible" 
via une `manyToOne`, puis une `oneToMany` vers votre entité `Bar`.

Toutes les liaisons peuvent être `unidirectionnelles` (exemple : une `manyToOne` n'a pas de `oneToMany` associée) 
ou `bidirectionnelles` (exemple : `une manyToOne` qui a une `oneToMany` associée).

Dans le cas des liaisons bidirectionnelles, il faut définir qui est le propriétaire (`owning side`) et qui est le côté inverse (`inverse side`) :
 * `manyToOne` et `oneToMany` : c'est la `manyToOne` qui est le owning side, et la oneToMany le inverse side
 * `oneToOne` : il faut choisir une des deux entités comme owning side, en ajoutant `inversedBy` côté owning side et `mappedBy` côté inverse side. Seule la table du owning side aura une clef étrangère vers le inverse side.
 * `manyToMany` : il faut choisir une des deux entités comme owning side, en ajoutant `inversedBy` côté owning side et `mappedBy` côté inverse side. Une table de liaison sera créé par Doctrine.

# Owning side

Doctrine ne gère que le owning side des relations : `manyToOne`, `manyToMany` owning side, et `oneToOne` owning side.

La raison est simple : dans votre base de données, le côté owning side est celui qui contient la clé étrangère.

Prenons pour exemple une entité `User`, qui est liée à une entité `Comment` : `User` > `oneToMany` > `Comment`.

Au niveau de votre base de données, c'est la table `comment` qui aura la clé étrangère `user_id`.

Le fait d'ajouter des `Comment` sur l'entité `User` sans appeler `Comment::setUser()` ne sert à rien.
Doctrine ne sauvegardera pas votre `Comment` avec la liaison vers `User`, et conservera donc `null` dans `Comment::$user`.

Au final, une requête de ce type sera générée :
```sql
INSERT INTO comment (user_id, message) VALUES (null, 'mon commentaire')
```

Grace à cet exemple, on comprend qu'un inverse side (`oneToMany`, `oneToOne` inverse side, `manyToMany` inverse side) 
n'est finalement qu'un lien pratique pour le développeur, et pas un lien réellement géré par Doctrine.

# Exemple de code

Beaucoup de bugs peuvent ne pas se voir rapidement quand on code une `oneToMany` :
 * tentative d'insertion du owning side avec `null` dans la clef étrangère
 * suppression des objets côté PHP mais pas en base de données

Reprenons notre exemple `User` > `oneToMany` > `Comment`.

## Owning side

```php
class User
{
    protected Collection $comments;

    public function __construct()
    {
        $this->comments = new ArrayCollection();
    }

    /** @param iterable<string|int, Comment> */
    public function setComments(iterable $comments): self
    {
        $this->clearComments();
        foreach ($comments as $comment) {
            $this->addComment($comment);
        }

        return $this;
    }

    public function addComment(Comment $comment): self
    {
        if ($this->comments->contains($comment) === false) {
            $this->comments->add($comment);
            $comment->setUser($this);
        }

        return $this;
    }

    public function getComments(): Collection
    {
        return $this->comments;
    }

    public function removeComment(Comment $comment): self
    {
        if ($this->comments->contains($comment)) {
            $this->comments->removeElement($comment);
            $comment->setUser(null);
        }

        return $this;
    }

    public function clearComments(): self
    {
        foreach ($this->getComments() as $comment) {
            $this->removeComment($comment);
        }
        $this->comments->clear();

        return $this;
    }
}
```

### Collection, ArrayCollection et PersistentCollection

Il faut utiliser `Collection` (qui est une interface, comme son nom de l'indique pas) comme typehint
pour la compatibilité avec `ArrayCollection` et `PersistentCollection`.

`ArrayCollection` est l'implémentation la plus simple de l'interface `Collection` de Doctrine,
qui permet de simplement faire une liste de quelque chose sans aucune vérification.

`PersistentCollection` est la classe qui sera instanciée par Doctrine quand on veut du lazy loading. 
Par exemple, si dans un `QueryBuilder` on ne charge que les entités `User` sans faire de liaison avec l'entité `Comment`,
alors Doctrine va remplacer ce qu'on a mis dans la propriété `User::$comments` par une instance de `PersistentCollection`.
Cette collection a accès à l'`EntityManager`, et dès qu'on va vouloir accéder à une autre propriété que l'identifiant 
d'une entité `Comment` alors `PersistentCollection` fera une requête pour récupérer le `Comment` demandé.

setComments() ne doit surtout pas faire $this->comments = new ArrayCollection(), 
sinon on ne supprime pas les liaisons du owning side Comment ! Il faut bien appeler clearComments() et addComment()
 * addComment() doit vérifier que le Comment qu'on veut ajouter n'existe pas déjà, pour éviter tout doublon
 * addComment() doit appeler Comment::setUser(), pour que le owning side Comment ait connaissance de la valeur à mettre dans la clef étrangère user_id
 * removeComment() doit appeler Comment::setUser(null), pour que le côté owning side Comment sache qu'il n'est plus lié à un User
 * clearComments() ne doit pas faire $this->comments = new ArrayCollection(), sinon, on ne supprime pas les liaisons du owning side Comment ! Il faut appeler removeComment()
 * clearComments() doit remettre à 0 le pointeur du tableau interne de Collection, via clear(). Sinon, le prochain appel à add() n'ajoutera pas le commentaire en clef 0, mais en clef 2 (si clearComments() a supprimé les commentaires 0 et 1 par exemple). /!\ Avant Doctrine 2.5.6, l'appel à Collection::reset() pouvait ne pas remettre à 0 les clefs, si le tableau ne contenait pas d'élément.
