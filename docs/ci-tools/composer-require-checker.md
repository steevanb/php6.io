# maglnet/composer-require-checker

[maglnet/composer-require-checker](https://github.com/maglnet/ComposerRequireChecker) est ou outil fait en PHP qui analyse votre `composer.json` à la recherche des `softs dependencies`.

Une soft dependency est une dépendance que vous utilisez dans votre projet mais qui n'est pas directement ajoutée via votre `composer.json` mais via une autre dépendance qui elle est dans votre `composer.json`.

`composer-require-checker` ira chercher tous les fichiers qui sont utilisables via l'autoload de Composer définit dans votre `composer.json` (clé `autoload` uniquement, la clé `autoload-dev` n'est pas lue) pour chercher leurs softs dependencies.

Dans les faits, une soft dependency n'est pas un problème : ça fonctionne. Mais si une mise à jour de cette autre dépendance supprime la dépendance que vous utilisez dans votre code : BOUM ;)

Il vérifie également que les extensions PHP requises dans votre `composer.json` sont bien installées.

# Exemple

Exemple avec l'installation par défaut de Symfony 5.2 :

```bash
mkdir /tmp/test
docker run --rm --interactive --tty --volume /tmp/test:/app --user $(id -u):$(id -g) composer:2.0 composer create-project symfony/skeleton /app
cd /tmp/test
docker run --tty --rm --volume $(pwd):/app steevanb/composer-require-checker:3.2.0
```

```bash
ComposerRequireChecker 3.2.0@cd6c6b91ee3c065e60a35ca1a67becb0ae86dbb7
The following 3 unknown symbols were found:
+---------------------------------------------------------------------------------+--------------------+
| Unknown Symbol                                                                  | Guessed Dependency |
+---------------------------------------------------------------------------------+--------------------+
| Symfony\Component\DependencyInjection\Loader\Configurator\ContainerConfigurator |                    |
| Symfony\Component\HttpKernel\Kernel                                             |                    |
| Symfony\Component\Routing\Loader\Configurator\RoutingConfigurator               |                    |
+---------------------------------------------------------------------------------+--------------------+
```

Il y a donc 3 softs dependencies, c'est-à-dire que dans le répertoire `src` 
des fichiers PHP utilisent ces 3 classes mais elles ne sont pas en dépendance directement dans votre `composer.json`.

Pour la précision : c'est `src/Kernel.php` le fautif,
et j'ai créé un [ticket de bug chez Symfony](https://github.com/symfony/symfony/issues/38448) pour leur indiquer mais ça n'a pas l'air de les intéresser plus que ça.

# Utilisation dans votre projet

`composer-require-checker` n'est pas une dépendance à mettre dans votre `require-dev`,
c'est un outil qui doit être lancé manuellement et donc il n'y a pas d'intérêt à ce qu'il soit installé en env de dev.

Si votre projet est en PHP 8.0 et n'a pas d'extensions PHP en  plus de celles installées par défaut avec l'image PHP officielle, 
alors vous pouvez utiliser l'image [steevanb/composer-require-checker](https://hub.docker.com/r/steevanb/composer-require-checker).

Cependant votre projet peut ne pas être en PHP 8.0 et ajouter des extensions.
Dans ce cas cette image ne fonctionnera pas : `composer-require-checker` vérifiera en premier lieu si les extensions PHP requises 
dans votre `composer.json` sont bien installées et comme ce ne sera pas le cas il s'arrêtera avant d'aller vérifier les softs dependencies.

Vous allez donc devoir créer votre propre image Docker avec votre version de PHP et vos dépendances installées, et installer `composer-require-checker` dedans.
Voilà un exemple : [Dockerfile](https://github.com/steevanb/docker-composer-require-checker/blob/master/docker/Dockerfile).

# Références

[maglnet/composer-require-checker](https://github.com/maglnet/ComposerRequireChecker)

![Docker](/images/icons/docker.png) [steevanb/composer-require-checker](https://hub.docker.com/r/steevanb/composer-require-checker)

[Issue #38448 chez symfony/Symfony](https://github.com/symfony/symfony/issues/38448)
