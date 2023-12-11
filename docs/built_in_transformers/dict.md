Replace values matched by dictionary keys

## Parameters

| Name             | Description                                                                                                                            | Default | Required | Supported DB types |
|------------------|----------------------------------------------------------------------------------------------------------------------------------------|---------|----------|--------------------|
| column           | The name of the column whose value will be affected                                                                                    |         | Yes      | any                |
| values           | map of value to replace in format: `{"string": "string"}`. The string with value `"\N"` is supposed to be a NULL value                 |         | No       | -                  |
| default          | default value if not any value has been matched with dict. The string with value `"\N"` is supposed to be NULL value. Default is empty |         | No       | -                  |
| fail_not_matched | fail if value is not matched with dict otherwise keep value                                                                            | `true`  | No       | -                  |
| validate         | perform encode-decode procedure using column type, ensuring that value has correct type                                                | `true`  | No       | -                  |

## Description

The `Dict` transformer uses provided dictionary (key-value) and replace the value by match provided in the `values`
preparer. The provided values must be compatible with PostgreSQL type format. If you want to validate the values format
before applying you can use `validate` parameter that will go through decoding procedure via PostgreSQL driver.
If there are no matches by key the error will be raised if there is no default value provided by `default` parameter. If
you want to change that behaviour change set `fail_not_matched: true`. In some cases it is unknown for driver
type `validation` operation is not available and may end up with error. For setting or matching `NULL` value use string
with `\N` sequence.

## Examples

### A. Replace marital status

This example replaces marital status `M` to `S` or `M` to `S`, raises an error if there is no match.

``` yaml title="Dict transformer example"
- schema: "humanresources"
  name: "employee"
  transformers:
    - name: "Dict"
      params:
        column: "maritalstatus"
        values: 
          "S": "M"
          "M": "S"
        validate: true
        fail_not_matched: true
```

### B. Validation error

This example shows validation error caused by type if the key and or value in dictionary.

``` yaml title="Dict transformer example"
- schema: "humanresources"
  name: "employee"
  transformers:
    - name: "Dict"
      params:
        column: "birthdate"
        values: 
          "1991-05-31aa": "a1991-05-31"
        default: "1991-jnd-fj"
        validate: true
        fail_not_matched: true
```

Caused validation warning with `error` severity

```json title="key validation error"
{
  "hash": "93337f8814822c0ca3cce20da11be2d7",
  "meta": {
    "Error": "\"unable to decode value: decoding error: invalid date format\"",
    "KeyValue": "1991-05-31aa",
    "ParameterName": "values",
    "SchemaName": "humanresources",
    "TableName": "employee",
    "TransformerName": "Dict"
  },
  "msg": "error validating values: error encoding key",
  "severity": "error"
}
```

```json title="value validation error"
{
  "hash": "77ddf7b878cfd368b53aa1b9158c512c",
  "meta": {
    "Error": "\"unable to decode value: decoding error: invalid date format\"",
    "ParameterName": "values",
    "SchemaName": "humanresources",
    "TableName": "employee",
    "TransformerName": "Dict",
    "ValueValue": "a1991-05-31"
  },
  "msg": "error validating values: error encoding value",
  "severity": "error"
}
```

```json title="default value validation err"
{
  "hash": "86a2c34ceffe7cb4606fc2b8d8d24eb0",
  "meta": {
    "ParameterValue": "1991-jnd-fj",
    "Error": "\"unable to decode value: decoding error: invalid date format\"",
    "ParameterName": "default_value",
    "SchemaName": "humanresources",
    "TableName": "employee",
    "TransformerName": "Dict"
  },
  "msg": "error validating \"default_value\"",
  "severity": "error"
}
```
