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
Generally, we need to combine data from multiple tables in our silver layer processing. The `.join()` method allows us to combine dataframes.

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

We can join the two together on the `player` column. This relates data from each table that corresponds to the same player:

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
Dataframes are _immutable_. Once we have a dataframe, that's it: we can't add or remove rows.

This is a bit of a blow sometimes, as we often want to add rows.

We can do this with `.union()`:

```python
more_scores = [
    {"player":"Sue",  "game":"galaga", "score": 22950},
    {"player":"Dave", "game":"scramble", "score": 9950}
]

more_scores_df = spark.createDataFrame(more_scores)

updated_scores_df = scores_df.union(more_scores_df)
display(updated_scores_df)
```

> Union requires the same column schema in both dataframes to work

Provided we have the same schema - same columns, same types - in both dataframes, `union()` will create a new dataframe combining the rows:

![Output of union of scores dataframe with another dataframe of additional scores](/images/union.png)

## Complex data types
So far, we have considered our data to be primitive types: things like strings, boolean or numbers. 

A lot of real-world data is interconnected. We don't have a single piece of information to describe a customer. We have several, related pieces. We wantto work with this data as a whole chunk, to match our concept of what that data represents.

There are several ways to do this, depending on the situation.

### Struct
Think about describing a _customer_ in a typical system.

There is plenty of information here. We might have a combination of name fields, date of birth, credit limit, preferred pronouns and often more.

We want to treat this data as a single chunk, so that it maps to our single concept of customer. 

In spark, we can define a `StructType`. This groups seperate named pieces of data of different data types.

Recall our scores data type = a list of python dictionaries:

```python
scores = [
  {"player":"Alan",  "game":"galaga", "score":14950},
  {"player":"Dan",   "game":"cricket", "score":110},
  {"player":"Rosie", "game":"snooker", "score":147},
]
```

We can define the structure of our scores data type using Spark's `StructType` and `StructField` objects:

```python
scores_struct = StructType([ \
  StructField("player", StringType, True),`
  StructField("game", StringType(), True),`
  StructField("score", IntegerType(), True),`
])
```

We see here that our `scores_struct` requires three fields `player`, `game` and `score`. Fields `player` and `game` are both of type string. Field `score` is of integer type, so must be used to hold integer values only. All three fields are `required = True` meaning they _must_ be present.

> Struct types are often known as a _schema_

Struct types easily convert to JSON data formats. 

### Array
In other cases, we don't have an internal structure of data. We just have _many_ of the _same thing_. 

This is recognisable as an _array_ of values, which idiomatic Python calls a _list_.

Apache Spark supports lists of values using `ArrayType`.

Here is some data relating game scores to players. But it's in a different format. This time, we have an entry for a single player playing a single game, but with a list of their scores achieved:

![Data featuring a list of integer games scores in a single row per player](/images/scored-games-list.png)

We can define a struct type as before, but define the `scores` field to be an `ArrayType`:

```python
scores_struct = StructType([ \
  StructField("player", StringType, True),`
  StructField("game", StringType(), True),`
  StructField("scores", ArrayType(), True),`
])
```

The eagle-eyed will spot that this structure not so much violates third normal form as runs a steam-roller over it, filling in the remains with asphalt.

Fortunately, if we need to work with this data in more normal columnar form, Apache Spark provides the `explode()` method.

#### Explode an array into separate rows
Explode will create a separate row for every element in the array. The non-array column values will be duplicated. The array values will be iterated through and placed into the rows.

It's easier to see it in action.  The code to run `explode()` is this:

```python
from pyspark.sql.functions import explode, col

scored_games_df = spark.createDataFrame(scored_games)

scores_per_play_df = scored_games_df.select(
    "player",
    "game",
    explode(col("scores")).alias("individual_game_score")
)

display(scores_per_play_df)
```

See how there are new rows in the table, each one with a new column holding score for an individual game. We have named this new column `individual_game_score`:


![Results of explode on data with array values](/images/explode.png)



### Map
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
