### Description

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
