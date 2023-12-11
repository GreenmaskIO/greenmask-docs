Generate a hash of the text value using the `Scrypt` hash function under the hood. `NULL` values are kept.

## Parameters

| Name   | Description                                                                   | Default | Required | Supported DB types |
|--------|-------------------------------------------------------------------------------|---------|----------|--------------------|
| column | The name of the column whose value will be affected                           |         | Yes      | text, varchar      |
| salt   | Salt for hash function, base64 encoded. By default, it is generated randomly. |         | No       | -                  |

## Examples

### A. Generate hash from job title

This example generates a hash from the `jobtitle` using the encoded string `"12345678"` into base64
`Zpmfe8F+LVMKjXiKEP2P8WfMW3aXlwilQftu1mOLESg=`.

``` yaml title="Hash transformer example"
- schema: "humanresources"
  name: "employee"
  transformers:
    - name: "Hash"
      params:
        column: "jobtitle"
        salt: 12345678
```

Expected results:

| column name | original value           | transformed                                  |
|-------------|--------------------------|----------------------------------------------|
| jobtitle    | Research and Development | Zpmfe8F+LVMKjXiKEP2P8WfMW3aXlwilQftu1mOLESg= |
