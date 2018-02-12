# Spark in the Dark Workshop: PySpark Functions Cheatsheet

## Dataframe Functions:

### Shown in Workshop

* **agg:**
* **collect():** Returns all the records as a list of Row. *Note: we only see this in the context of a SQL call, but it can be used outside of that.*
* **columns:** Returns all column names as a list.
* **count():** Returns the number of rows in this DataFrame.
* **distinct():** Returns a new DataFrame containing the distinct rows in this DataFrame.
* **drop(colName):** Returns a new DataFrame that drops the specified column. This is a no-op if schema doesn’t contain the given column name(s).
* **dropna():** Returns a new DataFrame omitting rows with null values.
* **dropDuplicates():** Return a new DataFrame with duplicate rows removed, optionally only considering certain columns.
* **fillna(value, subset=None):** Replace null values, alias for na.fill(). DataFrame.fillna() and DataFrameNaFunctions.fill() are aliases of each other.
* **filter(condition) / where(condition):** Filters rows using the given condition. `where()` is an alias for filter().
* **groupBy(cols):** Groups the DataFrame using the specified columns, so we can run aggregation on them. `groupby()` is an alias for `groupBy()`.
* **head(n):** Returns the first `n` rows.
* **orderBy(cols):** Return a new DataFrame sorted by the specified column(s).
* **printSchema():** Prints out the schema in the tree format.
* **rdd:** Returns the content as an `pyspark.RDD` of `Row`.
* **show(n, truncate=True):** Prints the first n rows to the console.
* **sort(cols):** Returns a new DataFrame sorted by the specified column(s).
* **withColumn(colName, column):** Returns a new DataFrame by adding a column or replacing the existing column that has the same name.

### Not Shown, but Possibly Helpful

* **alias:** Returns a new DataFrame with an alias set.
* **corr(col1, col2, method=None):** Calculates the correlation of two columns of a DataFrame as a double value. Currently only supports the Pearson Correlation Coefficient. DataFrame.corr() and DataFrameStatFunctions.corr() are aliases of each other.
* **dtypes:** Returns all column names and their data types as a list.
* **explain:** Prints the (logical and physical) plans to the console for debugging purpose.
* **first():** Returns the first row as a Row.
* **foreach(f):** Applies the f function to all Row of this DataFrame.
* **hint():** Specifies some hint on the current DataFrame.
* **limit():** Limits the result count to the number specified.
* **replace(to_replace, value=None, subset=None):** Returns a new DataFrame replacing a value with another value.
* **toDF(cols):** Returns a new class:DataFrame that with new specified column names
* **toJSON():** Converts a DataFrame into a RDD of string. Each row is turned into a JSON document as one element in the returned RDD.
* **withColumnRenamed(existing, new):** Returns a new DataFrame by renaming an existing column. This is a no-op if schema doesn’t contain the given column name.
* **write:** Interface for saving the content of the non-streaming DataFrame out into external storage.

## Column Functions:

* **asc():** Returns a sort expression based on the ascending order of the given column name.
* **between(lowerBound, upperBound):** A boolean expression that is evaluated to true if the value of this expression is between the given columns.
* **cast(dataType):** Convert the column into type dataType.
* **contains(val):** Return True/False if rows in Column contains this value.
* **endswith(val) / startswith(val):** Return a Boolean Column based on matching end / start of string.
* **getField():** An expression that gets a field by name in a StructField.
* **getItem():** An expression that gets an item at position ordinal out of a list, or gets an item by key out of a dict.
* **isNotNull() / isNull():** True if the current expression is not null / null.
* **otherwise() / when():** Evaluates a list of conditions and returns one of multiple possible result expressions. If Column.otherwise() is not invoked, None is returned for unmatched conditions.
* **substr():** Return a Column which is a substring of the column.

## Row functions:

* **asDict():** Return as an dict
