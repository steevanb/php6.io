---
title: Valeur par défaut des propriétés
mainMenu: doctrine
---

# Pas de valeur par défaut dans la signature des propriétés

Toutes les valeurs des propriétés doivent être initialisées dans `__construct()`, surtout pas dans la signature !

Même si vous n'utilisez pas les
[Partial Objects](https://www.doctrine-project.org/projects/doctrine-orm/en/2.9/reference/partial-objects.html)
de doctrine, on ne maîtrise pas totalement ce qui sera fait avec notre entité.

En effet, elle peut avoir été chargée par une librairie externe, avoir été modifiée dans un évènement, etc.

# Exemple du problème

```php
class Foo
{
    private int $defaultProperty = 1;
    
    public function __construct(private string $property)
    {
    }
    
    public function getDefaultProperty(): int
    {
        return $this->defaultProperty;
    }
    
    public  function getProperty(): string
    {
        return $this->property;
    }
}
```

```php
class FooRepository
{
    public function getPartial(): Foo
    {
        return $this
            ->createQueryBuilder('foo')
            ->select('PARTIAL foo.{property}')
    }
}
```

```php
$foo = $fooRepository->getPartial();

// On a bien la valeur en base de données pour $property, tout va bien
$foo->getProperty();

// On n'a pas la valeur en base de données pour $defaultProperty, on a la valeur qui a été définie dans le code source !
$foo->getDefaultProperty();
```

# Exemple de la bonne syntaxe

Pour ne pas avoir ce problème, on définit toutes les valeurs dans `__construct()`,
qui ne sera pas appelé par Doctrine lors de la création d'une entité : il utilise
[ReflectionClass::newInstanceWithoutConstructor](https://www.php.net/manual/en/reflectionclass.newinstancewithoutconstructor.php).

```php
class Foo
{
    private int $defaultProperty;
    
    public function __construct(private string $property)
    {
        $this->defaultProperty = 1;
    }
    
    public function getDefaultProperty(): int
    {
        return $this->defaultProperty;
    }
    
    public  function getProperty(): string
    {
        return $this->property;
    }
}
```

```php
$foo = $fooRepository->getPartial();

// On a bien la valeur en base de données pour $property, tout va bien
$foo->getProperty();

// PHP va lever une exception de type \Error : Typed property Foo::$defaultProperty must not be accessed before initialization
// Ce qui paraît cohérent : on n'a pas chargé cette donnée, mais on essaye d'y accéder alors qu'on ne devrait pas
$foo->getDefaultProperty();
```
