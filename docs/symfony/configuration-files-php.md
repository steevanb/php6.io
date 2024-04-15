---
title: Fichiers de configuration en PHP
mainMenu: symfony
---

# Pourquoi utiliser le format PHP pour vos fichiers de configuration ?
 
 - Depuis 6.0, c'est le format qui est mis en avant, avec un très gros travail dans Symfony et les bundles
 - Autocomplétion des configurations possibles grâce aux classes de Config
 - Pas besoin de lire de la documentation pour savoir comment configurer : tout est disponible via du code PHP
 - On peut faire des `if`, des `for` etc
 - On utilise réellement le fqcn des classes, les constantes, les enums etc
 - Les outils de CI peuvent valider le code ([phpstan](https://phpstan.org/) par exemple)
 - Pas besoin d'apprendre un nouveau format : on sait déjà faire du PHP

# Convertir ses fichiers de configuration en PHP via un outil

Vous pouvez utiliser [symplify/config-transformer](https://github.com/symplify/config-transformer).

Cependant, il n'utilise pas les classes `Config` précises, mais passe par `ContainerConfigurator`,
ce qui ne vous ajoute pas un des principaux atouts de la version PHP : l'autocomplétion.

Je vous conseille donc de passer par une convertion manuelle (voir ci-dessous).

# Convertir ses fichiers de configuration en PHP manuellement

### #1 Rafraîchir le cache

Pour que Symfony génère les classes PHP qui permettent la configuration en PHP, après avoir installé votre dépendance,
il faut rafraîchir le cache :

```bash
bin/console.php cache:warmup
```

### #2 Sauvegarder la configuration actuelle

Pour être sûr d'avoir la même configuration entre la version YAML et la version PHP,
vous pouvez conserver le résultat de la configuration en YAML
(remplacer `CONFIG_KEY` par la clé primaire de configuration, par exemple `doctrine_migrations`) :
```bash
bin/dev/console debug:config [CONFIG_KEY]
```

### #3 Désactiver la version YAML de votre configuration

Pendant que vous testez votre convertion,
faites bien attention : les 2 fichiers sont pris en compte (`foo.yaml` et `foo.php`).

Le fichier PHP étant lu en premier, puis le fichier YAML,
si une même configuration est présente dans la version PHP et la version YAML,
alors ce sera la version YAML qui sera la valeur finale.

Je vous conseille donc de mettre en commentaire le contenu complet du fichier YAML,
pour effectuer votre convertion sans que la version YAML soit prise en compte
tout en ayant un œil sur ce que vous devez convertir.

### #4 Classes Config

Symfony génère des classes `Config` dans `var/cache/dev/Symfony/Config`,
qui sont les classes PHP qui permettent de configurer votre package.

Si vous avez ignoré le répertoire `var` dans votre éditeur de code,
pensez à ne pas exclure `var/cache/dev/Symfony/Config`
pour avoir accès à l'autocomplétion dans vos fichiers de configuration PHP.

Par exemple, avec PHPStorm,
il faut faire click droit sur `var/cache/dev/Symfony/Config` > `Mark directory as` > `Cancel exclusion`.

Le nom de la classe `Config` est la version UpperCamelCase de votre clé de configuration, suffixée de `Config`.

Example :

```yaml
doctrine_migrations:
```
```php
use Symfony\Config\DoctrineMigrationsConfig;
```

### #5 Fichier de configuration PHP

Au même endroit que votre fichier YAML, créez un fichier PHP du même nom.

Ce nom n'est pas important pour Symfony, mais par convention,
le nom du fichier est le nom de la clé de configuration principale.

Ensuite, il faut effectuer la convertion :
chaque clé dans le YAML doit avoir une méthode correspondante, au format camelCase, dans la classe `Config`.

Exemple avec la version de base de `config/packages/framework.yaml` :
```yaml
framework:
    secret: '%env(APP_SECRET)%'
    session: true
    router:
        default_uri: http://localhost:10000

when@prod:
    framework:
        router:
            strict_requirements: null

when@test:
    framework:
        test: true
        session:
            storage_factory_id: session.storage.factory.mock_file
```

```php
<?php

declare(strict_types=1);

use Symfony\Config\FrameworkConfig;

return static function (FrameworkConfig $config): void {
    $config->secret($_ENV['APP_SECRET']);

    $config
        ->session()
        ->enabled(true);

    $config->router()->defaultUri('http://localhost:10000');

    if ($_ENV['APP_ENV'] === 'prod') {
        $config->router()->strictRequirements(null);
    }

    if ($_ENV['APP_ENV'] === 'test') {
        $config->test(true);

        $config
            ->session()
            ->storageFactoryId('session.storage.factory.mock_file');
    }
};

```

### #6 Vérifiez que la configuration n'a pas changé

Vous pouvez vérifier que la configuration n'a pas changé en exécutant `bin/dev/console debug:config [CONFIG_KEY]`,
et en comparant avec la version que vous avez exécuté avant la convertion (étape #1).

# Configuration accessible que dans certains environnements

Si un Bundle n'est pas installé dans tous les environnements,
alors la classe `Config` n'existera que dans les environnements dans lesquels le Bundle est installé.

Par exemple, `Symfony\Bundle\DebugBundle\DebugBundle` n'est installé que pour l'environnement `dev`.

Dans ce cas, vous pouvez faire comme ça :
```php
<?php

declare(strict_types=1);

use App\Environment\Environment;
use Symfony\Config\DebugConfig;

if ($_ENV['APP_ENV'] === 'dev') {
    return static function (DebugConfig $config): void {
    };
}
```
