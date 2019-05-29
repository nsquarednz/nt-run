# NT Run

A wrapper for Newman and Flyway, providing a simple automated test environment for newman capable of importing test data via flyway.

----------------------------------------------------------------------

# Newman 

#### (The cli companion for Postman)


[Newman](https://learning.getpostman.com/docs/postman/collection_runs/command_line_integration_with_newman/) is a command-line collection runner for Postman. It allows you to effortlessly run and test a Postman collection directly from the command-line. It is built with extensibility in mind so that you can easily integrate it with your continuous integration servers and build systems.


Tests in Postman are grouped together into "collections".
Each collection can contain multiple GET/POST/PUT/DELETE requests.
And each requests can contain multiple test that are run against it post execution.

----------------------------------------------------------------------

# Flyway 

#### (Version control for your database.)


[Flyway](https://flywaydb.org/) is an open-source database migration tool. It strongly favors simplicity and convention over configuration.

It is based around just 7 basic commands: Migrate, Clean, Info, Validate, Undo, Baseline and Repair.

Migrations can be written in SQL (database-specific syntax (such as PL/SQL, T-SQL, ...) is supported) or Java (for advanced data transformations or dealing with LOBs).

----------------------------------------------------------------------

# Configuration 


## Configuring flyway

This tool hooks into flyway to execute tests against an API connected to a database.

It is assumed that your project already has Flyway configured. If not, please start [here](https://flywaydb.org/documentation/).


## Creating/Editing newman tests

Tests are created and edited in [Postman](https://www.getpostman.com/).
See [POSTMAN.md](docs/POSTMAN.md) for a detailed walk-through on creating, modifying and exporting tests with Postman.


## Database test data

There is the option to apply test data to the database prior to executing the newman tests.

This is done by creating basic SQL scripts and placing them in the `db_test_data` folder.

SQL scripts must be named with __NO__ spaces or special characters. Instead use an underscore in-place of whitespace.


## Grouping tests

All test collections are stored in the sub-directory "tests".
Each collection __MUST__ be saved inside a newman group. Eg:
__"[ROOT]/tests/newman/tests/[NEWMAN_GROUP]"__

A newman group allows us to specify a single set of test data and newman cli arguments that is required for a group 
of collections.
This is configured with the `newman_group.config.json` file inside that group.

This is the current schema of the newman_group.config.json:

```
{
    "ignore_redirects":true,
    "database": {
        "clean_database_before_running_the_collections_in_this_group":true,
        "test_data":[
            "migration_1.sql",
            "migration_2.sql"
        ]
    }
}
```

  -  __ignore_redirects__
        Applies the `--ignore-redirects` flag to the newman command if set to __true__
  -  __database__
        A group of instructions for applying test data to the database
      - __clean_database_before_running_the_collections_in_this_group__
            Will force flyway to drop all tables and completely clear the database and start fresh
      - __test_data__
            A list of database schema change files located in `db_test_data` that will be applied as flyway 
            migrations prior to running the collections in the group. These are migrated in the order they are listed


## Config File

NT Run is configured with a .config file. This should be placed in your project's /tests directory. See the [example config file](.config.example).


## Executing tests from command-line

The `nt-run` script is a wrapper around newman and flyway that provides the ability to deploy 
test data and configure the newman command for each collection.

This should be executed within your projects 'tests' directory which contains a configured .config file.


```
Usage:      nt-run [OPTIONS] {FOLDER|COLLECTION}

Options:
             --help                                     -       Display this help message
             --all                                      -       Run all available collections
             --exit-on-fail                             -       Exit on first failure
             --list-collections                         -       List all available collections
             --list-groups                              -       List all available newman groups
             --location=<string>                        -       Execute the collection(s) at location. This can be a single collections or a newman group
             --teamcity                                 -       Add teamcity logging to test results

Examples:
             "./nt-run --all"                                                          -       Will execute all tests
             "./nt-run tests/group/SomeTestCollection.postman_collection.json"         -       Will execute the SomeTestCollection tests

```



