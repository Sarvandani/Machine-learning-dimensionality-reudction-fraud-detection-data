[![Made withJupyter](https://img.shields.io/badge/Made%20with-Jupyter-orange?style=for-the-badge&logo=Jupyter)](https://jupyter.org/try)
[![PyPI license](https://img.shields.io/pypi/l/ansicolortags.svg)](https://pypi.python.org/pypi/ansicolortags/)

# Dimensionality reudction of fraud detection data using Pyspark

The data of this work can be found in the following [link](https://www.kaggle.com/datasets/shivamb/vehicle-claim-fraud-detection).


```python
from pyspark.ml.feature import PCA, StringIndexer, VectorAssembler
from pyspark.sql import SparkSession
```
----

```python
# starting Spark session
spark = SparkSession.builder \
    .appName("Dimensionality Reduction with Spark for CSV") \
    .getOrCreate()

# Load CSV and see columns names and numbers
df = spark.read.format("csv").option("header", "true").load("./fraud_data.csv")  # Replace with the path to your CSV file
df.printSchema()
num_cols_df = print(len(df.columns))

```

    root
     |-- Month: string (nullable = true)
     |-- WeekOfMonth: string (nullable = true)
     |-- DayOfWeek: string (nullable = true)
     |-- Make: string (nullable = true)
     |-- AccidentArea: string (nullable = true)
     |-- DayOfWeekClaimed: string (nullable = true)
     |-- MonthClaimed: string (nullable = true)
     |-- WeekOfMonthClaimed: string (nullable = true)
     |-- Sex: string (nullable = true)
     |-- MaritalStatus: string (nullable = true)
     |-- Age: string (nullable = true)
     |-- Fault: string (nullable = true)
     |-- PolicyType: string (nullable = true)
     |-- VehicleCategory: string (nullable = true)
     |-- VehiclePrice: string (nullable = true)
     |-- FraudFound_P: string (nullable = true)
     |-- PolicyNumber: string (nullable = true)
     |-- RepNumber: string (nullable = true)
     |-- Deductible: string (nullable = true)
     |-- DriverRating: string (nullable = true)
     |-- Days_Policy_Accident: string (nullable = true)
     |-- Days_Policy_Claim: string (nullable = true)
     |-- PastNumberOfClaims: string (nullable = true)
     |-- AgeOfVehicle: string (nullable = true)
     |-- AgeOfPolicyHolder: string (nullable = true)
     |-- PoliceReportFiled: string (nullable = true)
     |-- WitnessPresent: string (nullable = true)
     |-- AgentType: string (nullable = true)
     |-- NumberOfSuppliments: string (nullable = true)
     |-- AddressChange_Claim: string (nullable = true)
     |-- NumberOfCars: string (nullable = true)
     |-- Year: string (nullable = true)
     |-- BasePolicy: string (nullable = true)
    
    33

--

```python
# see the contents of data for 10 columns
df.show(10)

```

    +-----+-----------+---------+------+------------+----------------+------------+------------------+------+-------------+---+-------------+--------------------+---------------+---------------+------------+------------+---------+----------+------------+--------------------+-----------------+------------------+------------+-----------------+-----------------+--------------+---------+-------------------+-------------------+------------+----+----------+
    |Month|WeekOfMonth|DayOfWeek|  Make|AccidentArea|DayOfWeekClaimed|MonthClaimed|WeekOfMonthClaimed|   Sex|MaritalStatus|Age|        Fault|          PolicyType|VehicleCategory|   VehiclePrice|FraudFound_P|PolicyNumber|RepNumber|Deductible|DriverRating|Days_Policy_Accident|Days_Policy_Claim|PastNumberOfClaims|AgeOfVehicle|AgeOfPolicyHolder|PoliceReportFiled|WitnessPresent|AgentType|NumberOfSuppliments|AddressChange_Claim|NumberOfCars|Year|BasePolicy|
    +-----+-----------+---------+------+------------+----------------+------------+------------------+------+-------------+---+-------------+--------------------+---------------+---------------+------------+------------+---------+----------+------------+--------------------+-----------------+------------------+------------+-----------------+-----------------+--------------+---------+-------------------+-------------------+------------+----+----------+
    |  Dec|          5|Wednesday| Honda|       Urban|         Tuesday|         Jan|                 1|Female|       Single| 21|Policy Holder|   Sport - Liability|          Sport|more than 69000|           0|           1|       12|       300|           1|        more than 30|     more than 30|              none|     3 years|         26 to 30|               No|            No| External|               none|             1 year|      3 to 4|1994| Liability|
    |  Jan|          3|Wednesday| Honda|       Urban|          Monday|         Jan|                 4|  Male|       Single| 34|Policy Holder|   Sport - Collision|          Sport|more than 69000|           0|           2|       15|       400|           4|        more than 30|     more than 30|              none|     6 years|         31 to 35|              Yes|            No| External|               none|          no change|   1 vehicle|1994| Collision|
    |  Oct|          5|   Friday| Honda|       Urban|        Thursday|         Nov|                 2|  Male|      Married| 47|Policy Holder|   Sport - Collision|          Sport|more than 69000|           0|           3|        7|       400|           3|        more than 30|     more than 30|                 1|     7 years|         41 to 50|               No|            No| External|               none|          no change|   1 vehicle|1994| Collision|
    |  Jun|          2| Saturday|Toyota|       Rural|          Friday|         Jul|                 1|  Male|      Married| 65|  Third Party|   Sedan - Liability|          Sport| 20000 to 29000|           0|           4|        4|       400|           2|        more than 30|     more than 30|                 1| more than 7|         51 to 65|              Yes|            No| External|        more than 5|          no change|   1 vehicle|1994| Liability|
    |  Jan|          5|   Monday| Honda|       Urban|         Tuesday|         Feb|                 2|Female|       Single| 27|  Third Party|   Sport - Collision|          Sport|more than 69000|           0|           5|        3|       400|           1|        more than 30|     more than 30|              none|     5 years|         31 to 35|               No|            No| External|               none|          no change|   1 vehicle|1994| Collision|
    |  Oct|          4|   Friday| Honda|       Urban|       Wednesday|         Nov|                 1|  Male|       Single| 20|  Third Party|   Sport - Collision|          Sport|more than 69000|           0|           6|       12|       400|           3|        more than 30|     more than 30|              none|     5 years|         21 to 25|               No|            No| External|             3 to 5|          no change|   1 vehicle|1994| Collision|
    |  Feb|          1| Saturday| Honda|       Urban|          Monday|         Feb|                 3|  Male|      Married| 36|  Third Party|   Sport - Collision|          Sport|more than 69000|           0|           7|       14|       400|           1|        more than 30|     more than 30|                 1|     7 years|         36 to 40|               No|            No| External|             1 to 2|          no change|   1 vehicle|1994| Collision|
    |  Nov|          1|   Friday| Honda|       Urban|         Tuesday|         Mar|                 4|  Male|       Single|  0|Policy Holder|   Sport - Collision|          Sport|more than 69000|           0|           8|        1|       400|           4|        more than 30|     more than 30|                 1|         new|         16 to 17|               No|            No| External|               none|          no change|   1 vehicle|1994| Collision|
    |  Dec|          4| Saturday| Honda|       Urban|       Wednesday|         Dec|                 5|  Male|       Single| 30|Policy Holder|   Sport - Collision|          Sport|more than 69000|           0|           9|        7|       400|           4|        more than 30|     more than 30|              none|     6 years|         31 to 35|               No|           Yes| External|             3 to 5|          no change|   1 vehicle|1994| Collision|
    |  Apr|          3|  Tuesday|  Ford|       Urban|       Wednesday|         Apr|                 3|  Male|      Married| 42|Policy Holder|Utility - All Perils|        Utility|more than 69000|           0|          10|        7|       400|           1|        more than 30|     more than 30|            2 to 4| more than 7|         36 to 40|               No|            No| External|             3 to 5|          no change|   1 vehicle|1994|All Perils|
    +-----+-----------+---------+------+------------+----------------+------------+------------------+------+-------------+---+-------------+--------------------+---------------+---------------+------------+------------+---------+----------+------------+--------------------+-----------------+------------------+------------+-----------------+-----------------+--------------+---------+-------------------+-------------------+------------+----+----------+
    only showing top 10 rows
    
----


```python
# Select the feature columns from the DataFrame
feature_cols = df.columns[1:33]  
print(feature_cols)
print(len(feature_cols))
```

    ['WeekOfMonth', 'DayOfWeek', 'Make', 'AccidentArea', 'DayOfWeekClaimed', 'MonthClaimed', 'WeekOfMonthClaimed', 'Sex', 'MaritalStatus', 'Age', 'Fault', 'PolicyType', 'VehicleCategory', 'VehiclePrice', 'FraudFound_P', 'PolicyNumber', 'RepNumber', 'Deductible', 'DriverRating', 'Days_Policy_Accident', 'Days_Policy_Claim', 'PastNumberOfClaims', 'AgeOfVehicle', 'AgeOfPolicyHolder', 'PoliceReportFiled', 'WitnessPresent', 'AgentType', 'NumberOfSuppliments', 'AddressChange_Claim', 'NumberOfCars', 'Year', 'BasePolicy']
    32

-----

```python
# Increas of columns is expected due to conversion from string to numeric values in the columns
# make new data frame and convert string columns to numeric representations
indexers = [StringIndexer(inputCol=col, outputCol=col+"_index").fit(df) for col in feature_cols]
indexed_df = df
for indexer in indexers:
    indexed_df = indexer.transform(indexed_df)

# Select the indexed feature columns for the vector assembly
indexed_feature_cols = [col+"_index" for col in feature_cols]

# Convert the indexed feature columns to a vector column
assembler = VectorAssembler(inputCols=indexed_feature_cols, outputCol="features")
df_with_features = assembler.transform(indexed_df)
num_cols_df = print(len(df_with_features.columns))

```

    66

------

```python
# Apply PCA for dimensionality reduction
pca = PCA(k=3, inputCol="features", outputCol="pcaFeatures")
pca_model = pca.fit(df_with_features)
reduced_df = pca_model.transform(df_with_features)

# Display the reduced dimensionality DataFrame
reduced_df.show()
num_cols_reduced_df = print(len(reduced_df.columns))

# Stop Spark session
spark.stop()
```

    +-----+-----------+---------+---------+------------+----------------+------------+------------------+------+-------------+---+-------------+--------------------+---------------+---------------+------------+------------+---------+----------+------------+--------------------+-----------------+------------------+------------+-----------------+-----------------+--------------+---------+-------------------+-------------------+------------+----+----------+-----------------+---------------+----------+------------------+----------------------+------------------+------------------------+---------+-------------------+---------+-----------+----------------+---------------------+------------------+------------------+------------------+---------------+----------------+------------------+--------------------------+-----------------------+------------------------+------------------+-----------------------+-----------------------+--------------------+---------------+-------------------------+-------------------------+------------------+----------+----------------+--------------------+--------------------+
    |Month|WeekOfMonth|DayOfWeek|     Make|AccidentArea|DayOfWeekClaimed|MonthClaimed|WeekOfMonthClaimed|   Sex|MaritalStatus|Age|        Fault|          PolicyType|VehicleCategory|   VehiclePrice|FraudFound_P|PolicyNumber|RepNumber|Deductible|DriverRating|Days_Policy_Accident|Days_Policy_Claim|PastNumberOfClaims|AgeOfVehicle|AgeOfPolicyHolder|PoliceReportFiled|WitnessPresent|AgentType|NumberOfSuppliments|AddressChange_Claim|NumberOfCars|Year|BasePolicy|WeekOfMonth_index|DayOfWeek_index|Make_index|AccidentArea_index|DayOfWeekClaimed_index|MonthClaimed_index|WeekOfMonthClaimed_index|Sex_index|MaritalStatus_index|Age_index|Fault_index|PolicyType_index|VehicleCategory_index|VehiclePrice_index|FraudFound_P_index|PolicyNumber_index|RepNumber_index|Deductible_index|DriverRating_index|Days_Policy_Accident_index|Days_Policy_Claim_index|PastNumberOfClaims_index|AgeOfVehicle_index|AgeOfPolicyHolder_index|PoliceReportFiled_index|WitnessPresent_index|AgentType_index|NumberOfSuppliments_index|AddressChange_Claim_index|NumberOfCars_index|Year_index|BasePolicy_index|            features|         pcaFeatures|
    +-----+-----------+---------+---------+------------+----------------+------------+------------------+------+-------------+---+-------------+--------------------+---------------+---------------+------------+------------+---------+----------+------------+--------------------+-----------------+------------------+------------+-----------------+-----------------+--------------+---------+-------------------+-------------------+------------+----+----------+-----------------+---------------+----------+------------------+----------------------+------------------+------------------------+---------+-------------------+---------+-----------+----------------+---------------------+------------------+------------------+------------------+---------------+----------------+------------------+--------------------------+-----------------------+------------------------+------------------+-----------------------+-----------------------+--------------------+---------------+-------------------------+-------------------------+------------------+----------+----------------+--------------------+--------------------+
    |  Dec|          5|Wednesday|    Honda|       Urban|         Tuesday|         Jan|                 1|Female|       Single| 21|Policy Holder|   Sport - Liability|          Sport|more than 69000|           0|           1|       12|       300|           1|        more than 30|     more than 30|              none|     3 years|         26 to 30|               No|            No| External|               none|             1 year|      3 to 4|1994| Liability|              4.0|            4.0|       2.0|               0.0|                   1.0|               0.0|                     2.0|      1.0|                1.0|     41.0|        0.0|             8.0|                  1.0|               2.0|               0.0|               0.0|            5.0|             3.0|               0.0|                       0.0|                    0.0|                     1.0|               6.0|                    4.0|                    0.0|                 0.0|            0.0|                      0.0|                      3.0|               2.0|       0.0|             1.0|(32,[0,1,2,4,6,7,...|[5.70877076561055...|
    |  Jan|          3|Wednesday|    Honda|       Urban|          Monday|         Jan|                 4|  Male|       Single| 34|Policy Holder|   Sport - Collision|          Sport|more than 69000|           0|           2|       15|       400|           4|        more than 30|     more than 30|              none|     6 years|         31 to 35|              Yes|            No| External|               none|          no change|   1 vehicle|1994| Collision|              0.0|            4.0|       2.0|               0.0|                   0.0|               0.0|                     3.0|      0.0|                1.0|      2.0|        0.0|             3.0|                  1.0|               2.0|               0.0|            6532.0|            6.0|             0.0|               3.0|                       0.0|                    0.0|                     1.0|               2.0|                    0.0|                    1.0|                 0.0|            0.0|                      0.0|                      0.0|               0.0|       0.0|             0.0|(32,[1,2,6,8,9,11...|[6532.00007918130...|
    |  Oct|          5|   Friday|    Honda|       Urban|        Thursday|         Nov|                 2|  Male|      Married| 47|Policy Holder|   Sport - Collision|          Sport|more than 69000|           0|           3|        7|       400|           3|        more than 30|     more than 30|                 1|     7 years|         41 to 50|               No|            No| External|               none|          no change|   1 vehicle|1994| Collision|              4.0|            1.0|       2.0|               0.0|                   3.0|               6.0|                     0.0|      0.0|                0.0|     21.0|        0.0|             3.0|                  1.0|               2.0|               0.0|            7643.0|            0.0|             0.0|               1.0|                       0.0|                    0.0|                     2.0|               0.0|                    2.0|                    0.0|                 0.0|            0.0|                      0.0|                      0.0|               0.0|       0.0|             0.0|(32,[0,1,2,4,5,9,...|[7643.00029756792...|
    |  Jun|          2| Saturday|   Toyota|       Rural|          Friday|         Jul|                 1|  Male|      Married| 65|  Third Party|   Sedan - Liability|          Sport| 20000 to 29000|           0|           4|        4|       400|           2|        more than 30|     more than 30|                 1| more than 7|         51 to 65|              Yes|            No| External|        more than 5|          no change|   1 vehicle|1994| Liability|              1.0|            5.0|       1.0|               1.0|                   4.0|               9.0|                     2.0|      0.0|                0.0|     39.0|        1.0|             1.0|                  1.0|               0.0|               0.0|            8754.0|           14.0|             0.0|               2.0|                       0.0|                    0.0|                     2.0|               1.0|                    3.0|                    1.0|                 0.0|            0.0|                      1.0|                      0.0|               0.0|       0.0|             1.0|(32,[0,1,2,3,4,5,...|[8754.00070366543...|
    |  Jan|          5|   Monday|    Honda|       Urban|         Tuesday|         Feb|                 2|Female|       Single| 27|  Third Party|   Sport - Collision|          Sport|more than 69000|           0|           5|        3|       400|           1|        more than 30|     more than 30|              none|     5 years|         31 to 35|               No|            No| External|               none|          no change|   1 vehicle|1994| Collision|              4.0|            0.0|       2.0|               0.0|                   1.0|               5.0|                     0.0|      1.0|                1.0|      8.0|        1.0|             3.0|                  1.0|               2.0|               0.0|            9865.0|            9.0|             0.0|               0.0|                       0.0|                    0.0|                     1.0|               3.0|                    0.0|                    0.0|                 0.0|            0.0|                      0.0|                      0.0|               0.0|       0.0|             0.0|(32,[0,2,4,5,7,8,...|[9865.00021409774...|
    |  Oct|          4|   Friday|    Honda|       Urban|       Wednesday|         Nov|                 1|  Male|       Single| 20|  Third Party|   Sport - Collision|          Sport|more than 69000|           0|           6|       12|       400|           3|        more than 30|     more than 30|              none|     5 years|         21 to 25|               No|            No| External|             3 to 5|          no change|   1 vehicle|1994| Collision|              2.0|            1.0|       2.0|               0.0|                   2.0|               6.0|                     2.0|      0.0|                1.0|     61.0|        1.0|             3.0|                  1.0|               2.0|               0.0|           10976.0|            5.0|             0.0|               1.0|                       0.0|                    0.0|                     1.0|               3.0|                    7.0|                    0.0|                 0.0|            0.0|                      3.0|                      0.0|               0.0|       0.0|             0.0|(32,[0,1,2,4,5,6,...|[10976.0008502726...|
    |  Feb|          1| Saturday|    Honda|       Urban|          Monday|         Feb|                 3|  Male|      Married| 36|  Third Party|   Sport - Collision|          Sport|more than 69000|           0|           7|       14|       400|           1|        more than 30|     more than 30|                 1|     7 years|         36 to 40|               No|            No| External|             1 to 2|          no change|   1 vehicle|1994| Collision|              3.0|            5.0|       2.0|               0.0|                   0.0|               5.0|                     1.0|      0.0|                0.0|     14.0|        1.0|             3.0|                  1.0|               2.0|               0.0|           12087.0|           12.0|             0.0|               0.0|                       0.0|                    0.0|                     2.0|               0.0|                    1.0|                    0.0|                 0.0|            0.0|                      2.0|                      0.0|               0.0|       0.0|             0.0|(32,[0,1,2,5,6,9,...|[12087.0003233404...|
    |  Nov|          1|   Friday|    Honda|       Urban|         Tuesday|         Mar|                 4|  Male|       Single|  0|Policy Holder|   Sport - Collision|          Sport|more than 69000|           0|           8|        1|       400|           4|        more than 30|     more than 30|                 1|         new|         16 to 17|               No|            No| External|               none|          no change|   1 vehicle|1994| Collision|              3.0|            1.0|       2.0|               0.0|                   1.0|               2.0|                     3.0|      0.0|                1.0|     20.0|        0.0|             3.0|                  1.0|               2.0|               0.0|           13198.0|            2.0|             0.0|               3.0|                       0.0|                    0.0|                     2.0|               4.0|                    6.0|                    0.0|                 0.0|            0.0|                      0.0|                      0.0|               0.0|       0.0|             0.0|(32,[0,1,2,4,5,6,...|[13198.0002604316...|
    |  Dec|          4| Saturday|    Honda|       Urban|       Wednesday|         Dec|                 5|  Male|       Single| 30|Policy Holder|   Sport - Collision|          Sport|more than 69000|           0|           9|        7|       400|           4|        more than 30|     more than 30|              none|     6 years|         31 to 35|               No|           Yes| External|             3 to 5|          no change|   1 vehicle|1994| Collision|              2.0|            5.0|       2.0|               0.0|                   2.0|              10.0|                     4.0|      0.0|                1.0|      0.0|        0.0|             3.0|                  1.0|               2.0|               0.0|           14309.0|            0.0|             0.0|               3.0|                       0.0|                    0.0|                     1.0|               2.0|                    0.0|                    0.0|                 1.0|            0.0|                      3.0|                      0.0|               0.0|       0.0|             0.0|(32,[0,1,2,4,5,6,...|[14309.0000473492...|
    |  Apr|          3|  Tuesday|     Ford|       Urban|       Wednesday|         Apr|                 3|  Male|      Married| 42|Policy Holder|Utility - All Perils|        Utility|more than 69000|           0|          10|        7|       400|           1|        more than 30|     more than 30|            2 to 4| more than 7|         36 to 40|               No|            No| External|             3 to 5|          no change|   1 vehicle|1994|All Perils|              0.0|            2.0|       6.0|               0.0|                   2.0|               7.0|                     1.0|      0.0|                0.0|     16.0|        0.0|             4.0|                  2.0|               2.0|               0.0|               1.0|            0.0|             0.0|               0.0|                       0.0|                    0.0|                     0.0|               1.0|                    1.0|                    0.0|                 0.0|            0.0|                      3.0|                      0.0|               0.0|       0.0|             2.0|(32,[1,2,4,5,6,9,...|[1.00027372201522...|
    |  Mar|          2|   Sunday|    Mazda|       Urban|       Wednesday|         Mar|                 3|  Male|       Single| 71|Policy Holder|  Sedan - All Perils|          Sedan|more than 69000|           0|          11|        7|       400|           3|        more than 30|     more than 30|              none| more than 7|          over 65|               No|            No| External|               none|          no change|   1 vehicle|1994|All Perils|              1.0|            6.0|       3.0|               0.0|                   2.0|               2.0|                     1.0|      0.0|                1.0|     50.0|        0.0|             2.0|                  0.0|               2.0|               0.0|            1112.0|            0.0|             0.0|               1.0|                       0.0|                    0.0|                     1.0|               1.0|                    5.0|                    0.0|                 0.0|            0.0|                      0.0|                      0.0|               0.0|       0.0|             2.0|(32,[0,1,2,4,5,6,...|[1112.00065166070...|
    |  Mar|          5|   Monday|    Honda|       Urban|          Monday|         Mar|                 5|  Male|      Married| 52|Policy Holder|   Sedan - Liability|          Sport| 20000 to 29000|           0|          12|       13|       400|           1|        more than 30|     more than 30|            2 to 4| more than 7|         41 to 50|               No|            No| External|               none|          no change|   1 vehicle|1994| Liability|              4.0|            0.0|       2.0|               0.0|                   0.0|               2.0|                     4.0|      0.0|                0.0|     28.0|        0.0|             1.0|                  1.0|               0.0|               0.0|            2223.0|           15.0|             0.0|               0.0|                       0.0|                    0.0|                     0.0|               1.0|                    2.0|                    0.0|                 0.0|            0.0|                      0.0|                      0.0|               0.0|       0.0|             1.0|(32,[0,2,5,6,9,11...|[2223.00052003853...|
    |  Jan|          3|   Friday|     Ford|       Urban|          Friday|         Jan|                 3|  Male|      Married| 28|Policy Holder|   Sedan - Liability|          Sport|more than 69000|           0|          13|       11|       400|           1|        more than 30|     more than 30|                 1|     7 years|         31 to 35|               No|            No| External|               none|          no change|   1 vehicle|1994| Liability|              0.0|            1.0|       6.0|               0.0|                   4.0|               0.0|                     1.0|      0.0|                0.0|      4.0|        0.0|             1.0|                  1.0|               2.0|               0.0|            3334.0|           10.0|             0.0|               0.0|                       0.0|                    0.0|                     2.0|               0.0|                    0.0|                    0.0|                 0.0|            0.0|                      0.0|                      0.0|               0.0|       0.0|             1.0|(32,[1,2,4,6,9,11...|[3334.00015843199...|
    |  Jan|          5|   Friday|    Honda|       Rural|       Wednesday|         Feb|                 1|  Male|       Single|  0|  Third Party|   Sedan - Collision|          Sedan|more than 69000|           0|          14|       12|       400|           3|        more than 30|     more than 30|              none|         new|         16 to 17|               No|            No| External|               none|          no change|   1 vehicle|1994| Collision|              4.0|            1.0|       2.0|               1.0|                   2.0|               5.0|                     2.0|      0.0|                1.0|     20.0|        1.0|             0.0|                  0.0|               2.0|               0.0|            4445.0|            5.0|             0.0|               1.0|                       0.0|                    0.0|                     1.0|               4.0|                    6.0|                    0.0|                 0.0|            0.0|                      0.0|                      0.0|               0.0|       0.0|             0.0|(32,[0,1,2,3,4,5,...|[4445.00032948971...|
    |  Jan|          5|   Monday|     Ford|       Urban|        Thursday|         Feb|                 1|  Male|      Married| 61|Policy Holder|   Sedan - Liability|          Sport|more than 69000|           0|          15|        3|       400|           1|        more than 30|     more than 30|              none| more than 7|         51 to 65|               No|            No| External|               none|          no change|   1 vehicle|1994| Liability|              4.0|            0.0|       6.0|               0.0|                   3.0|               5.0|                     2.0|      0.0|                0.0|     34.0|        0.0|             1.0|                  1.0|               2.0|               0.0|            5556.0|            9.0|             0.0|               0.0|                       0.0|                    0.0|                     1.0|               1.0|                    3.0|                    0.0|                 0.0|            0.0|                      0.0|                      0.0|               0.0|       0.0|             1.0|(32,[0,2,4,5,6,9,...|[5556.00055404215...|
    |  Aug|          4|  Tuesday|     Ford|       Urban|          Monday|         Aug|                 5|  Male|       Single| 38|Policy Holder|   Sedan - Liability|          Sport|more than 69000|           0|          16|       16|       400|           1|        more than 30|     more than 30|              none|     6 years|         36 to 40|               No|            No| External|               none|          no change|   1 vehicle|1994| Liability|              2.0|            2.0|       6.0|               0.0|                   0.0|              11.0|                     4.0|      0.0|                1.0|     18.0|        0.0|             1.0|                  1.0|               2.0|               0.0|            6088.0|            7.0|             0.0|               0.0|                       0.0|                    0.0|                     1.0|               2.0|                    1.0|                    0.0|                 0.0|            0.0|                      0.0|                      0.0|               0.0|       0.0|             1.0|(32,[0,1,2,5,6,8,...|[6088.00038258216...|
    |  Apr|          4| Thursday|     Ford|       Urban|       Wednesday|         May|                 1|  Male|      Married| 41|Policy Holder|  Sedan - All Perils|          Sedan|more than 69000|           0|          17|       15|       400|           4|        more than 30|     more than 30|              none|     7 years|         36 to 40|               No|            No| External|               none|          no change|   1 vehicle|1994|All Perils|              2.0|            3.0|       6.0|               0.0|                   2.0|               1.0|                     2.0|      0.0|                0.0|     11.0|        0.0|             2.0|                  0.0|               2.0|               0.0|            6199.0|            6.0|             0.0|               3.0|                       0.0|                    0.0|                     1.0|               0.0|                    1.0|                    0.0|                 0.0|            0.0|                      0.0|                      0.0|               0.0|       0.0|             2.0|(32,[0,1,2,4,5,6,...|[6199.00020762991...|
    |  Jul|          5|   Sunday|Chevrolet|       Urban|       Wednesday|         Aug|                 1|Female|      Married| 28|  Third Party|   Sedan - Collision|          Sedan| 20000 to 29000|           0|          18|        6|       400|           1|        more than 30|     more than 30|              none|     7 years|         31 to 35|               No|            No| External|             1 to 2|          no change|   1 vehicle|1994| Collision|              4.0|            6.0|       4.0|               0.0|                   2.0|              11.0|                     2.0|      1.0|                0.0|      4.0|        1.0|             0.0|                  0.0|               0.0|               0.0|            6310.0|           11.0|             0.0|               0.0|                       0.0|                    0.0|                     1.0|               0.0|                    0.0|                    0.0|                 0.0|            0.0|                      2.0|                      0.0|               0.0|       0.0|             0.0|(32,[0,1,2,4,5,6,...|[6310.00024240174...|
    |  May|          4| Thursday|  Pontiac|       Urban|          Monday|         May|                 5|  Male|       Single| 32|Policy Holder|   Sedan - Liability|          Sport| 20000 to 29000|           0|          19|        6|       400|           1|        more than 30|     more than 30|                 1|     7 years|         31 to 35|               No|            No| External|               none|          no change|   1 vehicle|1994| Liability|              2.0|            3.0|       0.0|               0.0|                   0.0|               1.0|                     4.0|      0.0|                1.0|      7.0|        0.0|             1.0|                  1.0|               0.0|               0.0|            6421.0|           11.0|             0.0|               0.0|                       0.0|                    0.0|                     2.0|               0.0|                    0.0|                    0.0|                 0.0|            0.0|                      0.0|                      0.0|               0.0|       0.0|             1.0|(32,[0,1,5,6,8,9,...|[6421.00019536511...|
    |  Apr|          4|   Monday|    Honda|       Urban|         Tuesday|         May|                 1|  Male|      Married| 30|  Third Party|   Sedan - Liability|          Sport|more than 69000|           0|          20|        2|       400|           2|        more than 30|     more than 30|            2 to 4|     6 years|         31 to 35|               No|            No| External|        more than 5|          no change|   1 vehicle|1994| Liability|              2.0|            0.0|       2.0|               0.0|                   1.0|               1.0|                     2.0|      0.0|                0.0|      0.0|        1.0|             1.0|                  1.0|               2.0|               0.0|            6533.0|            8.0|             0.0|               2.0|                       0.0|                    0.0|                     0.0|               2.0|                    0.0|                    0.0|                 0.0|            0.0|                      1.0|                      0.0|               0.0|       0.0|             1.0|(32,[0,2,4,5,6,10...|[6533.00007801976...|
    +-----+-----------+---------+---------+------------+----------------+------------+------------------+------+-------------+---+-------------+--------------------+---------------+---------------+------------+------------+---------+----------+------------+--------------------+-----------------+------------------+------------+-----------------+-----------------+--------------+---------+-------------------+-------------------+------------+----+----------+-----------------+---------------+----------+------------------+----------------------+------------------+------------------------+---------+-------------------+---------+-----------+----------------+---------------------+------------------+------------------+------------------+---------------+----------------+------------------+--------------------------+-----------------------+------------------------+------------------+-----------------------+-----------------------+--------------------+---------------+-------------------------+-------------------------+------------------+----------+----------------+--------------------+--------------------+
    only showing top 20 rows
    
    67



```python

```


```python

```


```python

```
