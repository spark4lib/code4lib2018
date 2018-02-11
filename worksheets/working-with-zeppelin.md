# Spark Practice: Working with `spark-shell` in Zeppelin Info

In this session, you will learn Zeppelin, Spark, & Spark SQL basics via primarily the DataFrames API. We are starting with a really simple dataset to focus on the tools. In the next session, we will use an expanded cultural heritage metadata set.

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

- `` - the default interpreter is set to be PySpark for our Workshop Docker Container.
- `%spark` - Spark interpreter to run Spark code written in Scala
- `%spark.sql` - Spark SQL interprter (to execute SQL queries against temporary tables in Spark)
- `%sh` - Shell interpreter to run shell commands
- `%angular` - Angular interpreter to run Angular and HTML code
- `%md` - Markdown for displaying formatted text, links, and images

To learn more about Zeppelin interpreters check out this [link](https://zeppelin.apache.org/docs/0.5.6-incubating/manual/interpreters.html).

### Check Your Environment

We want to make sure you have:

1. PySpark / Spark Working
2. You can load the sample data provided on the docker container
3. You know the Spark Version you're working with

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
