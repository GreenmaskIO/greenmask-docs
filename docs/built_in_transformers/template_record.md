### Description

Modify the record using gotemplate and apply changes via PostgreSQL driver. This transformer allows to implement
custom transformation logic.

!!! warning
    Note that `TemplateRecord` transformer distinguishes from `Template`. `Template` transformer applying go
    template **result** to the provided column, but `TemplateRecord` does not use template result, instead of
    the user defined template is able to apply the value via driver functions `.SetValue` or `.SetRawValue`.

| Name     | Description                                                                                                                                          | Default | Required | Supported DB types |
|----------|------------------------------------------------------------------------------------------------------------------------------------------------------|---------|----------|--------------------|
| column   | columns that supposed to be affected by the template. The list of columns will be checked for constraint violation                                   |         | False    | any                |
| template | gotemplate string                                                                                                                                    |         | Yes      |                    |
| validate | validate template result via PostgreSQL driver decoding procedure. This may cause an error if custom type does not has encode-decoder implementation | false   |          |                    |

``` yaml title="Template transformer example"
- name: "Template"
  params:
    columns: 
     - "updated_at"
    template: >
      {{ $val := .GetColumnValue "date_ts" }}
	  {{ if isNull $val }}
		{{ "2023-11-20 01:00:00" | .DecodeValueByColumn "date_ts" | dateModify "24h" | .SetColumnValue "date_ts" }}
	  {{ else }}
		 {{ "2023-11-20 01:00:00" | .DecodeValueByColumn "date_ts" | dateModify "48h" | .SetColumnValue "date_ts" }}
	  {{ end }}
    validate: true
```

Have a read about [provided functions and hints](../template_functions.md).

PostgreSQL driver functions provided by `Template` transformer:

* `.GetColumnType(name string) (typeName string, error err)` - returns string with column type.
* `.GetColumnValue` - returns a value and error for a specific column, encoded by PostgreSQL driver into
  any type and error It might be on of `int, float, time, string, bool` and `slice` or `map` of any type
* `.GetRawColumnValue` - returns a raw value as a string and error for a specific column
* `.SetColumnValue` - set new value to the column
* `.SetRawColumnValue` - set new raw value (string) to the column
* `.EncodeValueByColumn` - encodes value from any type to the string representation using provided column name
* `.DecodeValueByColumn` - decode value from raw string representation to golang type using provided column name
* `.EncodeValueByType` - encodes value from any type to the string representation using provided column type name
* `.DecodeValueByType` - decode value from raw string representation to golang type using provided type name

### Use cases
