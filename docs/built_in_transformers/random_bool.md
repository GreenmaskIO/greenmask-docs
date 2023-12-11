Generate random boolean values.

## Parameters

| Name      | Description                                                                  | Default | Required | Supported DB types |
|-----------|------------------------------------------------------------------------------|---------|----------|--------------------|
| column    | The name of the column whose value will be affected                          |         | Yes      | bool               |
| keep_null | Indicates whether NULL values should not be replaced with transformed values | `true`  | No       | -                  |

## Description

The RandomBool transformer generates a random boolean value. The behavior for `NULL` values can be
configured using the `keep_null` parameter.

## Examples

### A. Generate a Random Boolean for Salaried Flag

In this example, the `RandomBool` transformer generates a random boolean value for the `salariedflag` column.

``` yaml title="RandomBool transformer example"
- schema: "humanresources"
  name: "employee"
  transformers:
    - name: "RandomBool"
      params:
        column: "salariedflag"
```
