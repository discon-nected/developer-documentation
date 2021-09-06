---
category: DevelopInDepth
---
# Composer dependencies

Matomo requires various packages to run. Those requirements are managed using [Composer](https://getcomposer.org/).

## Adding new requirements

If for some reason a new requirement would be needed, we need to require it using composer.

```bash
composer require package/name ^1.2.3
```

If the required package is used for development only, it should be excluded for other environments using:

```bash
composer require-dev package/name ^1.2.3
```

## Update workflow

### Minor or patch release update

Almost all requirements in our `composer.json` are defined in a way that allow minor and/or patch release updates without any adjustments in `composer.json`.
As there are normally no problems expected for those updates there is an [automatic GitHub action](https://github.com/matomo-org/matomo/blob/4.x-dev/.github/workflows/composer-update.yml) that automatically checks for updates and creates a Pull Request if any are available.

### Major updates

Every 6 months we should check for possible major updates of our requirements. 

#### Checking for possible updates

Composer allows viewing all available updates for installed packages:

```bash
composer outdated
```

#### Updating the requirements

If there are major updates available we should go through them and update each. As some updates might require code changes on our side it is best to only update one requirement after the other. Otherwise it will be harder to identify why e.g. some code breaks or why some tests start failing.
Updating a single requirement can be achieved this way:

* Update the version number for the updated component in [matomo/composer.json](https://github.com/matomo-org/matomo/blob/4.x-dev/composer.json) if needed
* Execute `composer update --with-dependencies package/name`. You need to replace `package/name` with the name of the requirement, like `phpmailer/phpmailer`. 
* Commit & push `composer.json` and `composer.lock`.
* Check if any tests are failing and adjust the code / tests if needed.
* Create a PR and get it merged.

*Note: we use the `--with-dependencies` option, to ensure all dependencies are updated if needed.*

#### Problems with incompatible requirements

It might be possible that updating some dependencies might not work due to the PHP version requirements of Matomo. 
Some packages like `PHPUnit` might already require a higher PHP version than Matomo. 
This causes composer not to update to newer versions or a composer install to fail when updating the dependency in `composer.json`.

If an update to a newer version is mandatory there are basically two possible options
* Increase the PHP version requirement (which is usually only done for major releases)
* create a fork of the package and lower the minimum requirement (if the code really works with our minimum PHP requirement)

*Note: If you want to install newer requirements locally for testing purpose, you can achieve this using the `--ignore-platform-reqs` option of composer.*

### Updating internal packages

We are managing some of our [core components](/guides/core-components) and libraries through composer as well. Whenever we release a new version of those packages we should directly issue an update of the composer requirement in Matomo.

* Update the version number for the updated component in [matomo/composer.json](https://github.com/matomo-org/matomo/blob/4.x-dev/composer.json) if needed
* Execute `composer update matomo/$COMPONENT_NAME`. You need to replace `$COMPONENT_NAME` with the name of the component. For the cache component you may need to execute `composer update matomo/cache  matomo/doctrine-cache-fork`.
* Push `composer.json` and `composer.lock`.
* Create a PR and get it merged.