Add or subtract a random fraction to the original int value.

## Parameters

| Name   | Description                                                      | Default | Required | Supported DB types |
|--------|------------------------------------------------------------------|---------|----------|--------------------|
| column | The name of the column whose value will be affected              |         | Yes      | int2, int4, int8   |
| ratio  | The maximum random percentage for noise in value from `0` to `1` |         | Yes      | -                  |

## Description

The `NoiseInt` transformer multiplies the original integer value by a provided random value that is not higher than the
`ratio` parameter and adds it to the original value.

**Example**:

Original Value: `100`

Ratio: `0.1` (meaning add noise up to 10%)

Generated random value during execution: `0.05`

Transformation Result: `100 + (100 * 0.05) = 105`

The multiplied value might be either **added or subtracted**, resulting in `100 - (100 * 0.05) = 95`.

## Examples

### A. Noise Vocational Hours of Employee

In this example, the original value of `vacationhours` will be noised up to `50%`.

``` yaml title="NoiseInt transformer example"
- schema: "humanresources"
  name: "employee"
  transformers:
    - name: "NoiseInt"
      params:
        column: "vacationhours"
        ratio: 0.4
```
