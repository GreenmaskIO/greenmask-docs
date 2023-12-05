### Description

Generate random bool value.

Parameters:

| Name      | Description                                            | Default | Required | Supported DB types |
|-----------|--------------------------------------------------------|---------|----------|--------------------|
| column    | name of the column which value is going to be affected |         | Yes      | bool               |
| keep_null | do not replace NULL values to random value             | `true`  | No       | -                  |

``` yaml title="RandomBool transformer example"
- schema: "public"
  name: "order"
  transformers:
    - name: "RandomBool"
      params:
        column: "is_active"
        keep_null: false
```
