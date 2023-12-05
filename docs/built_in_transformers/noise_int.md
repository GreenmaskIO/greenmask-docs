### Description

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
