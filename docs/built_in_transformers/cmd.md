Transform data via external program using stdin and stdout interaction.

## Parameters

| Name               | Description                                                                                                                                                                                                                                                | Default           | Required | Supported DB types |
|--------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-------------------|----------|--------------------|
| columns            | A list of column names to be affected. If empty, the entire tuple is used. Read about the structure below                                                                                                                                                  |                   | Yes      | any                |
| executable         | The path to the executable file                                                                                                                                                                                                                            |                   | Yes      | -                  |
| args               | A list of parameters for the executable                                                                                                                                                                                                                    |                   | No       | -                  |
| driver             | The row driver with parameters that is used for interacting with cmd. The default is csv. `{"name": "text, csv, json", "params": { "format": "[text bytes]"}}`. The `text` and `bytes` options are available only for `json`                               | `{"name": "csv"}` | No       | -                  |
| validate           | Perform a decoding operation using the PostgreSQL driver for data received from the command to ensure the data format is correct.                                                                                                                          | `false`           | No       | -                  |
| timeout            | Timeout for sending and receiving data from the external command.                                                                                                                                                                                          | `2s`              | No       | -                  |
| expected_exit_code | The expected exit code on SIGTERM signal. If the exit code is unexpected, the transformation exits with an error.                                                                                                                                          | `0`               | No       | -                  |
| skip_on_behaviour  | Skip transformation call if one of the provided columns has a `null` value (`any`) or each of the provided columns has `null` values (`all`). This option works together with the `skip_on_null_input` parameter on columns. Possible values: `all`, `any` | `all`             | No       | -                  |

!!! warning

    The parameter `validate_output=true` may cause an error if the type does not have a PostgreSQL driver decoder 
    implementation. Most of the types, such as int, float, text, varchar, date, timestamp, etc., have encoders and 
    decoders, as well as inherited types like domain types based on them.

## Description

The `Cmd` transformer allows you to send original data to an external program via `stdin` and receive transformed data
from `stdout`. It supports various interaction formats such as `json`, `csv`, or plain `text` for one-column
transformations. The interaction is performed line by line, so at the end of each sent data, a new line symbol `\n` must
be included.

### Types of interaction modes

#### text

Textual driver. Uses only for one column transformation, you cannot provide here more than one column. The value encodes
into string laterally. For example `2023-01-03 01:00:00.0+03`.

#### json

json line driver. It has two formats `[text|bytes]` that can be passed through `driver.params`

`bytes` - use this format if the types are binary

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

* Each line is a JSON line with a map of attribute numbers to their values.
* d - the raw data base64 encoded. The base64 encoding is needed because it may be binary data
* n - is the value NULL or not

`text` - use this format if the types are not binary and might be represented as string literals

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

* Each line is a JSON line with a map of attribute numbers to their values.
* d - the raw data is represented as Unicode text.
* n - is the value NULL or not

#### csv

CSV driver (comma-separated). The count of attributes is the same as the table columns count, but the
columns that were not mentioned in the `columns` list are empty. The `NULL` value is represented as `\N`. Each
attribute is escaped by quote (`"`). For example, if the transformed table has attributes `id`, `title`, and
`created_at`, and we want to transform only `id` and `created_at`, then the CSV line is (where the second
attribute is empty)

``` csv title="csv line example"
"123","","2023-01-03 01:00:00.0+03"
```

Column object attributes:

* **name** - The name of the column. This value is required. Depending on the attributes below, this column may be used
  just as a value and is not affected in any way.
* **not_affected** - Indicates whether the column is affected in the transformation. This attribute is required for the
  validation procedure when Greenmask is called with `greenmask dump --validate`. Setting `not_affected=true` can be
  helpful when the command transformer transforms data depending on the value of another column. For example, if you
  want to generate an `updated_at` column value depending on the `created_at` column value, you can set `created_at`
  to `not_affected=true`. The default value is `not_affected=false`.
* **skip_original_data** - Indicates whether the original data is required for the transformer. This attribute can be
  helpful for decreasing the interaction time. One use case is when the command works as a generator and returns the
  value without relying on the original data. The default value is `skip_original_data=false`.
* **skip_on_null_input** - Specifies whether to skip transformation when the original value is `null`. This attribute
  works in conjunction with the `skip_on_behaviour` parameter. For example, if you have two affected columns with
  `skip_on_null_input=true` and one column is null, then when `skip_on_behaviour=any`, the transformation will be
  skipped, or when `skip_on_behaviour=and`, the transformation will be performed. The default is false.

## Examples

### A. Apply transformation performed by external command

In this example the columns `actual_arrival` and `scheduled_arrival` are transformed via external command transformer.

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

1. Validate the received data via decode procedure using the PostgreSQL driver. Note that this may cause an error if the
   type is not supported in the PostgreSQL driver.
2. Skip transformation (keep the values) if one of the affected columns (`not_affected=false`) has a null value.
3. If a column has a null value, then skip it. This works in conjunction with `skip_on_behaviour`. Since it has the
   value any, if one of the columns (`actual_arrival` or `scheduled_arrival`) has a `null` value, then skip the
   transformation call.
4. The format of JSON can be either `text` or `bytes`.
