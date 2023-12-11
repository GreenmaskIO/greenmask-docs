The `Replace` transformer allows you to replace original value by the provided

## Parameters

| Name      | Description                                                                                                                    | Default | Required | Supported DB types |
|-----------|--------------------------------------------------------------------------------------------------------------------------------|---------|----------|--------------------|
| column    | The name of the column whose value will be affected                                                                            |         | Yes      | any                |
| replace   | The value to replace                                                                                                           |         | Yes      | -                  |
| keep_null | Indicates whether NULL values should not be replaced with transformed values                                                   | `true`  | No       | -                  |
| validate  | Validate the value through driver decoding procedure. The value must match the column's type; otherwise, it may cause an error | `true`  | No       | -                  |

## Description

The `Replace` transformer can decode the provided value using the PostgreSQL driver for format validation purposes. To
enable this validation, set the validate parameter to true. The `RandomInt` transformer generates a random integer
within the specified `min` and `max` thresholds. The behavior for `NULL` values can be configured using the `keep_null`
parameter.

## Examples

### A. Updating the "jobtitle" Column

In this example, the provided `value: "programmer"` is validated through driver decoding. If the current value of the
`jobtitle` column is not `NULL`, it will be replaced with `programmer`. If the current value is `NULL`, it will
remain `NULL`.

``` yaml title="Replace transformer example"
- schema: "humanresources"
  name: "employee"
  transformers:
  - name: "Replace"
    params:
      column: "jobtitle"
      value: "programmer"
      keep_null: false
      validate: true
```

Expected results:

| column name | original value          | transformed |
|-------------|-------------------------|-------------|
| jobtitle    | Chief Executive Officer | programmer  |

### B. Validation error

This example causes error due to type in `DATE` value.

``` yaml title="Replace transformer example with misstake"
- schema: "humanresources"
  name: "employee"
  transformers:
  - name: "Replace"
    params:
      column: "hiredate"
      value: "programmer"
      keep_null: false
      validate: true
```

Expected results:

Causes validation warning with `error` severity

```json
{
  "hash": "b2d5380a7296b20ec7d01152931e1e33",
  "meta": {
    "Error": "invalid date format",
    "ParameterName": "value",
    "ParameterValue": "2009-01-a4",
    "SchemaName": "humanresources",
    "TableName": "employee",
    "TransformerName": "Replace",
    "TypeName": "date"
  },
  "msg": "error decoding \"value\" parameter from raw string to type",
  "severity": "error"
}
```
