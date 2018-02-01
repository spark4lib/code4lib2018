# Spark Practice: Working with spark-shell

## Getting Started

In our Github repository:
- see notebooks/shell-start.json & worksheets/shell-start.md

Getting to Zeppelin (from which we will run the shell, more on this in a second):
- Go to http://localhost:8080
- If Zeppelin is not running:
  - make sure you are running the workshop docker container:
    - ```shell
        $ docker run -p 8080:8080 --rm -v $PWD/logs:/logs -v $PWD/notebook:/notebook -e ZEPPELIN_LOG_DIR='/logs' -e ZEPPELIN_NOTEBOOK_DIR='/notebook' --name …
      ```
  - If Zeppelin is still not running, try restarting Zeppelin in your Docker container:
    - ```shell
        $ ssh into docker: docker exec -it zeppelin /bin/bash
      ```
    - then in that docker shell, run:
    - ```shell
        $ zeppelin daemon start: ./bin/zeppelin-daemon.sh start
      ```

## Our Dataset for this Simple Walk through:

| Name | Type | Format | URL | Description | Scott's Unofficial Score |
| ---- | ---- | ------ | --- | ----------- | ------------------------ |
| CMOA | Museum | JSON, CSV | https://github.com/cmoa/collection | N/A | 10 |
| Penn Museum | Museum | JSON, CSV, XML | N/A | JSON is poorly structed | 8 |
| Met Museum | Museum | CSV | N/A | Ugh. the met... | 5 |
| DigitalNZ Te Puna Web Directory | Library | XML | https://natlib.govt.nz/files/data/tepunawebdirectory.xml | MARC XML | 4 |
| Canadian Subject Headings | Library | RDF/XML | http://www.collectionscanada.gc.ca/obj/900/f11/040004/csh.rdf | Ugh, rdf | 2 |



## Working / Scratch area for running shell in notebook:

```
%sh
mkdir data
curl https://raw.githubusercontent.com/spark4lib/code4lib2018/2233b54d16975f68e99cdab6198623062308483d/data/shell-start-simple.csv >> data/shell-start-simple.csv
head data/shell-start-simple.csv
```

```
%pyspark
spark.read.csv(
    "data/shell-start-simple.csv", header=True, mode="DROPMALFORMED", inferSchema=True
)
```
