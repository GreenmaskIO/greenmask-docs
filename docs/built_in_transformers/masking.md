### Description

Mask a value using one of the masking behavior depending on your domain. Null values are kept.

Parameters:

| Name   | Description                                                                                                        | Default   | Required | Supported DB types |
|--------|--------------------------------------------------------------------------------------------------------------------|-----------|----------|--------------------|
| column | name of column which value is going to be affected                                                                 |           | Yes      | text, varchar      |
| type   | logical type of attribute (_default_, _password_, _name_, _addr_, _email_, _mobile_, _tel_, _id_, _credit_, _url_) | `default` | No       | -                  |

The _Masking_ transformer based on [ggwhite/go-masker](https://github.com/ggwhite/go-masker) and supports the next
masking rules:

|    Type     | Description                                                                                                                                                  |
|:-----------:|:-------------------------------------------------------------------------------------------------------------------------------------------------------------|
|   default   | returns `*` symbols with the same length. Example: input: `test1234` output: `********`                                                                      |
|    name     | mask the second letter and the third letter. Example: input: `ABCD` output: `A**D`                                                                           |
|  password   | always return `************`                                                                                                                                 |
|   address   | keep first 6 letters, mask the rest. Example: input: `Larnaca, makarios st` output: `Larnac*************`                                                    |
|    email    | keep domain and the first 3 letters. Example: input: `ggw.chang@gmail.com` output: `ggw****@gmail.com`                                                       |
|   mobile    | mask 3 digits from the 4'th digit. Example: input: `0987654321` output: `0987***321`                                                                         |
|  telephone  | remove `(`, `)`, ` `, `-` chart, and mask last 4 digits of telephone number, format to `(??)????-????`. Example: input: `0227993078` output: `(02)2799-****` |
|     id      | mask last 4 digits of ID number. Example: input: `A123456789` output: `A12345****`                                                                           |
| credit_cart | mask 6 digits from the 7'th digit. Example: input `1234567890123456` output `123456******3456`                                                               |
|     url     | mask the password part of the URL if exists. input: http://admin: `mysecretpassword@localhost:1234/uri` output: `http://admin:xxxxx@localhost:1234/uri`      |

``` yaml title="Masking transformer example"
- schema: "public"
  name: "order"
  transformers:
    - name: "Masking"
      params:
        column: "user_name"
        type: "name"
```
