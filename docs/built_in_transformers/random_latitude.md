The RandomLatitude transformer generates random latitude values for specified database columns. It's designed to support geographical data enhancements, particularly useful for applications requiring randomized but plausible geographical coordinates.

## Parameters

| Name      | Description                                          | Default | Required | Supported DB types |
|-----------|------------------------------------------------------|---------|----------|--------------------|
| column    | The name of the column to be affected.               |         | Yes      | float4, float8, numeric |
| keep_null | Indicates whether NULL values should be preserved.  | `false` | No       | -                  |

## Description

The `RandomLatitude` transformer utilizes the `faker` library to produce random latitude values within the range of -90 to +90 degrees. This transformer can be applied to columns designated to store geographical latitude information, enhancing data sets with randomized latitude coordinates.

## Examples

### A. Populate Random Latitude for Locations Table

This example demonstrates configuring the `RandomLatitude` transformer to populate the `latitude` column in a `locations` table with random latitude values.

```yaml
- schema: "public"
  name: "locations"
  transformers:
    - name: "RandomLatitude"
      params:
        column: "latitude"
        keep_null: false
```

With this configuration, the `latitude` column will be filled with random latitude values, replacing any existing non-NULL values. If `keep_null` is set to `true`, existing NULL values will be preserved.
