---
layout: documentation
title: Multi-Component Data Model
---

# Multi-Component Data Model #

Propel comes along with packaging capabilities that allow you to more easily integrate Propel into a packaged or modularized application.

## Muliple Schemas ##

You can use as many `schema.xml` files as you want. Schema files have to be named `(*.)schema.xml`, so names like `schema.xml`, `package1.schema.xml`, `core.package1.schema.xml` are all acceptable. These files _have_ to be located in your project directory.

Each schema file has to contain a `<database>` element with a `name` attribute. This name references the connection settings to be used for this database (and configured in the `runtime-conf.xml`), so separated schemas can share a common database name.

Whenever you call a propel build taks, Propel will consider all these schema files and build the classes (or the SQL) for all the tables.

## Understanding Packages ##

In Propel, a _package_ represents a group of models. This is a convenient way to organize your code in a modularized way, since classes and SQL files of a given package are be grouped together and separated from the other packages. By carefully choosing the package of each model, applications end up in smaller, independent modules that are easier to manage.

### Package Cascade ###

The package is defined in a configuration cascade. You can set it up for the whole project, for all the tables of a schema, or for a single table.

For the whole project, the main package is set in the `build.properties`:

{% highlight ini %}
propel.targetPackage = my_project
{% endhighlight %}

By default, all the tables of all the schemas in the project use this package. However, you can override the package for a given `<database>` by setting its `package` attribute:

{% highlight xml %}
<!-- in author.schema.xml -->
<database package="author" name="bookstore">
	<table name="author">
		<!-- author columns -->
	</table>
</database>

<!-- in book.schema.xml -->
<database package="book" name="bookstore">
	<table name="book">
		<!-- book columns -->
	</table>
	<table name="review">
		<!-- review columns -->
	</table>
</database>
{% endhighlight %}

In this example, thanks to the `package` attribute, the tables are grouped into the following packages:

* `my_project.author` package: `author` table
* `my_project.book` package: `book` and `review` tables

>**Warning**<br />If you separate tables related by a foreign key into separate packages (like `book` and `author` in this example), you must enable the `packageObjectModel` build property to let Propel consider other packages for relations.

You can also override the `package` attribute at the `<table>` element level.

{% highlight xml %}
<!-- in author.schema.xml -->
<database package="author" name="bookstore">
	<table name="author">
		<!-- author columns -->
	</table>
</database>

<!-- in book.schema.xml -->
<database package="book" name="bookstore">
	<table name="book">
		<!-- book columns -->
	</table>
	<table name="review" package="review">
		<!-- review columns -->
	</table>
</database>
{% endhighlight %}

This ends up in the following package:

* `my_project.author` package: `author` table
* `my_project.book` package: `book` table
* `my_project.review` package: `review` table

Notice that tables can end up in separated packages even though they belong to the same schema file.

_Tip_: You can use dots in a package name to add more package levels.

### Packages and Generated Model Files ###

The `package` attribute of a table translates to the directory in which Propel generates the Model classes for this table.

For instance, if no `package` attribute is defined at the database of table level, Propel places all classes according to the `propel.targetPackage` from the `build.properties`:

{% highlight text %}
build/
  classes/
    my_project/
      om/
      map/
      Author.php
      AuthorPeer.php
      AuthorQuery.php
      Book.php
      BookPeer.php
      BookQuery.php
      Review.php
      ReviewPeer.php
      ReviewQuery.php
{% endhighlight %}

You can further tweak the location where Propel puts the created files by changing the `propel.output.dir` build property. By default this property is set to:

{% highlight ini %}
propel.output.dir = ${propel.project.dir}/build
{% endhighlight %}

You can change it to use any other directory as your build directory.

If you set up packages for `<database>` elements, Propel splits up the generated model classes into subdirectories named after the package attribute:

{% highlight text %}
build/
  classes/
    my_project/
      author/
        om/
        map/
        Author.php
        AuthorPeer.php
        AuthorQuery.php
      book/
        om/
        map/
        Book.php
        BookPeer.php
        BookQuery.php
        Review.php
        ReviewPeer.php
        ReviewQuery.php
{% endhighlight %}

And of course, if you specialize the `package` attribute per table, you can have one table use its own package:

{% highlight text %}
build/
  classes/
    my_project/
      author/
        om/
        map/
        Author.php
        AuthorPeer.php
        AuthorQuery.php
      book/
        om/
        map/
        Book.php
        BookPeer.php
        BookQuery.php
      review/
        om/
        map/
        Review.php
        ReviewPeer.php
        ReviewQuery.php
{% endhighlight %}

### Packages And SQL Files ###

Propel also considers packages for SQL generation. In practice, Propel generates one SQL file per package. Each file contains the CREATE TABLE SQL statements necessary to create all the tables of a given package.

So by default, all the tables end up in a single SQL file:

{% highlight text %}
build/
  sql/
    schema.sql
{% endhighlight %}

If you specialize the `package` for each `<database>` element, Propel uses it for SQL files:

{% highlight text %}
build/
  sql/
    author.schema.sql // contains CREATE TABLE author
    book.schema.sql   // contains CREATE TABLE book and CREATE TABLE review
{% endhighlight %}

And, as you probably expect it, a package overridden at the table level also acocunts for an independent SQL file:

{% highlight text %}
build/
  sql/
    author.schema.sql // contains CREATE TABLE author
    book.schema.sql   // contains CREATE TABLE book
    review.schema.sql // contains CREATE TABLE review
{% endhighlight %}

## Understanding The packageObjectModel Build Property ##

The `propel.packageObjectModel` build property enables the "packaged" build process. This modifies the build tasks behavior by joining `<database>` elements of the same name - but keeping their packages separate. That allows to split a large schema into several files, regardless of foreign key dependencies, since Propel will join all schemas using the same database name.

To switch this on, simply add the following line to the `build.properties` file in your project directory:

{% highlight text %}
propel.packageObjectModel = true
{% endhighlight %}

## The Bookstore Packaged Example ##

In the bookstore-packaged example you'll find the following schema files:

 * author.schema.xml
 * book.schema.xml
 * club.schema.xml
 * media.schema.xml
 * publisher.schema.xml
 * review.schema.xml
 * log.schema.xml

Each schema file has to contain a `<database>` tag that has its `package` attribute set to the package name where _all_ of the tables in this schema file/database belong to.

For example, in the bookstore-packaged example the `author.schema.xml` contains the following `<database>` tag:

{% highlight text %}
<database package="core.author" name="bookstore" [...]>
{% endhighlight %}

That means, that the Author OM classes will be created in a subdirectory `core/author/` of the build output directory.

You can have more than one schema file that belong to one package. For example, in the the bookstore-packaged example both the `book.schema.xml` and `media.schema.xml` belong to the same package "core.book". The generated OM classes for these schemas will therefore end up in the same `core/book/` subdirectory.

### The OM build ###

To run the packaged bookstore example build simply go to the `propel/test/fixtures/bookstore-packages/` directory and type:

{% highlight text %}
../../../generator/bin/propel-gen om
{% endhighlight %}

This should run without any complaints. When you have a look at the projects/bookstore-packaged/build/classes directory, the following directory tree should have been created:
{% highlight text %}
addon/
   club/
      BookClubList.php
      BookClubListPeer.php
      BookListRel.php
      BookListRelPeer.php
core/
   author/
      Author.php
      AuthorPeer.php
   book/
      Book.php
      BookPeer.php

      Media.php
      MediaPeer.php
   publisher/
      Publisher.php
      PublisherPeer.php
   review/
      Review.php
      ReviewPeer.php
util/
   log/
      BookstoreLog.php
      BookstoreLogPeer.php
{% endhighlight %}

(The additional subdirectories map/ and om/ in each of these directories have been omitted for clarity.)

## The SQL build ##

From the same schema files, run the SQL generation by calling:

{% highlight text %}
../../../generator/bin/propel-gen sql
{% endhighlight %}

Then, have a look at the `build/sql/` directory: you will see that for each package (that is specified as a package attribute in the schema file database tags), one sql file has been created:

* addon.club.schema.sql
* core.author.schema.sql
* core.book.schema.sql
* core.publisher.schema.sql
* core.review.schema.sql
* util.log.schema.sql

These files contain the CREATE TABLE SQL statements necessary for each package.

When you now run the insert-sql task by typing:

{% highlight text %}
../../../generator/bin/propel-gen insert-sql
{% endhighlight %}

these SQL statements will be executed on a SQLite database located in the Propel/generator/test/ directory.
