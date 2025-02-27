# Using Spark SQL
We can use familiar SQL statements in Databricks, aloowing us to work with relational data in the form of tables and views. 

Spark SQL extends ANSI SQL syntax to cover Databricks specific features. These include support for complex data types in columns and the concept of [managed tables](data-storage.md).

## Working with Spark SQL

### Convert dataframe to temporary view
It is often useful to work with dataframe objects using SQL. To do this, we must either save the dataframe as a table, or convert it to a temporary view.

Let's start with our dataframe from earlier listing scores. We'll create it in memory first:

```python
%python
column_names = ["Player", "Score", "Out"]

scores_df = spark.createDataFrame([
      ("Alan", 0, "Yes"), 
      ("Rosie", 117, "No"), 
      ("Dan", 89, "Yes"),
      ("Tom", 89, "No")
    ], column_names)
```

Calling `createOrReplaceTempView()` on the dataframe converts it to a temporary view:

```python
scores_df.createOrReplaceTempView("scores")
```

This creates a temporary view named `scores`. We can treat the view as a normal table and use SQL against it.

Let's find the (_rather poor - ed_) players who were out for a duck (no runs):

```sql
select * from scores where score = 0 and out = 'Yes'
```

This shows the following list of miserable failures, batters who we hope will do better in future:

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
Spark _extends DDL_ syntax to allow us to create, populate and query tables using complex data types:

```sql

```

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

## Handling existing tables
Sometimes a table already exists of the name we want to use.

We need to decide what to do with the existing table and its data.

We have three options:

- __DROP TABLE__ remove the table if we know it is there
- __CREATE OR REPLACE TABLE__ will delete all existing data in the table and replace it
- __CREATE TABLE IF NOT EXISTS__ will take no action if the table already exists

Choose the one which fits your needs, based on keeping existing data or deleting it.

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

