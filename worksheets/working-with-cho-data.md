# Spark Practice: Working with CHO Data in Zeppelin

In this session, we want you to apply what we learned in Spark Shell Introduction (Spark SQL, PySpark, & Zeppelin) for analyzing some provided Cultural Heritage Organization (CHO) data.

You will be working in small groups on this.

Table of Contents:

1. [Getting Started](#getting-started)
1. [Part 1: Creating a Dataframe Instance from a CHO CSV](#part-1-creating-a-dataframe-instance-from-a-cho-csv)
1. [Part 2: Simple Analysis of your CHO Dataset](#part-2-simple-analysis-of-your-cho-dataset)
1. [Part 3: Creating Derivative DataFrames for your CHO ](#part-3-creating-derivative-dataframes-for-your-cho)
1. [Part 4: Using SQL to Analyze Your Data](#part-4-using-sql-to-analyze-your-data)
1. [Part 5: Simple Data Visualization](#part-5-simple-data-visualization)

## Getting Started

### Check Your Environment (Again)

This should all be the same. But just to make sure, we want to make sure you have:

1. PySpark / Spark Working;
2. You can load the sample data provided on the docker container;
3. You know the Spark Version you're working with.

The following paragraph (running bash shell commands) should produce something like the following:

```sh
> println("Spark Version: " + sc.version)
> val penndata = sc.textFile("penn.csv")
> val cmoadata = sc.textFile("cmoa.csv")
> println("penn count: " + penndata.count)
> println("cmoadata count: " + cmoadata.count)
Spark Version: 2.1.0
penndata: org.apache.spark.rdd.RDD[String] = penn.csv MapPartitionsRDD[2538] at textFile at <console>:28
cmoadata: org.apache.spark.rdd.RDD[String] = cmoa.csv MapPartitionsRDD[2540] at textFile at <console>:27
penn count: 379317
cmoadata count: 34596
```

### CHO Datasets to Choose From

See [the CHO dataset files here](../sample-data/). FYI, you'll need to unzip the Penn dataset using the `%sh` interpreter in Zeppelin (so that it unzips in your Docker container instance).

| Name | Institution Type | Format | URL |
| ---- | ---------------- | ------ | --- |
| CMOA | Museum | CSV | https://github.com/cmoa/collection |
| Penn Museum | Museum | CSV | https://www.penn.museum/collections/data.php |

## Part 1: Creating a Dataframe Instance from a CHO CSV

Using what we learned from the previous session, load one of the CHO datasets into a DataFrame. Start by having the schema inferred.

Do some basic exploration of the Dataframe to figure out if it is adequately loaded:
1. `.printSchema()`
2. `.show(10)` (you'll want to use truncate this time)
3. `.columns` to view the loaded columns
4. Check out some values for specific columns using `.select(colName)` & `.distinct()`
5. Consider making your own Schema & loading the CSV into a DataFrame with that Schema
6. Perform a `count()` to know how many rows are loaded
7. Get a sampling of the DataFrame using `.sample(withReplacement, fraction, seed=None)`

Make sure you feel confident in your DataFrame and the data loaded.

## Part 2: Simple Analysis of your CHO Dataset

Now use the [functions](functions-list.md) we've learned to try to start answering some questions about your CHO data. For example:

1. Select (`.select(colName, colName, ...)`) & Filter (`.filter(colName conditional)`) a couple subsets of columns to get a sense of the data.
2. Try ordering by a couple of fields (`.orderBy(colName)`) and see if that works out.
3. Apply a filter for finding rows where a column is or is not null (`.filter(pennDf.date_made.isNotNull())` or `.filter(pennDf.date_made.isNull())`). Sometimes it is just as important to get identifiers of records where a field is Null as it is to ignore empty fields in computations.
4. Use `groupBy(colName)` and Aggregate functions (`count()` or `agg({"colName": "mean"})`) to get numbers related to selected fields (i.e. what is the average height of resources by period or the number of resource by curatorial section?)
5. Use `distinct()`, `count()`, AND `countDistinct` along with ordering (`orderBy()`) to get rankings of values.

## Part 3: Creating Derivative DataFrames for your CHO

Think of some dataframes that could be good to feed into SQL queries or visualizations. Then consider how you need to derive them using what we learned before. For example:

1. Get DataFrame with Curatorial, ID, Culture, URL, & Period, Dropping Nulls (using `dropna()` & `select()`)
2. Get DataFrame with little description (no Creator, Title, or Description) using `filter()` & `isNull()`
3. Use `explode()` to break out new rows for each `object_name` breaking on the `|` delimiter.
4. Make new columns (with `withcolumns()` and `col.like(regex)`) to capture if AD or BC date (or other criteria).

## Part 4: Using SQL to Analyze Your Data

[Here](http://cse.unl.edu/~sscott/ShowFiles/SQL/CheatSheet/SQLCheatSheet.html) is a SQL Cheatsheet in case you need it.

First, use `df.createOrReplaceTempView("ViewName")` to create the temporary views.

Then Use SQL (via `spark.sql("""SQL query""")`) to analyze and query your data.

## Part 5: Simple Data Visualization

Use the `sql` interpreter in Zeppelin to run SQL queries against our previously defined Temporary SQL View (just enter it where the table name normally goes in a SQL Query).

This gives us immediate access to the SQL-driven visualizations available in Zeppelin.
