TanoConsulting Ibexa-Migration-Bundle
=====================================

The replacement of [kaliop/ezmigrationbundle](https://github.com/kaliop-uk/ezmigrationbundle) for Ibexa 4.

This bundle makes it easy to programmatically deploy changes to Ibexa database structure and contents.

It is inspired by the [DoctrineMigrationsBundle](http://symfony.com/doc/current/bundles/DoctrineMigrationsBundle/index.html)

You can think of it as the great-grandson of the legacy [ezxmlinstaller](https://github.com/ezsystems/ezxmlinstaller) extension.

*WORK IN PROGRESS*

Please read [WHATSNEW.md](WHATSNEW.md) for details!

*Many thanks to [Wizhippo](https://github.com/wizhippo) for having started this version, and kindly donated the code!*

## Requirements

* PHP 7.4 or later.

* Ibexa 4.0 (aka. Ibexa DXP).


## Installation

ATM there is no stable release. The `master` branch is now `main`, so you have to run:

    composer require 'tanoconsulting/ibexa-migration-bundle:dev-main'

Then make sure that the bundle is active, generally by editing `config/bundles.php`

### Checking that the bundle is installed correctly

If you run `php bin/console` you should see the following new commands in the list:

    kaliop
      kaliop:migration:generate
      kaliop:migration:mass_migrate
      kaliop:migration:migrate
      kaliop:migration:migration
      kaliop:migration:resume
      kaliop:migration:status

This indicates that the bundle has been installed and registered correctly.

### Updating the bundle

To get the latest version, you can update the bundle to the latest available version by using `composer`

    composer update tanoconsulting/ibexa-migration-bundle

## Getting started

All commands accept the standard Symfony/eZPlatform options, although some of them might not have any effect on the
command's execution.

### Generating a new, empty migration definition file

The bundle provides a command to easily generate a new blank migration definition file.

For example:

    php bin/console kaliop:migration:generate --format=yml

The above command will place a new yml skeleton file in the `MigrationsDefinitions` directory in `./src/`.

If the directory does not exist then the command will create it for you.
If the command is successful it will create a new yml file named with the following pattern: `YYYYMMDDHHMMSS_placeholder.yml`.
You are encouraged to rename the file and change the `placeholder` part to something more meaningful, but please keep
the timestamp part and underscore, as well as the extension.

_(the contents of the skeleton Yaml file are stored as twig template)_

### Listing all migrations and their status

To see all the migrations definitions available in the system and whether they have been applied or not simply run the
status command in your eZPlatform root directory:

    php bin/console kaliop:migration:status

The list of migrations which have been already applied is stored in the database, in a table named `kaliop_migrations`.
The bundle will automatically create the table if needed.
In case you need to use a different name for that table, you can change the Symfony parameter `ez_migration_bundle.table_name`.

### Applying migrations

To apply all available migrations run the migrate command in your eZPlatform root directory:

     php bin/console kaliop:migration:migrate

NB: if you just executed the above command and got an error message because the migration definition file that you had
just generated is invalid, do not worry - that is by design. Head on to the next paragraph...

NB: migrations execute by the default as the admin user with ID 14. Without this user account in the database, you must
specify the use of another admin account by passing the `-a` flag.

#### Applying a single migration file

To apply a single migration run the migrate command passing it the path to its definition, as follows:

    php bin/console kaliop:migration:migrate --path=src/MyNamespace/MyBundle/MigrationVersions/20160803193400_a_migration.yml

NB: you can specify as well a folder with the `--path` flag, in which case all the migration definitions contained in that
folder will be executed.

### Editing migration files

So far so good, but what kind of actions can be actually done using a migration?

Each migration definition consists of a series of steps, where each step defines an action.

A simple example of a migration to create a 'folder' content is:

    -
        mode: create
        type: content
        content_type: folder
        parent_location: 2
        attributes:
            name: hello world

In a Yaml migration, you can define the following types of actions:
- creation, update and deletion of Contents
- creation, update and deletion of ContentTypes
- creation, update and  deletion of ContentTypeGroups
- deletion of Content Versions
- creation and deletion of Languages
- creation, update and deletion of Locations
- creation, update and deletion of ObjectStates
- creation, update and deletion of ObjectStateGroups
- creation, update and deletion of Roles
- creation, update and deletion of Sections
- creation and deletion of Tags (from the Netgen Tags Bundle)
- creation, update and deletion of Users
- creation, update and deletion of UserGroups
- purging and recovering Contents from the Trash
- creation, appending, copy, renaming and deletion of files
- execution of SQL queries
- execution of command-line scripts
- execution of methods of Symfony services
- execution of http calls
- sending of email
- looping over arrays and executing one of the above actions on each element
- canceling, snoozing or suspending the migration itself

The docs describing all supported parameters are in the [DSL Language description](Resources/doc/DSL/README.md)

### Custom migrations

For more specific needs, you can also use 2 other types of migrations:
- SQL migrations
- PHP migrations

#### SQL migrations

Example command to generate an SQL migration definition:

     php bin/console kaliop:migration:generate create-new-table --format=sql

This will create the following file, which you are free to edit:

    ./src/MigrationsDefinitions/2021XXYYHHMMSS_mysql_create-new-table.sql

*NB* if you rename the sql file, keep in mind that the type of database to which it is supposed to apply is the part
of the filename between the first and second underscore characters.
If you later try to execute that migration on an eZPublish installation running on, say, PostgreSQL, the migration
will fail. You are of course free to create a specific SQL migration for a different database type.

The Migration bundle itself imposes no limitations on the type of databases supported, but as it is based on the
Doctrine DBAL, it will only work on the databases that Doctrine supports.

*NB* you can also save the SQL statement to execute in a yml-formatted migration file. This gives you access to more
options, such as setting and resolving references. Yml-formatted migration files do not need to have the db type in their name.

*NB* if the SQL statement (or statements) in your migration is too long, the migration might fail or be only partially
applied, in some cases (such as when using MySQL) without even reporting an error. If you need to execute multiple, long
queries, you are better off splitting them, either in many .sql migrations, or in a single .yml migration with sql steps.

#### PHP migrations

If the type of manipulation that you need to do is too complex for either YML or SQL, you can use a php class as
migration definition. To generate a PHP migration definition, execute:

     php bin/console kaliop:migration:generate AMigrationClass --format=php

This will create the following file, which you are free to edit:

    ./src/MigrationsDefinitions/2021XXYYHHMMSS_AMigrationClass.php

As you can see in the generated definition, the php class to be used for a migration needs to implement a specific
interface. The Symfony DIC container is passed to the migration class so that it can access from it all the services,
parameters and other thing that it needs.

*NB* if you rename the php file, keep in mind that the filename and the name of the class it contains are tied - the
standard autoloading mechanism of the application does not apply when loading the migration definition. This is also
the reason why the php classes used as migrations should not use namespaces.

*NB* it is also possible to run any method of any existing Symfony service just by declaring it as migration step
in a yaml migration. See the [relevant DSL](Resources/doc/DSL/Service.yml) for details.

### Re-executing failed migrations

The easiest way to re-execute a migration in 'failed' status, is to remove it from the migrations table:

    php bin/console kaliop:migration:migration migration_name --delete

After removing the information about the migration from the migrations table, running the `migrate` command will execute it again.


## Usage of transactions / rolling back changes

By default the bundle runs each migration in a database transaction.
This means that if a step fails, all the previous steps get rolled back, and the database is left in its original state.
This is a safety feature built in by design;
* if you prefer the migration steps to be executed in separate transactions the easiest way is to create a separate
  migration file for each step
* you can use the command-line flag `-u` to disable usage of transactions by the migrate command

Note also that by default the `migrate` command stops on the 1st failed migration, but it can be executed with a flag
to allow it to continue and execute all available migrations even in case of failures.

As for rolling back changes: given the nature of the eZPublish API, rolling back changes to Content is not an easy feat.
As such, the bundle does not provide built-in support for rolling back the database to the version it had before
applying a given migration. We recommend always taking a database snapshot before applying migrations, and use it in
case you need to roll back your changes. Another approach consists in writing a separate migration to undo the changes.


## Customizing the migration logic via Event Listeners

An easy way to hook up custom logic to the execution of migrations - without having to implement your own customized
action executors - is to use Event Listeners.

Two events are fired *for each step* during execution of migrations:

    * ez_migration.before_execution => listeners receive a BeforeStepExecutionEvent event instance
    * ez_migration.step_executed => listeners receive a StepExecutedEvent event instance

An event is fired only in case a migration fails because a step throws a specific migration abort exception:

    * ez_migration.migration_aborted => listeners receive a MigrationAbortedEvent event instance

An event is fired when a migration is being generated using the `kaliop:migration:generate` command, allowing to alter
the data that will be serialized as migration steps:

    * ez_migration.migration_generated => listeners receive a MigrationGeneratedEvent event instance

In order to act on those events, you will need to declare tagged services, such as for ex:

    my.step_executed_listener:
        class: my\helper\StepExecutedListener
        tags:
            - { name: kernel.event_listener, event: ez_migration.step_executed, method: onStepExecuted }

and the corresponding php class:

    use Kaliop\eZMigrationBundle\API\Event\StepExecutedEvent;

    class StepExecutedListener
    {
        public function onStepExecuted(StepExecutedEvent $event)
        {
            // do something...
        }
    }


## Known Issues and limitations

* unlike the Doctrine Migrations Bundle, this bundle does not support rollback of changes. Read above for the reason why.

* if you are using the Doctrine Migrations Bundle to manage your schema, you will get spurious sql created to handle the
  database tables belonging to Kaliop Migrations Bundle.
  For the moment, the best workaround is to use the `filter-expression` parameter on the command-line when running
  `doctrine:migrations:diff` and friends, with a value of `kaliop_migrations_*`

* if you get fatal errors when running a migration stating that a node or object has not been found, it is most likely
  related to how the dual-kernel works in eZPublish, and the fact that the legacy and Symfony kernels use a separate
  connection to the database. Since the migration bundle by default wraps all database changes for a migration in a
  database transaction, when the Slots are fired which allow the legacy kernel to clear its caches, the legacy kernel
  cannot see the database changes applied by the Symfony kernel, and, depending on the specific Slot in use, might
  fail with a fatal error.
  The simplest workaround is to disable usage of transactions by passing the `-u` flag to the `migrate` command.

* similar symptoms can manifest themselves when you are using the Solr Search Engine Bundle.
  In this case the problem is compounded the fact that, even if a node or object is sent to Solr from within a database
  transaction, the Solr search index might be configured to only commit received data within a short time delay.
  A known workaround involve:
    - separate your migration steps into separate migrations
    - running the migrations each in its own transaction (and process) by using the `-p` flag to the `migrate` command
    - adding `sleep` migration steps to migrations 2 .. N
    - and/or configuring Solr to always commit changes to the index immediately (eg. disable `commitwithin`)

* when using SOLR in multi-core configurations and getting a `java.lang.NegativeArraySizeException` error, you will have
  to set a lower value than the default 2147483647 for parameter `ez_migration_bundle.query_limit`

* if you get fatal errors without any error message when running a migration which involves a lot of content changes,
  such as f.e. altering a contentType with many contents, it might be that you are running out of memory for your
  php process.
  Known workarounds involve:
    - increase the maximum amount of memory allowed for the php script by running it with option '-d memory_limit=-1'
    - execute the migration command using a Symfony environment which has reduced logging and kernel debug disabled:
      the default configuration for the `dev` environment is known to leak memory

* if you get fatal errors with the message 'You cannot create a service ("request") of an inactive scope ("request")',
  take a look at the following issue for a possible explanation and ideas for workarounds:
  https://jira.ez.no/browse/EZP-24691

* when updating a Role, you have to specify in the migration *all* the policies for it. Any existing policies that are not
  in the yml file will be removed.
  To make it easy to create a migration for updating a role, please use the `migration:generate` command using the `--type=role` flag

* take care when creating content types: the eZPublish API, used internally by the migration bundle, will let you use dash
  characters in the content type identifiers, even if the resulting content types will then be unusable, eg.

  Example of an invalid definition:

            type: ezstring
            name: Topbar-hover-color
            identifier: topbar-hover-color


## Frequently asked questions

### How can I update a specific content which has a different Id on dev, test and prod environments?

A: use the 'reference/set' migration step to define a reference for the desired content Id, and use a Symfony parameter
to store a different value for each Symfony environment. For example:

    -
        type: reference
        mode: set
        identifier: content_id_ref
        value: '%a.parameter.name%'

    -
        type: content
        mode: update
        match:
            content_id: "reference:content_id_ref"
        etc: ...

### How to update an existing Role to change its policies?

When using a migration to update a Role, you must define ALL its policies. Any not defined will be removed.
The safest and simplest way to make sure that you do not forget any of the existing policies is to first generate an
update migration that has the complete specification of the role as it currently is defined, and then edit manually.

Example command to create such a migration:

    php ezpublish/console kaliop:migration:generate --type=role --mode=update --match-type=identifier --match-value=Anonymous bundleName

### When dumping a Content into a yml migration via the `generate` command, the list of attributes is empty

A: this is most likely due to using a bad language configuration


## Extending the bundle

### Supporting custom migrations

The bundle has been designed to be easily extended in many ways, such as:
* adding support for custom/complex field-types
* adding support for completely new actions in the Yml definitions
* adding support for a new file format for storing migration definitions
* adding support for a new resolver for the custom references in the migration definitions
* taking over the way that the migrations definitions are loaded from the filesystem or stored in the database
* etc...

Following Symfony best practices, for the first 4 options in the list above all you need to do is to create a service
and give it an appropriate tag (the class implementing service should of course implement an appropriate interface).

To find out the names of the tags that you need to implement, as well as for all the other services which you can
override, take a look at the [services.yml file](Resources/config/services.yml).

### Running tests

The bundle uses PHPUnit to run functional tests.

#### Running tests in a working eZPublish / eZPlatform installation

To run the tests:

    export KERNEL_CLASS=App\Kernel (or whatever you renamed it to)
    export APP_ENV=behat (or whatever your environment is)

    bin/phpunit --stderr -c vendor/tanoconsulting/ibexa-migration-bundle/phpunit.xml.dist

*NB* the tests do *not* mock interaction with the database, but create/modify/delete many types of data in it.
As such, there are good chances that running tests will leave stale/broken data.
It is recommended to run the tests suite using a dedicated eZPublish installation or at least a dedicated database.

#### Setting up a dedicated test environment for the bundle

A safer choice to run the tests of the bundle is to set up a dedicated environment, similar to the one used when the test
suite is run on Travis.
The advantages are multiple: on one hand you can start with any version of eZPublish you want; on the other you will
be more confident that any tests you add or modify will also pass on Travis.
The disadvantages are that you will need Docker and Docker-compose, and that the environment you will use will look
quite unlike a standard eZPublish setup! Also, it will take a considerable amount of disk space and time to build.

Steps to set up a dedicated test environment and run the tests in it:

    # if you have a github auth token, it is a good idea to copy it now to Tests/docker/data/.composer/auth.json
    git clone --depth 1 --branch 0.2.0 https://github.com/tanoconsulting/euts.git teststack

    ./teststack/teststack build
    ./teststack/teststack runtests
    ./teststack/teststack stop

You can also run a single test case:

    ./teststack/teststack runtests ./Tests/phpunit/01_CollectionsTest.php

Note: this will take some time the 1st time your run it, but it will be quicker on subsequent runs.
Note: make sure to have enough disk space available.

In case you want to run manually commands, such as the symfony console:

    ./teststack/teststack console cache:clear

Or easily get to a database shell prompt:

    ./teststack/teststack dbconsole

Or command-line shell prompt to the Docker container where tests are run:

    ./teststack/teststack shell

The tests in the Docker container run using the version of debian/php/eZPlatform kernel specified in the file
`.euts.env`.
If you want to test against a different version, feel free to:
- add to it any required var (see file `teststack/.euts.env.example` as guidance)
- rebuild the test stack
- run tests the usual way

You can even keep multiple test stacks available in parallel, by using different env files, eg:
- create a file `.euts.env.local` and add to it any required env var, starting with a unique `COMPOSE_PROJECT_NAME`
- build the new test stack via `./teststack/teststack. -e .euts.env.local build`
- run the tests via: `./teststack/teststack -e .euts.env.local runtests`


[![License](https://poser.pugx.org/tanoconsulting/ibexa-migration-bundle/license)](https://packagist.org/packages/tanoconsulting/ibexa-migration-bundle)
[![Latest Stable Version](https://poser.pugx.org/tanoconsulting/ibexa-migration-bundle/v/stable)](https://packagist.org/packages/tanoconsulting/ibexa-migration-bundle)
[![Total Downloads](https://poser.pugx.org/tanoconsulting/ibexa-migration-bundle/downloads)](https://packagist.org/packages/tanoconsulting/ibexa-migration-bundle)

[![Build Status](https://github.com/tanoconsulting/ibexa-migration-bundle/actions/workflows/ci.yml/badge.svg)](https://github.com/tanoconsulting/ibexa-migration-bundle/actions/workflows/ci.yml)
[![Code Coverage](https://codecov.io/gh/tanoconsulting/ibexa-migration-bundle/branch/main/graph/badge.svg)](https://codecov.io/gh/tanoconsulting/ibexa-migration-bundle/tree/main)