# Spark Practice: Working with spark-shell

## Getting Started



## Our Dataset for this Simple Walk through:

| Name | Institution Type | Format | URL | Description |
| ---- | ---------------- | ------ | --- | ----------- |
| CMOA | Museum | JSON, CSV | https://github.com/cmoa/collection |
| Penn Museum | Museum | JSON, CSV, XML | https://www.penn.museum/collections/data.php | JSON is poorly structured |
| Met Museum | Museum | CSV |  | ¯\_(ツ)_/¯ |
| DigitalNZ Te Puna Web Directory | Library | XML | https://natlib.govt.nz/files/data/tepunawebdirectory.xml,MARC | XML |
| Canadian Subject Headings | Library | RDF/XML | http://www.collectionscanada.gc.ca/obj/900/f11/040004/csh.rdf | "Ugh, rdf" |
| DPLA | Aggregator | CSV,JSON,XML | dp.la |


## Run Spark-Shell Yourself


$ docker pull gettyimages/spark
$ docker run -p 3030:8080 gettyimages/spark
Go to http://localhost:3030 to see your Spark Dashboard

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
5.

```
%pyspark
spark.read.csv(
    "data/shell-start-simple.csv", header=True, mode="DROPMALFORMED", inferSchema=True
)
```
