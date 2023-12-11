Generate a random date in a specified interval.

## Parameters

| Name      | Description                                                                                                                                         | Default | Required | Supported DB types           |
|-----------|-----------------------------------------------------------------------------------------------------------------------------------------------------|---------|----------|------------------------------|
| column    | name of the column which value is going to be affected                                                                                              |         | Yes      | date, timestamp, timestamptz |
| min       | The minimum threshold date for the random value. The format depends on the column type                                                              |         | Yes      | -                            |
| max       | The maximum threshold date for the random value. The format depends on the column type.                                                             |         | Yes      | -                            |
| truncate  | Truncate the date to the specified part (`nano`, `second`, `minute`, `hour`, `day`, `month`, `year`). Truncate operation is not applied by default. |         | No       | -                            |
| keep_null | Indicates whether NULL values should not be replaced with transformed values.                                                                       | `true`  | No       | -                            |

## Description

The `RandomDate` transformer generates a random date within the provided interval, starting from `min` to `max`. It
can also perform date truncation up to the specified part of the date. The format of dates in the `min` and `max`
parameters must adhere to PostgreSQL types, including `DATE`, `TIMESTAMP WITHOUT TIMEZONE`,
or `TIMESTAMP WITH TIMEZONE`. The behavior for `NULL` values can be configured using the `keep_null` parameter. Truncate
operation `truncate` is not applied by default.

## Example

### A. Generate `modification` date

In this example, a random timestamp without timezone is generated for the `modifieddate` column within the range from
`2011-05-31 00:00:00` to `2013-05-31 00:00:00`, and the portion of the random value after the day is truncated.

``` yaml title="RandomDate transformer example"
- schema: "sales"
  name: "salesorderdetail"
  transformers:
    - name: "RandomDate"
      params:
        column: "modifieddate"
        keep_null: false
        min: "2011-05-31 00:00:00"
        max: "2013-05-31 00:00:00"
        truncate: "day"
```

Expected results:

| column name  | original value      | transformed         |
|--------------|---------------------|---------------------|
| modifieddate | 2007-06-23 00:00:00 | 2005-12-08 00:00:00 |
