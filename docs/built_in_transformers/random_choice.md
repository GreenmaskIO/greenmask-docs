Replace values randomly chosen from a provided list.

## Parameters

| Name      | Description                                                                                                                | Default | Required | Supported DB types |
|-----------|----------------------------------------------------------------------------------------------------------------------------|---------|----------|--------------------|
| column    | The name of the column whose value will be affected                                                                        |         | Yes      | any                |
| values    | A list of values in any format. The string with value `\N` is treated as a NULL value                                      |         | Yes      | -                  |
| validate  | Perform a decoding procedure via the PostgreSQL driver using the column type to ensure that the value has the correct type |         |          |                    |
| keep_null | Indicates whether NULL values should not be replaced with transformed values                                               |         |          |                    |

## Description

The `RandomChoice` transformer replaces a randomly chosen value from the list provided in the `values` parameter. You
can use the `validate` parameter to validate the `values` format before applying the transformation, which involves a
decoding procedure via the PostgreSQL driver. The behavior for `NULL` values can be configured using the `keep_null`
parameter.

## Examples

### A. Randomly Choosing from Provided Dates

In this example, the provided values undergo validation through PostgreSQL driver decoding, and one value is randomly
chosen from the list.

```yaml
    - schema: "humanresources"
      name: "jobcandidate"
      transformers:
        - name: "RandomChoice"
          params:
            column: "modifieddate"
            validate: true
            values:
              - "2023-12-21 07:41:06.891"
              - "2023-12-21 07:41:06.896"
```

### B. Validation error

This example raises a validation error due to a type mismatch in the `TIMESTAMP` value `2023-12-21a 07:41:06.891`.

```yaml
    - schema: "humanresources"
      name: "jobcandidate"
      transformers:
        - name: "RandomChoice"
          params:
            column: "modifieddate"
            validate: true
            values:
              - "2023-12-21a 07:41:06.891"
```

```json
{
  "hash": "7f7ae501ca350c6277d017962ebe5b97",
  "meta": {
    "Error": "\"unable to decode value: decoding error: parsing time \"2023-12-21a 07:41:06.891\" as \"2006-01-02 15:04:05.999999999\": cannot parse \"a 07:41:06.891\" as \" \"\"",
    "ParameterItemValue[0]": "2023-12-21a 07:41:06.891",
    "ParameterName": "values",
    "SchemaName": "humanresources",
    "TableName": "jobcandidate",
    "TransformerName": "RandomChoice"
  },
  "msg": "error validating value: driver decoding error",
  "severity": "error"
}
```
