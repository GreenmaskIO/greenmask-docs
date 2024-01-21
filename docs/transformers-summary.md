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
    * External `Custom Transformers`. Create [custom transformers](custom-transformers.md) using the Greenmask toolkit,
      supporting the custom
      transformer interaction protocol.

These categories provide a comprehensive framework for applying various obfuscation techniques to sensitive data.

## List of transformers

I've added all the transformers from the `FakerTransformersDes` map to the list of transformers. You can see them in the
table below:
I've added all the transformers from the `FakerTransformersDes` map to the list of transformers. You can see them in the
table below:

| Name                                                             | Description                                                                                                                                 | PostgreSQL Type                          | Transformation Method |
|------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------|------------------------------------------|-----------------------|
| [Cmd](built_in_transformers/cmd.md)                              | Transform data via external program using stdin and stdout interaction                                                                      | any                                      | user-defined          |
| [Dict](built_in_transformers/dict.md)                            | Replace values matched by dictionary keys                                                                                                   | any                                      | replace               |
| [Hash](built_in_transformers/dict.md)                            | Generate a hash of the text value                                                                                                           | text                                     | replace               |
| [Json](built_in_transformers/json.md)                            | Change a JSON document using `delete` and `set` operations                                                                                  | json                                     | edit                  |
| [Mask](built_in_transformers/masking.md)                         | Mask a value (insert `*` into original value) using one of the masking behaviors depending on your domain                                   | text                                     | edit                  |
| [NoiseDate](built_in_transformers/noise_date.md)                 | Add or subtract a random duration in the provided `ratio` interval to the original date value                                               | date, timestamp                          | noise                 |
| [NoiseFloat](built_in_transformers/noise_float.md)               | Add or subtract a random fraction to the original float value                                                                               | float                                    | noise                 |
| [NoiseInt](built_in_transformers/noise_int.md)                   | Add or subtract a random fraction to the original int value                                                                                 | int                                      | noise                 |
| [RandomBool](built_in_transformers/random_bool.md)               | Generate random boolean values                                                                                                              | boolean                                  | random                |
| [RandomChoice](built_in_transformers/random_choice.md)           | Replace values randomly chosen from a provided list                                                                                         | any                                      | replace               |
| [RandomFloat](built_in_transformers/random_float.md)             | Generate a random float within the provided interval.                                                                                       | float                                    | random                |
| [RandomInt](built_in_transformers/random_int.md)                 | Generate a random integer within the provided interval                                                                                      | int                                      | random                |
| [RandomString](built_in_transformers/random_string.md)           | Generate a random string using the provided characters within the specified length range.                                                   | text                                     | random                |
| [RandomUuid](built_in_transformers/random_uuid.md)               | Generate random uuid                                                                                                                        | text, uuid                               | random                |
| [RandomLatitude](built_in_transformers/random_latitude.md)       | Generate a random latitude value.                                                                                                           | float4, float8, numeric                  | random                |
| [RandomLongitude](built_in_transformers/random_longitude.md)     | Generate a random longitude value.                                                                                                          | float4, float8, numeric                  | random                |
| [RandomUnixTime](built_in_transformers/random_unix_time.md)      | Generate a random Unix timestamp.                                                                                                           | int4, int8, numeric                      | random                |
| [RandomMonthName](built_in_transformers/random_month_name.md)    | Generate the name of a random month.                                                                                                        | text, varchar                            | random                |
| [RandomYearString](built_in_transformers/random_year_string.md)  | Generate a random year as a string.                                                                                                         | text, varchar, int2, int4, int8, numeric | random                |
| [RandomDayOfWeek](built_in_transformers/random_day_of_week.md)   | Generate a random day of the week.                                                                                                          | text, varchar                            | random                |
| [RandomDayOfMonth](built_in_transformers/random_day_of_month.md) | Generate a random day of the month.                                                                                                         | text, varchar, int2, int4, int8, numeric | random                |
| [RandomCentury](built_in_transformers/random_century.md)         | Generate a random century.                                                                                                                  | text, varchar                            | random                |
| [RandomTimezone](built_in_transformers/random_timezone.md)       | Generate a random timezone.                                                                                                                 | text, varchar                            | random                |
| [RandomEmail](built_in_transformers/random_email.md)             | Generate a random email address.                                                                                                            | text, varchar                            | random                |
| [RandomMacAddress](built_in_transformers/random_mac_address.md)  | Generate a random MAC address.                                                                                                              | text, varchar, macaddr, macaddr8         | random                |
| [RandomDomainName](built_in_transformers/random_domain_name.md)  | Generate a random domain name.                                                                                                              | text, varchar                            | random                |
| [RandomURL](built_in_transformers/random_url.md)                 | Generate a random URL.                                                                                                                      | text, varchar                            | random                |
| [RandomUsername](built_in_transformers/random_username.md)       | Generate a random username.                                                                                                                 | text, varchar                            | random                |
| [RandomIPv4](built_in_transformers/random_ipv4.md)               | Generate a random IPv4 address.                                                                                                             | text, varchar, inet                      | random                |
| [RandomIPv6](built_in_transformers/random_ipv6.md)               | Generate a random IPv6 address.                                                                                                             | text, varchar, inet                      | random                |
| [RandomPassword](built_in_transformers/random_password.md)       | Generate a random password.                                                                                                                 | text, varchar                            | random                |
| [RandomWord](built_in_transformers/random_word.md)               | Generate a random word.                                                                                                                     | text, varchar                            | random                |
| [RandomSentence](built_in_transformers/random_sentence.md)       | Generate a random sentence.                                                                                                                 | text, varchar                            | random                |
| [RegexpReplace](built_in_transformers/regexp_replace.md)         | Replace string using regular expression                                                                                                     | text                                     | replace               |
| [Replace](built_in_transformers/replace.md)                      | Replace original value by the provided                                                                                                      | any                                      | replace               |
| [SetNull](built_in_transformers/set_null.md)                     | Set a `NULL` value to the column                                                                                                            | any                                      | replace               |
| [Template](built_in_transformers/template.md)                    | Execute a Go template and apply the result to a specified column                                                                            | any                                      | replace               |
| [TemplateRecord](built_in_transformers/template_record.md)       | Modify records using a Go template and apply changes via the PostgreSQL driver. Provides you a way to implement custom transformation logic | any                                      | user-defined          |
| [RealAddress](built_in_transformers/real_address.md)             | Generate real addresses for specified database columns using the `faker` library.                                                           | any                                      | user-defined          |