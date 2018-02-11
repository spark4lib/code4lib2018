# Spark Practice: Working with spark-shell

## Steps Shown in Walk-through

#### Run a PySpark Interpreter Shell

```shell
$ pyspark
```

You should see something like the following:

```
$ pyspark
Python 3.4.2 (default, Oct  8 2014, 10:45:20)
[GCC 4.9.1] on linux
Type "help", "copyright", "credits" or "license" for more information.
Setting default log level to "WARN".
To adjust logging level use sc.setLogLevel(newLevel). For SparkR, use setLogLevel(newLevel).
18/02/09 15:34:31 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
Welcome to
      ____              __
     / __/__  ___ _____/ /__
    _\ \/ _ \/ _ `/ __/  '_/
   /__ / .__/\_,_/_/ /_/\_\   version 2.2.1
      /_/

Using Python version 3.4.2 (default, Oct  8 2014 10:45:20)
SparkSession available as 'spark'.
>>>
```

Notes:
- we are running in the PySpark interpreter to see spark-shell. Once could also run a Scala interpreter.
- This automatically makes a SparkSession for us; when coding, you'll need to start this yourself.

#### Check out the Spark Dashboard

If you're running the command below (i.e. you're running your docker container on port 3030), at https://localhost:3030 should be your Spark Dashboard.

![Image of Spark Dashboard with no jobs running](../images/spark-dashboard.png).

#### Our Dataset for this Simple Walk through:

See [the CSV file here](../sample-data/small-sample.csv).

| Name | Institution Type | Format | URL | Description |
| ---- | ---------------- | ------ | --- | ----------- |
| CMOA | Museum | JSON, CSV | https://github.com/cmoa/collection |
| Penn Museum | Museum | JSON, CSV, XML | https://www.penn.museum/collections/data.php | JSON is poorly structured |
| Met Museum | Museum | CSV |  | ¯\_(ツ)_/¯\n |
| DigitalNZ Te Puna Web Directory | Library | XML | https://natlib.govt.nz/files/data/tepunawebdirectory.xml,MARC | XML |
| Canadian Subject Headings | Library | RDF/XML | http://www.collectionscanada.gc.ca/obj/900/f11/040004/csh.rdf | "Ugh, rdf" |
| DPLA | Aggregator | CSV,JSON,XML | dp.la |

#### In our Spark-Shell (PySpark Interpreter), Read a CSV File

Read the CSV using the PySpark read & CSV libraries:

```py
> spark.read.csv("sample-data/small-sample.csv")
DataFrame[_c0: string, _c1: string, _c2: string, _c3: string, _c4: string, _c5: string]
```

We see it makes a dataframe. But what if we want to see the data? Run `.show()` on that data to see what was read in?

```py
> spark.read.csv("sample-data/small-sample.csv").show()
+--------------------+----------------+--------------+--------------------+--------------------+--------------+
|                 _c0|             _c1|           _c2|                 _c3|                 _c4|           _c5|
+--------------------+----------------+--------------+--------------------+--------------------+--------------+
|                Name|Institution Type|        Format|                 URL|         Description|Informal Score|
|                CMOA|          Museum|     JSON, CSV|https://github.co...|                null|            10|
|         Penn Museum|          Museum|JSON, CSV, XML|https://www.penn....|JSON is poorly st...|             7|
|          Met Museum|          Museum|           CSV|                null|            ¯_(ツ)_/¯|          null|
|                  ,3|            null|          null|                null|                null|          null|
|DigitalNZ Te Puna...|         Library|           XML|https://natlib.go...|            MARC XML|             3|
|Canadian Subject ...|         Library|       RDF/XML|http://www.collec...|            Ugh, rdf|             4|
|                DPLA|     Aggregator |  CSV,JSON,XML|               dp.la|                null|           100|
+--------------------+----------------+--------------+--------------------+--------------------+--------------+
```

Great! But... not entirely. What about headers?


```py
> spark.read.csv("sample-data/small-sample.csv", header=True).show()
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

That's better.

### Simple Analysis & Other Options for Reading a CSV File

But hold up, we seem to have trouble with a pesky newline in one of the cells. Let's try `DROPMALFORMED` mode and see if that helps.

```py
> spark.read.csv("sample-data/small-sample.csv", header=True, mode="DROPMALFORMED").show()
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

Well, that's better, but we don't want to lose the Met's row. Let's try the `multiLine` option.

```py
> spark.read.csv("sample-data/small-sample.csv", header=True, multiLine=True).show()
+--------------------+----------------+--------------+--------------------+--------------------+---------------+
|                Name|Institution Type|        Format|                 URL|         Description|Informal Score
+--------------------+----------------+--------------+--------------------+--------------------+---------------+
|                CMOA|          Museum|     JSON, CSV|https://github.co...|                null|            10
|         Penn Museum|          Museum|JSON, CSV, XML|https://www.penn....|JSON is poorly st...|             7
|          Met Museum|          Museum|           CSV|                null|          ¯_(ツ)_/¯
|             3
|DigitalNZ Te Puna...|         Library|           XML|https://natlib.go...|            MARC XML|             3
|Canadian Subject ...|         Library|       RDF/XML|http://www.collec...|            Ugh, rdf|             4
|                DPLA|     Aggregator |  CSV,JSON,XML|               dp.la|                null|           100
+--------------------+----------------+--------------+--------------------+--------------------+---------------+
```

That's... better? That column still looks wonky. Let's inspect the column, then inspect the schema.:

```py
> spark.read.csv("sample-data/small-sample.csv", header=True, multiLine=True).count()
```

Now, do we have the correct number of non-header rows? 6?

```py
> spark.read.csv("sample-data/small-sample.csv", header=True, multiLine=True).count()
6
```

Good! Now, is that `3` really in that `Name` column? Let's view the Name column:

```py
> spark.read.csv("sample-data/small-sample.csv", header=True, multiLine=True).select("Name").show()
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
```

Good! And the `Informal Score` column?

```py
> spark.read.csv("sample-data/small-sample.csv", header=True, multiLine=True).select(sample["Name"], sample["Informal Score"]).show()

Traceback (most recent call last):
  File /usr/spark-2.2.1/python/pyspark/sql/utils.py, line 63, in deco
    return f(*a, **kw)
  File /usr/local/lib/python3.4/dist-packages/py4j-0.10.6-py3.4.egg/py4j/protocol.py, line 320, in get_return_value
    format(target_id, ".", name), value)
py4j.protocol.Py4JJavaError: An error occurred while calling o146.apply.
...
pyspark.sql.utils.AnalysisException: Cannot resolve column name Informal Score among (Name, Institution Type, Format, URL, Description, Informal Score);
```

Uh Oh. What is our schema? Note, it is currently being inferred by the library.

```py
> sample.printSchema()
root
 |-- Name: string (nullable = true)
 |-- Institution Type: string (nullable = true)
 |-- Format: string (nullable = true)
 |-- URL: string (nullable = true)
 |-- Description: string (nullable = true)
: string (nullable = true)
```

First, let's use a little regular Python to make sure we have appropriate quoting. Note you can still run regular Python within your Pyspark interpreter:

```py
> import csv
> with open('sample-data/small-sample.csv') as fh:
    test = csv.reader(fh)
    with open('sample-data/small-sample-quoted.csv', 'w') as fout:
      test_write = csv.writer(fout, quotechar='"', quoting=csv.QUOTE_ALL)
      for row in test:
        test_write.writerow(row)
```

Next, let's pass in a specific schema. First we need to create one with the specific datatypes declared:

```
> from pyspark.sql.types import *
> customSchema = StructType([
    StructField("Name", StringType(), True),
    StructField("Institution Type", StringType(), True),
    StructField("Format", StringType(), True),
    StructField("URL", StringType(), True),
    StructField("Description", StringType(), True),
    StructField("Informal Score", DecimalType(), True)])
```

And now pass that schema in with our other options to read this CSV file into a dataframe:

```
> sample = spark.read.csv("sample-data/small-sample.csv", header=True, multiLine=True schema=customSchema)
> sample.show()
+--------------------+----------------+--------------+--------------------+--------------------+--------------+
|                Name|Institution Type|        Format|                 URL|         Description|Informal Score|
+--------------------+----------------+--------------+--------------------+--------------------+--------------+
|                CMOA|          Museum|     JSON, CSV|https://github.co...|                  10|          null|
|         Penn Museum|          Museum|JSON, CSV, XML|https://www.penn....|JSON is poorly st...|             7|
|          Met Museum|          Museum|           CSV|                null|           ¯_(ツ)_/¯
|             3|
|DigitalNZ Te Puna...|         Library|           XML|https://natlib.go...|            MARC XML|             3|
|Canadian Subject ...|         Library|       RDF/XML|http://www.collec...|            Ugh, rdf|             4|
|                DPLA|      Aggregator|  CSV,JSON,XML|               dp.la|                 100|          null|
+--------------------+----------------+--------------+--------------------+--------------------+--------------+
```

This looks fixed now, hooray! But let's call up

```
>>> sample.select(sample["Name"], sample["Informal Score"]).show()
+--------------------+--------------+
|                Name|Informal Score|
+--------------------+--------------+
|                CMOA|          null|
|         Penn Museum|             7|
|          Met Museum|             3|
|DigitalNZ Te Puna...|             3|
|Canadian Subject ...|             4|
|                DPLA|          null|
+--------------------+--------------+
```

>>> sample.select(sample["Name"], sample["Informal Score"], sample['Description']).show()
+--------------------+--------------+--------------------+
|                Name|Informal Score|         Description|
+--------------------+--------------+--------------------+
|                CMOA|          null|                  10|
|         Penn Museum|             7|JSON is poorly st...|
|          Met Museum|             3|           ¯_(ツ)_/¯
|
|DigitalNZ Te Puna...|             3|            MARC XML|
|Canadian Subject ...|             4|            Ugh, rdf|
|                DPLA|          null|                 100|
+--------------------+--------------+--------------------+


## Run the Spark-Shell Walk-through Yourself

1. I am using the Getty Images docker container for this section (Thanks, Getty!) for sake of consistency:
    ```bash
    $ docker pull gettyimages/spark
    ```
2. Start docker:
    ```bash
    $ docker run --name start-shell -p 3030:8080 gettyimages/spark
    ```
3. SSH into the Docker bash shell: :
    ```bash
    $ docker exec -it start-shell /bin/bash
    ```
4. Grab our small sample data and check out that it transferred okay
    ```bash
    $ mkdir sample-data
    $ curl https://raw.githubusercontent.com/spark4lib/code4lib2018/master/sample-data/small-sample.csv > sample-data/small-sample.csv
    $ head sample-data/small-sample.csv
    ```
5. Start pyspark
   ```bash
   $ pyspark
   ```
6. Run the steps above yourself!

sample_rdd = sample.rdd


Dataframe functions:

* alias
* collect()
* corr(col1, col2, method=None)
* cube
* dropna
* dtypes
* explain
* first()
* foreach(f)
* hint
* limit()
* replace
* sort(cols)
* toDF(cols)
* toJSON()
* withColumn()
* withColumnRenamed()
* write


* agg
* columns
* count()
* distinct
* drop
* dropDuplicates
* fillna(value, subset=None)
* filter(condition) / where(condition)
* groupBy(cols)
* head()
* orderBy(cols)
* printSchema()
* rdd
* show(n)

Col functions:

* asc()
* between(lowerBound, upperBound)
* cast()
* contains()
* endswith() / startswith()
* getFields()
* getItem()
* isNotNull() / isNull()
* otherwise() / when()
* substr()

Row functions:

* asDict()
