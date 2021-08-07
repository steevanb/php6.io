---
title: self vs static
mainMenu: php
---

# Contexte

Quand vous utilisez des méthodes statiques dans des classes, 
à l'intérieur de la classe vous pouvez utiliser `self::foo()` ou `static::foo()` pour appeler ces méthodes.

Il y a une différence fondamentale entre ces deux mots-clés : 
`self` ne prend pas en compte les surcharges alors que `static` les prend en compte.

`static` est l'équivalent de `$this` en dynamique alors que `self` n'a pas d'équivalent en dynamique.

# Version de PHP minimale pour self et static

Le mot-clé `self` a été ajouté en PHP 5.0.

Le mot-clé `static` a été ajouté en PHP 5.3, vous devriez être largement à jour pour pouvoir l'utiliser ;)

# Exemple avec self

```php
class Foo
{
    public static function outputInformations()
    {
        echo __CLASS__;
    }
    
    public static function callMe()
    {
        self::outputInformations(); // On utilise self dans la classe parente
    }
}

class Bar extends Foo
{
    public static function outputInformations()
    {
        echo __CLASS__;
    }
}

Bar::callMe(); // Affiche "Foo"
```

Ici on voit bien que l'utilisation de `self` ne prend pas en compte que la classe `Bar`
a surchargé la méthode `outputInformations()` alors qu'on s'attend à ce que ce soit le cas.

[Testez vous-même](http://sandbox.onlinephpfunctions.com/code/46b99442e9cb63eca9164e66f84bd6051c6d95e6)

# Exemple avec static

```php
class Foo
{
    public static function outputInformations()
    {
        echo __CLASS__;
    }
    
    public static function callMe()
    {
        static::outputInformations(); // On utilise static dans la classe parente
    }
}

class Bar extends Foo
{
    public static function outputInformations()
    {
        echo __CLASS__;
    }
}

Bar::callMe(); // Affiche "Bar"
```

Ici on voit vien que l'utilisation de `static` prend en compte que la classe `Bar`
a surchargé la méthode `outputInformations()` : c'est l'équivalent du comportement de `$this`.

[Testez vous-même](http://sandbox.onlinephpfunctions.com/code/46b99442e9cb63eca9164e66f84bd6051c6d95e6)

# Pourquoi ne pas avoir corrigé self ?

On peut se demander pourquoi PHP n'a pas corrigé le mot-clé `self` au lieu de créer un nouveau mot-clé.

`self` ayant un comportement bien précis, 
il est fortement probable que des librairies ou du code dans des projets fonctionne grâce à ce comportement.

Changer `self` pour prendre en compte les surcharges aurait un impact potentiellement énorme sur le code existant.

Pour éviter un BC break en PHP 5.3, le mot-clé `static` a été ajouté.

# Pourquoi on voit encore self dans les librairies ?

J'ai demandé à [Nicolas Grekas](https://twitter.com/nicolasgrekas) (numéro deux de Symfony) 
pourquoi dans Symfony ils utilisaient encore `self`. 

La réponse vient de la philosophie de Symfony de ne pas gérer les surcharges de leur code :
les méthodes `protected`, `protected static` etc peuvent être soumises à des BC break sans qu'on en soit informé.

Seules les méthodes publiques sont soumises aux BC break.

Dans leur vision ce n'est pas important,
dans les faits ça peut être parfois problématique quand on veut surcharger une méthode statique ou une constante.

# Conclusion

N'utilisez __jamais__ `self`, utilisez __toujours__ `static` !

`self` est presque un bug des vieilles versions de PHP qui compilaient le statique "trop tôt" 
alors que les surcharges n'étaient pas encore parsées.

A partir de PHP 5.3 le mot-clé `static` a été ajouté pour compiler la partie statique 
d'une classe après que les surcharges soient parsées : c'est ce qu'on appelle le 
[late static binding](https://www.php.net/manual/fr/language.oop5.late-static-bindings.php).

Avant PHP 5.3 on n'avait aucun moyen de prendre en compte les surcharges dans du statique.

# Sources

[php.net](https://www.php.net/manual/fr/language.oop5.late-static-bindings.php)
