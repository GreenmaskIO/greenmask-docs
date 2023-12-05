### Description

Generate a random string using the provided characters.

Parameters:

| Name       | Description                                        | Default                                                | Required | Supported DB types |
|------------|----------------------------------------------------|--------------------------------------------------------|----------|--------------------|
| column     | name of column which value is going to be affected |                                                        | Yes      | text, varchar      |
| min_length | min length of string                               |                                                        | Yes      | -                  |
| max_length | max length of string                               |                                                        | Yes      | -                  |
| symbols    | the characters range for random string             | `abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ` | No       | -                  |
| keep_null  | do not replace NULL values to random value         | `true`                                                 | No       | -                  |

``` yaml title="RandomString transformer example"
- schema: "public"
  name: "order"
  transformers:
    - name: "RandomString"
      params:
        column: "description"
        min_length: 10
        max_length: 100
        symbols: "1234567890-"
```
