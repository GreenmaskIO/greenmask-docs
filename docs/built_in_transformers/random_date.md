### Description

Generate a random date in the provided interval

Parameters:

| Name      | Description                                                                                                                                                       | Default | Required | Supported DB types           |
|-----------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------|----------|------------------------------|
| column    | name of the column which value is going to be affected                                                                                                            |         | Yes      | date, timestamp, timestamptz |
| min       | min threshold date (and/or time) of random value. The format depends on the column type.                                                                          |         | Yes      | -                            |
| max       | max threshold date (and/or time) of random value. The format depends on the column type.                                                                          |         | Yes      | -                            |
| truncate  | truncate date till the part (_year_, _month_, _day_, _hour_, _second_, _millisecond_, _microsecond_, _nanosecond_). Truncate operation is not applied by default. |         | No       | -                            |
| keep_null | do not replace NULL values to random value                                                                                                                        | `true`  | No       | -                            |

``` yaml title="RandomDate transformer example"
- schema: "public"
  name: "order"
  transformers:
    - name: "RandomDate"
      params:
        column: "created_at"
        keep_null: false
        min: "2023-01-03 01:00:00.0+03"
        max: "2023-01-04 00:00:00.0+03"
        truncate: "second"
```
