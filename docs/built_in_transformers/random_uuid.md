Generate random uuid using version 4.

## Parameters

| Name      | Description                                                                  | Default | Required | Supported DB types  |
|-----------|------------------------------------------------------------------------------|---------|----------|---------------------|
| column    | The name of the column whose value will be affected                          |         | Yes      | text, varchar, uuid |
| keep_null | Indicates whether NULL values should not be replaced with transformed values | `true`  | No       | -                   |

# Description

The `RandomUuid` transformer generates a random UUID. The behavior for `NULL` values can be configured using
the `keep_null` parameter.

## Examples

### A. Updating the "rowguid" Column

This example replaces original UUID value of column `rowguid` to randomly generated.

``` yaml title="RandomUuid transformer example"
- schema: "humanresources"
  name: "employee"
  transformers:
  - name: "RandomUuid"
    params:
      column: "rowguid"
      keep_null: false
```

Expected results:

| column name | original value                       | transformed                          |
|-------------|--------------------------------------|--------------------------------------|
| rowguid     | f01251e5-96a3-448d-981e-0f99d789110d | 0211629f-d197-4187-8a87-095ec4f51977 |
