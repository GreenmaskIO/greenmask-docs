### Description

Generate random uuid using version 4.

| Name      | Description                                        | Default | Required | Supported DB types  |
|-----------|----------------------------------------------------|---------|----------|---------------------|
| column    | name of column which value is going to be affected |         | Yes      | text, varchar, uuid |
| keep_null | do not replace NULL values to random value         | `true`  | No       | -                   |

``` yaml title="RandomUuid transformer example"
- schema: "public"
  name: "order"
  transformers:
  - name: "RandomUuid"
    params:
      column: "transcaction_id"
      keep_null: false
```
