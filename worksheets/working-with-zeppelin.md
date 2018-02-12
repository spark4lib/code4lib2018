# Spark Practice: Working with `spark-shell` in Zeppelin Info

In this session, you will learn Zeppelin, Spark, & Spark SQL basics via primarily the DataFrames API. We are starting with a really simple dataset to focus on the tools. In the next session, we will use an expanded cultural heritage metadata set.

Table of Contents:

1. [Getting Started](#getting-started)
1. [Part 1: Creating a Dataframe Instance from a CSV](#part-1-creating-a-dataframe-instance-from-a-csv)
1. [Part 2: Simple Analysis of our Dataset via Dataframe API](#part-2-simple-analysis-of-our-dataset-via-dataframe-api)
1. [Part 3: Creating New Dataframes to Expand or Rework our Original Dataset](#part-3-creating-new-dataframes-to-expand-or-rework-our-original-dataset)
1. [Part 4: Using SQL to Analyze our Dataset](#part-4-using-sql-to-analyze-our-dataset)
1. [Part 5: Simple Data Visualization](#part-5-simple-data-visualization)

## Getting Started

### Datasets? DataFrames?

A **Dataset** is a distributed collection of data. Dataset provides the benefits of strong typing, ability to use powerful lambda functions with the benefits of (Spark SQL’s) optimized execution engine. A Dataset can be constructed from JVM objects and then manipulated using functional transformations (map, flatMap, filter, etc.). The Dataset API is available in Scala and Java.

A **DataFrame** is a Dataset organized into named columns. It is conceptually equivalent to a table in a relational database or a data frame in R/Python, but with richer optimizations under the hood. DataFrames can be constructed from a wide array of sources such as: structured data files, tables in Hive, external databases, or existing RDDs. The DataFrame API is available in Scala, Java, Python, and R. In Scala and Java, a DataFrame is represented by a Dataset of Rows. In the Scala API, DataFrame is simply a type alias of Dataset[Row]. (Note that in Scala type parameters (generics) are enclosed in square brackets.)

[[source](http://spark.apache.org/docs/2.0.0/sql-programming-guide.html#datasets-and-dataframes)]

### How to Run a Paragraph in a Zeppelin Note(book)

To run a paragraph in a Zeppelin notebook, you can either click the `play` button (blue triangle) on the right-hand side or click on the paragraph simply press `Shift + Enter`.

### What are Zeppelin Interpreters?

In the following paragraphs we are going to execute Spark code, run shell commands to download and move files, run sql queries etc. Each paragraph will start with `%` followed by an interpreter name, e.g. `%spark2` for a Spark 2.x interpreter. Different interpreter names indicate what will be executed: code, markdown, html etc.  This allows you to perform data ingestion, munging, wrangling, visualization, analysis, processing and more, all in one place!

Throughout this notebook we will use the following interpreters:

- The default interpreter is set to be PySpark for our Workshop Docker Container.
- `%spark` - Spark interpreter to run Spark code written in Scala
- `%spark.sql` - Spark SQL interprter (to execute SQL queries against temporary tables in Spark)
- `%sh` - Shell interpreter to run shell commands
- `%angular` - Angular interpreter to run Angular and HTML code
- `%md` - Markdown for displaying formatted text, links, and images

To learn more about Zeppelin interpreters check out this [link](https://zeppelin.apache.org/docs/0.5.6-incubating/manual/interpreters.html).

### Check Your Environment

We want to make sure you have:

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

### Our Dataset for this Simple Walk through:

See [the CSV file here](../sample-data/small-sample.csv).

| Name | Institution Type | Format | URL | Description |
| ---- | ---------------- | ------ | --- | ----------- |
| CMOA | Museum | JSON, CSV | https://github.com/cmoa/collection |
| Penn Museum | Museum | JSON, CSV, XML | https://www.penn.museum/collections/data.php | JSON is poorly structured |
| Met Museum | Museum | CSV |  | ¯\_(ツ)_/¯\n |
| DigitalNZ Te Puna Web Directory | Library | XML | https://natlib.govt.nz/files/data/tepunawebdirectory.xml,MARC | XML |
| Canadian Subject Headings | Library | RDF/XML | http://www.collectionscanada.gc.ca/obj/900/f11/040004/csh.rdf | "Ugh, rdf" |
| DPLA | Aggregator | CSV,JSON,XML | dp.la |

## Part 1: Creating a Dataframe Instance from a CSV

Spark

```
> spark.read.csv("small-sample.csv")
DataFrame[_c0: string, _c1: string, _c2: string, _c3: string, _c4: string]
```

This uses `pyspark.sql.DataFrameReader` to load from external storage systems into a DataFrame. For the `csv` module particularly, it loads a CSV file and returns the result as a DataFrame. The above command returns the DataFrame instance.

Using `.show()` method on that DataFrame instance, we can see a pretty print version of the data:

```
> spark.read.csv("small-sample.csv").show()
+--------------------+----------------+--------------+--------------------+--------------------+
|                 _c0|             _c1|           _c2|                 _c3|                 _c4|
+--------------------+----------------+--------------+--------------------+--------------------+
|                Name|Institution Type|        Format|                 URL|         Description|
|                CMOA|          Museum|     JSON, CSV|https://github.co...|                null|
|         Penn Museum|          Museum|JSON, CSV, XML|https://www.penn....|JSON is poorly st...|
|          Met Museum|          Museum|           CSV|                null|            ¯_(ツ)_/¯|
|                   "|            null|          null|                null|                null|
|DigitalNZ Te Puna...|         Library|           XML|https://natlib.go...|            MARC XML|
|Canadian Subject ...|         Library|       RDF/XML|http://www.collec...|            Ugh, rdf|
|                DPLA|     Aggregator |  CSV,JSON,XML|               dp.la|                null|
+--------------------+----------------+--------------+--------------------+--------------------+
```

We want to fix a few things about this data being read in, using the `pyspark.sql.csv` module's available options:

### Adding a Header

We want to take the first row as a header.

```
> spark.read.csv("code4lib2018/sample-data/small-sample.csv", header=True).show()
+--------------------+----------------+--------------+--------------------+--------------------+--------------+
|                Name|Institution Type|        Format|                 URL|         Description|Informal Score|
+--------------------+----------------+--------------+--------------------+--------------------+--------------+
|                CMOA|          Museum|     JSON, CSV|https://github.co...|                null|            10|
|         Penn Museum|          Museum|JSON, CSV, XML|https://www.penn....|JSON is poorly st...|             7|
|          Met Museum|          Museum|           CSV|                null|            ¯_(ツ)_/¯|          null|
|                  ,3|            null|          null|                null|                null|          null|
|DigitalNZ Te Puna...|         Library|           XML|https://natlib.go...|            MARC XML|             3|
|Canadian Subject ...|         Library|       RDF/XML|http://www.collec...|            Ugh, rdf|             4|
|                DPLA|     Aggregator |  CSV,JSON,XML|               dp.la|                null|           100|
+--------------------+----------------+--------------+--------------------+--------------------+--------------+
```

### Adding (& removing) DROPMALFORMED mode

We see there is a newline causing us some issues. We don't want to leave those out now, but it is an opportunity to learn how `DROPMALFORMED` mode can help us.

```
> spark.read.csv("code4lib2018/sample-data/small-sample.csv", header=True, mode="DROPMALFORMED").show()
+--------------------+----------------+--------------+--------------------+--------------------+--------------+
|                Name|Institution Type|        Format|                 URL|         Description|Informal Score|
+--------------------+----------------+--------------+--------------------+--------------------+--------------+
|                CMOA|          Museum|     JSON, CSV|https://github.co...|                null|            10|
|         Penn Museum|          Museum|JSON, CSV, XML|https://www.penn....|JSON is poorly st...|             7|
|DigitalNZ Te Puna...|         Library|           XML|https://natlib.go...|            MARC XML|             3|
|Canadian Subject ...|         Library|       RDF/XML|http://www.collec...|            Ugh, rdf|             4|
|                DPLA|     Aggregator |  CSV,JSON,XML|               dp.la|                null|           100|
+--------------------+----------------+--------------+--------------------+--------------------+--------------+
```

### Working Around the New Lines / Multiline CSV

With the Spark Shell (CLI) walk through, we could use the `multiLine` qualifier. However, that causes an error in our Zeppelin notebook, since we're running version 2.1.x and that qualifier was added in Spark 2.2.x.

For the sake of the rest of this example, we run a simple Python script within our PySpark notebook context to remove the newlines:

```py
import csv

# we will see a cleaner way using pyspark to handle this issue later
# though really, use proper quoting with Spark 2.2.x & multifile=True

with open('code4lib2018/sample-data/small-sample.csv') as fh:
    test = csv.reader(fh)
    with open('code4lib2018/sample-data/small-sample-stripped.csv', 'w') as fout:
        test_write = csv.writer(fout, quoting=csv.QUOTE_ALL)
        for row in test:
            new_row = [val.replace("\r\n", "") for val in row]
            test_write.writerow(new_row)
```

This saves a newlines-free CSV for the sake of this notebook in `code4lib2018/sample-data/small-sample-stripped.csv`.

### Handling Schemas

Right now, we're inferring the schema via the `sparl.read.csv` module. `inferSchema` is set to `True` by default. We can see what it is guessing by using `printSchema()` on the DataFrame instance:

```
> spark.read.csv("code4lib2018/sample-data/small-sample-stripped.csv", header=True, inferSchema=True).printSchema()
root
 |-- Name: string (nullable = true)
 |-- Institution Type: string (nullable = true)
 |-- Format: string (nullable = true)
 |-- URL: string (nullable = true)
 |-- Description: string (nullable = true)
 |-- Informal Score: integer (nullable = true)
```

We want to update a few things though - make the score a `DecimalType` instead of an `IntegerType`; and make the `Name` required (i.e. `nullable = false`). So we first define our own Schema, importing the dataTypes we need:

```py
>from pyspark.sql.types import *
> customSchema = StructType([
    StructField("Name", StringType(), True),
    StructField("Institution Type", StringType(), True),
    StructField("Format", StringType(), True),
    StructField("URL", StringType(), True),
    StructField("Description", StringType(), True),
    StructField("Informal Score", DecimalType(), True)])
> sampleDf = spark.read.csv("code4lib2018/sample-data/small-sample-stripped.csv", header=True, schema=customSchema)
> sampleDf.show()
+--------------------+----------------+--------------+--------------------+--------------------+--------------+
|                Name|Institution Type|        Format|                 URL|         Description|Informal Score|
+--------------------+----------------+--------------+--------------------+--------------------+--------------+
|                CMOA|          Museum|     JSON, CSV|https://github.co...|                null|            10|
|         Penn Museum|          Museum|JSON, CSV, XML|https://www.penn....|JSON is poorly st...|             7|
|          Met Museum|          Museum|           CSV|                null|            ¯_(ツ)_/¯|             3|
|DigitalNZ Te Puna...|         Library|           XML|https://natlib.go...|            MARC XML|             3|
|Canadian Subject ...|         Library|       RDF/XML|http://www.collec...|            Ugh, rdf|             4|
|                DPLA|     Aggregator |  CSV,JSON,XML|               dp.la|                null|           100|
+--------------------+----------------+--------------+--------------------+--------------------+--------------+
```

At this point, we've saved our DataFrame instance of the CSV as `sampleDf`.


## Part 2: Simple Analysis of our Dataset via Dataframe API

Here are a few ways to explore our DataFrame instance:

### View Subsets of DataFrame Records

```
> sampleDf.head(2)
[Row(Name=u'CMOA', Institution Type=u'Museum', Format=u'JSON, CSV', URL=u'https://github.com/cmoa/collection', Description=None, Informal Score=Decimal('10')), Row(Name=u'Penn Museum', Institution Type=u'Museum', Format=u'JSON, CSV, XML', URL=u'https://www.penn.museum/collections/data.php', Description=u'JSON is poorly structured', Informal Score=Decimal('7'))]
```

### View Number & List of Columns in a DataFrame

```
> sampleDf.columns
['Name', 'Institution Type', 'Format', 'URL', 'Description', 'Informal Score']
> len(sampleDf.columns)
6
```

### Show Specific Columns

`.select(colName)` lets you select one or more columns in a new DataFrame, which you can then `show()` (or process otherwise):

```
> sampleDf.select("Name").show()
+--------------------+
|                Name|
+--------------------+
|                CMOA|
|         Penn Museum|
|          Met Museum|
|DigitalNZ Te Puna...|
|Canadian Subject ...|
|                DPLA|
+--------------------+
> sampleDf.select("Name", "Informal Score").show()
+--------------------+--------------+
|                Name|Informal Score|
+--------------------+--------------+
|                CMOA|            10|
|         Penn Museum|             7|
|          Met Museum|             3|
|DigitalNZ Te Puna...|             3|
|Canadian Subject ...|             4|
|                DPLA|           100|
+--------------------+--------------+
```

### Column Filter versus Select

`select` lets you get a subset of columns in a new DataFrame. You can add a statement to a `select` function, and it will return the result of that.

`filter`, however, lets you get a subset of columns in a new Dataframe, but takes a conditional statement and only returns rows from those columns that return `True`.

```
> scoresBooleanDf = sampleDf.select('Name', 'URL', sampleDf['Informal Score'] > 5)
> scoresBooleanDf.show()
+--------------------+--------------------+--------------------+
|                Name|                 URL|(Informal Score > 5)|
+--------------------+--------------------+--------------------+
|                CMOA|https://github.co...|                true|
|         Penn Museum|https://www.penn....|                true|
|          Met Museum|                null|               false|
|DigitalNZ Te Puna...|https://natlib.go...|               false|
|Canadian Subject ...|http://www.collec...|               false|
|                DPLA|               dp.la|                true|
+--------------------+--------------------+--------------------+
> highScoresDf = sampleDf.filter(sampleDf['Informal Score'] > 5)
> highScoresDf.select('Name', 'URL', 'Informal Score').show()
+-----------+--------------------+--------------+
|       Name|                 URL|Informal Score|
+-----------+--------------------+--------------+
|       CMOA|https://github.co...|            10|
|Penn Museum|https://www.penn....|             7|
|       DPLA|               dp.la|           100|
+-----------+--------------------+--------------+
```

### Number of Rows in a DataFrame

`count()` returns the number of records in a DataFrame.

```
> numSampleDf = sampleDf.count()
> numHighScoresDf = highScoresDf.count()
> print("Percentage of Datasets with a Score at or above 5: " + str(float(numHighScoresDf)/float(numSampleDf)*100) + "%")
Percentage of Datasets with a Score at or above 5: 50.0%
```

### Out of the Box Stats on your Dataframe

`.describe()` lets you get out of the box (so to speak) statistics on your DataFrame. You can run it on your entire DataFrame, or on selected subsets:

```
> sampleDf.describe().show()
+-------+-----------+----------------+------+--------------------+--------------------+-----------------+
|summary|       Name|Institution Type|Format|                 URL|         Description|   Informal Score|
+-------+-----------+----------------+------+--------------------+--------------------+-----------------+
|  count|          6|               6|     6|                   5|                   4|                6|
|   mean|       null|            null|  null|                null|                null|          21.1667|
| stddev|       null|            null|  null|                null|                null|38.71649088782022|
|    min|       CMOA|     Aggregator |   CSV|               dp.la|JSON is poorly st...|                3|
|    max|Penn Museum|          Museum|   XML|https://www.penn....|            ¯_(ツ)_/¯|              100|
+-------+-----------+----------------+------+--------------------+--------------------+-----------------+

> sampleDf.describe('Informal Score').show()
+-------+-----------------+
|summary|   Informal Score|
+-------+-----------------+
|  count|                6|
|   mean|          21.1667|
| stddev|38.71649088782022|
|    min|                3|
|    max|              100|
+-------+-----------------+
```

### Order Responses

`orderBy()` lets you order your DataFrame according to a given column. It defaults to ascending order, and `.desc()` applied to the column used for ordering can reverse that:

```
> sampleDf.orderBy("Informal Score").show()
+--------------------+----------------+--------------+--------------------+--------------------+--------------+
|                Name|Institution Type|        Format|                 URL|         Description|Informal Score|
+--------------------+----------------+--------------+--------------------+--------------------+--------------+
|DigitalNZ Te Puna...|         Library|           XML|https://natlib.go...|            MARC XML|             3|
|          Met Museum|          Museum|           CSV|                null|            ¯_(ツ)_/¯|             3|
|Canadian Subject ...|         Library|       RDF/XML|http://www.collec...|            Ugh, rdf|             4|
|         Penn Museum|          Museum|JSON, CSV, XML|https://www.penn....|JSON is poorly st...|             7|
|                CMOA|          Museum|     JSON, CSV|https://github.co...|                null|            10|
|                DPLA|     Aggregator |  CSV,JSON,XML|               dp.la|                null|           100|
+--------------------+----------------+--------------+--------------------+--------------------+--------------+
> sampleDf.orderBy(sampleDf["Informal Score"].desc()).show()
+--------------------+----------------+--------------+--------------------+--------------------+--------------+
|                Name|Institution Type|        Format|                 URL|         Description|Informal Score|
+--------------------+----------------+--------------+--------------------+--------------------+--------------+
|                DPLA|     Aggregator |  CSV,JSON,XML|               dp.la|                null|           100|
|                CMOA|          Museum|     JSON, CSV|https://github.co...|                null|            10|
|         Penn Museum|          Museum|JSON, CSV, XML|https://www.penn....|JSON is poorly st...|             7|
|Canadian Subject ...|         Library|       RDF/XML|http://www.collec...|            Ugh, rdf|             4|
|          Met Museum|          Museum|           CSV|                null|            ¯_(ツ)_/¯|             3|
|DigitalNZ Te Puna...|         Library|           XML|https://natlib.go...|            MARC XML|             3|
+--------------------+----------------+--------------+--------------------+--------------------+--------------+
```

### Group Responses

Like with SQL, you can Group your DataFrame as well. Here, we make a new DataFrame off of `sampleDf`. This new DataFrame only contains `Institution Type`, groups by it and shows the `count()` for each:

```
> URLGroupDf = sampleDf.groupBy("Institution Type").count()
> URLGroupDf.show()
+----------------+-----+
|Institution Type|count|
+----------------+-----+
|         Library|    2|
|     Aggregator |    1|
|          Museum|    3|
+----------------+-----+
> URLGroupDf.printSchema()
root
 |-- Institution Type: string (nullable = true)
 |-- count: long (nullable = false)
```

### Group Values then Compute an Aggregate

`count` is already a type of Aggregate function. However, we can use `agg` to then pass in our own special function with set formulas.

```
> formatGroupDf = sampleDf.groupBy("Institution Type").agg({"Informal Score": 'mean'})
> formatGroupDf.show()
+----------------+-------------------+
|Institution Type|avg(Informal Score)|
+----------------+-------------------+
|         Library|             3.5000|
|     Aggregator |           100.0000|
|          Museum|             6.6667|
+----------------+-------------------+
> formatGroupDf.printSchema()
root
 |-- Institution Type: string (nullable = true)
 |-- avg(Informal Score): decimal(14,4) (nullable = true)
```

### Get Distinct Values in a Column of our DataFrame

`distinct()` returns only unique values in a Column of a DataFrame. Note: this returns a notably bad response of distinct values that we use in the next section.

```
> distinctFormatDf = sampleDf.select("Format").distinct()
> distinctFormatDf.show()
+--------------+
|        Format|
+--------------+
|           CSV|
|           XML|
|     JSON, CSV|
|JSON, CSV, XML|
|       RDF/XML|
|  CSV,JSON,XML|
+--------------+
> distinctFormatDf.count()
6
```

### Combining Aggregates, SQL Functions, & Distinct Values

```
> countDistinctDF = sampleDf.select("Name", "Institution Type", "Format").groupBy("Institution Type").agg(countDistinct("Format"))
> countDistinctDF.show()
+----------------+----------------------+
|Institution Type|count(DISTINCT Format)|
+----------------+----------------------+
|         Library|                     2|
|     Aggregator |                     1|
|          Museum|                     3|
+----------------+----------------------+
```

## Part 3: Creating New Dataframes to Expand or Rework our Original Dataset

Now we want to focus on deriving new DataFrames from our original `sampleDf`.

### Create a new DataFrame without Duplicate Values

`dropDuplicates()` will remove Rows with duplicate values from the select Column(s).

```
> scoresDf = sampleDf.select('Informal Score').dropDuplicates()
> scoresDf.show()
+--------------+
|Informal Score|
+--------------+
|             7|
|            10|
|           100|
|             3|
|             4|
+--------------+
```

### DataFrame with Dropped Null Values

```
> noNullDf = sampleDf.select('Name', 'URL').dropna()
> noNullDf.show()
+--------------------+--------------------+
|                Name|                 URL|
+--------------------+--------------------+
|                CMOA|https://github.co...|
|         Penn Museum|https://www.penn....|
|DigitalNZ Te Puna...|https://natlib.go...|
|Canadian Subject ...|http://www.collec...|
|                DPLA|               dp.la|
+--------------------+--------------------+
```

### DataFrame with Filled Null Values

```
> nonNullDf = sampleDf.select('Name', 'URL').fillna('No URL')
> nonNullDf.show()
+--------------------+--------------------+
|                Name|                 URL|
+--------------------+--------------------+
|                CMOA|https://github.co...|
|         Penn Museum|https://www.penn....|
|          Met Museum|              No URL|
|DigitalNZ Te Puna...|https://natlib.go...|
|Canadian Subject ...|http://www.collec...|
|                DPLA|               dp.la|
+--------------------+--------------------+
```

### DataFrame with Rows where Given Column *is* Null

```
> filterNonNullDF = sampleDf.filter(sampleDf.URL.isNull()).sort("Name")
> filterNonNullDF.show()
+----------+----------------+------+----+-----------+--------------+
|      Name|Institution Type|Format| URL|Description|Informal Score|
+----------+----------------+------+----+-----------+--------------+
|Met Museum|          Museum|   CSV|null|   ¯_(ツ)_/¯|             3|
+----------+----------------+------+----+-----------+--------------+
```

### Handling `Format` (Multivalue fields)

#### Split `Format` into Arrays of Values

```
> from pyspark.sql.functions import split
> sampleDf = sampleDf.withColumn("Format Array", split(sampleDf.Format, ","))
> sampleDf.show()
+--------------------+----------------+--------------+--------------------+--------------------+--------------+------------------+
|                Name|Institution Type|        Format|                 URL|         Description|Informal Score|      Format Array|
+--------------------+----------------+--------------+--------------------+--------------------+--------------+------------------+
|                CMOA|          Museum|     JSON, CSV|https://github.co...|                null|            10|      [JSON,  CSV]|
|         Penn Museum|          Museum|JSON, CSV, XML|https://www.penn....|JSON is poorly st...|             7|[JSON,  CSV,  XML]|
|          Met Museum|          Museum|           CSV|                null|            ¯_(ツ)_/¯|             3|             [CSV]|
|DigitalNZ Te Puna...|         Library|           XML|https://natlib.go...|            MARC XML|             3|             [XML]|
|Canadian Subject ...|         Library|       RDF/XML|http://www.collec...|            Ugh, rdf|             4|         [RDF/XML]|
|                DPLA|     Aggregator |  CSV,JSON,XML|               dp.la|                null|           100|  [CSV, JSON, XML]|
+--------------------+----------------+--------------+--------------------+--------------------+--------------+------------------+
> sampleDf.printSchema()
root
 |-- Name: string (nullable = true)
 |-- Institution Type: string (nullable = true)
 |-- Format: string (nullable = true)
 |-- URL: string (nullable = true)
 |-- Description: string (nullable = true)
 |-- Informal Score: decimal(10,0) (nullable = true)
 |-- Format Array: array (nullable = true)
 |    |-- element: string (containsNull = true)
```

#### Explode `Format` into Multiple Rows of Values

```
> from pyspark.sql.functions import explode
> sampleDf = sampleDf.withColumn("Format", explode(split("Format", ",")))
> sampleDf.show()
+--------------------+----------------+-------+--------------------+--------------------+--------------+------------------+
|                Name|Institution Type| Format|                 URL|         Description|Informal Score|      Format Array|
+--------------------+----------------+-------+--------------------+--------------------+--------------+------------------+
|                CMOA|          Museum|   JSON|https://github.co...|                null|            10|      [JSON,  CSV]|
|                CMOA|          Museum|    CSV|https://github.co...|                null|            10|      [JSON,  CSV]|
|         Penn Museum|          Museum|   JSON|https://www.penn....|JSON is poorly st...|             7|[JSON,  CSV,  XML]|
|         Penn Museum|          Museum|    CSV|https://www.penn....|JSON is poorly st...|             7|[JSON,  CSV,  XML]|
|         Penn Museum|          Museum|    XML|https://www.penn....|JSON is poorly st...|             7|[JSON,  CSV,  XML]|
|          Met Museum|          Museum|    CSV|                null|            ¯_(ツ)_/¯|             3|             [CSV]|
|DigitalNZ Te Puna...|         Library|    XML|https://natlib.go...|            MARC XML|             3|             [XML]|
|Canadian Subject ...|         Library|RDF/XML|http://www.collec...|            Ugh, rdf|             4|         [RDF/XML]|
|                DPLA|     Aggregator |    CSV|               dp.la|                null|           100|  [CSV, JSON, XML]|
|                DPLA|     Aggregator |   JSON|               dp.la|                null|           100|  [CSV, JSON, XML]|
|                DPLA|     Aggregator |    XML|               dp.la|                null|           100|  [CSV, JSON, XML]|
+--------------------+----------------+-------+--------------------+--------------------+--------------+------------------+
```

#### Create New Columns With Booleans if Format among Format Values

```
> sampleDf.select("Format").distinct().show()
+-------+
| Format|
+-------+
|    CSV|
|    XML|
|    CSV|
|    XML|
|RDF/XML|
|   JSON|
+-------+
> sampleDf = sampleDf.withColumn('CSV', sampleDf.Format.like("%CSV%"))
> sampleDf = sampleDf.withColumn('XML', sampleDf.Format.like("%XML%"))
> sampleDf = sampleDf.withColumn('RDF', sampleDf.Format.like("%RDF%"))
> sampleDf = sampleDf.withColumn('JSON', sampleDf.Format.like("%JSON%"))
> sampleDf.show()
+--------------------+----------------+-------+--------------------+--------------------+--------------+------------------+-----+-----+-----+-----+
|                Name|Institution Type| Format|                 URL|         Description|Informal Score|      Format Array|  CSV|  XML|  RDF| JSON|
+--------------------+----------------+-------+--------------------+--------------------+--------------+------------------+-----+-----+-----+-----+
|                CMOA|          Museum|   JSON|https://github.co...|                null|            10|      [JSON,  CSV]|false|false|false| true|
|                CMOA|          Museum|    CSV|https://github.co...|                null|            10|      [JSON,  CSV]| true|false|false|false|
|         Penn Museum|          Museum|   JSON|https://www.penn....|JSON is poorly st...|             7|[JSON,  CSV,  XML]|false|false|false| true|
|         Penn Museum|          Museum|    CSV|https://www.penn....|JSON is poorly st...|             7|[JSON,  CSV,  XML]| true|false|false|false|
|         Penn Museum|          Museum|    XML|https://www.penn....|JSON is poorly st...|             7|[JSON,  CSV,  XML]|false| true|false|false|
|          Met Museum|          Museum|    CSV|                null|            ¯_(ツ)_/¯|             3|             [CSV]| true|false|false|false|
|DigitalNZ Te Puna...|         Library|    XML|https://natlib.go...|            MARC XML|             3|             [XML]|false| true|false|false|
|Canadian Subject ...|         Library|RDF/XML|http://www.collec...|            Ugh, rdf|             4|         [RDF/XML]|false| true| true|false|
|                DPLA|     Aggregator |    CSV|               dp.la|                null|           100|  [CSV, JSON, XML]| true|false|false|false|
|                DPLA|     Aggregator |   JSON|               dp.la|                null|           100|  [CSV, JSON, XML]|false|false|false| true|
|                DPLA|     Aggregator |    XML|               dp.la|                null|           100|  [CSV, JSON, XML]|false| true|false|false|
+--------------------+----------------+-------+--------------------+--------------------+--------------+------------------+-----+-----+-----+-----+
```

#### Transform DataFrame to RDD & Use `map`

```
> sampleRdd = sampleDf.select("Format").rdd.map(lambda x: x[0].split(","))
> sampleRdd.take(5)
[[u'JSON'], [u' CSV'], [u'JSON'], [u' CSV'], [u' XML']]
```

### Drop a Column

```
> sampleDf = sampleDf.drop('Format Array')
> sampleDf.show()
+--------------------+----------------+-------+--------------------+--------------------+--------------+-----+-----+-----+-----+
|                Name|Institution Type| Format|                 URL|         Description|Informal Score|  CSV|  XML|  RDF| JSON|
+--------------------+----------------+-------+--------------------+--------------------+--------------+-----+-----+-----+-----+
|                CMOA|          Museum|   JSON|https://github.co...|                null|            10|false|false|false| true|
|                CMOA|          Museum|    CSV|https://github.co...|                null|            10| true|false|false|false|
|         Penn Museum|          Museum|   JSON|https://www.penn....|JSON is poorly st...|             7|false|false|false| true|
|         Penn Museum|          Museum|    CSV|https://www.penn....|JSON is poorly st...|             7| true|false|false|false|
|         Penn Museum|          Museum|    XML|https://www.penn....|JSON is poorly st...|             7|false| true|false|false|
|          Met Museum|          Museum|    CSV|                null|            ¯_(ツ)_/¯|             3| true|false|false|false|
|DigitalNZ Te Puna...|         Library|    XML|https://natlib.go...|            MARC XML|             3|false| true|false|false|
|Canadian Subject ...|         Library|RDF/XML|http://www.collec...|            Ugh, rdf|             4|false| true| true|false|
|                DPLA|     Aggregator |    CSV|               dp.la|                null|           100| true|false|false|false|
|                DPLA|     Aggregator |   JSON|               dp.la|                null|           100|false|false|false| true|
|                DPLA|     Aggregator |    XML|               dp.la|                null|           100|false| true|false|false|
+--------------------+----------------+-------+--------------------+--------------------+--------------+-----+-----+-----+-----+
```

### Remove Whitespace Around Values in a Column

```
> from pyspark.sql.functions import regexp_replace
> sampleDf = sampleDf.withColumn("Format", regexp_replace(sampleDf.Format, "\s+", ""))
> sampleDf.select("Format").distinct().show()
+-------+
| Format|
+-------+
|    CSV|
|    XML|
|RDF/XML|
|   JSON|
+-------+
```

### Create Derivative Dataframe Based on Conditional Using `where()`

```
> CSVsampleDf = sampleDf.where((sampleDf.Format == "CSV"))
> CSVsampleDf.show()
+-----------+----------------+------+--------------------+--------------------+--------------+----+-----+-----+-----+
|       Name|Institution Type|Format|                 URL|         Description|Informal Score| CSV|  XML|  RDF| JSON|
+-----------+----------------+------+--------------------+--------------------+--------------+----+-----+-----+-----+
|       CMOA|          Museum|   CSV|https://github.co...|                null|            10|true|false|false|false|
|Penn Museum|          Museum|   CSV|https://www.penn....|JSON is poorly st...|             7|true|false|false|false|
| Met Museum|          Museum|   CSV|                null|            ¯_(ツ)_/¯|             3|true|false|false|false|
|       DPLA|     Aggregator |   CSV|               dp.la|                null|           100|true|false|false|false|
+-----------+----------------+------+--------------------+--------------------+--------------+----+-----+-----+-----+
```

## Part 4: Using SQL to Analyze our Dataset

To have a more dynamic experience, let’s create a temporary (in-memory) view that we can query against and interact with the resulting data in a table or graph format. The temporary view will allow us to execute SQL queries against it.

Note that the temporary view will reside in memory as long as the Spark session is alive. [Here](http://cse.unl.edu/~sscott/ShowFiles/SQL/CheatSheet/SQLCheatSheet.html) is a SQL Cheatsheet in case you need it.

### Create Temporary SQL View from a DataFrame

```
> sampleDf.createOrReplaceTempView("sampleDataView")
```

### Write & Run a Spark SQL Query

This will take a Spark SQL Query then run it, using `collect()` to return Rows from our specified DataFrame that match the query:

```
> sparkQuery = spark.sql("SELECT * FROM sampleDataView LIMIT 20")
> sparkQuery.collect()
[Row(Name=u'CMOA', Institution Type=u'Museum', Format=u'JSON', URL=u'https://github.com/cmoa/collection', Description=None, Informal Score=Decimal('10'), CSV=False, XML=False, RDF=False, JSON=True), Row(Name=u'CMOA', Institution Type=u'Museum', Format=u'CSV', URL=u'https://github.com/cmoa/collection', Description=None, Informal Score=Decimal('10'), CSV=True, XML=False, RDF=False, JSON=False), Row(Name=u'Penn Museum', Institution Type=u'Museum', Format=u'JSON', URL=u'https://www.penn.museum/collections/data.php', Description=u'JSON is poorly structured', Informal Score=Decimal('7'), CSV=False, XML=False, RDF=False, JSON=True), Row(Name=u'Penn Museum', Institution Type=u'Museum', Format=u'CSV', URL=u'https://www.penn.museum/collections/data.php', Description=u'JSON is poorly structured', Informal Score=Decimal('7'), CSV=True, XML=False, RDF=False, JSON=False), Row(Name=u'Penn Museum', Institution Type=u'Museum', Format=u'XML', URL=u'https://www.penn.museum/collections/data.php', Description=u'JSON is poorly structured', Informal Score=Decimal('7'), CSV=False, XML=True, RDF=False, JSON=False), Row(Name=u'Met Museum', Institution Type=u'Museum', Format=u'CSV', URL=None, Description=u'\xaf_(\u30c4)_/\xaf', Informal Score=Decimal('3'), CSV=True, XML=False, RDF=False, JSON=False), Row(Name=u'DigitalNZ Te Puna Web Directory', Institution Type=u'Library', Format=u'XML', URL=u'https://natlib.govt.nz/files/data/tepunawebdirectory.xml', Description=u'MARC XML', Informal Score=Decimal('3'), CSV=False, XML=True, RDF=False, JSON=False), Row(Name=u'Canadian Subject Headings', Institution Type=u'Library', Format=u'RDF/XML', URL=u'http://www.collectionscanada.gc.ca/obj/900/f11/040004/csh.rdf', Description=u'Ugh, rdf', Informal Score=Decimal('4'), CSV=False, XML=True, RDF=True, JSON=False), Row(Name=u'DPLA', Institution Type=u'Aggregator ', Format=u'CSV', URL=u'dp.la', Description=None, Informal Score=Decimal('100'), CSV=True, XML=False, RDF=False, JSON=False), Row(Name=u'DPLA', Institution Type=u'Aggregator ', Format=u'JSON', URL=u'dp.la', Description=None, Informal Score=Decimal('100'), CSV=False, XML=False, RDF=False, JSON=True), Row(Name=u'DPLA', Institution Type=u'Aggregator ', Format=u'XML', URL=u'dp.la', Description=None, Informal Score=Decimal('100'), CSV=False, XML=True, RDF=False, JSON=False)]
```

Here's another Spark SQL Example:

```
> sparkQuery = spark.sql("""SELECT `Institution Type`, COUNT(DISTINCT(Format)) AS NumFormats FROM sampleDataView GROUP BY `Institution Type`""")
> sparkQuery.collect()
> for n in sparkQuery.collect():
>    n
Row(Institution Type=u'Library', NumFormats=2)
Row(Institution Type=u'Aggregator ', NumFormats=3)
Row(Institution Type=u'Museum', NumFormats=3)
```

## Part 5: Simple Data Visualization

Use the `sql` interpreter in Zeppelin to run SQL queries against our previously defined Temporary SQL View (just enter it where the table name normally goes in a SQL Query).

This gives us immediate access to the SQL-driven visualizations available in Zeppelin.


This is the end of this session. In the [next session, you'll be working in small groups to take some of the things learned above and apply them to current, CHO (cultural heritage organization) data](working-with-cho-data.md). 




----
