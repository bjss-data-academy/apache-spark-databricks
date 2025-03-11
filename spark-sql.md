# Working with Spark SQL
We can use familiar SQL to work with tables and views in Databricks.

Spark SQL adds extensions for databricks specific features. Examples are support for complex data types and Databricks [managed tables](data-storage.md).

## Working with Spark SQL
We can enter SQL directly into a Notebook. 

Before we can use SQL, we need some tables (or views) to work on. 

Sounds like a good excuse to practice converting a dataframe to a view.

### Convert dataframe to temporary view
It is often useful to work with dataframe objects using SQL. 

To do this, we must either save the dataframe as a table, or convert it to a temporary view.

Let's start with our scores dataframe from earlier. We'll create the dataframe:

```python
column_names = ["Player", "Score", "Out"]

scores_df = spark.createDataFrame([
      ("Alan", 0, "Yes"), 
      ("Rosie", 117, "No"), 
      ("Dan", 89, "Yes"),
      ("Tom", 89, "No")
    ], column_names)
```

We can convert to a temporary view by calling `createOrReplaceTempView()`:

```python
scores_df.createOrReplaceTempView("scores")
```

The view is named `"scores"`- passed in as the method parameter. 

We can write SQL queries against this view. 

Let's find the (_rather poor - ed_) players who were out for a duck (no runs):

```sql
select * from scores where score = 0 and out = 'Yes'
```

Giving the following list of miserable failures, batters who we hope will do better in future:

![Results of SQL statement](/images/useless-batters.png)

## Exploring data
We can find out data is available to us by ubnsing the `SHOW`, and `DESCRIBE` commands.

`SHOW CATALOGS` lists all Catalogs available to us:

![Show catalogs output](/images/show-catalogs.png)

Inside a catalogue, we can see what schemas are available, using `SHOW SCHEMA':

![Show schemas output](/images/show-schemas.png)

> Note the `USE` keyword to select one of the catalogs we listed before

We can then show all tables inside one of those schemas, using `SHOW TABLES`:

![Show tables output](/images/show-tables.png)

The table schema can be viewed using `DESCRIBE`:

![Describe table output](/images/describe-table.png)

Read more at [Databricks Documentation](https://docs.databricks.com/aws/en/discover/database-objects)

## Spark extensions to SQL
Spark extends SQL to support Databricks specific features:

- New column types STRUCT, ARRAY, MAP, JSON
- Dot notation to access STRUCT fields
- Colon notation to access MAP fields
- MANAGED and EXTERNAL tables

For more information, see [Databricks Spark SQL reference](https://docs.databricks.com/aws/en/sql/language-manual/)

## Creating tables
We can use Spark extended DDL (Data Definition Lnaguage) to create tables:

```sql
CREATE TABLE scores (
      player STRING,
      game STRING,
      score INTEGER );
```

We can then use standard SQL DML (Data Manipulation Language) to insert rows:

```sql
INSERT INTO scores VALUES ('Alan', 'scramble', 10000);
INSERT INTO scores VALUES ('Rosie', 'tiddlywinks', 15);
```

and execute queries:

```sql
SELECT * FROM scores ORDER BY score DESC;
```

## Spark DDL extensions
Spark _extends DDL_ syntax to allow us to create, populate and query tables using complex data types.

Let's create a table `game_results` suitable for holding a raw piece of JSON data we've ingested from an API.

Assume an example of the JSON structure is:

```json
{
  "scramble": [{ "playedOn": "2022-01-01", "score": 9950}]
}
```

> You might find a [JSV](https://tour.json-schema.org/content/01-Getting-Started/01-Your-First-Schema) defining the data schema. 
> 
> It would be found in the API documentation

We create a table with a column `game_results` to hold that structure:

```sql
CREATE TABLE IF NOT EXISTS game_results (
  player STRING, 
  game_history MAP<STRING, ARRAY<STRUCT<date STRING, score INTEGER>>>
);

INSERT INTO game_results VALUES (
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
    game_results 
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


Our silver layer processing often deals with such situations. We ingest complex data structures in bronze, as that is their original format. Silver layer then transforms those structures to something easier for analytics to work with.

## Handling existing tables
Sometimes a table already exists of the name we want to use.

We need to decide what to do with the existing table and its data.

We have three options:

- `DROP TABLE` remove the table if we know it is there
- `CREATE OR REPLACE TABLE` will delete all existing data in the table and replace it
- `CREATE TABLE IF NOT EXISTS` will take no action if the table already exists

Choose the one which fits your needs, based on keeping existing data or deleting it.

> Take care deleting data in managed tables. You can't get it back.

## Converting data files into tables
Have some data in files, like some CSV or JSON files? We can convert to tables directly.

Convert a CSV file `parts_data.csv` stored at `/mnt/data/parts_data.csv/` to a table:

```sql
CREATE TABLE parts
USING CSV
LOCATION '/mnt/data/parts_data.csv/';
```

## Auto-incrementing identity columns
We can mark a column to be auto-generated. Perfect for generating surrogate keys:

```sql
   CREATE TABLE defects
      id GENERATED ALWAYS AS IDENTITY,
      description STRING,
      reported_on DATE
```

Values for column `id` will be auto-generated.

## Using CTAS - Create Table As Select
`CREATE TABLE AS SELECT` (CTAS) is very useful. It creates a new table from the results of a SQL query. 

This is a common task in silver layer processing. We have some raw data. We want some part of that raw data, perhaps only a few columns with some aggregate information. We might want to combine that data with reference tables. CTAS is the perfect tool for the job.

Supppose our Bronze layer ingested raw data into tables `scores` and `contacts`.

![Contents of tables scores and contacts](/images/scores-contacts.png)

In our Silver layer, we want to work with three columns:

- the Player's _name_
- their _email_ address
- their _score_

We can create a new table `player_summary` using the CTAS syntax:

```sql
CREATE TABLE player_summary AS
SELECT s.Player, s.Score, c.Email
FROM scores s
JOIN contacts c
ON s.Player = c.Player;
```

Resulting in:

![Rows of new table player_summary](/images/player-summary.png)

> Handy: CREATE OR REPLACE TABLE <name> AS [...]


## Creating External tables
[External tables](/architecture.md) store their data outside of our Databricks system, perhaps on an external cloud provider, or corporate on-premises hardware.

We can mark a table in Databricks as being externally hosted:

```sql
CREATE TABLE parts
(
  id GENERATED ALWAYS AS IDENTITY,
  name STRING
)
LOCATION 's3://<bucket-path>/<table-directory>';
```

The key is to use the `LOCATION` keyword to specify an accessible path to the storage location.

> DROP TABLE and DELETE FROM will NOT delete data in the external table

For more, see [External Tables - Databricks Documentation](https://docs.databricks.com/aws/en/tables/external)

## Working with Unity Catalog and Schemas
Typically, the tables and views we work with will be grouped into a _schema_ managed overall by a _Unity Catalog_.

We will need permissions granting from whoever can grant them. That's outside scope.

Once we have permissions, the `USE` keyword specifies which catalog and schema we want to use.

Assuming a catalog named `bjss` and a schema named `training`:

```
%sql
USE CATALOG bjss;
USE SCHEMA training;
```

We can then reference table `scores`:

```sql
%sql
select * from scores;
```

## Using SQL in Python
We can use SQL statements directly against the dataframe object in Python:

![SQL statement in Python call](/images/sql-in-python.png)

This can be useful as it allows SQL to be generated inside Python code. 

# Further reading
Databricks documentation:
- [CREATE TABLE options](https://docs.databricks.com/aws/en/sql/language-manual/sql-ref-syntax-ddl-create-table-using)

For more information on SQL see:
- [Fundamentals of SQL](https://github.com/bjssacademy/fundamentals-sql/tree/main)
- [SQL for Data Engineering](https://github.com/bjss-data-academy/sql-for-data-engineering/blob/main/README.md)

# Next
It's important to know how Databricks manages data to get good results:

[Data Management in Databricks](/architecture.md)

[Back to Contents](/contents.md)
