# Transforming Data using DataFrames
The main task of Spark code is to build our [medallion architecture](https://github.com/bjss-data-academy/data-engineering-fundamentals/blob/main/medallion-architecture.md) and transform raw data into useful business insights.

Spark provides many features and functions to do that, organised around dataframes. By design, they resemble SQL commands.

## Select 
Like SQL [SELECT..FROM](https://sqlbolt.com/lesson/select_queries_introduction), the `select()` method on dataframe allows us to choose which columns to work with.

Let's start with a dataframe:

```python
scores = [
  {"player":"Alan",  "game":"galaga", "score":14950},
  {"player":"Dan",   "game":"cricket", "score":110},
  {"player":"Rosie", "game":"snooker", "score":147},
]

scores_df = spark.createDataFrame(scores)
```

And select the `game` column:

```python
game_column_df = scores_df.select("game")
```

The returned dataframe contains the value of the `game` column:

![New dataframe contents after selecting game column](/images/game-column-df.png)

## Filter / Where
A dataframe can be filtered to contain only rows matching a filter condition.

We can use _either_ the `filter()` or the `where()` method for this - they are synonyms. 

Using filter to find only rows with a score greater than 120:

```python
over_120_df = scores_df.filter("score > 120")
```

Giving:

![Output of filter by score greater than 120](/images/score-over-120.png)

## Limit
We often want to return a small number of rows, perhaps for a summary view, or a list of best-matching results. We can do this with the `limit()` method.

Limiting our search for scores over 120 to just one:

```python
one_over_120_df = scores_df.filter("score > 120").limit(1)
```

Giving:

![Results of limit to one method showing a single result row](/images/limit-1.png)

> Note: limit() imposes no particular order on rows

## Order by
Filtered rows in a dataframe are _unordered_. We use the `orderBy()` method to impose an ordering on the result set.

We can get our scores in descending order:

```python
from pyspark.sql.functions import col

scores_highest_first_df = scores_df.orderBy(col("score").desc())
```

Returning

![All rows in descending order of score](scores-descending.png)

## Sort
p78

order by


## Distinct
p76

## Join

## Union
p77

## Complex data types
## Built-in functions
## User Defined Functions
## Performance ranking
## How Spark executes functions

## Grouping rows
collect_set
collect_list
array_distinct

## Splitting rows
pivot
explode

## Wide and Narrow transformations
p19

## Lazy evaluation
p19

# Labs

# Next
[Back to Contents](/contents.md)
