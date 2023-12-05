### Description

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
