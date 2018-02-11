# Spark in the Dark Workshop: PySpark Functions Cheatsheet

## Dataframe Functions:

### Shown in Workshop

* **agg:**
* **collect():** Returns all the records as a list of Row. *Note: we only see this in the context of a SQL call, but it can be used outside of that.*
* **columns:** Returns all column names as a list.
* **count():** Returns the number of rows in this DataFrame.
* **distinct():** Returns a new DataFrame containing the distinct rows in this DataFrame.
* **drop(colName):** Returns a new DataFrame that drops the specified column. This is a no-op if schema doesn’t contain the given column name(s).
* **dropDuplicates:**
* **fillna(value, subset=None):**
* **filter(condition) / where(condition):**
* **groupBy(cols):**
* **head():**
* **orderBy(cols):**
* **printSchema():**
* **rdd:**
* **show(n):**

### Not Shown, but Possibly Helpful

* **alias:** Returns a new DataFrame with an alias set.
* **corr(col1, col2, method=None):**
* **cube:**
* **dropnav:**
* **dtypes:**
* **explain:**
* **first():**
* **foreach(f):**
* **hint:**
* **limit():**
* **replace:**
* **sort(cols):**
* **toDF(cols):**
* **toJSON():**
* **withColumn():**
* **withColumnRenamed():**
* **write:**

## Column Functions:

* **asc():**
* **between(lowerBound, upperBound):**
* **cast():**
* **contains():**
* **endswith() / startswith():**
* **getFields():**
* **getItem():**
* **isNotNull() / isNull():**
* **otherwise() / when():**
* **substr():**

## Row functions:

* **asDict():**
