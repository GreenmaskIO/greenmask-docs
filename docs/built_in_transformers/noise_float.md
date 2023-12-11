Add or subtract a random fraction to the original float value.

## Parameters

| Name      | Description                                                                                              | Default | Required | Supported DB types                                |
|-----------|----------------------------------------------------------------------------------------------------------|---------|----------|---------------------------------------------------|
| column    | The name of the column whose value will be affected                                                      |         | Yes      | float4 (real), float8 (double precision), numeric |
| ratio     | The maximum random percentage for noise in value from 0 to 1. Example: 0.1 (meaning add noise up to 10%) |         | Yes      | -                                                 |
| precision | The precision of the noised float value (number of digits after the decimal point)                       | `4`     | No       | -                                                 |

## Description

The `NoiseFloat` transformer multiplies the original float value by a provided random value that is not higher than
the `ratio` parameter and adds it to the original value with the option to specify the precision via the `precision`
parameter.

Example:

Original Value: `12.56`

Ratio: `0.9` (meaning add noise up to 90%)

Generated random value during execution: `0.75`

Transformation Result: `12.58 + (12.58 * 0.75) = 22.015`

The multiplied value might be either **added or subtracted**, resulting in `12.58 - (12.58 * 0.75) = 3.1449`

If the precision is set to 2 (`precision: 2`), the value will be rounded to 3.15.

## Examples

### A. Adding Noise to the Price of a Purchase

In this example, the original value of `standardprice` will be noised up to `50%` and rounded up to `2` decimals.

``` yaml title="NoiseDate transformer example"
- schema: "purchasing"
  name: "productvendor"
  transformers:
    - name: "NoiseFloat"
      params:
        column: "standardprice"
        ratio: 0.5
        precision: 2
```
