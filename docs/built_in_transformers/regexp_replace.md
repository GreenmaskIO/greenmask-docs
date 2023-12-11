Replace string using regular expression

## Parameters

Parameters:

| Name    | Description                                                                                         | Default | Required | Supported DB types |
|---------|-----------------------------------------------------------------------------------------------------|---------|----------|--------------------|
| column  | The name of the column whose value will be affected                                                 |         | Yes      | text, varchar      |
| regexp  | The regular expression pattern to search for in the column's value                                  |         | Yes      | -                  |
| replace | The replacement value. This value may be replaced with a captured group from the `regexp` parameter |         | Yes      | -                  |

## Description

The syntax of the regular expressions accepted is the same general syntax used by Perl, Python, and other languages.
More precisely, it is the syntax accepted by RE2 and described at [golang doc](https://golang.org/s/re2syntax), except
for `\C`.

## Examples

### A. Removing leading prefix from loginid column value

In this example, the original `loginid` in format `adventure-works\{{ id_name }}` replacing to `{{ id_name }}`.

``` yaml title="RegexpReplace transformer example"
- schema: "humanresources"
  name: "employee"
  transformers:
  - name: "RegexpReplace"
    params:
      column: "loginid"
      regexp: "adventure-works\\\\(.*)"
      replace: "$1"
```

!!! note

    YAML has control symbols, and using them without escaping may cause an error. In this example, the prefix 
    of 'id' is separated by the `\` symbol. Since this symbol is a control symbol, we must escape it using `\\`. 
    However, the '\' symbol is also a control symbol for regular expressions, which is why we need to 
    double-escape it as `\\\\`.

Expected results:

| column name | original value       | transformed |
|-------------|----------------------|-------------|
| loginid     | adventure-works\ken0 | ken0        |
