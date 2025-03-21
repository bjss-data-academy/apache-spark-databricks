# Transforming Data using DataFrames
The main task of Spark code is to build our [medallion architecture](https://github.com/bjss-data-academy/data-engineering-fundamentals/blob/main/medallion-architecture.md) and transform raw data into useful business insights.

We saw how raw data can be ingested in the Bronze layer, using raw files in [Volumes](/architecture.md) and reading raw files into [DataFrames](/spark-dataframe.md)

The silver layer focusses on _filtering_, _cleaning_ and _augmenting_ data. 

![medallion architecture with silver and gold layers emphasised](/images/medallion-silver-gold-emphasised.png)

Typical Silver layer tasks include:

- Removing unnecessary columns
- Removing unnecessary rows of data
- Adding aggregate quantities
- Joining multiple data sources together
- Cross-referencing to reference data sets

Spark transform methods see most use in the silver and gold layers, as we augment our data and create the top-level business aggregates that give insights:

Many Spark features resemble SQL commands, and can be used as methods on a dataframe object.

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

## Inner Join
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

## Left Join
What happens when some users are _missing_ from the contacts table? It happens.

If we used the join as above, we would _never see_ the scores of those missing contacts.

Sometimes this is perfect for our application. 

Other times, not so much. We really want a row with the player's score in it, even if we have no contact data.

This is a job for _left join_. 

We add an extra parameter named `how` to the `join` method call:

```python
scores_contacts_df = scores_df.join(contacts_df, on="player", how="left")
```

That `how="left"` causes the new dataframe `scores_scontacts_df` to contain _every_ row of `scores_df` joined to _all matching rows_ of contacts_df. We get a `null` in columns where no matching row exists.

As an example, suppose we only have contact details for Alan and Rosie:

```python
contacts_df = spark.createDataFrame([
    ("Alan", "al@example.com"), 
    ("Rosie", "rosie@example.com"),
    ], ["Player", "Email"])
```

When we run the left join code on `scores_df` and `contacts_df`, we see this output:

![Output of left join contains nulls where we have no matching information](/images/left-join.png)

You can see the `null` values in columns where no matching data exists. But the rows _are still there_.

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

We can define a struct to represent a custom data type that exactly holds _those_ fields with _those_ data types:

```python
scores_struct = StructType([
  StructField("player", StringType(), True),
  StructField("game", StringType(), True),
  StructField("score", IntegerType(), True),
])
```

We see here that our `scores_struct` requires three fields `player`, `game` and `score`. Fields `player` and `game` are both of type string. Field `score` is of integer type, so must be used to hold integer values only. All three fields are `required = True` meaning they _must_ be present.

> Struct types are often known as a _schema_

Struct types easily convert to JSON data formats. 

### Navigating struct columns
Dataframe columns can hold a struct. And that struct itself can hold other structs, giving us _nested data_.

Spark allows us to navigate around individual data items in those structs.

Let's take the example struct above and modify ot slightly. We will create a dataframe that has two columns - the `player` name, and a `results` column. This `results` column will hold a struct containing the `game` played and the `score` obtained:

```python
from pyspark.sql.types import StructType, StructField, StringType, IntegerType

scores = [
  {"player":"Alan",  "results":{"game":"galaga", "score":14950}},
  {"player":"Rosie", "results":{"game":"snooker", "score":147}},
]

game_score_pair = StructType([
  StructField("game", StringType(), True),
  StructField("score", IntegerType(), True),
])

scores_struct = StructType([
  StructField("player", StringType(), True),
  StructField("results", game_score_pair, True),
])

scores_df = spark.createDataFrame(scores, schema=scores_struct)

display(scores_df)
```

This gives us nested data - column `results` contains a struct with fields `game` and `score`:

![Nested data - results column contains struct](/images/nested-struct.png)

We can use the _dot notation_ in our queries to navigate this nested struct:

```python
from pyspark.sql.functions import col

galaga_scores_df = scores_df.filter(col("results.game") == "snooker")\
    .select("player", "results.score")

display(galaga_scores_df)
```

The above query shows us all the players who scored for snooker. Note the _dot notation_ in `results.game` and 'results.score`, allowing us to _reach inside_ the struct. 

The output is a list of matching rows, in the usual manner:

![Output of rows selected using dot notation in the query](/images/struct-query.png)

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

### Explode an array into separate rows
Explode will create a separate row for every element in the array. 

For each new row:
- Non-array column values will be duplicated
- array values will be iterated through and placed _individually_ into the rows

It's easier to see it in action. The code to run `explode()` is this:

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

### Pivot rows to columns
Think of some rows of data where one column has different values. It can be useful to combine them into a single row with multiple columns, one column for each of the values.

In our scores example, we have a column `game`. This stores the game that was played to get the scores.

We can convert rows with different games into columns using `pivot`.

Let's set up some rows of player scores for a number of games:

```python
scored_games = [
    {"player":"Alan", "game":"scramble", "score": 99950 },
    {"player":"Alan", "game":"scramble", "score": 1050 },
    {"player":"Alan", "game":"scramble", "score": 50 },
    {"player":"Alan", "game":"scramble", "score": 0 },
    {"player":"Rosie","game":"scrabble", "score": 131},
    {"player":"Rosie","game":"scrabble", "score": 99},
    {"player":"Rosie","game":"scrabble", "score": 131},
    {"player":"Rosie","game":"scramble", "score": 78},
    {"player":"Rosie","game":"duck hunt", "score": 12},
]

scored_games_df = spark.createDataFrame(scored_games)
```

Now we can use `.pivot()` to combine total scores for games into a column for that game:

```python
game_columns_df = scored_games_df\
    .groupBy("player", "game")\
    .pivot("game")\
    .sum("score")

score_summary_df = game_columns_df.select("player", "duck hunt", "scrabble", "scramble")
```

Giving us this resulting dataframe:

![Output of pivot on game column, to show total scores per game in one row per game](/images/pivot.png)

> There is one new column for every distinct value in the `game` column

We can see a sequence of methods to call:

- `groupBy`to create a grouped data object we can apply `pivot` to
- `pivot` specifying the column to convert row values to new columns
- an _aggregate method_ used to populate the value for the new column

Using the above groupBy and pivot, we get one row per-game per-player.

### Map
There's some truth in the saying that 

> if the only tool you have is Python, every problem looks like a dictionary

(even though I just made it up).

Python has very convenient support for the dictionary data type, allowing us to create key-value pairs easily. 

key-value data is oftenm used for _lookup tables_, and these appear frequently as reference data.

Spark can work with columns that are dictionaries. Again, the eagle-eyed will spot this is the exact opposite of third normal form, but alas, real data often do be that way. Sometimes for good reason; sometimes not.

Here is a dataframe using a dictionary to store facts about each of our players:

```python
player_reference = [
    {"player": "Alan", "facts":{"hobby":"watching paint dry", "vegan": False}},
    {"player": "Dan", "facts":{"hobby":"painting dry watches", "vegan": True}},
    {"player": "Rosie", "facts":{"hobby":"drying watch paint", "vegan": False}},
]

player_reference_df = spark.createDataFrame(player_reference)

display(player_reference_df)
```

![Dataframe with a map type column of facts](/images/map-column.png)

### Explode map values into separate rows
We can use `explode` to separate out each key-value pair into a row of its own:

```python
from pyspark.sql.functions import col, explode; 

separate_facts_df = player_reference_df.select("player", explode(col("facts")))

display(separate_facts_df)
```

Resulting in the following new rows:

![New rows created one per map-value](/images/map-explode.png)

## Combining rows
We can combine multiple rows into single rows using `collect_set` and `collect_list` methods. This is often useful after a `groupBy`, or after using some window partitioning.

### collect_list
Specify a column. All values in that column will be combined into an array. This is complementary to `explode` earlier.

Using our player data from before:

```python
from pyspark.sql.functions import collect_list

scored_games = [
    {"player":"Alan", "game":"scramble", "score": 99950 },
    {"player":"Alan", "game":"scramble", "score": 1050 },
    {"player":"Alan", "game":"scramble", "score": 50 },
    {"player":"Alan", "game":"scramble", "score": 0 },
    {"player":"Rosie","game":"scrabble", "score": 131},
    {"player":"Rosie","game":"scrabble", "score": 99},
    {"player":"Rosie","game":"scrabble", "score": 131},
    {"player":"Rosie","game":"scramble", "score": 78},
    {"player":"Rosie","game":"duck hunt", "score": 12},
]

scored_games_df = spark.createDataFrame(scored_games)

combined_scores_df = scored_games_df.groupBy("player", "game")\
    .agg(collect_list("score").alias("all_scores"))

display(combined_scores_df)
```

We see fewer rows. Scores for individual games have been combined into an array. The array is stored in a column named `all_scores`, using the `alias` method:

![Results of collect_list gathering player scores per game into an array](/examples/collect-list.png)

### collect_set
We don't awlays want to combine _all_ the data values into an array. Sometimes, we want only _distinct_ values.

Method `collect_set` does this for us.

In our player scores example, `collect_set` could be used to create an array column with all the games player by a player.

Using the same `scored_games_df` as before:

```python
from pyspark.sql.functions import collect_set

player_games_df = scored_games_df.groupBy("player")\
    .agg(collect_set("game").alias("games_played"))

display(player_games_df)
```

We create a new column `games_played` holding an array of games:

![Results of collect_set to combine games played into one array column](/images/collect-set.png)

### array_distinct
Closely related to `collect_set` in terms of function is `array_distinct`. 

Again, this is a way to _de-dupe_ - remove duplicate values from - an array of data.

We can get a list of all the different games played in our dataset using `array_distinct`:

```python
from pyspark.sql.functions import array_distinct, collect_list

games_played_df = scored_games_df.agg(array_distinct(collect_list("game")).alias("games_played"))

display(games_played_df)
```

This returns a dataframe with a single column containing all the games found in the data:

![Column with array of all distinct games](/images/array-distinct.png)

### flatten
`flatten()` combines the values from multiple separate arrays into a single array.


## Wide and Narrow transformations
Transformations can be either _wide_ or _narrow_.

The difference lies in how the data is shuffled across the network. 

### Narrow Transformations
In a narrow transform, each input partition contributes to only one output partition. 

No data is shuffled across the network in this case. It is all kept local to the executor node. 

> __Narrow__: fastest performance

### Wide Transformations
Data from one partition may contribute to several output partitions. These may be on different nodes, requiring a shuffle operation. Data will be transferred over the network. This is a slow operation.

### Wide vs Narrow Summary
| Aspect             | Narrow Transformations                           | Wide Transformations                        |
|--------------------|--------------------------------------------------|--------------------------------------------|
| **Data Movement**   | No data shuffle between partitions             | Requires data shuffle between partitions  |
| **Performance**     | More efficient                                 | Less efficient due to shuffling           |
| **Examples**        | `map()`, `filter()`, `flatMap()`               | `groupByKey()`, `reduceByKey()`, `join()`  |

> Minimize wide transformations to improve performance

## Lazy evaluation
Spark uses _Lazy Evaluation_ for its transform processing. 

This means that transforms are:
- not executed in the order the code specifies them
- not executed until triggered by an action

This is very different to normal imperative code in Python, and crucial to understand.

Spark code first builds an _Execution Plan_ of which order transforms will be executed in. This can then be optimised by the _Adaptive Query Execution_ engine.

No work is done until a trigger is executed. These include methods like `collect()`, `count()`, and `save()`, amongst others.

The optimised execution plan runs, the transformations are applied and the results calculated _after_ the trigger executes.

To see this in action with some code:

```python
df = spark.createDataFrame(data, columns)

# Applying transformations (still no computation)
df_filtered = df.filter(col("Age") > 30)  # Filter transformation (lazy)
df_selected = df_filtered.select("Name", "Age")  # Select transformation (lazy)
```

Nothing has run yet: No data has been processed. To cause that, we need to execute a trigger:

```python
# Action triggers the computation
df_selected.show()  # This triggers the actual execution
```

Now the optimised execution plan will run. 

# Labs
TODO

# Next
We can work with tables and trabsform data using Spark SQL:

[Working with Spark SQL](/spark-sql.md)

[Back to Contents](/contents.md)
