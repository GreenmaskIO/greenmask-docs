## Methods

Greenmask categorizes obfuscation methods into the following categories:

* `random` - Generate a new value within a provided interval.
* `noise` - Add or subtracts a randomly generated noisy value to the original.
* `replace` - Replace the original value with a provided value.
* `edit` - Partially or fully change the original value.
* `user-defined` - Custom obfuscation logic implemented by `Greenmask` users. There are three ways to implement custom
  obfuscation logic:
    * External Program and `Cmd` Transformer Usage: Utilize an external program in conjunction with the Cmd transformer.
    * `TemplateRecord` transformer Usage: Perform multi-column transformations using pre-defined template functions.
    * External `Custom Transformers`. Create [custom transformers](custom-transformers.md) using the Greenmask toolkit, supporting the custom
      transformer interaction protocol.

These categories provide a comprehensive framework for applying various obfuscation techniques to sensitive data.

## List of transformers

| Name                                                       | Description                                                                                                                                 | PostgreSQL type | Transformation method |
|------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------|-----------------|-----------------------|
| [Cmd](built_in_transformers/cmd.md)                        | Transform data via external program using stdin and stdout interaction                                                                      | any             | user-defined          |
| [Dict](built_in_transformers/dict.md)                      | Replace values matched by dictionary keys                                                                                                   | any             | replace               |
| [Hash](built_in_transformers/dict.md)                      | Generate a hash of the text value                                                                                                           | text            | replace               |
| [Json](built_in_transformers/json.md)                      | Change a JSON document using `delete` and `set` operations                                                                                  | json            | edit                  |
| [Mask](built_in_transformers/masking.md)                   | Mask a value (insert `*` into original value) using one of the masking behaviors depending on your domain                                   | text            | edit                  |
| [NoiseDate](built_in_transformers/noise_date.md)           | Add or subtract a random duration in the provided `ratio` interval to the original date value                                               | date, timestamp | noise                 |
| [NoiseFloat](built_in_transformers/noise_float.md)         | Add or subtract a random fraction to the original float value                                                                               | float           | noise                 |
| [NoiseInt](built_in_transformers/noise_int.md)             | Add or subtract a random fraction to the original int value                                                                                 | int             | noise                 |
| [RandomBool](built_in_transformers/random_bool.md)         | Generate random boolean values                                                                                                              | boolean         | random                |
| [RandomChoice](built_in_transformers/random_choice.md)     | Replace values randomly chosen from a provided list                                                                                         | any             | replace               |
| [RandomFloat](built_in_transformers/random_float.md)       | Generate a random float within the provided interval.                                                                                       | float           | random                |
| [RandomInt](built_in_transformers/random_int.md)           | Generate a random integer within the provided interval                                                                                      | int             | random                |
| [RandomString](built_in_transformers/random_string.md)     | Generate a random string using the provided characters within the specified length range.                                                   | text            | random                |
| [RandomUuid](built_in_transformers/random_uuid.md)         | Generate random uuid                                                                                                                        | text, uuid      | random                |
| [RegexpReplace](built_in_transformers/regexp_replace.md)   | Replace string using regular expression                                                                                                     | text            | replace               |
| [Replace](built_in_transformers/replace.md)                | Replace original value by the provided                                                                                                      | any             | replace               |
| [SetNull](built_in_transformers/set_null.md)               | Set a `NULL` value to the column                                                                                                            | any             | replace               |
| [Template](built_in_transformers/template.md)              | Execute a Go template and apply the result to a specified column                                                                            | any             | replace               |
| [TemplateRecord](built_in_transformers/template_record.md) | modify records using a Go template and apply changes via the PostgreSQL driver. Provides you a way to implement custom transformation logic | any             | user-defined          |
 | [RealAddress](built_in_transformers/real_address.md)       | Generate real addresses for specified database columns using the `faker` library.                                                           | any             | random                |
