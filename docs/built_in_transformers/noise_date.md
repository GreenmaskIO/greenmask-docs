### Description

Add some random value (shift value) in the provided interval. The value might be shifted either to before this
date or after.

Parameters:

| Name     | Description                                                                                                                                                                                                   | Default | Required | Supported DB types           |
|----------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------|----------|------------------------------|
| column   | name of column which value is going to be affected                                                                                                                                                            |         | Yes      | date, timestamp, timestamptz |
| ratio    | max random duration for noise. The value has [PostgreSQL interval format](https://www.postgresql.org/docs/current/datatype-datetime.html#DATATYPE-INTERVAL-INPUT). Example: `1 year 1 mons 1 day 01:01:01.01` |         | Yes      | -                            |
| truncate | truncate date till the part (_year_, _month_, _day_, _hour_, _second_, _millisecond_, _microsecond_, _nanosecond_). Truncate operation is not applied by default.                                             |         | No       | -                            |

``` yaml title="NoiseDate transformer example"
- schema: "public"
  name: "order"
  transformers:
    - name: "NoiseDate"
      params:
        column: "created_at"
        ratio: "1 year 1 mons 1 day 01:01:01.01"
        truncate: "microsecond"
```
