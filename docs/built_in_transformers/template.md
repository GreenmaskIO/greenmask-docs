### Description

Execute go template and apply the template result to the column

| Name     | Description                                                       | Default | Required | Supported DB types |
|----------|-------------------------------------------------------------------|---------|----------|--------------------|
| column   | name of column which value is going to be affected                |         | Yes      | any                |
| template | gotemplate string                                                 |         | Yes      |                    |
| validate | validate template result via PostgreSQL driver decoding procedure | false   |          |                    |

``` yaml title="Template transformer example"
- schema: "public"
  name: "order"
  transformers:
  - name: "Template"
    params:
      column: "updated_at"
      template: >
        {{- now | .EncodeValue -}}
      validate: true
```

Have a read about [provided functions and hints](../template_functions.md).

PostgreSQL driver functions provided by `Template` transformer:

* `.GetColumnType` - returns string with column type.
* `.GetValue` - returns column value (that was assigned in the `column` parameter) encoded by PostgreSQL driver into
  any type and error. It might be on of `int, float, time, string, bool` and `slice` or `map` of any type
* `.GetRawValue` - return raw column value (that was assigned in the `column` parameter) as a string and error
* `.GetColumnValue` - returns a value and error for a specific column, encoded by PostgreSQL driver into
  any type and error It might be on of `int, float, time, string, bool` and `slice` or `map` of any type
* `.GetColumnValue` - returns a raw value as a string and error for a specific column
* `.EncodeValue` - encodes value from any type to the string representation using type that has assigned column in
  `column` parameter
* `.DecodeValue` - decode value using from raw string representation to golang type (such
  as `int, float, time, string, bool` and `slice` or `map` of any type) using type that has assigned column in `column`
  parameter.
* `.EncodeValueByColumn` - encodes value from any type to the string representation using provided column name
* `.DecodeValueByColumn` - decode value using from raw string representation to golang type using provided column name
* `.EncodeValueByType` - encodes value from any type to the string representation using provided column type name
* `.DecodeValueByType` - decode value using from raw string representation to golang type using provided type name


### Use cases
