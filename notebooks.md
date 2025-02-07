# Using notebooks in Databricks
We can use an interactive programming style inside Databricks with _notebooks_.

Databricks Notebooks will be familar to users of Jupyter notebooks.

This guide builds on [Getting Started with Databricks](https://github.com/bjss-data-academy/getting-started-databricks/blob/main/README.md)

If you are not familiar with how notebooks work, review the guide first.

## Magic Commands
Notebooks have _magic commands_ that allow you to override the default language, and to provide some utilities.

A magic command starts with a percent sign __%__

## Selecting a programming language
To select a programming language, enter one of the following as the first line:

- %python
- %r
- %scala
- %sql
- %java

## Writing formatted text using markdown
Magic command __md__ causes the text to be rendered following markdown rules:

```text
%md
# Heading 1
- Bullet 1
- Bullet 2
## Heading 2
[web link](http://example.com)
![Image link (c) Alamy stock](https://c7.alamy.com/comp/2GAN3YE/pomeranian-eats-a-banana-dog-eating-fruit-on-white-background-pomeranian-elite-isolate-food-2GAN3YE.jpg)
```

- Clicking outside notebook for viewed version
- Click inside text to pull up editor

## Working with the file system
Magic command __%fs__ allows use of file system commands. Handy to poke around and see what we have.

```text
%fs
ls
```
will run the `ls` (list all visible files) command.

More information: [databricks documentation](https://docs.databricks.com/en/dev-tools/databricks-utils.html#dbutils-fs)

## Executing shell commands
__%sh__ will cause a shell command to run on the Spark Driver.

_note: only applies to the driver node, not any worker_

## Installing Python libraries
__%pip__ uses the familiar `pip` installer to add Python libraries to your code.

## Working with Spark SQL
__%sql__ allows use of Spark SQL. 

Very good to explore a SQL dataset and see what's there.

```sql
select * from users u where u.name = 'Alan' 
```

# Next
[Back to Contents](/contents.md)
