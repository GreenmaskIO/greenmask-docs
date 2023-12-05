### Description

Set NULL value to the column

| Name   | Description                                        | Default | Required | Supported DB types |
|--------|----------------------------------------------------|---------|----------|--------------------|
| column | name of column which value is going to be affected |         | Yes      | any                |

``` yaml title="SetNull transformer example"
- schema: "public"
  name: "order"
  transformers:
  - name: "SetNull"
    params:
      column: "updated_at"
```
