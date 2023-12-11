# Configuration

The configuration is organized into six sections:

* **common** - Settings that can be utilized for both the `dump` and `restore` commands.
* **log** - Configuration for the logging subsystem.
* **storage** - Configuration for the storage locations where dumps are stored
* **dump** - Configuration for the dump command. This section includes pg_dump options and transformation settings.
* **restore** - Configuration for the restore command. It contains pg_restore options and additional restoration
  scripts.
* **custom_transformers** - Definitions of custom transformers that interact through stdin and stdout. Once a custom
  transformer is configured, it becomes accessible via the `greenmask list-transformers` command.

## common

* **pg_bin_path** - Path to the PostgreSQL binaries. Note that the PostgreSQL server version must match the provided
  binaries.
* **tmp_dir** - Temporary directory for storing the TOC (Table of Contents) fi

!!! note

    Greenmask exclusively manages data dumping and data restoration processes, delegating schema dumping to the 
    `pg_dump `utility and schema restoration to the `pg_restore` utility. Both `pg_dump` and `pg_restore` rely on a 
    toc.dat file located in a specific directory, which contains metadata and object definitions. Therefore, 
    the `tmp_dir` parameter is essential for storing the toc.dat file during the dumping or restoration procedure. It's 
    important to note that all artifacts in this directory will be automatically deleted once the Greenmask 
    command is completed.

## log

The log section of the configuration allows you to specify the following settings:

* **level** - Specifies the level of logging, which can be one of the following: debug, info, or error. The default
  level is info.
* **format** - Defines the logging format, which can be either json or text. The default format is text.

## storage

The storage section enables you to configure the storage driver for storing the dumped data. Currently,
two storage options are supported: **directory** and **s3**.

### directory

Directory storage refers to a filesystem directory where the dump data will be stored.

Parameters:

* **path** - Specifies the path to the directory where the dumps will be stored on the filesystem.

``` yaml title="direcroty storage config example"
directory:
    path: "/home/user_name/storage_dir" # (1)
```

### s3

The S3 storage configuration in Greenmask allows you to store dump data in an S3-like remote storage service,
such as Amazon S3 or Azure Blob Storage. Here are the parameters you can configure for S3 storage:

* **endpoint** - Override the default AWS endpoint to a custom one for making requests.
* **bucket** - The name of the bucket where the dump data will be stored.
* **prefix** - A prefix for objects in the bucket, specified in path format.
* **region** - The region of the S3 service.
* **storage_class** - The storage class for performing object requests.
* **disable_ssl** - Disable SSL for interactions (default is false).
* **access_key_id** - Access key for authentication.
* **secret_access_key** - Secret access key for authentication.
* **session_token** - Session token for authentication.
* **role_arn** - Amazon resource name for role-based authentication.
* **session_name** - Role session name to uniquely identify a session.
* **max_retries** - The number of retries on request failures.
* **cert_file** - Path to the SSL certificate for making requests.
* **max_part_size** - Maximum part length for one request.
* **concurrency** - The number of goroutines to use in parallel for each Upload call when sending parts.
* **use_list_objects_v1** - Use the old v1 ListObjects request instead of v2.
* **force_path_style** - Force the request to use path-style addressing (e.g., `http://s3.amazonaws.com/BUCKET/KEY`)
  instead of virtual hosted bucket addressing (e.g., `http://BUCKET.s3.amazonaws.com/KEY`).
* **use_accelerate** - Enable S3 Accelerate feature.

``` yaml title="s3 storage config eample for Minio runned in the Docker"
s3:
  endpoint: "http://localhost:9000"
  bucket: "testbucket"
  region: "us-east-1"
  access_key_id: "Q3AM3UQ867SPQQA43P2F"
  secret_access_key: "zuf+tfteSlswRu7BJ86wekitnifILbZam1KYY3TG"
```

## dump

The dump command section in the Greenmask configuration file is used to configure parameters for the `greenmask dump`
command. It includes the following parameters:

* **pg_dump_options** - A map of pg_dump options. These options are used for configuring the behavior of the
  underlying [pg_dump command](commands.md#dump). You can refer to the list of supported pg_dump options in the
  Greenmask dump command documentation.
* **transformation** - This section contains configuration for applying transformations to table columns during the
  dump operation. It includes the following sub-parameters:

    * **schema** - The schema name of the table.
    * **name** - The name of the table.
    * **query** - An optional parameter for specifying a custom query to be used in the COPY command. By default,
      the entire table is dumped, but you can use this parameter to set a custom query.

        !!! warning

            Be cautious when using the query parameter, as it may lead to constraint violation errors during 
            restoration, and Greenmask currently cannot handle query validation.

    * **columns_type_override** - Allows you to override the column types explicitly. You can associate a column
      with another type that is supported by your transformer. This is useful when the transformer works strictly with
      specific types of columns. For example, if a column named `post_code` is of type TEXT, but the RandomInt
      transformer works only with INT family types, you can override it as shown in the example provided.
      ``` yaml title="column type overriden example"
        columns_type_override:
          post_code: "int4"  # (1)
      ```
      { .annotate }

           1. Change the data type of the post_code column to INT4 (INTEGER).

    * **apply_for_inherited** - An optional parameter to apply the same transformation to all partitions if the table
      is partitioned. This can save you from defining the transformation for each partition manually.

        !!! warning

            It is recommended to use the `--load-via-partition-root` parameter when dealing with partitioned tables, 
            as the partition key value might change.

    * **transformers** - A list of transformers to apply to the table, along with their parameters. Each transformation
      item includes the following sub-parameters:

        * **name** - The name of the transformer.
        * **params** - A map of the provided transformer parameters.

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

Here is an example configuration for the **dump** section:

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

1. Override the **post_code** column type to **int4 (INTEGER)**. This is necessary because the **post_code** column
   originally has a TEXT type, but it contains values that resemble integers. By explicitly overriding the type
   to **int4**, we ensure compatibility with transformers that work with integer types, such as **RandomInt**.
2. After the type is overridden we can apply a compatible transformer

## validate

The **validate** section in the configuration file is used to specify parameters for the `greenmask validate`
command. Here is an example of the validate section configuration:

```yaml title="example of validate section config"
validate:
  tables: # (1)
    - "orders"
    - "public.cart"
  data: true # (2)
  diff: true # (3)
  rows_limit: 10 # (4)
  resolved_warnings: # (5)
    - "8d436fae67b2b82b36bd3afeb0c93f30"
  format: "horizontal" # (6)
```
{ .annotate }

1. A list of tables to validate. If this list is not empty, the validation operation will only be performed for
   the specified tables. Tables can be written with or without the schema name (e.g., **"public.cart"** or
   **"orders"**).
2. Specifies whether to perform data transformation for a limited set of rows. If set to true, data transformation
   will be performed, and the number of rows transformed will be limited to the value specified in the `rows_limit`
   parameter (default is 10).
3. Specifies whether to perform diff operations for the transformed data. If set to `true`, the validation process
   will **find the differences between the original and transformed data**. More details can be found in the validate
   [command documentation](commands.md/#validate).
4. Limits the number of rows to be transformed during validation. The default limit is `10` rows, but you can change
   it by modifying this parameter.
5. A hash list of resolved warnings. These warnings have been addressed and resolved in a previous validation run.
6. Specifies the format of the transformation output. Possible values are `[horizontal|vertical]`. The default format
   is `horizontal`. You can choose the format that suits your needs. More details can be found in the [validate
   command documentation](commands.md/#validate).

## restore

The *restore* section in the configuration file is used to specify parameters for the `greenmask restore` command.
It contains `pg_restore` settings and custom script execution settings. Here are the available parameters:

Parameters:

* **pg_restore_options** - A map of pg_restore options. These options are used to configure the behavior of
  the pg_restore utility during the restoration process. You can refer to the list of supported pg_restore options
  in the [greenmask restore command documentation]().
* **scripts** - A map of custom scripts to be executed during different restoration stages. Each script is
  associated with a specific restoration stage and includes the following attributes:
    * **[pre-data|data|post-data]** - The name of the restoration stage when the script should be executed.
        * **name** - The name of the script.
        * **when** - Specifies when to execute the script, which can be either **"before"** or **"after"** the
          specified restoration stage.
        * **query** - An SQL query string to be executed.
        * **query_file** - The path to an SQL query file to be executed.
        * **command** - A command with parameters to be executed. It is provided as a list, where the first item is the
          command name.

As mentioned in [the architecture](architecture.md/#backing-up), a backup contains three sections: pre-data, data,
and post-data. The custom script execution allows you to customize and control the restoration process by executing
scripts or commands at specific stages. The available restoration stages and their corresponding execution conditions
are as follows:

* **pre-data** - Scripts or commands can be executed before or after restoring the pre-data section.
* **data** - Scripts or commands can be executed before or after restoring the data section.
* **post-data** - Scripts or commands can be executed before or after restoring the post-data section.

Each stage can have a **"when"** condition with possible values:

* **before** - Execute the script or SQL command before the mentioned restoration stage.
* **after** - Execute the script or SQL command after the mentioned restoration stage.

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

1. **List of pre-data stage scripts:** This section contains scripts that are executed before or after the restoration of 
   the pre-data section. The scripts include SQL queries and query files.
2. **List of data stage scripts**: This section contains scripts that are executed before or after the restoration of the 
   data section. The scripts include shell commands with parameters and SQL query files.
3. **List of post-data stage scripts**: This section contains scripts that are executed before or after the restoration 
   of the post-data section. The scripts include SQL queries and query files.
4. **Command in the first argument and the parameters in the rest of the list**: When specifying a command to be 
   executed in the scripts section, you provide the command name as the first item in a list, followed by any 
   parameters or arguments for that command. The command and its parameters are provided as a list within the 
   script configuration.
