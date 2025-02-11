# Starting with DataFrames
The core concept in Apache Spark for working with data is the [_Dataframe_](https://spark.apache.org/docs/latest/api/python/reference/pyspark.sql/api/pyspark.sql.DataFrame.html#pyspark.sql.DataFrame).

A Dataframe is an [object](https://github.com/bjss-data-academy/python-essentials/blob/main/06-objects.md) that does the heavy lifting of managing and analysing data for us. 

We don't need to see the code inside a DataFrame in order to use it. But it is good to know some of the benefits it provides

## Benefits of DataFrames

- _Universal abstraction_. Once we have data in a dataframe, we can work on it in a uniform way. 
- _Storage abstraction_ it does not matter where the data came from or goes to
- _In-memory_ the data resides in system memory, making processing faster
- _Optimised for Columnar data_ the in-memory data is organised to make operations on whole columns efficient. This matches data lake usage well

A dataframe object will manage large amounts of data for us, and provide it in an easy-to-use way. It is optimised to work with data where we want to know soething about entire _columns_ rather than individual rows. 

### Why optimise for columns?
A lot of business-level analysis relates to a single value: total _profit_, average _volume_, shortest _lead time_. These are all aggregate quntities of individual businees facts.

An example queryt a business would like to answer is _what is the average spend across all customers?_

To answer this query, we would be looking at a column of _sold-price_ for each product sold, and then sum the total. The dataframe object makes that sum efficient to perform.

Because many top-level queries aggregate values across all rows in a single column, it makes sense to optimise processing for columns of data.

## Reading data into a dataframe
We can read data in several popular formats into a dataframe.

Let's read in a CSV file:

```python
%python
scores_df = spark.read.format("csv") 
  .option("header", "true") 
  .option("inferSchema", "true") 
  .load("/path/to/file/cricket_scores.csv")
```

It's one line of code. Let's break down each piece:

- `spark.read.format("csv")` read a file, and expect CSV format
- `option("header", "true")` expect the first line to be column headers and discard
- `option("inferSchema", "true")` work out the data type for each value of data
- `load("/path/to/file/cricket_scores.csv")` load the file `cricket_scores.csv` from the specified path

This line returns an instance of a _dataframe_ object that contains the data in the CSV file. We assign that to Python variable `scores_df` for later use.

## Displaying the dataframe content
We can take a look at the data inside the dataframe by typing `display(scores_df)` in a Python notebook window. We will see a tabular output window showing the data:

![Tabular output of display(scores_df)](/images/display-scores-df.png)

## Using SQL like methods
SQL we know and love from the relational database world. Dataframes provide familiar SQL-like methods to work with data.

Let's select all rows where the player did not go out for a duck (scored at least one run):

```python
runners_df = scores_df.\
    select("Player", "Score", "Out")\
    .where("Score > 0")\
    .orderBy("Score", ascending=False)

display(runners_df)
```

Which shows:

![Tabular output of Spark SQL](/images/select-scores-df.png)

The `select` method on dataframe is the entry point to running SQL style methods. `select` will return a new dataframe object, containing rows matching the query. Above, we had a `where` clause to filter out zero scoring rows, and an `orderBy` clause, for results in descending score order.

The methods chain together in a style similar to an [SQL query](https://github.com/bjss-data-academy/sql-for-data-engineering/blob/main/README.md). We'll look at Spark SQL support in the next section, and how this seamlessly works with dataframes.

## Working with columns
Dataframe objects have plenty of ways of transforming data. We'll cover transforming data in more depth in the next section.

For now, let's look at some basic features of working with columns in a dataframe. 

### Rename column
Changing the name of a column is useful and easy. 

Let's rename the column _Out_ to _bowled_out_.

```python
renamed_scores_df = scores_df.withColumnRenamed("out", "bowled_out")
```

Displaying the dataframe shows the new column name:

![Renamed column as displayed](/images/renamed-column-df.png)

### Add new column
As we transform our data through the silver layer, it's often useful to add new columns to hold the results of our transforms.

Let's add a column named _best_player_ into our data frame:

```python
from pyspark.sql.functions import lit

added_column_df = renamed_scores_df.withColumn("best_player", lit(False))
```

The main work is done by calling method `withColumn` on the original dataframe. This returns a new dataframe object with the added column.

You'll notice two arguments passed in:

- The name of the new column
- An initial value for that column in each row

In this case, we are initialising the column to the constant boolean value _False_. We do this using Spark's `lit()` function, short for _literal value_. The import in the first line pulls in the `lit` function from its library so we can use it.

Displaying the new dataframe shows the new column:

![Result of adding a new column when displayed](/images/added-column-df.png)

### Convert data type of column
-  https://spark.apache.org/docs/latest/api/python/reference/pyspark.sql/data_types.html

### Column aggregation functions
- business purpose olap oltp - see overall results
- parquet by default ??



## Writing data from a dataframe
input and output
- read and write many formats
- csv
- sql - Spark SQL is a thing see link
- json
- parquet - columnar datab adds acid, versioning

# Next
[Transforming Data](/transforming-data.md)

[Back to Contents](/contents.md)
