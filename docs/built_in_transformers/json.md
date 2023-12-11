Change a JSON document using `delete` and `set` operations. `NULL` values are kept

## Parameters

| Name       | Description                                                   | Default | Required | Supported DB types |
|------------|---------------------------------------------------------------|---------|----------|--------------------|
| column     | The name of the column whose value will be affected           |         | Yes      | json, jsonb        |
| operations | A list of operations that contains editing `delete` and `set` |         | Yes      | -                  |

## Description

The `Json` transformer applies a list of changing operations `set` or `delete` on a JSON document. The value might
be constant provided by the `value` parameter or dynamic provided by the `value_template` parameter where the template
resulting data is applied as a result. Parameters `value` and `value_template` are required only for the set operation.

### Operations structure

* **operation** - name of operation. Can be on of `set` or `delete`
* **path** - path to the object within the document
* **value** - value to assign by provided path. Required only for `set` operation
* **value_template** - go template string that result assigning by the path
* **error_not_exist** - raise an error if the key by path does not exist. Disabled by default.

### Path syntax

The Json transformer is based on [tidwall/sjson](https://github.com/tidwall/sjson) and supports the same path syntax.

**Syntax rules**:

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

### Template functions

#### .GetPath

Returns the current path with which the operation is applying.

Signature:

`.GetPath() (path string)`

Return Value:

* `path` - Current operation path

Example:

```gotemplate
{{- if eq .GetPath "title" -}}
default title
{{- else -}}
some title
{{- end -}}
```

#### .GetOriginalValue

Returns the original value on which the current operation path is pointing. If the value by the path does not exist, it
returns `nil`.

Signature:

`.GetOriginalValue() (value any)`

Return Value:

* `value` - original value

Example:

```gotemplate
{{- if eq .GetOriginalValue 123 -}}
{{- randomInt 1 100 -}}
{{- else -}}
0
{{- end -}}
```

#### .OriginalValueExists

Returns a boolean value that indicates whether the path exists or not.

Signature:

`.OriginalValueExists() (exists bool)`

Return Value:

* `exists` - existing flag

Example:

```gotemplate
{{- if eq .OriginalValueExists false -}}
123
{{- else -}}
{{- .GetOriginalValue -}}
{{- end -}}
```

#### .GetColumnValue

Returns a value for a specific column, encoded by PostgreSQL driver into any type and error It might be on
of `int`, `float`, `time`, `string`, `bool` and `slice` or `map` of any type. Read
details [here](template_record.md/#getcolumnvalue).

#### .GetRawColumnValue

Returns a raw value as a string for a specific column. Read
details [here](template_record.md/#getrawcolumnvalue).

#### .EncodeValueByColumn

Encode a value of any type into its raw string representation using the specified column name. This encoding operation
is carried out through the PostgreSQL driver. Causes an error if the type is not compatible with column type. Read
details [here](template_record.md/#encodevaluebycolumn).

#### .DecodeValueByColumn

Decode a value from its raw string representation to a Golang type using the specified column name. This decoding
operation is executed through the PostgreSQL driver and may result in an error if the type is not compatible with the
column type. Read details [here](template_record.md/#decodevaluebycolumn).

#### .EncodeValueByType

Encode a value of any type into its string representation using the specified type name. This encoding operation
is carried out through the PostgreSQL driver and may result in an error if the type is not compatible with the column
type. Read details [here](template_record.md/#encodevaluebytype).

#### .DecodeValueByType

Decode a value from its raw string representation to a Golang type using the specified type name. This decoding
operation is performed through the PostgreSQL driver and may result in an error if the type is not compatible with the
column type. Read details [here](template_record.md/#decodevaluebytype).

## Examples

### A. Changing json document

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
            path: "seats"
            error_not_exist: True
            value_template: "{{ randomInt 100 400 }}"
          - operation: "set"
            path: "details.preperties.1"
            value: {"name": "somename", "description": null}
          - operation: "delete"
            path: "values.:2"
```
