### Description

Change json document using delete and set operations. Null values are kept.

Parameters:

| Name       | Description                                             | Default | Required | Supported DB types |
|------------|---------------------------------------------------------|---------|----------|--------------------|
| column     | name of column which value is going to be affected      |         | Yes      | json, jsonb        |
| operations | list of operations that contains editing delete and set |         | Yes      | -                  |

Operations structure:

* **operation** - name of operation. Can be on of `set` or `delete`
* **path** - path to the object within the document
* **value** - value to assign by provided path. Required only for `set` operation

The _Json_ transformer is based on [tidwall/sjson](https://github.com/tidwall/sjson) and supports the same path syntax.

Syntax rules:

* A path is a series of keys separated by a dot. The dot and colon characters can be escaped with `\`.
* The -1 key can be used to append a value to an existing array
* Normally numbers in the path are used for accessing the array, but if your document has int keys in the
  map colon symbol `:` before the number is required. For instance, we have a document

    ``` json
    {
      "values": {
        1: "test1",
        2: "test2"
      }
    }
    ```
  In this case, a path syntax for editing key `2` is `"values.:2"`

``` yaml title="Json transformer example"
- schema: "bookings"
  name: "aircrafts_data"
  transformers:
    - name: "Json"
      params:
        column: "model"
        operations:
          - operation: "set"
            path: "en"
            value: "Boeing 777-300-2023"
          - operation: "set"
            path: "details.preperties.1"
            value: {"name": "somename", "description": null}
          - operation: "delete"
            path: "values.:2"
```