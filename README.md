# Boring stuff instruction

## 1. Install development tools and configure an autoloader via `composer.json`:

```json
{
    "require": {
        "php": "~5.5.0"
    },
    "require-dev": {
        "behat/behat": "~3.0.13",
        "phpspec/phpspec": "~2.1.0-RC1",
        "phpunit/phpunit": "~4.2.6"
    },
    "autoload": {
        "psr-0": {
            "SymfonyLive": "src/"
        }
    }
}
```

## 2. Bring in Symfony2 standard distribution into the project:

Download Symfony2 standard edition to the framework folder up one folder (say `N` to "Acme demo
bundle"):

```bash
composer create-project symfony/framework-standard-edition ../framework/ "~2.5.3"
```

Copy the `app` and `web` folders from the standard edition to the project:

```bash
cp -r ../framework/app . && cp -r ../framework/web
```

Replace your `composer.json` with the next block and then run `composer update`:

```json
{
    "require": {
        "php": "~5.5.0",
        "symfony/symfony": "~2.5.4",
        "doctrine/orm": "~2.2.3",
        "doctrine/doctrine-bundle": "~1.2",
        "twig/extensions": "~1.0",
        "symfony/assetic-bundle": "~2.3",
        "symfony/swiftmailer-bundle": "~2.3",
        "symfony/monolog-bundle": "~2.4",
        "sensio/distribution-bundle": "~3.0",
        "sensio/framework-extra-bundle": "~3.0",
        "incenteev/composer-parameter-handler": "~2.0"
    },
    "require-dev": {
        "behat/behat": "~3.0.13",
        "behat/mink": "~1.5.0",
        "behat/mink-extension": "~2.0.0",
        "behat/symfony2-extension": "~2.0.0",
        "behat/mink-browserkit-driver": "~1.1.0",
        "phpspec/phpspec": "~2.1.0-RC1",
        "phpunit/phpunit": "~4.2.0",
        "sensio/generator-bundle": "~2.3"
    },
    "autoload": {
        "psr-0": {
            "SymfonyLive": "src/"
        }
    },
    "scripts": {
        "post-root-package-install": [
            "SymfonyStandard\\Composer::hookRootPackageInstall"
        ],
        "post-install-cmd": [
            "Incenteev\\ParameterHandler\\ScriptHandler::buildParameters",
            "Sensio\\Bundle\\DistributionBundle\\Composer\\ScriptHandler::buildBootstrap",
            "Sensio\\Bundle\\DistributionBundle\\Composer\\ScriptHandler::clearCache",
            "Sensio\\Bundle\\DistributionBundle\\Composer\\ScriptHandler::installAssets",
            "Sensio\\Bundle\\DistributionBundle\\Composer\\ScriptHandler::installRequirementsFile",
            "Sensio\\Bundle\\DistributionBundle\\Composer\\ScriptHandler::removeSymfonyStandardFiles"
        ],
        "post-update-cmd": [
            "Incenteev\\ParameterHandler\\ScriptHandler::buildParameters",
            "Sensio\\Bundle\\DistributionBundle\\Composer\\ScriptHandler::buildBootstrap",
            "Sensio\\Bundle\\DistributionBundle\\Composer\\ScriptHandler::clearCache",
            "Sensio\\Bundle\\DistributionBundle\\Composer\\ScriptHandler::installAssets",
            "Sensio\\Bundle\\DistributionBundle\\Composer\\ScriptHandler::installRequirementsFile",
            "Sensio\\Bundle\\DistributionBundle\\Composer\\ScriptHandler::removeSymfonyStandardFiles"
        ]
    },
    "extra": {
        "symfony-app-dir": "app",
        "symfony-web-dir": "web",
        "incenteev-parameters": {
            "file": "app/config/parameters.yml"
        }
    }
}
```

## 3. Configure online attendee behat suite by replacing `behat.yml` and running `behat --init`

```yml
default:
    suites:
        attendee:
            contexts: [ AttendeeContext ]
            filters:  { role: conference attendee }
        online_attendee:
            contexts: [ OnlineAttendeeContext ]
            filters:  { role: conference attendee, tags: critical }
    extensions:
        Behat\Symfony2Extension: ~
        Behat\MinkExtension:
            default_session: 'symfony2'
            sessions:
                symfony2: { symfony2: ~ }
```

## 4. Generate Symfony2 bundle

```bash
app/console generate:bundle \
    --namespace=SymfonyLive/Framework/PersonalScheduleBundle \
    --dir=src \
    --bundle-name=PersonalScheduleBundle \
    --no-interaction
```

## 5. Improve behat debugging in Symfony2

Add to the end of `app/config/config_test.yml`:

```yml
services:
    listener.exception_rethrow:
        class: SymfonyLive\Framework\Test\ExceptionRethrowListener
        tags:
            - { name: kernel.event_listener, event: kernel.exception, method: onKernelException }
```

Create `src/SymfonyLive/Framework/Test/ExceptionRethrowListener.php`:

```php
<?php

namespace SymfonyLive\Framework\Test;

use Symfony\Component\HttpKernel\Event\GetResponseForExceptionEvent;
use Symfony\Component\HttpKernel\Exception\HttpExceptionInterface;

class ExceptionRethrowListener
{
    public function onKernelException(GetResponseForExceptionEvent $event)
    {
        $exception = $event->getException();

        if ($exception instanceof HttpExceptionInterface) {
            return;
        }

        throw $exception;
    }
}
```

## 6. Add the conference repository service definition to
`src/SymfonyLive/Framework/PersonalScheduleBundle/Resources/config/services.xml`:

```xml
<?xml version="1.0" ?>

<container xmlns="http://symfony.com/schema/dic/services"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://symfony.com/schema/dic/services http://symfony.com/schema/dic/services/services-1.0.xsd">

    <services>

        <service id="symfony_live.conference_repository"
                 class="SymfonyLive\Conference\ConferenceRepository"
                 factory-service="doctrine.orm.entity_manager"
                 factory-method="getRepository">
            <argument>SymfonyLive\Conference\Conference</argument>
        </service>

    </services>

</container>
```

## 7. Replace the Doctrine mapping configuration in the `app/config/config.yml`:

```yml
# Doctrine Configuration
doctrine:
    dbal:
        driver:   "%database_driver%"
        host:     "%database_host%"
        port:     "%database_port%"
        dbname:   "%database_name%"
        user:     "%database_user%"
        password: "%database_password%"
        charset:  UTF8
        # if using pdo_sqlite as your database driver, add the path in parameters.yml
        # e.g. database_path: "%kernel.root_dir%/data/data.db3"
        # path:     "%database_path%"

    orm:
        auto_generate_proxy_classes: "%kernel.debug%"
        auto_mapping: false
        mappings:
            symfony_live:
                is_bundle: false
                type:      xml
                dir:       %kernel.root_dir%/../src/SymfonyLive/Framework/Doctrine/mapping
                prefix:    SymfonyLive
```

## 8. Configure development database (sqlite) by adding next lines to `app/config/config_dev.yml`:

```yml
doctrine:
    dbal:
        driver:   pdo_sqlite
        host:     127.0.0.1
        port:     ~
        dbname:   symfony
        user:     root
        password: ~
        charset:  UTF8
        path:     "%kernel.root_dir%/../data.db3"
```

## 9. Define Doctrine mapping for Conference and TalkSchedule:

In `src/SymfonyLive/Framework/Doctrine/mapping/Conference.Conference.orm.xml`:

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<doctrine-mapping xmlns="http://doctrine-project.org/schemas/orm/doctrine-mapping"
                  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
                  xsi:schemaLocation="http://doctrine-project.org/schemas/orm/doctrine-mapping
        http://doctrine-project.org/schemas/orm/doctrine-mapping.xsd">
    <entity name="SymfonyLive\Conference\Conference"
            repository-class="SymfonyLive\Framework\Doctrine\DoctrineConferenceRepository">
        <id name="name" type="string"/>
        <one-to-many field="talkSchedules"
                     mapped-by="conference"
                     target-entity="SymfonyLive\Conference\TalkSchedule">
            <cascade>
                <cascade-persist/>
                <cascade-remove/>
            </cascade>
        </one-to-many>
    </entity>
</doctrine-mapping>
```

In `src/SymfonyLive/Framework/Doctrine/mapping/Conference.TalkSchedule.orm.xml`:

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<doctrine-mapping xmlns="http://doctrine-project.org/schemas/orm/doctrine-mapping"
                  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
                  xsi:schemaLocation="http://doctrine-project.org/schemas/orm/doctrine-mapping
        http://doctrine-project.org/schemas/orm/doctrine-mapping.xsd">
    <entity name="SymfonyLive\Conference\TalkSchedule">
        <id name="conference" association-key="true"/>
        <id name="talk" type="object"/>
        <id name="slot" type="object"/>
        <id name="track" type="object"/>
        <many-to-one field="conference" target-entity="SymfonyLive\Conference\Conference">
            <join-column name="conference" referenced-column-name="name"/>
        </many-to-one>
    </entity>
</doctrine-mapping>
```

## 10. Add mapping for personal schedule

In `src/SymfonyLive/Framework/Doctrine/mapping/Attendee.PersonalSchedule.orm.xml`:

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<doctrine-mapping xmlns="http://doctrine-project.org/schemas/orm/doctrine-mapping"
                  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
                  xsi:schemaLocation="http://doctrine-project.org/schemas/orm/doctrine-mapping
        http://doctrine-project.org/schemas/orm/doctrine-mapping.xsd">

    <entity name="SymfonyLive\Attendee\PersonalSchedule"
            repository-class="SymfonyLive\Framework\Doctrine\DoctrineScheduleRepository">
        <id name="conference" association-key="true"/>

        <many-to-one field="conference" target-entity="SymfonyLive\Conference\Conference">
            <join-column name="conference" referenced-column-name="name"/>
        </many-to-one>

        <many-to-many field="talkSchedules" target-entity="SymfonyLive\Conference\TalkSchedule">
            <join-table name="personal_talk_schedules">
                <join-columns>
                    <join-column name="conference" referenced-column-name="conference"/>
                </join-columns>
                <inverse-join-columns>
                    <join-column name="talk" referenced-column-name="talk" unique="true"/>
                    <join-column name="slot" referenced-column-name="slot" unique="true"/>
                    <join-column name="track" referenced-column-name="track" unique="true"/>
                </inverse-join-columns>
            </join-table>
        </many-to-many>
    </entity>
</doctrine-mapping>
```
