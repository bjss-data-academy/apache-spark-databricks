# Data Management in Databricks
Databricks organises data in a specific way:

![Data orgnaisation in Databricks](/images/databricks-data-organisation.png)

### Unity Catalog
At the top, the `Unity Catalog` provides a unified approach to _data governance_. That's a fancy word for things like permissions, retention policy, encryption and so on.

We can have as many Unity Catalogs as we need. For most enterprises, having a single one works best.

### Schemas
Within a Catalog we have many `schema`. These group together the various kinds of data resources we need.

### Tables
Tables hold relational data, familiar to us from regular database products. 

### Views
Views are based on tables, again familiar from the relational database world.

In Databricks, views can provide _fine-grained security_. It is possible to create views restrictied to specific columns of a table. Views can _mask_ column values, making them unreadable - good for sensitive information.

### Volumes
Volumes allow file-based data to be stored and managed. Raw data such as CSV or JSON files can be placed inside a volume.

### Functions
User-Defined Functions (UDFs) can be created to work on data in customised ways. The code is managed by Databricks, and access controls can be applied.

### AI Models
Data can be transformed using AI models. These too can be managed as part of the Unity Catalog, just like any other data asset. 

## Managed and external Tables
Databricks offers two levels of support for tables - _managed_ tables or _external_ tables.

Tables in Databricks are made up of _data_ plus table _metadata_:

- __Data__: values inside rows and columns
- __Metadata__: information about how and where that data is stored

Databricks manages the metadata for both types. It has to; Databricks needs to know about the data it is dealing with.

The difference between the two table types lies in how and where the data is managed.

### Managed Tables

- Databricks controls metadata
- Databricks controls data

![Showing data and metadata insde Databricks management](/images/managed-table.png)  

In a managed table, Databricks fully controls the data that is stored inside the table along with metadata. It decides where the data is stored. It decides how best to work with that data.

> DROP TABLE deletes the metadata and the data. Data is permanently gone.

### External Tables

- Databricks stores and controls metadata
- External systems store and control data
- Databricks has no control over the external data, only access to it

![Showing Databricks managing only metadata with an external provider managing data](/images/unmanaged-external-table.png)

With an external table, Databricks still holds metadata information about where that data is held and how to acces it. But that's all.

The data itself is fully managed outside of databricks. 

Examples of such external storage include cloud storage and on-premise database products.

> DROP TABLE deletes the metadata only. External data is _unaffected_

If we drop an external table in databricks, the external data is unaffected. Only Databricks metadata is removed.

> We can restore external data - the metadata can be recreated

### Delta Tables
Delta tables are a Databricks data storage format that builds on the open-source [Parquet](https://github.com/apache/parquet-format) file format to add:

- ACID transactions
- Time-travel (versioned history of data at points in time)
- High performance append and delete of rows
- High performance data caching
- Schema enforcement

The secret sauce is to add a _transaction log_ to the raw parquet file storage:

![Delta table showing parquet file data rows with column metadata plus three entries in a transaction log](/images/delta-table-internals.png)

A Delta Table stores data in the column-friendly Parquet format. It then adds a transaction log to overcome limitations of the Parquet format. Databricks  computes the result of the Parquet contents plus all transaction log variations to determine what the current dataset is.

_Appending data_ becomes a simple addition to the transaction log. This is much faster than reading the whole parquet file, modifying it in-memory then writing it out again.

_ACID transactions_ are also a simple addition to the transaction log. 

_Time-travel_ involves working through the transaction log in time sequence, until we hit the desired date. We now have a snapshot of what the data was at that time.

The transaction log is stored in a folder _delta_log_ as a series of numbered `json` files.

> Delta tables are the default format in Databricks

## Working with Spark SQL
SQL only works on tables and views. To use SQL with a dataframe, we must first convert it to a table or a view.

TODO
TODO - repetition to remove, rearrange to use scored_df as running example - clarify text and floe
TODO

### Creating an empty table
This follows standard ANSI SQL DDL syntax, with the ability to work with complex data types.

A simple example of creating a basic `User` table is:

```sql
CREATE TABLE User (
    id BIGINT,
    name STRING,
    email_address STRING,
    bio STRING
)
```

> This creates the table `User` as a _managed_ table, using Delta format.

Rows can be inserted in the normal way:

```sql
INSERT INTO User VALUES (1, "Alan", "al@example.com", "Author Java OOP Done Right, Test-Driven Deveopment in Java. Co-author Nokia Bounce, Fun School 2, Red Arrows")
```

### Create an external table
If our user data was stored outside Databricks, we could access it as an _external table_.

The syntax to create external tables uses the `LOCATION` keyword, to specify where the data is stored:

```sql
CREATE TABLE external_user
LOCATION '/mnt/external_storage/external_user/';
```
## Dataframes 
In-memory processing

## Partitions

## Liquid Clustering

## Shuffle

## Skew

## Skip index

