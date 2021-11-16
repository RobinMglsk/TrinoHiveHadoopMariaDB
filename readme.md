# A Hive / Hadoop / Mariadb / Trino sandbox
A docker-compose version of the sandbox that is used for https://www.linkedin.com/learning/presto-essentials-data-science.

The Hive / Hadoop part is based on: https://hshirodkar.medium.com/apache-hive-on-docker-4d7280ac6f8e. The Trino part is just the docker image with a custom hive and mysql catalog connector.

## Basics
### Start everything
```sh
docker-compose up
```
### Load test db data
Login into the hive server.
```sh
docker exec -it hive-server /bin/bash
```
Create the test database
```sh
cd /employee
hive -f employee_table.hql
```

Import the test data from a csv file.
```sh
hadoop fs -put employee.csv hdfs://namenode:8020/user/hive/warehouse/testdb/db/employee 
```
### Verify data
Login to Trino server and start trino-cli
```sh
docker exec -t trino /bin/bash
trino
```

Check catalogs
```sql
show catalogs;
```

Check schemas
```sql
show schemas from hive;
```

Check tables
```sql
show tables from hive.testdb;
```

Check employee data we just imported
```sql
select * from hive.testdb.employees;
```

## Using Trino's TPCH connector to create and seed tables
You can create a new table on in your mysql db by doing:
```sql
create table mysql.test.customer as select * from tpch.tiny.customer limit 100;
```
Or do the same but to a hive data store:
```sql
create table hive.testdb.customer as select * from tpch.tiny.customer limit 100;
```
This wil use the TPCH catalog (which generates dummy data) and copy 100 rows into the recipient.

## Exporting data
By running following command you can run a query and export the result to a csv file.
```sh
trino --execute "select * from mysql.test.customer" --output-format CSV > output.csv
```
Other output formats are: `ALIGNED`, `VERTICAL`, `TSV`, `TSV_HEADER`, `CSV`, `CSV_HEADER`, `CSV_UNQUOTED`, `CSV_HEADER_UNQUOTED`, `JSON`, `NULL`