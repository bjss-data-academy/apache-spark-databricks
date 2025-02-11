# Using Spark SQL
Overview of Spark SQL
Convert Dataframe to temporary view
Basic SQL queries
Defining a schema
Managed versus Unmanaged tables
Creating tables with SQL DDL
Creating views
Metadata: What's inside this database?
Reading tables into dataframes
Reading CSV into table

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

We can also use SQL programmatically, using `spark.sql`:

![SQL statement in Python call](/images/sql-in-python.png)

This can be useful as it allows SQL to be generated inside Python code. 


