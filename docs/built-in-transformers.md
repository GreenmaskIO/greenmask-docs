## Dict 

Replace values matched by dictionary keys

Parameters:

| Name             | Description                                                                                                                            | Default | Required | Supported DB types |
|------------------|----------------------------------------------------------------------------------------------------------------------------------------|---------|----------|--------------------|
| column           | name of column which value is going to be affected                                                                                     |         | Yes      | any                |
| values           | map of value to replace in format: `{"string": "string"}`. The string with value `"\N"` is supposed to be a NULL value                 |         | No       | -                  |
| default          | default value if not any value has been matched with dict. The string with value `"\N"` is supposed to be NULL value. Default is empty |         | No       | -                  |
| fail_not_matched | fail if value is not matched with dict otherwise keep value                                                                            | `false` | No       | -                  |
| validate         | perform encode-decode procedure using column type, ensuring that value has correct type                                                | `true`  | No       | -                  |

For setting or matching _NULL_ value use string with `\N` sequence.

``` yaml title="Dict transformer example"
- schema: "public"
  name: "order"
  transformers:
    - name: "Dict"
      params:
        column: "type_name"
        values: 
          "type1": "random_type1"
          "type2": "random_type2"
        default: "unknown_type"
        validate: true
        fail_not_matched: false
```

## Hash

Generate the hash of the text value using the Scrypt hash function under the hood. Null values are kept.

Parameters:

| Name   | Description                                                                    | Default | Required | Supported DB types |
|--------|--------------------------------------------------------------------------------|---------|----------|--------------------|
| column | name of column which value is going to be affected                             |         | Yes      | text, varchar      |
| salt   | random salt for hash function as a text string. By default generates randomly. |         | No       | -                  |

``` yaml title="Hash transformer example"
- schema: "bookings"
  name: "booking"
  transformers:
    - name: "Hash"
      params:
        column: "name"
        salt: "12345678"
```

## Json

Change json document using delete and set operations. Null values are kept.

!!! note
    Greenmask currently provides basic features. It's expected that later Json transformer will be integrated with
    gotemplate that might be more flexible with editing Json document.

Parameters:

| Name       | Description                                             | Default | Required | Supported DB types |
|------------|---------------------------------------------------------|---------|----------|--------------------|
| column     | name of column which value is going to be affected      |         | Yes      | json, jsonb        |
| operations | list of operations that contains editing delete and set |         | Yes      | -                  |

Operations structure:

* **operation** - name of operation. Can be on of `set` or `delete`
* **path** - path to the object within the document
* **value** - value to assign by provided path. Required only for `set` operation

The _Json_ transformer is based on [tidwall/sjson](https://github.com/tidwall/sjson) and supports the same path syntax.

Syntax rules:

* A path is a series of keys separated by a dot. The dot and colon characters can be escaped with `\`.
* The -1 key can be used to append a value to an existing array
* Normally numbers in the path are used for accessing the array, but if your document has int keys in the
  map colon symbol `:` before the number is required. For instance, we have a document

    ``` json
    {
      "values": {
        1: "test1",
        2: "test2"
      }
    }
    ```
  In this case, a path syntax for editing key `2` is `"values.:2"`

``` yaml title="Json transformer example"
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
            path: "details.preperties.1"
            value: {"name": "somename", "description": null}
          - operation: "delete"
            path: "values.:2"
```

## Masking

Mask a value using one of the masking behavior depending on your domain. Null values are kept.

Parameters:

| Name   | Description                                                                                                        | Default   | Required | Supported DB types |
|--------|--------------------------------------------------------------------------------------------------------------------|-----------|----------|--------------------|
| column | name of column which value is going to be affected                                                                 |           | Yes      | text, varchar      |
| type   | logical type of attribute (_default_, _password_, _name_, _addr_, _email_, _mobile_, _tel_, _id_, _credit_, _url_) | `default` | No       | -                  |

The _Masking_ transformer based on [ggwhite/go-masker](https://github.com/ggwhite/go-masker) and supports the next
masking rules:

|    Type     | Description                                                                                                                                                  |
|:-----------:|:-------------------------------------------------------------------------------------------------------------------------------------------------------------|
|   default   | returns `*` symbols with the same length. Example: input: `test1234` output: `********`                                                                      |
|    name     | mask the second letter and the third letter. Example: input: `ABCD` output: `A**D`                                                                           |
|  password   | always return `************`                                                                                                                                 |
|   address   | keep first 6 letters, mask the rest. Example: input: `Larnaca, makarios st` output: `Larnac*************`                                                    |
|    email    | keep domain and the first 3 letters. Example: input: `ggw.chang@gmail.com` output: `ggw****@gmail.com`                                                       |
|   mobile    | mask 3 digits from the 4'th digit. Example: input: `0987654321` output: `0987***321`                                                                         |
|  telephone  | remove `(`, `)`, ` `, `-` chart, and mask last 4 digits of telephone number, format to `(??)????-????`. Example: input: `0227993078` output: `(02)2799-****` |
|     id      | mask last 4 digits of ID number. Example: input: `A123456789` output: `A12345****`                                                                           |
| credit_cart | mask 6 digits from the 7'th digit. Example: input `1234567890123456` output `123456******3456`                                                               |
|     url     | mask the password part of the URL if exists. input: http://admin: `mysecretpassword@localhost:1234/uri` output: `http://admin:xxxxx@localhost:1234/uri`      |

``` yaml title="Masking transformer example"
- schema: "public"
  name: "order"
  transformers:
    - name: "Masking"
      params:
        column: "user_name"
        type: "name"
```

## NoiseDate

Add some random value (shift value) in the provided interval. The value might be shifted either to before this
date or after.

Parameters:

| Name     | Description                                                                                                                                                                                                   | Default | Required | Supported DB types           |
|----------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------|----------|------------------------------|
| column   | name of column which value is going to be affected                                                                                                                                                            |         | Yes      | date, timestamp, timestamptz |
| ratio    | max random duration for noise. The value has [PostgreSQL interval format](https://www.postgresql.org/docs/current/datatype-datetime.html#DATATYPE-INTERVAL-INPUT). Example: `1 year 1 mons 1 day 01:01:01.01` |         | Yes      | -                            |
| truncate | truncate date till the part (_year_, _month_, _day_, _hour_, _second_, _millisecond_, _microsecond_, _nanosecond_). Truncate operation is not applied by default.                                             |         | No       | -                            |

``` yaml title="NoiseDate transformer example"
- schema: "public"
  name: "order"
  transformers:
    - name: "NoiseDate"
      params:
        column: "created_at"
        ratio: "1 year 1 mons 1 day 01:01:01.01"
        truncate: "microsecond"
```

## NoiseFloat

Multiply the float value by the provided random value that is not higher than `ratio` parameter and add it to the original
value.

Example:

* Original: `12.56`
* Ratio: `0.9`
* Generated random value during execution: `0.75`

Transformation result: `12.58 + (12.58 * 0.75) = 22.015`  (1)
{ .annotate }

1. The multiplied value might be either added or subtracted `12.58 - (12.58 * 0.75) = 3.1449`

Parameters:

| Name      | Description                                                                   | Default | Required | Supported DB types |
|-----------|-------------------------------------------------------------------------------|---------|----------|--------------------|
| column    | name of column which value is going to be affected                            |         | Yes      | float4, float8     |
| ratio     | max random percentage for noise. Example: `0.1` (meaning add noise up to 10%) |         | Yes      | -                  |
| precision | precision of noised float value (number of digits after coma)                 | `4`     | No       | -                  |

``` yaml title="NoiseDate transformer example"
- schema: "public"
  name: "order"
  transformers:
    - name: "NoiseFloat"
      params:
        column: "total_price"
        ratio: 0.1
        precision: 2
```

## NoiseInt

Multiply the int value by the provided random value that is not higher than `ratio` parameter and add it to the 
original value.

Example:

* Original: `100`
* Ratio: `0.1`
* Generated random value during execution: `0.05`

Transformation result: `100 + (100 * 0.05) = 105`  (1)
{ .annotate }

1. The multiplied value might be either added or subtracted `100 - (100 * 0.05) = 95`

Parameters:

| Name   | Description                                                                   | Default | Required | Supported DB types |
|--------|-------------------------------------------------------------------------------|---------|----------|--------------------|
| column | name of column which value is going to be affected                            |         | Yes      | int2, int4, int8   |
| ratio  | max random percentage for noise. Example: `0.1` (meaning add noise up to 10%) |         | Yes      | -                  |

``` yaml title="NoiseInt transformer example"
- schema: "public"
  name: "order"
  transformers:
    - name: "NoiseInt"
      params:
        column: "amount"
        ratio: 0.1
```

## RandomBool

Generate random bool value.

Parameters:

| Name      | Description                                            | Default | Required | Supported DB types |
|-----------|--------------------------------------------------------|---------|----------|--------------------|
| column    | name of the column which value is going to be affected |         | Yes      | bool               |
| keep_null | do not replace NULL values to random value             | `true`  | No       | -                  |

``` yaml title="RandomBool transformer example"
- schema: "public"
  name: "order"
  transformers:
    - name: "RandomBool"
      params:
        column: "is_active"
        keep_null: false
```

## RandomDate

Generate a random date in the provided interval

Parameters:

| Name      | Description                                                                                                                                                       | Default | Required | Supported DB types           |
|-----------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------|----------|------------------------------|
| column    | name of the column which value is going to be affected                                                                                                            |         | Yes      | date, timestamp, timestamptz |
| min       | min threshold date (and/or time) of random value. The format depends on the column type.                                                                          |         | Yes      | -                            |
| max       | max threshold date (and/or time) of random value. The format depends on the column type.                                                                          |         | Yes      | -                            |
| truncate  | truncate date till the part (_year_, _month_, _day_, _hour_, _second_, _millisecond_, _microsecond_, _nanosecond_). Truncate operation is not applied by default. |         | No       | -                            |
| keep_null | do not replace NULL values to random value                                                                                                                        | `true`  | No       | -                            |

``` yaml title="RandomDate transformer example"
- schema: "public"
  name: "order"
  transformers:
    - name: "RandomDate"
      params:
        column: "created_at"
        keep_null: false
        min: "2023-01-03 01:00:00.0+03"
        max: "2023-01-04 00:00:00.0+03"
        truncate: "second"
```

## RandomFloat

Generate a random float in the provided interval

Parameters:

| Name      | Description                                                                 | Default | Required | Supported DB types |
|-----------|-----------------------------------------------------------------------------|---------|----------|--------------------|
| column    | name of column which value is going to be affected                          |         | Yes      | float4, float8     |
| min       | min float threshold of random value. The value range depends on column type |         | Yes      | -                  |
| max       | max float threshold of random value. The value range depends on column type |         | Yes      | -                  |
| precision | precision of random float value (number of digits after coma)               | `4`     | No       | -                  |
| keep_null | do not replace NULL values to random value                                  | `true`  | No       | -                  |

``` yaml title="RandomFloat transformer example"
- schema: "public"
  name: "order"
  transformers:
    - name: "RandomFloat"
      params:
        column: "total_price"
        keep_null: false
        min: 10000.99
        max: 0.50
        precision: 2
```

## RandomInt

Generate a random int in the provided interval

Parameters:

| Name      | Description                                                               | Default | Required | Supported DB types |
|-----------|---------------------------------------------------------------------------|---------|----------|--------------------|
| column    | name of column which value is going to be affected                        |         | Yes      | int2, int4, int8   |
| min       | min int threshold of random value. The value range depends on column type |         | Yes      | -                  |
| max       | max int threshold of random value. The value range depends on column type |         | Yes      | -                  |
| keep_null | do not replace NULL values to random value                                | `true`  | No       | -                  |

``` yaml title="RandomInt transformer example"
- schema: "public"
  name: "order"
  transformers:
    - name: "RandomInt"
      params:
        column: "amount"
        keep_null: true
        min: 1
        max: 100
```

## RandomString

Generate a random string using the provided characters.

Parameters:

| Name       | Description                                        | Default                                                | Required | Supported DB types |
|------------|----------------------------------------------------|--------------------------------------------------------|----------|--------------------|
| column     | name of column which value is going to be affected |                                                        | Yes      | text, varchar      |
| min_length | min length of string                               |                                                        | Yes      | -                  |
| max_length | max length of string                               |                                                        | Yes      | -                  |
| symbols    | the characters range for random string             | `abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ` | No       | -                  |
| keep_null  | do not replace NULL values to random value         | `true`                                                 | No       | -                  |

``` yaml title="RandomString transformer example"
- schema: "public"
  name: "order"
  transformers:
    - name: "RandomString"
      params:
        column: "description"
        min_length: 10
        max_length: 100
        symbols: "1234567890-"
```

## RandomUuid

Generate random uuid using version 4.

| Name      | Description                                        | Default | Required | Supported DB types  |
|-----------|----------------------------------------------------|---------|----------|---------------------|
| column    | name of column which value is going to be affected |         | Yes      | text, varchar, uuid |
| keep_null | do not replace NULL values to random value         | `true`  | No       | -                   |

``` yaml title="RandomUuid transformer example"
- schema: "public"
  name: "order"
  transformers:
  - name: "RandomUuid"
    params:
      column: "transcaction_id"
      keep_null: false
```

## RegexpReplace

Replace string using regular expression

The syntax of the regular expressions accepted is the same general syntax used by Perl, Python, and other languages.
More precisely, it is the syntax accepted by RE2 and described at [golang doc](https://golang.org/s/re2syntax), except for `\C`.

Parameters:

| Name    | Description                                                                         | Default | Required | Supported DB types |
|---------|-------------------------------------------------------------------------------------|---------|----------|--------------------|
| column  | name of column which value is going to be affected                                  |         | Yes      | text, varchar      |
| regexp  | regular expression                                                                  |         | Yes      | -                  |
| replace | replacement value. The value may be replaced with the group from `regexp` parameter |         | Yes      | -                  |

``` yaml title="RegexpReplace transformer example"
- schema: "public"
  name: "order"
  transformers:
  - name: "RegexpReplace"
    params:
      column: "description"
      regexp: "(Hello)\s*world\s*(\!+\?)"
      replace: "$1 Mr NoName $2"
```

## Replace

Replace a column value with the provided.

| Name      | Description                                                                                                | Default | Required | Supported DB types |
|-----------|------------------------------------------------------------------------------------------------------------|---------|----------|--------------------|
| column    | name of column which value is going to be affected                                                         |         | Yes      | any                |
| replace   | value to replace. The value must be literally the same as the column type otherwise it may cause an error. |         | Yes      | -                  |
| keep_null | do not replace NULL values to random value                                                                 | `true`  | No       | -                  |

``` yaml title="Replace transformer example"
- schema: "public"
  name: "order"
  transformers:
  - name: "Replace"
    params:
      column: "created_at"
      value: "2023-11-07 15:22:21.885053+00"
      keep_null: false
```

## SetNull

Set NULL value to the column

| Name   | Description                                        | Default | Required | Supported DB types |
|--------|----------------------------------------------------|---------|----------|--------------------|
| column | name of column which value is going to be affected |         | Yes      | any                |

``` yaml title="SetNull transformer example"
- schema: "public"
  name: "order"
  transformers:
  - name: "SetNull"
    params:
      column: "updated_at"
```

## Cmd

Transform data via external program using stdin and stdout interaction.

| Name               | Description                                                                                                                                                                                                                                       | Default           | Required | Supported DB types |
|--------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-------------------|----------|--------------------|
| columns            | affected column names. If empty use the whole tuple. Read about structure below                                                                                                                                                                  |                   | Yes      | any                |
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
