# Using Spark SQL
We can use familiar SQL statements in Databricks, aloowing us to work with relational data in the form of tables and views. 

Spark SQL extends ANSI SQL syntax to cover Databricks specific features. These include support for complex data types in columns and the concept of managed tables.

Databricks provides an enhanced SQL syntax that works with tables and views. This superset of SQL - Spark SQL - includes Databricks extended features, such as support for complex data types in columns and managed tables.

## Managed and External tables in Databricks
Databricks offers two levels of support for tables - _managed_ tables or _external_ tables.

Tables in Databricks are made up of data plus table metadata:

- Data is the actual values inside rows and columns
- Metadata is information about how and where that data is stored

Databricks manages the metadata for both types. It has to really - Databricks needs to know about the data it is dealing with.

The difference between the two types lies in how and where the data is managed.

### Managed Tables

- Databricks controls metadata
- Databricks controls data

![Showing data and metadata insde Databricks management](/images/managed-table.png)  

In a managed table, Databricks fully controls the data that is stored inside the table along with metadata. It decides where the data is stored. It decides how best to work with that data.

> DROP TABLE deletes the metadata and the data. Data is permanently gone.

### External Tables

- Databricks stores and controls metadata
- External systems store and control data
- Databricks has no control over the external data, only access to it

![Showing Databricks managing only metadata with an external provider managing data](/images/unmanaged-external-table.png)

With an external table, Databricks still holds metadata information about where that data is held and how to acces it. But that's all.

The data itself is fully managed outside of databricks. 

Examples of such external storage include cloud storage and on-premise database products.

> DROP TABLE deletes the metadata only. External data is _unaffected_

If we drop an external table in databricks, the external data is unaffected. Only Databricks metadata is removed.

> We can restore external data - the metadata can be recreated

### Delta Tables
Delta tables are a Databricks data storage format that builds on the open-source [Parquet](https://github.com/apache/parquet-format) file formay to add:

- ACID transactions
- Time-travel (versioned history of data at points in time)
- High performance append and delete of rows
- High performance data caching
- Schema enforcement

The secret sauce is to add a _transaction log_ to the raw parquet file storage:

![Delta table showing parquet file data rows with column metadata plus three entries in a transaction log](/images/delta-tables-interbals.png)

> Delta tables are the preferred format in Databricks

### Other table formats

TODO TODO 


## Working with Spark SQL
SQL only works on tables and views. To use SQL with a dataframe, we must first convert it to a table or a view.

TODO
TODO - repetition to remove, rearrange to use scored_df as running example - clarify text and floe
TODO

### Creating an empty table
This follows standard ANSI SQL DDL syntax, with the ability to work with complex data types.

A simple example of creating a basic `User` table is:

```sql
CREATE TABLE User (
    id BIGINT,
    name STRING,
    email_address STRING,
    bio STRING
)
```

> This creates the table `User` as a _managed_ table, using Delta format.

Rows can be inserted in the normal way:

```sql
INSERT INTO User VALUES (1, "Alan", "al@example.com", "Author Java OOP Done Right, Test-Driven Deveopment in Java. Co-author Nokia Bounce, Fun School 2, Red Arrows")
```

### Create an external table
If our user data was stored outside Databricks, we could access it as an _external table_.

The syntax to create external tables uses the `LOCATION` keyword, to specify where the data is stored:

```sql
CREATE TABLE external_user
LOCATION '/mnt/external_storage/external_user/';
```

### Convert dataframe to temporary view
It is often useful to work with dataframe objects using SQL. To do this, we must either save the dataframe as a table, or convert it to a temporary view.

Converting to a temporary view is a single method call on the dataframe you want to convert:

```python
scores_df.createOrReplaceTempView("scores")
```

The `createOrReplaceTempView` method of dataframe makes an in-memory SQL view. It is a simple matter to run SQL against it in a new notebook. 

Assuming our dataframe is our `scores-df` from earlier, let's find the (_rather poor - ed_) players who were out for a duck (no runs):

```sql
select * from scores where score = 0 and out = 'Yes'
```

Showing the following list of miserable failures, batters who hope to do better in future:

![Results of SQL statement](/images/useless-batters.png)

## Basic SQL queries
Once we have a table or view, all the usual SQL basics apply. 

For more information, see [SQL for Data Engineering](https://github.com/bjss-data-academy/sql-for-data-engineering/blob/main/README.md) and [Fundamentals of SQL](https://github.com/bjssacademy/fundamentals-sql/tree/main)

![SQL statement in Python call](/images/sql-in-python.png)

This can be useful as it allows SQL to be generated inside Python code. 

## Spark extensions to SQL
Spark adds some new features to SQL:

- New column types STRUCT, ARRAY, MAP, JSON
- Dot notation to access STRUCT fields
- Colon notation to access MAP fields

For more information, see [Databricks Spark SQL reference](https://docs.databricks.com/aws/en/sql/language-manual/)

## Creating tables with SQL DDL
We can use SQL DDL (Data Definition Lnaguage) to create tables:

```sql
USE `2870560854981942`.`spark-training`;

CREATE TABLE `spark-training`.scores (player STRING, game STRING, score INTEGER);
```

We can then use standard SQL DML (Data Manipulation Language) to insert rows:

```sql
INSERT INTO `spark-training`.scores VALUES ('Alan', 'scramble', 10000);
INSERT INTO `spark-training`.scores VALUES ('Rosie', 'tiddlywinks', 15);
```

and execute queries:

```sql
SELECT * FROM `spark-training`.scores ORDER BY score DESC;
```

### Spark DDL extensions
Spark _extends DDL_ syntax to allow us to create, populate and query tables using complex data types.

Here is a more complex - and highly contrived - example. 

Let's create and populate a table with a column holding a map of arrays of struct:

```sql
CREATE TABLE IF NOT EXISTS `spark-training`.examples (
  player STRING, 
  game_history MAP<STRING, ARRAY<STRUCT<date STRING, score INTEGER>>>
);

INSERT INTO `spark-training`.examples VALUES (
  "Alan", 
  MAP("scramble", ARRAY(
    STRUCT("2022-01-01", 9950), 
    STRUCT("2025-02-21", 119500)
  ))
);
```

This gives us a table with a complex data type for the `game_history` column:

![Output of rows in our complex schema](/images/complex-create.png)

We can run the following Spark SQL query to find the highest score achieved by all Scramble game players:

```sql
WITH exploded_game_history AS (
  SELECT 
    player, 
    game, 
    results
  FROM 
    `spark-training`.examples 
    LATERAL VIEW explode(game_history) AS game, results
),
exploded_results AS (
  SELECT
    player,
    game,
    result.date,
    result.score
  FROM 
    exploded_game_history
    LATERAL VIEW explode(results) AS result
)

SELECT player, max(score) FROM exploded_results WHERE game = 'scramble' GROUP BY player;
```

which gives a much more straightforward result:

![Results of complex query](/images/complex-query.png)

While contrived, this _could_ appear in real-life following ingestion of JSON structured data provided by a REST or GraphQL API. 

Our silver layer processing would want to work with that nested data to simplify access. Simplified results would populate the Gold layer tables.

## CTAS - Create Table As Select
TODO TODO TODO


# TODO
vvvv TODO  vvvvv

create or replace table

Managed versus Unmanaged tables [external]
deletion rules

Creating views
Metadata: What's inside this database?
Reading tables into dataframes
Reading CSV into table

We can also use SQL programmatically, using `spark.sql`:

