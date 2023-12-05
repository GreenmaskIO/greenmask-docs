### Description

Replace a column value with the provided.

| Name      | Description                                                                                                | Default | Required | Supported DB types |
|-----------|------------------------------------------------------------------------------------------------------------|---------|----------|--------------------|
| column    | name of column which value is going to be affected                                                         |         | Yes      | any                |
| replace   | value to replace. The value must be literally the same as the column type otherwise it may cause an error. |         | Yes      | -                  |
| keep_null | do not replace NULL values to random value                                                                 | `true`  | No       | -                  |

``` yaml title="Replace transformer example"
- schema: "public"
  name: "order"
  transformers:
  - name: "Replace"
    params:
      column: "created_at"
      value: "2023-11-07 15:22:21.885053+00"
      keep_null: false
```
