---
title: Mapping Doctrine en PHP
mainMenu: symfony
---

# Pourquoi utiliser le format PHP pour le mapping Doctrine ?
 
 - Depuis Symfony 6.0, c'est le format qui est mis en avant, avec un très gros travail dans Symfony et les bundles
 - Autocomplétion des configurations possibles grâce à `ClassMetadataBuilder`
 - Pas besoin de lire de la documentation pour savoir comment configurer : tout est disponible via du code PHP
 - On peut faire des `if`, des `for` etc
 - On utilise réellement le fqcn des classes, les constantes, les enums etc
 - Les outils de CI peuvent valider le code ([phpstan](https://phpstan.org/) par exemple)
 - Pas besoin d'apprendre un nouveau format : on sait déjà faire du PHP

# Convertir ses fichiers de configuration en PHP manuellement

### #1 Correction du FQCN de PHPDriver

Dans [doctrine-bundle 2.12.0](https://github.com/doctrine/DoctrineBundle),
[la configuration du FQCN de PHPDriver](https://github.com/doctrine/DoctrineBundle/blob/2.12.0/config/orm.xml#L36)
n'est pas bonne : elle indique `Doctrine\ORM\Mapping\Driver\PHPDriver`,
mais [la classe a été déplacée en 2.3.0](https://github.com/doctrine/orm/blob/2.3/UPGRADE.md#metadata-drivers)
vers `Doctrine\Common\Persistence\Mapping\Driver\PHPDriver`.

Vous allez donc avoir cette exception : `Attempted to load class "PHPDriver" from namespace "Doctrine\ORM\Mapping\Driver". Did you forget a "use" statement for "Doctrine\Persistence\Mapping\Driver\PHPDriver"?`

Il faut donc changer la configuration comme suit :
```php
# config/packages/doctrine.php
use Doctrine\Persistence\Mapping\Driver\PHPDriver;
use Symfony\Component\DependencyInjection\Loader\Configurator\ContainerConfigurator;

return static function (ContainerConfigurator $containerConfigurator): void {
    $containerConfigurator
        ->parameters()
        ->set('doctrine.orm.metadata.php.class', PHPDriver::class);
}
```

### #2 Configuration de Doctrine

Pour indiquer à Doctrine que le mapping sera fait en PHP dans `config/doctrine` :
```php
# config/packages/doctrine.php
use Symfony\Config\DoctrineConfig;

return static function (DoctrineConfig $config): void {
    $config
        ->orm()
        ->entityManager('default')
        ->mapping('App')
        ->type('php')
        ->isBundle(false)
        ->dir('%kernel.project_dir%/config/doctrine/mapping')
        ->prefix('App\\Entity')
        ->alias('App');
}
```

### #3 Création du fichier de mapping

Comme pour les autres fichiers de mapping, le nom du fichier doit avoir ce format :
 - Namespace de l'entité dont les `\` sont remplacés par des `.`
 - Nom de la classe
 - Extension : `.php`

Exemple pour l'entité `App\Entity\Foo` : `config/doctrine/App.Entity.Foo.php`.

### #4 $metadata

Doctrine est assez gentil : il nous envoie une variable `$metadata` quand il inclut notre fichier de configuration,
qui est déjà configurée pour notre entité.

Pour que PHPStorm (et d'autres outils comme phpstan) sache que cette variable existe, on peut ajouter une PHPDoc :

```php
<?php

declare(strict_types=1);

use Doctrine\ORM\Mapping\ClassMetadata;

/** @var ClassMetadata $metadata */
```

### #5 Création du builder

A partir de là, vous pouvez configurer votre entité avec `$metadata`.

Par exemple, pour ajouter un champ, vous pouvez utiliser `mapField()`.

Cependant, vous allez passer par des gros tableaux, bien compliqués à écrire
(il faut trouver dans la doc toutes les clés possibles, les valeurs possibles etc) :
ce n'est pas très optimal et très difficile à relire / valider.

N'allez pas sur Twitter (ouais, je vais rester sur ce nom) pour insulter les mères de tout le monde :
Doctrine est sympa, il nous a fait une classe
[ClassMetadataBuilder](https://github.com/doctrine/orm/blob/3.1.x/src/Mapping/Builder/ClassMetadataBuilder.php)
qui permet de passer par des objets et pas des array.

```php
<?php

declare(strict_types=1);

use Doctrine\ORM\{
    Mapping\Builder\ClassMetadataBuilder,
    Mapping\ClassMetadata
};

/** @var ClassMetadata $metadata */

$builder = new ClassMetadataBuilder($metadata);
```

[Documentation de ClassMetadataBuilder](https://www.doctrine-project.org/projects/doctrine-orm/en/3.1/reference/php-mapping.html).

### #6 A vous de jouer

A votre tour : c'est le moment de configurer le mapping de votre entité, via `$builder`.

Comme je suis assez sympa, je vous montre un exemple pour un identifiant :

```php
<?php

declare(strict_types=1);

use Doctrine\DBAL\Types\Types;
use Doctrine\ORM\{
    Mapping\Builder\ClassMetadataBuilder,
    Mapping\ClassMetadata
};

/** @var ClassMetadata $metadata */

$builder = new ClassMetadataBuilder($metadata);

$builder
    ->createField('id', Types::INTEGER)
    ->makePrimaryKey()
    ->generatedValue()
    ->option('unsigned', true)
    ->build();
```

# Validation du mapping

Vous pouvez exécuter la commande `bin/console doctrine:mapping:info`
pour vérifier que Doctrine trouve bien vos fichiers de mapping.

La commande `bin/console doctrine:schema:validate` exécutera `doctrine:mapping:info`,
et en plus, vérifiera que votre base de données correspond à votre mapping :
qu'il ne manque pas de migrations, et qu'elles ont toutes été jouées.

Si vous voulez voir ce que Doctrine veut faire de votre mapping dans votre base de données :
`bin/console doctrine:schema:update --dump-sql`.
