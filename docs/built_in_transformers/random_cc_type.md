The RandomCCType transformer is designed to populate specified database columns with random credit card types. This tool is essential for applications that require the simulation of financial transaction data, testing payment processing systems, or anonymizing credit card type information in datasets.

## Parameters

| Name       | Description                                           | Default | Required | Supported DB Types |
|------------|-------------------------------------------------------|---------|----------|--------------------|
| column     | The name of the column to be affected.                |         | Yes      | text, varchar      |
| keep_null  | Indicates whether NULL values should be preserved.   | `false` | No       | -                  |

## Description

Utilizing a predefined list of credit card types (e.g., Visa, MasterCard, American Express, Discover), the `RandomCCType` transformer injects random credit card type names into the designated database column. This feature allows for the creation of realistic and varied financial transaction datasets by simulating a range of credit card types without using real card data.

## Examples

### Populate Random Credit Card Types for a Transactions Table

This example shows how to configure the `RandomCCType` transformer to populate the `card_type` column in a `transactions` table with random credit card types. It's a straightforward method for simulating diverse payment methods across transactions.

```yaml
- schema: "public"
  name: "transactions"
  transformers:
    - name: "RandomCCType"
      params:
        column: "card_type"
        keep_null: false
```

In this configuration, the `card_type` column will be updated with random credit card types for each entry, replacing any existing non-NULL values. If the `keep_null` parameter is set to true, existing NULL values in the column will be preserved, maintaining the integrity of records where card type information is not applicable.
