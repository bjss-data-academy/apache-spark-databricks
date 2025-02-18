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

## OrderBy / Sort
Filtered rows in a dataframe are _unordered_. We use either of the `orderBy()` or `sort()` methods to impose an ordering on the result set.

We can get our scores in descending order:

```python
from pyspark.sql.functions import col

scores_highest_first_df = scores_df.orderBy(col("score").desc())
```

Returning

![All rows in descending order of score](/images/scores-descending.png)

## Distinct
Where multiple equal data values exist in a column, we often need to find the set of unique values in there. 

Unique values are returned using `distinct()`:

```python
favourite_books = [
  {"title":"Tom the Racer"},
  {"title":"Java OOP Done Right"},
  {"title":"Databricks for dummies"},
  {"title":"Java OOP Done Right"},
]

favourite_books_df = spark.createDataFrame(favourite_books)

unique_titles_df = favourite_books_df.distinct()
```

Which returns all unique values of column `title`:

![Unique values of title in favourite books dataframe](/images/unique-titles.png)

### dropDuplicates()
See also [dropDuplicates()](https://spark.apache.org/docs/latest/api/python/reference/pyspark.sql/api/pyspark.sql.DataFrame.dropDuplicates.html) for a way to specify which combination of columns needs to be unique.

## Join
Generally, we want to be combining data from multiple tables in our silver layer processing. The `.join()` method allows us to combine dataframes.

We'll start with two related tables. `scores_df` once again holds the score information for a number of players. Additional reference table `contacts_df` holds email contact details for players:

```python
scores = [
  {"player":"Alan",  "game":"galaga", "score":14950},
  {"player":"Dan",   "game":"cricket", "score":110},
  {"player":"Rosie", "game":"snooker", "score":147},
]

scores_df = spark.createDataFrame(scores)

contacts = [
    {"player":"Dan",   "email":"dan@example.com"},
    {"player":"Rosie", "email":"rosie@example.com"},
    {"player":"Alan",  "email":"alan@example.com"},
]
```

We can join the two together on the player column. This relates data from each table that corresponds to the same player:

```python
contacts_df = spark.createDataFrame(contacts)

scores_contacts_df = scores_df.join(contacts_df, "player")
display(scores_contacts_df)
```

Giving us an output:

![Output of scores and contacts datarames joined together](/images/joined-df.png)

We can filter and select columns as we need. Here is a query that will return the email address of the highest scoring player:

```python
winner_email = scores_df.join(contacts_df, "player")\
    .select("email")\
    .orderBy(col("score").desc())\
    .limit(1)

display(winner_email)
```

giving

![Finding email of highest scoring player](/images/winner-email.png)

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
