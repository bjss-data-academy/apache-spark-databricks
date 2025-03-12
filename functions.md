# Functions
Functions are another way to transform data. 

So far, we've mostly worked with methods on a dataframe object. Thos emthods implicitly operate on the data inside that specific dataframe.

Free-standing functions operate only on the parameters you send them. For example:

```python
y = cos(0)
```

passes the value of 0 to the `cos` function. That returns the cosine of whatever you pass in. in this case, the cosine of zero. The result is one.

Pure functions like this are independent of any object. They have no memory of the past. They operate only on the data they are given, return a result, and promptly forget all about it.

## Built-in functions
Spark has a large library of useful functions. 

You can read about them all [here](https://spark.apache.org/docs/latest/api/sql/index.html)

There are categories for
- maths functions: examples cos, sin, abs, sqrt
- string manipulation: examples trim, substr, levenstein, upper
- array manipulation: examples array_contains, array_append, array_zip
- date manipulation: examples unix_timestamp, date_from_unix_date, date_format, day_of_month
- regular expressions
- statistics: examples variance, avg, aggregate

And many more, too many to list. 

Spend some time familiarising with the functions made available - they will simplify your work across many common Data Engineering tasks.

## User Defined Functions
What if we need a function to do a job for us, but it isn't already provided?

We can write our own function as a _User Defined Function+ or _UDF_ for short.

We can write a normal Python function and turn it into a UDF by registering it. This allows the function to be used inside SQL statements - a very powerful thing - and dataframes.

### Defining a UDF
Let's write a Python function to prepend the word "Hello " to a piece of text:

```Python
def hello_to( text ):
   return "Hello " + text
```

Simple enough. Pass in _Alan_ and return _Hello Alan_.

We can register this as a UDF:

```python
spark.udf.register("hello_to", hello_to)
```

Now we can use this function as if it were built-in.

### Using a UDF with SQL
We can use the UDF in a SQL query. 

Assuming a table `users` with the string column `name`, we can write:

```sql
%sql
select name, hello_to(name) as greeting from users
```

Which will return a list of rows consisting of name and the text "Hello " <name>:

```text
name    | greeting
--------+-------------
Alan    | Hello Alan
Dan     | Hello Dan
Rosie   | Hello Rosie
```

### Using a UDF with a DataFrame in Python
Assuming the same `users` table above, we can convert to a dataframe and run the UDF:

```python
users_df = spark.table("users")
display( users_df.select("name", hello_to("name").alias("greeting")))
```

We get a similar result as for SQL, only based around a dataframe.

## Performance ranking
Unsurprisingly, different kinds of functions have different execution speeds.

The built-in functions work the best, having the tightest integration with Spark. Other functions are less integrated, and perform differently:

![Function type performance ranking](/images/udf-performance-ranking.png)

Functions _usually_ perform better from __SQL__ than from Python, Spark or Java. This happens because the Catalyst optimiser is working for the SQL versions, and some data transfers can be avoided.

## How Spark executes functions

# Next
Working with real-time streaming data:

[Working with Streaming Data](/streaming.md)

[Back to Contents](/contents.md)
