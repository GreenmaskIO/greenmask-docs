### Description

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
