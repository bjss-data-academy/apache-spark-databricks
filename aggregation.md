# Transforming Data using DataFrames
The main task of Spark code is to build our [medallion architecture](https://github.com/bjss-data-academy/data-engineering-fundamentals/blob/main/medallion-architecture.md) and transform raw data into useful business insights.

Spark provides many features and functions to do that, organised around dataframes. By design, they resemble SQL commands.

## Select 
Like SQL `SELECT..FROM`, the `select()` method on dataframe allows us to choose which columns to work with.

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
