The `SetNull` transformer enables you to assign a `NULL` value to the column.

## Parameters

| Name   | Description                                         | Default | Required | Supported DB types |
|--------|-----------------------------------------------------|---------|----------|--------------------|
| column | The name of the column whose value will be affected |         | Yes      | any                |

## Description

Use that transformer for setting `NULL` value to column. This transformer generates warning if the column that is
transformed has `NOT NULL` constraint.

```json title="NULL constraint violation warning"
{
  "hash": "5a229ee964a4ba674a41a4d63dab5a8c",
  "meta": {
    "ColumnName": "jobtitle",
    "ConstraintType": "NotNull",
    "ParameterName": "column",
    "SchemaName": "humanresources",
    "TableName": "employee",
    "TransformerName": "SetNull"
  },
  "msg": "transformer may produce NULL values but column has NOT NULL constraint",
  "severity": "warning"
}
```

## Examples

### A. Set null value to updated_at column

``` yaml title="SetNull transformer example"
- schema: "humanresources"
  name: "employee"
  transformation:
    - name: "SetNull"
      params:
        column: "jobtitle"
```

Expected results:

| column name | original value          | transformed |
|-------------|-------------------------|-------------|
| jobtitle    | Chief Executive Officer | NULL        |
