Mask a value using one of the masking behaviors depending on your domain. `NULL` values are kept.

## Parameters

| Name   | Description                                                                                                     | Default   | Required | Supported DB types |
|--------|-----------------------------------------------------------------------------------------------------------------|-----------|----------|--------------------|
| column | The name of the column whose value will be affected                                                             |           | Yes      | text, varchar      |
| type   | Data type of attribute (`default`, `password`, `name`, `addr`, `email`, `mobile`, `tel`, `id`, `credit`, `url`) | `default` | No       | -                  |

## Description

The `Masking` transformer replaces characters with asterisk `*` symbols depending on the provided data type. If the
value is `NULL`, it is kept unchanged. It is based on [ggwhite/go-masker](https://github.com/ggwhite/go-masker) and
supports the next masking rules:

|    Type     | Description                                                                                                                                                  |
|:-----------:|:-------------------------------------------------------------------------------------------------------------------------------------------------------------|
|   default   | Returns `*` symbols with the same length. Example: input: `test1234` output: `********`                                                                      |
|    name     | Mask the second letter and the third letter. Example: input: `ABCD` output: `A**D`                                                                           |
|  password   | Always return `************`                                                                                                                                 |
|   address   | Keep first 6 letters, mask the rest. Example: input: `Larnaca, makarios st` output: `Larnac*************`                                                    |
|    email    | Keep domain and the first 3 letters. Example: input: `ggw.chang@gmail.com` output: `ggw****@gmail.com`                                                       |
|   mobile    | Mask 3 digits from the 4'th digit. Example: input: `0987654321` output: `0987***321`                                                                         |
|  telephone  | Remove `(`, `)`, ` `, `-` chart, and mask last 4 digits of telephone number, format to `(??)????-????`. Example: input: `0227993078` output: `(02)2799-****` |
|     id      | Mask last 4 digits of ID number. Example: input: `A123456789` output: `A12345****`                                                                           |
| credit_cart | Mask 6 digits from the 7'th digit. Example: input `1234567890123456` output `123456******3456`                                                               |
|     url     | Mask the password part of the URL if exists. input: `http://admin:mysecretpassword@localhost:1234/uri` output: `http://admin:xxxxx@localhost:1234/uri`       |

## Example

### A. Masking employee national id number

This example masks the national ID number of an employee.

``` yaml title="Masking transformer example"
- schema: "humanresources"
  name: "employee"
  transformers:
    - name: "Masking"
      params:
        column: "nationalidnumber"
        type: "id"
```

Expected results:

| column name      | original value | transformed |
|------------------|----------------|-------------|
| nationalidnumber | 295847284      | 295847****  |

