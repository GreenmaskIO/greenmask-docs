### Description

Replace string using regular expression

The syntax of the regular expressions accepted is the same general syntax used by Perl, Python, and other languages.
More precisely, it is the syntax accepted by RE2 and described at [golang doc](https://golang.org/s/re2syntax), except for `\C`.

Parameters:

| Name    | Description                                                                         | Default | Required | Supported DB types |
|---------|-------------------------------------------------------------------------------------|---------|----------|--------------------|
| column  | name of column which value is going to be affected                                  |         | Yes      | text, varchar      |
| regexp  | regular expression                                                                  |         | Yes      | -                  |
| replace | replacement value. The value may be replaced with the group from `regexp` parameter |         | Yes      | -                  |

``` yaml title="RegexpReplace transformer example"
- schema: "public"
  name: "order"
  transformers:
  - name: "RegexpReplace"
    params:
      column: "description"
      regexp: "(Hello)\s*world\s*(\!+\?)"
      replace: "$1 Mr NoName $2"
```
