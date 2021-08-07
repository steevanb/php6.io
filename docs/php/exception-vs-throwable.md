---
title: \Exception vs \Throwable
mainMenu: php
---

# \Exception avant PHP 7.0

Avant PHP 7.0, la classe principale de toutes les exceptions était `\Exception`, 
que ce soit des exceptions levées par PHP ou par votre code.

Ce catch gérait donc toutes les exceptions possibles :
```php
try {
} catch (\Exception $exception) {
}
```

# \Error depuis PHP 7.0

PHP 7.0 a remplacé certaines erreurs qu'on ne pouvait pas catcher 
(ce n'étaient pas des exceptions mais l'équivalent d'un appel à 
[trigger_error()](https://www.php.net/manual/fr/function.trigger-error.php)) par des exceptions.

Par exemple quand on veut inclure un fichier PHP qui contient une erreur de syntaxe, 
en PHP < 7.0 on devait gérer avec `set_error_handler()` (ce qui était compliqué)
et depuis PHP 7.0 on a une exception `\ParseError` levée.

Pour ne pas ajouter un BC break énorme sur tous les `catch (\Exception)`, 
qui auraient donc récupérés de nouvelles exceptions qu'ils ne savent pas traiter,
les exceptions levées par le moteur PHP étendent maintenant de `\Error` et pas de `\Exception`.

Dans votre code, vous ne pouvez pas lever d'exception de type `\Error` et ses enfants, elles sont réservées au moteur PHP.

Vous pouvez toujours lever des exceptions de type `\Exception` et ses enfants, comme avant.

# \Throwable pour les gouverner toutes

Toutes les classes d'exceptions sont donc soit des enfants de `\Exception` ou de `\Error`,
ce qui nous permet de séparer les exceptions du moteur PHP de celles du code.

Si on veut catcher n'importe quelle exception, 
qu'elle soit de type `\Error` ou `\Exception` (et leurs enfants), 
on a une classe parente à `\Error` et `\Exception` qui est `\Throwable`.

Ce qui nous donne ce diagramme de classes pour les exceptions :

 * \Throwable
   * \Error
     * \CompileError
       * \ParseError
     * ...
   * \Exception
     * ...

# Exemples

[Testez vous-même](http://sandbox.onlinephpfunctions.com/code/ca8c2f3bb6e8dfbb478e2bd07e6b5ef3889a7151)

## Catcher \Exception

```php
try {
    throw new \Exception('\Exception');
} catch (\Exception $exception) {
    echo '\Exception levée et catchée via catch (\Exception $exception).';
}
```

## Tout catcher via \Throwable

```php
try {
    throw new \Exception('\Exception');
} catch (\Throwable $exception) {
    echo '\Exception levée et catchée via catch (\Throwable $exception).';
}
```

## Différence entre \Exception et \Throwable

```php
function foo($foo, $bar) {}

try {
    foo('only one param given');
} catch (\Exception $exception) {
    echo 'Ne devrait jamais passer là, l\'exception levée étend de \Error et pas \Exception.';
} catch (\Throwable $exception) {
    echo 'foo() mal appelée et catchée via (\Throwable $exception) (depuis PHP 7.1.0).';
}
```

# Sources

[php.net](https://www.php.net/manual/fr/language.errors.php7.php#language.errors.php7.hierarchy)
