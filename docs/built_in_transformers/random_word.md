The RandomWord transformer populates specified database columns with random words. Ideal for simulating textual content, enhancing linguistic datasets, or anonymizing text in databases.

## Parameters

| Name       | Description                                           | Default | Required | Supported DB Types |
|------------|-------------------------------------------------------|---------|----------|--------------------|
| column     | The name of the column to be affected.                |         | Yes      | text, varchar      |
| keep_null  | Indicates whether NULL values should be preserved.   | `false` | No       | -                  |

## Description

The `RandomWord` transformer employs a mechanism to inject random words into a designated database column, supporting the generation of linguistically plausible and contextually diverse text. This transformer is particularly beneficial for creating rich text datasets for development, testing, or educational purposes without specifying the language, focusing on versatility and ease of use.

## Examples

### Populate Random Words for a Content Table

This example demonstrates configuring the `RandomWord` transformer to populate the `tag` column in a `content` table with random words. It's a straightforward approach to adding varied textual data for tagging or content categorization.

```yaml
- schema: "public"
  name: "content"
  transformers:
    - name: "RandomWord"
      params:
        column: "tag"
        keep_null: false
```

In this setup, the `tag` column will be updated with random words for each entry, replacing any existing non-NULL values. If `keep_null` is set to true, existing NULL values in the column will remain unchanged, maintaining data integrity for records where textual data is not applicable.
