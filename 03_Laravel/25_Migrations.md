# Migrations

## Important configuration preface
In `/app/Providers/AppServiceProvider.php` update the `boot` method adding the following Schema line:

```php
public function boot()
{
    \Schema::defaultStringLength(191);
}
```

This change is necessary for the work we'll do in this note set. More details as to what this change is doing will be discussed in lecture.

<small markdown=1>[ref: Laravel docs: Index Lengths & MySQL / MariaDB](https://laravel.com/docs/migrations#creating-indexes)</small>


## What are migrations?
With your application's local database created and your connection to that database confirmed, it's time to build your tables.

Rather than building tables with raw SQL queries or using a tool like phpMyAdmin, we'll use Laravel migrations.

__Migrations are PHP scripts that describe alterations to your database&mdash; whether it be adding tables, dropping tables, adding new columns to tables, etc.__

The benefits of managing your database with migrations include...

+ The scripts can be tracked in version control so you have a history of changes over time.
+ When you deploy your application to a new/different server, you can easily run your migrations to create a  copy of your database for that server.
+ Similarly, new developers can quickly build copies of your database on their local servers.

<br>

**Before proceeding, if you created a `books` table when reading the *Databases Primer* notes, delete that table now.**

<img src='http://making-the-internet.s3.amazonaws.com/laravel-how-to-delete-a-table-in-phpmyadmin@2x.png' style='max-width:1000px; width:100%' alt=''>


## Generate a new migration file
Artisan has several commands to help with migrations.

First, we'll prompt Artisan to generate a new migration file.
```bash
$ php artisan make:migration create_books_table
```

The migration name (`create_books_table`) should describe what you're doing. In this example we're creating a table called `books`.

After you run the above command, navigate to `/database/migrations/` where you should see a new file that's named something like `2017_04_06_072743_create_books_table.php`.

In `/database/migrations/` you'll also see 2 pre-existing migrations that come with Laravel by default: `create_users_table` and `create_password_resets_table`. These tables support Laravel's user authentication features, which will be discussed more at a later date.

Note that each migration filename is prefixed with a unique timestamp&mdash; this ensures migrations are run in the same order in which they were created.


## Write your first migration
Open up your the books migration file, and you should see something like this:
```php
<?php

use Illuminate\Support\Facades\Schema;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Database\Migrations\Migration;

class CreateBooksTable extends Migration
{
    /**
     * Run the migrations.
     *
     * @return void
     */
    public function up()
    {
        Schema::create('books', function (Blueprint $table) {
            $table->increments('id');
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
        Schema::dropIfExists('books');
    }
}
```

Every generated migration has this boilerplate code with a `up` and `down` method.

+ In the `up` method you'll describe some change to the database (adding a new table, adding fields to a table, etc.).
+ In the `down` method you'll reverse any changes made in the `up` method.


## Design the books table
Before writing any code in the `up` method, we have to decide what fields the table will need, and what MySQL data type each field in the table should be.

To do this, we'll refer to the plan we came up with in the *Database Primer* notes:

<img src='http://making-the-internet.s3.amazonaws.com/laravel-books-table-design@2x.png' style='max-width:480px; width:100%' alt='books table design'>

To understand the MySQL data types chosen each field, [refer again to this table](https://github.com/susanBuck/dwa15-fall2017/blob/master/03_Laravel/24_Common_MySQL_Data_Types.md).

For all our tables/fields, we'll follow these naming conventions:

+ Table names are plural.
+ Tables and field names are written in `snake_case`.

With the `books` table design in mind, it's time to write the migration code that will create this table.


## up - Build the table
Modify the `up` method so it looks like this:

```php
public function up() 
{

	Schema::create('books', function (Blueprint $table) {

		# Increments method will make a Primary, Auto-Incrementing field.
		# Most tables start off this way
		$table->increments('id');

		# This generates two columns: `created_at` and `updated_at` to
		# keep track of changes to a row
		$table->timestamps();

		# The rest of the fields...
		$table->string('title');
		$table->string('author')->nullable(); # Example of a modifier
		$table->integer('published');
		$table->string('cover');
		$table->string('purchase_link');

		# FYI: We're skipping the 'tags' field for now; more on that later.
	});
}
```

Note how the Laravel `Schema` component is used to create the `books` table.

The above code includes comments for the `Schema` methods we're using, but be sure to read the [Schema Documentation](http://laravel.com/docs/migrations#creating-tables) for full details.



## down - Drop the table
The &ldquo;undo&rdquo; action for creating a new table is simple&mdash; just drop the table:

```php
public function down()
{
    Schema::dropIfExists('books');
}
```



## Run your migrations
When you've completed writing the code for your migration, you'll use Artisan to run it:

```bash
$ php artisan migrate
```

Example output:
```bash
Migration table created successfully.
Migrated: 2014_10_12_000000_create_users_table
Migrated: 2014_10_12_100000_create_password_resets_table
Migrated: 2017_04_06_072743_create_books_table
```

FYI: The first time you run `php artisan migrate` it will create a `migrations` table which will be used to keep track of what migrations you've run. 

That should do the trick. Examine your new `books` table in phpMyAdmin to make sure it matches the design you were aiming for.

Moving forward, you should not make *any* structural changes to your database tables via phpMyAdmin. phpMyAdmin should now only be used to view tables and data. Any structural changes to tables need to be made with migrations. If you make changes via phpMyAdmin, your database/tables will be out of sync with your migrations.


## Fresh
Getting your migrations right the first time can be challenging, especially if you're starting a new project with several new tables. You may forget fields you need or change your mind about the data type and modifiers for a given fields. There may also be problems in your migration files that cause a migration to fail partway through, leaving you with an incomplete database.

Given this, there's an easy way to &ldquo;blank slate&rdquo; your database and start over with the following Artisan command:
```bash
$ php artisan migrate:fresh
```

This will drop all your tables and then run your migrations again (presumably after you've edited them to correct whatever problem existed).

The `migrate:fresh` command is useful when working locally, or in the early stages of a project when you don't have real data being entered into a production database. When a project is released into the real-world though, you don't want to run `php artisan migrate:fresh` on production since it will whipe out all your real-world data.

To avoid accidentally doing this, Artisan will warn you when the command is run on your production environment:
```xml
root@DWA-Fall-2017:/var/www/html/foobooks# php artisan migrate:fresh

**************************************
*     Application In Production!     *
**************************************

 Do you really wish to run this command? (yes/no) [no]:
```



## Altering tables 
Once a table has been created and used in a live-production setting, any future changes to that table should also be made via a migration.

For example, let's imagine down the road you wanted to add a `page_count` field to the books table. In that situation, you'd do the following (you don't have to do this now since this is purely hypothetical):

```bash
$ php artisan make:migration add_page_count_field_to_books_table
```

In the resulting migration, your up method would look something like this:
```php
public function up() {
    Schema::table('books', function (Blueprint $table) {
        $table->integer('page_count');
    });
}
```

And your down method would look something like this:
```php
public function down() {
    Schema::table('books', function (Blueprint $table) {
        $table->dropColumn('page_count');
    });
}
```

When you next run `php artisan migrate`, it would recognize this new migration that has not been run, and it would run it, altering your `books` table.

Using migrations in this way effectively creates a version control system for your database tables, allowing you to manage changes over time and across different environments. 

In this course, we don't need to worry about these kind of alteration migrations because we're not dealing with real data. Given that, it's more straightfoward to just make edits to your existing &ldquo;create table&rdquo; migrations and re-run them. 

*To be discussed in lecture: Factors to consider when altering tables in real world environments that have real world data.*
