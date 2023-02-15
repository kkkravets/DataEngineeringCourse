## Question 1
Queries for creating an external table using the fhv 2019 data and then counting vehicle records for 2019:
```sql
CREATE OR REPLACE EXTERNAL TABLE `dataops-376306.trips_data_all.fhv_2019`
OPTIONS (
  format = 'CSV',
  uris = ['gs://dtc_data_lake_dataops-376306/week_3/fhv_tripdata_2019-*.csv.gz']
);
SELECT COUNT(*) FROM `dataops-376306.trips_data_all.fhv_2019`;
```

## Question 2

Query to create Materialized BQ table from External table

```sql
CREATE OR REPLACE TABLE `dataops-376306.trips_data_all.bq_fhv_2019` AS
SELECT * FROM `dataops-376306.trips_data_all.fhv_2019`;
```
Queries for External and Materialized BQ tables to count the distinct number of affiliated_base_number (and compare amounts of processed data):
``` sql
SELECT DISTINCT COUNT(Affiliated_base_number) FROM `dataops-376306.trips_data_all.fhv_2019` 
```
``` sql
SELECT DISTINCT COUNT(Affiliated_base_number) FROM `dataops-376306.trips_data_all.bq_fhv_2019` 
```
## Question 3
Query to count how many records have both a blank (null) PUlocationID and DOlocationID in the entire dataset?

```sql
SELECT COUNT(*)
FROM `dataops-376306.trips_data_all.bq_fhv_2019` 
WHERE PUlocationID IS NULL AND DOlocationID IS NULL
```

## Questions 4 and 5

*What is the best strategy to optimize the table if query always filter by pickup_datetime and order by affiliated_base_number?*

Partition the table by pickup_datetime: partitioning allows BigQuery to scan only the partitions that contain relevant data, which reduces the amount of data scanned and improves query performance. Partitioning on pickup_datetime ensures that only the partitions containing data for a particular time range will be scanned, based on the query's filter criteria.
Cluster the table by affiliated_base_number: clustering organizes data within each partition based on a clustering column, which allows BigQuery to skip over irrelevant data during query processing. Clustering on affiliated_base_number will group together rows with the same affiliated_base_number, which means that BigQuery can read fewer data blocks when processing a query that sorts by this column.

Query to create such table:

```sql
CREATE OR REPLACE TABLE `dataops-376306.trips_data_all.bq_fhv_2019_part_clust`
PARTITION BY DATE(`pickup_datetime`)
CLUSTER BY `Affiliated_base_number` AS
SELECT * FROM `dataops-376306.trips_data_all.bq_fhv_2019`
```

*Now we need to write a query to retrieve the distinct affiliated_base_number between pickup_datetime 2019/03/01 and 2019/03/31 (inclusive)*.
Query for ordinary BQ table:

``` sql
SELECT DISTINCT `Affiliated_base_number`
FROM `dataops-376306.trips_data_all.bq_fhv_2019`
WHERE `pickup_datetime` BETWEEN '2019-03-01'AND '2019-03-31'
```
This gives around 647MB processed bytes

Query for partitioned table:

``` sql
SELECT DISTINCT `Affiliated_base_number`
FROM `dataops-376306.trips_data_all.bq_fhv_2019_part_clust`
WHERE `pickup_datetime` BETWEEN '2019-03-01'AND '2019-03-31'
```
This gives around 24MB processed bytes