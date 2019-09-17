---
id: 3-my-first-crud
title: 3. My First CRUD
---

## Objective

Create a CRUD (Create, Read, Update Delete) of Products that will be accessible through the `api/products` endpoint. The goal is to learn the following: 
 
 - How to configure a database
 - A workflow to create API Endpoints
 - Good API architectural practices
 - What are controllers, models, factories and migrations.

## Prerequisites

- Have [PostgreSQL](https://www.postgresql.org) installed
- Have [Postman](https://www.getpostman.com) installed

## Workflow

In the day to day you will be working across all the different layers of your application. During the first times working that way, it can be daunting, since there is still a lot you don't know nor how it works. Nevertheless, I think is important to get to know the basics of all of this layers so you can start perceiving the value of the framework's architecture. In a typical day you will do something like this:

1. Understand the client necessity
1. Define the API Contract (Endpoints)
1. Create tables in the database (if necessary) using Migrations
1. Create Routes
1. Create Controllers
1. Create Models
1. Create a Factory for testing
1. Create Tests for each endpoint
1. Create a Request in Postman so you can share it across your team 
   
If you don't know some of this concepts don't worry. We will do a deep dive on each of them later. But for know let's try to create a CRUD following the above workflow.

## One time database configuration (Hopefully)

Database configurations is usually a one time thing to do in every project. It's a little complex but once it is done is done.
  
### 1. Enter to the PostgreSQL console

If your database is running locally you should be able to run the following command to enter to the postgresql console:

```shell
# You can run it in any directory
psql -U postgres 
# Change "postgres" for the name of your user if postgres is not it.

```

### 2. Check what databases you already have 

Inside the postgreSQL shell run this command to show all the databases that PostgreSQL have
```shell
\l
```

### 3. Creating our Development and Testing Databases

When you are working, you usually need a development database and a testing database. The latter should only be used when you run your tests and the former is the one you used to manually validate the changes you are doing into the application. The test database should always have an empty state before each test-run to ensure that there is not strange data that could interfere with the test data.  

To create a database run this command inside the PostgreSQL console.

```shell
CREATE DATABASE laravel;
```

> "laravel" is the name of this database but you can change it to anything you want.

Lets see our new database by running `\l`:

```shell
postgres=# \l
                                         List of databases
            Name            |  Owner   | Encoding |  Collate   |   Ctype    |   Access privileges
----------------------------+----------+----------+------------+------------+-----------------------
 laravel                    | postgres | UTF8     | en_US.utf8 | en_US.utf8 |
 postgres                   | postgres | UTF8     | en_US.utf8 | en_US.utf8 |
 template0                  | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
                            |          |          |            |            | postgres=CTc/postgres
 template1                  | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
                            |          |          |            |            | postgres=CTc/postgres
```

The "laravel" database will be used as your development database. Lets create the test database. 

```shell
create database laravel_test
```

> As before "laravel_test" is just a name and it can be changed.

### Configuring the databases in Laravel

By default, laravel knows how to communicate with mysql, sqlite and postgres by using specific drivers. We only need to tell laravel which driver should it use by specifying the driven name on the `.env` file.  This file is where we should put everything that is environment dependent. For example, an application can (and should) use a different database depending if it is being tested using automated tests(testing environment), if it is being used by the mobile team to build new features (staging environment), or if it is deployed to production (production environment)

Add the following to your `.env` file.

```
# .env
DB_CONNECTION=pgsql
DB_HOST=127.0.0.1
DB_PORT=5432
DB_DATABASE=laravel
DB_USERNAME=postgres
DB_PASSWORD=
``` 

Now lets do the same but for our testing environment. Lets create a `env.testing` file that laravel will use when we run our test using `phpunit`. Just copy everything in your `.env` file into `.env.testing`.

The database part of `env.testing` should look something like this:

```
DB_CONNECTION=pgsql
DB_HOST=127.0.0.1
DB_PORT=5432
DB_DATABASE=laravel_test
DB_USERNAME=postgres
DB_PASSWORD=
```

Now let's test if our database connections are setup correctly. Run the following command in the project root directory to start a laravel console:

```shell
php artisan tinker
```

> The laravel console runs by default using the variables inside the `.env` file. So every change you make here will be reflected in your `development` environment.

Inside the laravel console run this command to check if we are connected to the database:

```
DB::connection()->getPdo()
```

If everything is correctly connected you should see something like this:
 
```
=> PDO {#2960
     inTransaction: false,
     attributes: {
       CASE: NATURAL,
       ERRMODE: EXCEPTION,
       PERSISTENT: false,
       DRIVER_NAME: "pgsql",
       SERVER_INFO: "PID: 984; Client Encoding: UTF8; Is Superuser: on; Session Authorization: postgres; Date Style: ISO, MDY",
       ORACLE_NULLS: NATURAL,
       CLIENT_VERSION: "11.5",
       SERVER_VERSION: "11.4 (Debian 11.4-1.pgdg90+1)",
       STATEMENT_CLASS: [
         "PDOStatement",
       ],
       EMULATE_PREPARES: false,
       CONNECTION_STATUS: "Connection OK; waiting to send.",
       DEFAULT_FETCH_MODE: BOTH,
     },
   }

```

Now validate your testing environment database by running this command. The laravel console will use the variables inside `.env.testing`.

```shell
php artisan tinker --env=testing 
```

When you run `DB::connection()->getPdo()` you should get the same result as before. If everything is working lets start making our first CRUD!  

## A CRUD of Products

- Run this command in your project root:

```shell
php artisan make:model Product -mcr
```

The above command will:

1. Create a model called `Product.php` inside the `app/` directory.
2. Create a controller called `ProductController.php` inside the `app/Http/Controllers/` directory.
3. Create a migration called `xxxx_xx_xx_xxxx_create_products_table.php` inside the `database/migrations/` directory.
4. Create a factory called `ProductFactory.php` inside the `database/factories/` directory.


### Create Product Table

If we open the `xxxx_xx_xx_xxxx_create_products_table.php` you will see something like this:

```php
<?php

use Illuminate\Support\Facades\Schema;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Database\Migrations\Migration;

class CreateProductsTable extends Migration
{
    /**
     * Run the migrations.
     *
     * @return void
     */
    public function up()
    {
        Schema::create('products', function (Blueprint $table) {
            $table->bigIncrements('id');
            $table->timestamps();
        });
    }

    /**
     * Reverse the migrations.
     *
     * @return void
     */
    public function down()
    {
        Schema::dropIfExists('products');
    }
}
```

Lets add columns to store the name and price of the products. Your migration should look something like this:

```php
<?php

use Illuminate\Support\Facades\Schema;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Database\Migrations\Migration;

class CreateProductsTable extends Migration
{
    /**
     * Run the migrations.
     *
     * @return void
     */
    public function up()
    {
        Schema::create('products', function (Blueprint $table) {
            $table->bigIncrements('id');
            $table->timestamps();
            // A string to store the name
            $table->string('name');
            // A decimal to store the price
            $table->decimal('price');
        });
    }

    /**
     * Reverse the migrations.
     *
     * @return void
     */
    public function down()
    {
        Schema::dropIfExists('products');
    }
}
```

Save the file and run this command in your project root directory to apply the changes in your database:

```shell
php artisan migrate
```

### Create Product

- Create a Test (and make it fail)
- Add Route
- Return the correct HTTP Status
- Return the correct body structure and data 
- Verify the product is stored in the database
 
### List All Products

- Create a Test
- Add Route
- Return the correct HTTP Status
- Return the correct structure body and data

### Show Product
- Create a Test
- Add Route
- Return the correct HTTP Status
- Return the correct structure body and data

### Update Product
- Create a Test (and make it fail)
- Add Route
- Return the correct HTTP Status
- Return the correct body structure and data 
- Verify is updated in the database

### Delete Product
- Create a Test (and make it fail)
- Add Route
- Return the correct HTTP Status
- Return the correct body structure and data 
- Verify is removed from the database

