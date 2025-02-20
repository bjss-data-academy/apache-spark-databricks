# Using Spark SQL
Databricks allows us to use Spark with any supported programming language. 

At the time of writing, these are:

- Python
- SQL
- R
- Scala
- Java

The most popular in Data Engineering at BJSS are Python and SQL.

The point to note is that the _same functionality_ can be achieved from _any_ language. Anything we can do with a dataframe in Python can be done in SQL.

SQL only works on tables and views. To use SQL with a dataframe, we must first convert it to a table or a view.

## Convert dataframe to temporary view
We can save our dataframe object as a temporary view, allowing us to run SQL queries against it.

This is a single method call on the dataframe you want to convert:

```python
scores_df.createOrReplaceTempView("scores")
```

The `createOrReplaceTempView` method of dataframe makes the table. Then it is a simple matter to run SQL against it in a new notebook. Let's find the (_rather poor - ed_) players who were out for a duck (no runs):

```sql
select * from scores where score = 0
```

Showing the following list of miserable failure, batters who hope to do better in future:

![Results of SQL statement](/images/useless-batters.png)

## Basic SQL queries
Once we have a table or view, all the usual SQL basics apply. 

Fr more information, see [SQL for Data Engineering](https://github.com/bjss-data-academy/sql-for-data-engineering/blob/main/README.md) and [Fundamentals of SQL](https://github.com/bjssacademy/fundamentals-sql/tree/main)


![SQL statement in Python call](/images/sql-in-python.png)

This can be useful as it allows SQL to be generated inside Python code. 

## Creating tables with SQL DDL
We can use standard SQL DDL (Data Definition Lnaguage) to create tables:

TODO TODO TODO

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

