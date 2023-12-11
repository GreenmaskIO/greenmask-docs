The RandomString transformer generates a random string using the provided characters within the specified length range.

## Parameters

| Name       | Description                                                                  | Default                                                | Required | Supported DB types |
|------------|------------------------------------------------------------------------------|--------------------------------------------------------|----------|--------------------|
| column     | The name of the column whose value will be affected                          |                                                        | Yes      | text, varchar      |
| min_length | The minimum length of the generated string                                   |                                                        | Yes      | -                  |
| max_length | The maximum length of the generated string                                   |                                                        | Yes      | -                  |
| symbols    | The range of characters that can be used in the random string                | `abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ` | No       | -                  |
| keep_null  | Indicates whether NULL values should not be replaced with transformed values | `true`                                                 | No       | -                  |

## Description

The `RandomString` transformer generates a random string with a length between `min_length` and `max_length` using the
characters specified in the symbols string as the possible set of characters. The behavior for `NULL` values can be
configured using the `keep_null` parameter.

## Examples

### A. Generate random string for `accountnumber`

In this example, a random string is generated for the `accountnumber` column with a length range from `9` to `12`. The
character set used for generation includes `1234567890ABCDEFGHIJKLMNOPQRSTUVWXYZ`.

``` yaml title="RandomString transformer example"
- schema: "purchasing"
  name: "vendor"
  transformers:
    - name: "RandomString"
      params:
        column: "accountnumber"
        min_length: 9
        max_length: 12
        symbols: "1234567890ABCDEFGHIJKLMNOPQRSTUVWXYZ"
```

Expected results:

| column name   | original value | transformed |
|---------------|----------------|-------------|
| accountnumber | AUSTRALI0001   | 96B82A65548 |
