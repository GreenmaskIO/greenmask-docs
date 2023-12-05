### Description

Replace values matched by dictionary keys

Parameters:

| Name             | Description                                                                                                                            | Default | Required | Supported DB types |
|------------------|----------------------------------------------------------------------------------------------------------------------------------------|---------|----------|--------------------|
| column           | name of column which value is going to be affected                                                                                     |         | Yes      | any                |
| values           | map of value to replace in format: `{"string": "string"}`. The string with value `"\N"` is supposed to be a NULL value                 |         | No       | -                  |
| default          | default value if not any value has been matched with dict. The string with value `"\N"` is supposed to be NULL value. Default is empty |         | No       | -                  |
| fail_not_matched | fail if value is not matched with dict otherwise keep value                                                                            | `false` | No       | -                  |
| validate         | perform encode-decode procedure using column type, ensuring that value has correct type                                                | `true`  | No       | -                  |

For setting or matching _NULL_ value use string with `\N` sequence.

``` yaml title="Dict transformer example"
- schema: "public"
  name: "order"
  transformers:
    - name: "Dict"
      params:
        column: "type_name"
        values: 
          "type1": "random_type1"
          "type2": "random_type2"
        default: "unknown_type"
        validate: true
        fail_not_matched: false
```
