Generates real addresses for specified database columns using the `faker` library. It supports customization of the generated address format through Go templates.

## Parameters

| Name      | Description                                                                                   | Default | Required | Supported DB types |
|-----------|-----------------------------------------------------------------------------------------------|---------|----------|--------------------|
| columns   | Specifies the affected column names along with additional properties for each column. Details: |         | Yes      | Various            |

- **Column Properties**:
    - `name`: The name of the column (string, required, description: column name).
    - `template`: Go template string for formatting real address attributes (string, required).
    - `keep_null`: Indicates whether NULL values should be kept as NULL (bool, optional).

## Template Value Descriptions

The `template` parameter allows for the injection of real address attributes into a customizable template. Here are the possible values you can include in your template:

- `{{.Address}}`: Street address or equivalent.
- `{{.City}}`: City name.
- `{{.State}}`: State, province, or equivalent region name.
- `{{.PostalCode}}`: Postal or ZIP code.
- `{{.Latitude}}`: Geographic latitude.
- `{{.Longitude}}`: Geographic longitude.

These placeholders can be combined and formatted as desired within the template string to generate custom address formats.

## Description

The `RealAddress` transformer uses the `faker` library to generate realistic addresses, which can then be formatted according to a specified template and applied to selected columns in a database. It allows for the generated addresses to replace existing values or to preserve NULL values, based on the transformer's configuration.

## Examples

### A. Generate Real Addresses for User Profiles

This example shows how to configure the `RealAddress` transformer to generate real addresses for the `address` column in a `employee` table, using a custom format.

```yaml
- schema: "humanresources"
  name: "employee"
  transformers:
    - name: "RealAddress"
      params:
        columns:
          - name: "address"
            template: "{{.Address}}, {{.City}}, {{.State}} {{.PostalCode}}"
            keep_null: false
```

This configuration will generate real addresses with the format "Street Address, City, State PostalCode" and apply them to the `address` column, replacing any existing non-NULL values.
