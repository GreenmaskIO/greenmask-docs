### Description

Transform data via external program using stdin and stdout interaction.

| Name               | Description                                                                                                                                                                                                                                       | Default           | Required | Supported DB types |
|--------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-------------------|----------|--------------------|
| columns            | affected column names. If empty use the whole tuple. Read about structure below                                                                                                                                                                   |                   | Yes      | any                |
| executable         | path to executable file                                                                                                                                                                                                                           |                   | Yes      | -                  |
| args               | list of parameters for executable                                                                                                                                                                                                                 |                   | No       | -                  |
| driver             | row driver with parameters that is used for interacting with cmd. The default is csv. `{"name": "text\| csv\| json", "params": { "format": "[text\|bytes]"}}`                                                                                     | `{"name": "csv"}` | No       | -                  |
| validate           | try to encode-decode data received from cmd in order to ensure the data format is correct                                                                                                                                                         | `false`           | No       | -                  |
| timeout            | timeout for sending and receiving data from cmd                                                                                                                                                                                                   | `2s`              | No       | -                  |
| expected_exit_code | expected exit code on SGTERM signal. If the exit code is unexpected then the greenmask exits with error                                                                                                                                           | `0`               | No       | -                  |
| skip_on_behaviour  | skip transformation call if one of the provided columns has a null value (_any_) or each of the provided column has null values (_all_). This option works together with `skip_on_null_input` parameter on columns. Possible values: _all_, _any_ | `all`             | No       | -                  |

Type of modes:

* **text** - textual format. Each value is represented by the string unicode literal with new line. The tuple
  delimiter is a new line symbol (`\n`)
* **json** - json line interaction. It has two formats `[text|bytes]` that can be passed through `driver.params`
    * **bytes** - use this format if the typs is binary
      ``` json
        {
          1: {
            "d": "aGVsbG8gd29ybHNeODcxMjE5MCUlJSUlJQ==",
            "n": false,
          },
          2: {
            "d": "aGVsbG8gd29ybHNeODcxMjE5MCUlJSUlJQ==",
            "n": false,
          }
        }
      ```
      where:

        * each line is the json line with the map of attribute numbers to their value.
        * d - the raw data base64 encoded. The base64 encoding is needed because it may be binary data
        * n - is the value NULL or not

    * **text** - use this format if the types are not binary and might be represented by string literal
      ``` json
        {
          1: {
            "d": "some_value1",
            "n": false,
          },
          2: {
            "d": "some_value2",
            "n": false,
          }
        }
      ```
      where:

        * each line is the json line with the map of attribute numbers to their value.
        * d - the raw data is represented in unicode text.
        * n - is the value NULL or not

* **csv** - csv format (coma separated). The count of attributes are the same as the table attribute count, but the columns
  that was not mentioned in `columns` list are empty. The NULL value represents by `\N` value. Each attribute is
  escaped by quote (`"`). Imagine transformed table has attributes id, title and created_at and we want to transform
  only id and created_at then the csv line is (where the second attribute is empty)
  ``` csv title="csv line example"
  "123","","2023-01-03 01:00:00.0+03"
  ```


Column object attributes:

* **name** - name of the column. The value is required. Depending on the attributes below this column may be used just as a value and is not
  affected somehow.
* **not_affected** - shows is column affected in transformation. This is required for the validation procedure when
  greenmask called with `greenmask dump --validate`. The `not_affected=true` might be helpful is the command transformer
  transforms the data depending on the value of some another column. For instance, you might want to generate
  `updated_at` column value depending on `created_at` column value, so in this case `created_at` is `not_affected=true`.
  The default value is `not_affected=false`
* **skip_original_data** - shows is the original data is required for transformer. This might be helpful for decreasing the
  interaction time. One of the usage cases is command works as a generator and returns the value and does not rely on
  the original data. The default value `skip_original_data=false`
* **skip_on_null_input** - skip transformation on null original value. Works together with `skip_on_behaviour`
  parameter. In case we have two affected columns we may use this logic. In case both columns are
  `skip_on_null_input=true` and one column is null, then when `skip_on_behaviour=any` then skip transformation or when
  `skip_on_behaviour=and` then the transformation will be performed.

``` yaml title="Cmd transformer example"
- name: "Cmd"
  params:
    executable: "cmd_test.sh"
    driver: 
      name: "json"
      params: 
        format: "bytes" # (4) 
    timeout: "60s"
    validate_output: true # (1)
    expected_exit_code: -1
    skip_on_behaviour: "any" # (2)
    columns:
      - name: "actual_arrival"
        skip_original_data: true
        skip_on_null_input: true # (3)
      - name: "scheduled_arrival"
        skip_original_data: true
        skip_on_null_input: true # (3)
```
{ .annotate }

1. Validate the received data via encode-decode procedure via go driver. Note - it may cause an error if the type is
   not supported in the golang `pgx` driver.
2. Skip transformation (keep the values) if one of the affected columns has null value
3. In the column has `null` value than skip it. Works together with `skip_on_behaviour`. Since it has `any` value then
   if one of column (actual_arrival or scheduled_arrival) has null value then skip the transformation call
4. The `format` of json can be either `text` or `bytes`

!!! warning
The parameter `validate_output=true` may cause an error if it's the type does not have `pgx` driver encode-decode
implementation. Most of the types such as int, float, text, varchar, date, timestamp, etc. have encoders and
decoders as well as inherited types like domains type based on them.  
