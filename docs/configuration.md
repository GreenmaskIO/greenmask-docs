# Configuration

The configuration is divided into 6 sections:

* **common** - setting that can be used either for `dump` or `restore` command
* **log** - setting for logging subsystem
* **storage** - setting for storages where dumps is going to be stored. Currently available - S3 and Directory
* **dump** - setting for dump command. Contains pg_dump options and transformation
* **restore** - setting for restore option. Contains pg_restore opting and additional restoration scripts
* **custom_transformers** - definition of custom transformers that interact via (stdin-stdout). Once the custom
  transformer has been connected it will be also available via `greenmask list-transformers`

## common

* **pg_bin_path** - path to the PostgreSQL binaries. Note - the PostgreSQL server must be the same version as a
  provided binaries
* **tmp_dir** - temporal directory for storing TOC file.

!!! note

    Greenmask controls only data dumping and data restoration and delegates schema dumping to pg_dump util and schema 
    restoration to pg_restore util. Both pg_dump and pg_restore require toc.dat file in some directory that contains 
    metadata and object definition. According to that tmp_dir parameter is required for storing `toc.dat` file during 
    the dumping or restoration procedure. All the artifacts will be deleted once the command is completed.

## log

* **level** - level of logging. Can be one of - debug, info, or error. Default is info.
* **format** - logging format. Can be one of - json or text. The default is text.

## storage

Setup storage driver for storing the dumped data. Currently supported storages - **directory** and **S3**.

### directory

Filesystem directory where dump will be stored.

Parameters:

* **path** - path to the directory where dumps is going to be stored

``` yaml title="direcroty storage config example"
directory:
    path: "/home/user_name/storage_dir" # (1)
```

### s3

S3-like remote storage. It might be azure, amazon S3, and so on.

Parameters:

* **endpoint** - override default AWS endpoint to custom for using it in requests
* **bucket** - the name of the bucket where dump is going to be stored
* **prefix** - prefix for objects in the bucket in path format
* **region** - region of the service
* **storage_class** - storage class for performing objects requests
* **disable_ssl** - disable SSL in the interaction - default false
* **access_key_id** - access key
* **secret_access_key** - secret access key
* **session_token** - session token
* **role_arn** - amazon resources name
* **session_name** - role session name to uniquely identify a session when the same role is assumed by different
  principals or for different reasons.
* **max_retries** - count of retries on the failure of the request
* **cert_file** - path to the SSL cert for performing the requests
* **max_part_size** - max part length for one request
* **concurrency** - the number of goroutines to spin up in parallel per call to Upload when sending parts
* **use_list_objects_v1** - Use old v1 ListObjects request instead of v2
* **force_path_style** - force the request to use path-style addressing, i.e., `http://s3.amazonaws.com/BUCKET/KEY`.
  By default, the S3 client will use virtual hosted bucket addressing when
  possible (`http://BUCKET.s3.amazonaws.com/KEY`).
* **use_accelerate** - enable S3 Accelerate feature

``` yaml title="s3 storage config eample for Minio runned in the Docker"
s3:
  endpoint: "http://localhost:9000"
  bucket: "testbucket"
  region: "us-east-1"
  access_key_id: "Q3AM3UQ867SPQQA43P2F"
  secret_access_key: "zuf+tfteSlswRu7BJ86wekitnifILbZam1KYY3TG"
```

!!! warning

    Greenmask is beta in early access and cannot guarantee that all the S3-like storages will be working properly. 
    Please create a feature request for your storage if you think it's popular and worthy. If any bug or lack of 
    setting will be found do not hesitate to create an Issue in GitHub

## dump

Section for `greenmask dump` command. Contains pg_dump parameters and transformation config.

Parameters:

* **pg_dump_options** - map of pg_dump options. See list of the supported pg_dump option
  in [greenmask dump command](commands.md#dump)
* **transformation** - contains tables and a list of transformers that apply a transformation on the column value:

    * **schema** - table schema name
    * **name** - table name
    * **query** - overridden query for COPY command. By default, the whole table is dumped. Use this parameter if
      you want to set a custom query.

        !!! warning

            Be careful with **query**  it may cause constraint violation errors during restoration. 
            Greenmask currently cannot handle a query validation

    * **columns_type_override** - Column type override. An associate  column with another type that supports your
      transformer. Most of the greenmask
      transformers work strictly with listed types of columns. If you know that column value contains laterally the
      same
      value as the transformer-supported type, then you can explicitly override it. For instance, post_code is a TEXT
      value, but `RandomInt` transformer works only with INT family types, then you can override it
      ``` yaml title="column type overriden example"
        columns_type_override:
          post_code: "int4"  # (1)
      ```
      { .annotate }

           1. override post_code column type to int4 (INTEGER)
  
    * **apply_for_inherited** - apply for the same transformation for all partitions if the table is partitioned.
      In this case, you do not need to define transformation for each partition manually.

        !!! warning

            It is recommented to use `load-via-partition-root` parameter because partition key value might be changed.

    * **transformers** - list of transformers on the table with provided parameters:
      ``` yaml title="transformers config example"
         transformers:
          - name: "RandomDate"
            params:
              min: "2023-01-01 00:00:00.0+03"
              max: "2023-01-02 00:00:00.0+03"
              column: "scheduled_departure"

          - name: "NoiseDate"
            params:
              ratio: "01:00:00"
              column: "scheduled_arrival"
      ```
      Each transformation item contains:
        * **name** - transformer name
        * **parameters** - map of the provided transformer parameter

``` yaml title="example dump section config"
dump:
  pg_dump_options: 
    dbname: "host=/run/postgresql user=postgres dbname=demo"
    jobs: 10
    exclude-schema: "(\"teSt\"*|test*)"
    table: "bookings.flights"
    load-via-partition-root: true

  transformation:
    - schema: "bookings"
      name: "flights"
      query: "select * from bookings.flights3 limit 1000000" 
      columns_type_override:
        post_code: "int4" # (1)
      transformers:
        - name: "RandomDate"
          params:
            min: "2023-01-01 00:00:00.0+03"
            max: "2023-01-02 00:00:00.0+03"
            column: "scheduled_departure"

        - name: "NoiseDate"
          params:
            ratio: "01:00:00"
            column: "scheduled_arrival"

        - name: "RegexpReplace"
          params:
            column: "status"
            regexp: "On Time"
            replace: "Delayed"

        - name: "RandomInt" # (2)
          params:
            column: "post_code"
            min: "11"
            max: "99"

    - schema: "bookings"
      name: "aircrafts_data"
      transformers:
        - name: "Json"
          params:
            column: "model"
            operations:
              - operation: "set"
                path: "en"
                value: "Boeing 777-300-2023"
              - operation: "set"
                path: "crewSize"
                value: 10

        - name: "NoiseInt"
          params:
            ratio: 0.9
            column: "range"
```
{ .annotate }

1. override **post_code** column type to int4 (INTEGER). Since **post_code** column has a **TEXT** type but contains
   int like value we would override the type explicitly and apply a transformer that works with INT, for instance
   **RandomInt**
2. After the type is overridden we can apply a compatible transformer

## validate

Section for `greenmask validate` command

```yaml title="example of validate section config"
validate:
  tables: # (1)
    - "orders"
    - "public.cart"
  data: true # (2)
  diff: true # (3)
  rows_limit: 10 # (4)
  resolved_warnings:  # (5)
    - "8d436fae67b2b82b36bd3afeb0c93f30"
  format: "horizontal" # (6)
```
{ .annotate }

1. list of tables to validate. If not empty only for listed tables the validate operation will be performed. The table
   might be written (`public.cart`) with schema or without (`orders`)
2. perform data transformation for the limited rows set. The limit by default is 10 but may be changed via `rows_limit`
   parameter
3. perform diff operations for the transformed data. See details in [validate command](commands.md/#validate)
4. limit the rows for transformation. The default is `10`
5. hash list of the resolved warnings
6. format of transformation output. Possible values `[horizontal|vertical]` Default is `horizontal`. See details
   in [validate command](commands.md/#validate)


## restore

Restoration config section. Contains pg_restore setting and custom scripts execution

Parameters:

* **pg_restore_options** - map of pg_dump options. See list of the supported pg_dump option
  in [greenmask dump command](commands.md/#restore)
* **scripts** - map of scripts executed one by one in assigned restoration stage
  * **[pre-data|data|post-data]** - name of the stage when execute
    * **name** - of the script
    * **when** - condition when execute the command. Might be **before** or **after**
    * **query** - SQL query string
    * **query_file** - SQL query file path
    * **command** - command with parameters. Provided as a list where the first item is command name

As was described in [Architecture](architecture.md/#backing-up) a backup contains three section - pre-data, data and
post-data. For customization purposes and controlling the restoration process Greenmask introduce custom script
execution that might be applied in some stages.

Restoration stages list when SQL script or command might be executed:

* **pre-data** - execute before or after restoring pre-data section
* **data** - execute before or after restoring data section
* **post-data** - execute before or after restoring post-data section

Each stage has a when condition with possible values:

* **before** - execute script/sql-command before mentioned stage
* **after** - execute script/sql-command after mentioned stage

``` yaml title="scripts definition example"
scripts:
  pre-data: # (1)
    - name: "pre-data before script [1] with query"
      when: "before"
      query: "create table script_test(stage text)"
    - name: "pre-data before script [2]"
      when: "before"
      query: "insert into script_test values('pre-data before')"
    - name: "pre-data after test script [1]"
      when: "after"
      query: "insert into script_test values('pre-data after')"
    - name: "pre-data after script with query_file [1]"
      when: "after"
      query_file: "pre-data-after.sql"
  data: # (2)
    - name: "data before script with command [1]"
      when: "before"
      command: # (4)
        - "data-after.sh"
        - "param1"
        - "param2"
    - name: "data after script [1]"
      when: "after"
      query_file: "data-after.sql"
  post-data: # (3)
    - name: "post-data before script [1]"
      when: "before"
      query: "insert into script_test values('post-data before')"
    - name: "post-data after script with query_file [1]"
      when: "after"
      query_file: "post-data-after.sql"
```
{ .annotate }

1. list of pre-data stage scripts
2. list of data stage scripts
3. list of post-data stage scripts
4. command in the first argument and the parameters in the rest of the list
