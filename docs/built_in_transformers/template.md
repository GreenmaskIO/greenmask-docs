The `Template` Transformer allows you to execute a Go template and apply the result to a specified column.

## Parameters

| Name     | Description                                                                                                                                                          | Default | Required | Supported DB types |
|----------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------|----------|--------------------|
| column   | The name of the column whose value will be affected                                                                                                                  |         | Yes      | any                |
| template | A Go template string                                                                                                                                                 |         | Yes      | -                  |
| validate | Validate the template result using the PostgreSQL driver decoding procedure. This may cause an error if a custom type does not have an encode-decoder implementation | false   | No       | -                  |

## Description

The `Template` transformer utilizes Go templates and **applying the template result to the column**. Go's template
system is designed to be extensible, enabling developers to access data objects and incorporate custom functions
programmatically. For more information, you can refer to the
official  [Go Template documentation](https://pkg.go.dev/text/template).

For more information about available template functions, refer to the [functions and hints](../template_functions.md).

!!! warning

    Pay attention to the whitespaces when designing the template. Use dash-wrapped - brackets {{- -}} for 
    trimming the spaces. For instance the value `"2023-12-19"` is not the same as `" 2023-12-19  "` and it may cause 
    an error when restoring.

### Template functions

#### .GetColumnType

Returns a string with the column type. Works similar with `TemplateRecord` `.GetColumnType` function. Read
details [here](template_record.md/#getcolumntype).

#### .GetValue

Returns the column value for column assigned in the `column` parameter, encoded by the PostgreSQL driver into any
type along with any associated error. Supported types include `int`, `float`, `time`, `string`, `bool`, as
well as `slice` or `map` of any type.

Signature:

`.GetValue() (value any, err error)`

Return Value:

* `value` - any type that was encoded from original raw value into golang type. Usually it is `int`, `float`, `time`,
  `string`, `bool` and `slice` or `map` of any type
* `err` - An error if there's an issue

Example

```gotemplate
{{ .GetValue  }}
```

#### .GetRawValue

Returns a raw value as a string for column assigned in the `column` parameter.

Signature:

`.GetRawColumnValue(name string) (value string, err error)`

Return Value:

* `value` - A string raw value
* `err` - An error if there's an issue

Example

```gotemplate
{{ .GetRawValue }}
```

#### .GetColumnValue

Returns a value for a specific column, encoded by PostgreSQL driver into any type and error It might be on
of `int`, `float`, `time`, `string`, `bool` and `slice` or `map` of any type. Read
details [here](template_record.md/#getcolumnvalue).

#### .GetRawColumnValue

Returns a raw value as a string for a specific column. Read
details [here](template_record.md/#getrawcolumnvalue).

#### .EncodeValue

Encode a value of any type into its string representation using the type assigned to the table column specified in the
`column` parameter. This encoding operation is performed through the PostgreSQL driver and may result in an error if the
provided value's type is not compatible with the column's type.

Signature:

`.EncodeValue(value any) (res any, err error)`

Parameters:

* `value` - A value of a specific data type

Return Value:

* `res` - A raw value, represented as a string or Null value.
* `err` - An error if there's an issue

Consider a scenario where we have a column named `id` with a data type of `int`, and we've specified this column using
the transformer parameter `column: "id"`.

In the example below, an integer value `123` is converted into the string `123` to match the int data type assigned to
the `id` column. Subsequently, this string "`123`" is concatenated with `456`. The template yields a result of `123456`,
which is then assigned to the id column.

```gotemplate
{{- 123 | .EncodeValue | cat "567" -}}
```

#### .DecodeValue

Decode a value from its raw string representation to a Golang type using the data type assigned to the table column
specified in the `column` parameter. This decoding operation is performed through the PostgreSQL driver and may result
in an error if the specified Golang type is not compatible with the column's data type.

Signature:

`.DecodeValueByColumn(value any) (res any, err error)`

Parameters:

* `value` - A raw value, represented as a string or Null value.

Return Value:

* `res` - A value of a specific data type
* `err` - An error if there's an issue

Example:

Let's consider a scenario where we have a column named `created_at` with a data type of `timestamp`, and we've specified
this column using the transformer parameter `column: "created_at"`.

In this example, a raw value `"2023-12-17 18:50:34.277372+00"` is decoded using the `timestamptz` PostgreSQL data type,
which is assigned to the `created_at` column. Subsequently, it is encoded back. This operation serves as a raw value
verification process. If there are any mistakes in the timestamp value format, the `.DecodeValue` function will produce
an error.

The template generates a result of `2023-12-17 18:50:34.277372+00`, which is subsequently assigned to the
`created_at` column.

```gotemplate
{{ "2023-12-17 18:50:34.277372+00" | .DecodeValue | .EncodeValue }}
```

For example, this template will result in an error:

```gotemplate
{{ "2023asda-12-17 18:50:34.277372+00" | .DecodeValue | .EncodeValue }}
```

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

### A. Update "firstname" Column

Table structure

![person-person-schema.png](..%2Fassets%2Fbuilt_in_transformers%2Fperson-person-schema.png)

**Change Rule**

We want to modify the "firstname" column using the following rule:

* If the current value of the `firstname` column is equal to `Terri` replace it with `Mary`.
* For all other cases, generate a random name.

**Using Template Function**

To generate random names, you can make use of the `fakerFirstName` template function, which is designed to create
synthetic names.

```yaml title="Template transformer example"
- schema: "humanresources"
  name: "employee"
  transformation:
    - name: "Template"
      params:
        column: "firstname"
        template: >
          {{- if eq .GetValue "Terri" -}}
            Mary
          {{- else -}}
            {{- fakerFirstName -}} Jr
          {{- end -}}

        validate: true
```

Expected results:

Case 1 (non Terri value)

| column name | original value | transformed |
|-------------|----------------|-------------|
| firstname   | Ken            | Mike        |

Case 2 (is Terry)

| column name | original value | transformed |
|-------------|----------------|-------------|
| firstname   | Terri          | Mary        |
