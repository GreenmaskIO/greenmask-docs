### Description

Generate the hash of the text value using the Scrypt hash function under the hood. Null values are kept.

Parameters:

| Name   | Description                                                                    | Default | Required | Supported DB types |
|--------|--------------------------------------------------------------------------------|---------|----------|--------------------|
| column | name of column which value is going to be affected                             |         | Yes      | text, varchar      |
| salt   | random salt for hash function as a text string. By default generates randomly. |         | No       | -                  |

``` yaml title="Hash transformer example"
- schema: "bookings"
  name: "booking"
  transformers:
    - name: "Hash"
      params:
        column: "name"
        salt: "12345678"
```
