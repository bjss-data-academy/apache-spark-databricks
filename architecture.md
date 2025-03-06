# Data Management in Databricks
Whenever we have big data, we have a big problem managing it.

Think about all the permissions that need setting. And data retention policies. Audit controls. Across every different data source. And every different cloud data store.

It's a lot of work.

To help pull this together, Databricks provides the _Unity Catalog_. 

This is a unified way of organising data across clouds and clusters, and adding permissions.

We get the typical benefits of a central approach:

- Global settings
- Consistent approach
- Disparate sources treated the same way

## Unity Catalog Structure
Databricks organises data in a three-level structure underneath a Unity Catalog as follows:

![Data organisation in Databricks](/images/databricks-data-organisation.png)

### Unity Catalog
At the top, the `Unity Catalog` provides a unified approach to _data governance_. 

Governance a fancy word for things like permissions, retention policy, encryption and so on.

We can have as many Unity Catalogs as we need. For most enterprises, having a single one works best, to give a place for Global settings affecting every data asset the enterprise owns.

### Schemas
Within a Catalog we have many `schema`. These group together the various kinds of data resources we need.

### Tables
Tables hold relational data, familiar to us from regular database products. 

### Views
Views are virtual tables based on a SQL query, again familiar from the relational database world.

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

# Data Governance
_Data Governance_ is about how we protect our confidential data and comply with data regulations.

Corporate users of data often have regulations like GDPR and HIPAA to comply with. They generally have internal policies on who can see what, when.

Governance includes controlling data access, enforcing data retention policies, audit trails, deletion, exncryption and more.

All this is tricky with small systems. But it gets more of a problem at scale. Everything is simply 'more'. More data. More cloud stoarge accounts. More users. More restrictions. More to enforce.

Databricks Unity Catalog is aimed at the problem of managing all this. 

## Three level governance
We can grant permissions to the _securable items_ in our data structure. 

As the structure is three levels deep, starting from the Unity Catalog at the top, we need permissions to use each level.

If we want to create a table, we need permissions on the catalog, schema and table to achieve this:

![Showing three levels of permissions required to create a table](/images/three-level-permissions.png)

Normal SQL `GRANT` syntax is used to set permissions, as follows.

### Grant use of catalog
The starting point: allow use of the Unity Catalog to a group `allowed-user-group`:

```sql
  GRANT USE CATALOG ON CATALOG main TO `allowed-user-group`;
```

### Grant use of schema
Allow use of a named schema inside that Unity Catalog to that group:

```sql
  GRANT USE SCHEMA ON SCHEMA main.default TO `allowed-user-group`;
```

### Grant create table
Allow fine-grained actions inside that schema to that group. Here, we allow `CREATE TABLE` to be used:
```sql
  GRANT CREATE TABLE ON SCHEMA main.default TO `allowed-user-group`;
```

### Problem: Missing grant
Here's a common gotcha - Watch out for missing any of the required three levels of permissions:

```sql
  GRANT USE CATALOG ON CATALOG main TO `allowed-user-group`;
  GRANT CREATE TABLE ON SCHEMA main.default TO `allowed-user-group`;
```

This will _not_ allow tables to be created on that schema.

Q: Why?
A: There is no permission to use the schema at all. This overrides the ability to create a table.

### Other permissions
We can easily `REVOKE` a `GRANT`, as well as secure other pieces of our Databricks data.

For more details, see [Manage Privileges in Unity Catalog](https://docs.databricks.com/aws/en/data-governance/unity-catalog/manage-privileges/?language=SQL)

# Further Reading
- [Delta tables - Databricks](https://docs.databricks.com/aws/en/delta/tutorial)
- [Transaction log - Databricks](https://www.databricks.com/blog/2019/08/21/diving-into-delta-lake-unpacking-the-transaction-log.html)
- [Unity Catalog Privileges](https://docs.databricks.com/aws/en/data-governance/unity-catalog/manage-privileges/privileges)
  
# Next
[Back to Contents](/contents.md)
