Generate a random float within the provided interval.

## Parameters

| Name      | Description                                                                            | Default | Required | Supported DB types                                |
|-----------|----------------------------------------------------------------------------------------|---------|----------|---------------------------------------------------|
| column    | The name of the column whose value will be affected                                    |         | Yes      | float4 (real), float8 (double precision), numeric |
| min       | The minimum threshold for the random value. The value range depends on the column type |         | Yes      | -                                                 |
| max       | The maximum threshold for the random value. The value range depends on the column type |         | Yes      | -                                                 |
| precision | The precision of the random float value (number of digits after the decimal point).    | `4`     | No       | -                                                 |
| keep_null | Indicates whether NULL values should not be replaced with transformed values.          | `true`  | No       | -                                                 |

## Description

The `RandomFloat` transformer generates a random `float` value within the provided interval, starting from `min` to
`max`, with the option to specify the precision via `precision` parameter. The behavior for `NULL` values can be
configured using the `keep_null` parameter.

## Examples

### A. Generate random price

In this example, the `RandomFloat` transformer generates random prices in the range from `0.1` to `7000` while
maintaining a precision of up to 2 digits.

``` yaml title="RandomFloat transformer example"
- schema: "sales"
  name: "salesorderdetail"
  transformers:
    - name: "RandomFloat"
      params:
        column: "unitprice"
        min: 0.1
        max: 7000
        precision: 2
```

Expected results:

| column name | original value | transformed |
|-------------|----------------|-------------|
| unitprice   | 2024.994       | 6806.5      |
