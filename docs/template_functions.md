# Template functions and hints

Greenmask uses [sprig template functions](https://masterminds.github.io/sprig/) under the hood that provided you
variety of ways to implement almost any logic. Read the sprig documentation for getting knowledge of all available
functions. Greenmask extends it by providing additional function and transformation function that repeats logic
of [built-in transformers](transformers-summary.md) and allows you to implement comprehensive and complicated logic for
your relation.

Currently, go templates utilized in the next transformers:

* Template
* TemplateRecord
* Json

## List of the custom functions

There is a list of the all provided template functions grouped by categories.

### PostgreSQL driver

#### Template transformer functions

Read details [here](built_in_transformers/template.md/#template-functions).

#### RecordTemplate transformer functions

Read details [here](built_in_transformers/template_record.md/#template-functions).

#### Json transformer functions

Read details [here](built_in_transformers/json.md/#template-functions).

#### Json transformation

#### null

Returns a `NULL` value that could be used together with Driver encoding-decoding operations.

Example:

```gotemplate
{{ if eq .GetColumnValue "category" "film" }}
{{ null | .SetColumnValue "category" }}
{{ end }}
```

#### isNull

Returns true if the checking value is `NULL`

Example:

```gotemplate
{{ if  isNull .GetColumnValue "category"  }}
{{ "replaced_category" | .SetColumnValue "category" }}
{{ end }}
```

#### isNotNull

Returns true if the checking value is not `NULL`

Example:

```gotemplate
{{ if  isNotNull .GetColumnValue "category"  }}
{{ .GetColumnValue "category" | cat "prefix_" | .SetColumnValue "category" }}
{{ end }}
```

#### sqlCoalesce

Works as `coalesce` function in any SQL. It allows you to choose the first not NULL argument from provided

Example:

```gotemplate
{{ sqlCoalesce .GetColumnValue "category" "other_not_null_value" }}
```

In this case if column `category` has a NULL value, than the string with value `"other_not_null_value"` can be chosen.

### Json

#### jsonExists

Test that json contains the value by path. Return true if the path exists.

Example:

```gotemplate
{{ $jsonValue := "{\"a\": 1}" }}
{{ if jsonExists "a" $jsonValue }}
{{ mustJsonSet "a" 0 $jsonValue | .SetColumnValue "json_doc" }}
{{ end }}
```

#### mustJsonGet

Get the json attribute value by path and raise an error if the path does not exist

Example:

```gotemplate
{{ $jsonValue := "{\"a\": 1}" }}
{{ if jsonExists "a" $jsonValue }}
{{ mustJsonGet "a" $jsonValue | jsonSet "b" | .SetColumnValue "json_doc" }}
{{ end }}
```

#### mustJsonGetRaw

Get json attribute raw value by path and raise an error if the path does not exist

Example:

```gotemplate
{{ $jsonValue := "{\"a\": 1}" }}
{{ if jsonExists "a" $jsonValue }}
{{ mustJsonGetRaw "a" $jsonValue | jsonSetRaw "b" | .SetRawColumnValue "json_doc" }}
{{ end }}
```

#### jsonGet

Get json attribute value by path, return nil if value does not exist

Example:

```gotemplate
{{ $jsonValue := "{\"a\": 1}" }}
{{ if eq jsonGet "b" nil }}
{{ jsonSet "b" 2 | .SetColumnValue "json_doc" }}
{{ end }}
```

#### jsonGetRaw

Get json attribute raw value by path, return nil if value does not exist

```gotemplate
{{ $jsonValue := "{\"b\": 1}" }}
{{ if eq jsonGetRaw "a" nil }}
{{ jsonSetRaw "b" "2" | .SetColumnValue "json_doc" }}
{{ end }}
```

#### jsonSet

Set the value to the json document by path

Example:

```gotemplate
{{ $jsonValue := "{\"a\": 1}" }}
{{ if jsonExists "a" $jsonValue }}
{{ mustJsonGet "a" $jsonValue | jsonSet "b" | .SetColumnValue "json_doc" }}
{{ end }}
```

#### jsonSetRaw

Set the raw value to the json document by path

Example:

```gotemplate
{{ $jsonValue := "{\"a\": 1}" }}
{{ if jsonExists "a" $jsonValue }}
{{ mustJsonGetRaw "a" $jsonValue | jsonSetRaw "b" | .SetRawColumnValue "json_doc" }}
{{ end }}
```

#### jsonDelete

Delete attribute from the json document by path

Example:

```gotemplate
{{ $jsonValue := "{\"a\": 1, \"b\": 2}" }}
{{ if jsonExists "a" $jsonValue }}
{{ mustJsonDelete "a" | .SetColumnValue "json_doc" }}
{{ end }}
```

#### jsonValidate

Validate json document syntax and raise error if there is any issues

Example:

```gotemplate
{{ "{\"a\": 1, \"b\": 2, }" | jsonValidate }}
```

#### jsonIsValid

Validate json and return true if valid

Example:

```gotemplate
{{ $jsonValue := "{\"a\": 1}" }}
{{ if eq jsonIsValid $jsonValue false }}
{{ fail "json document is invalid" }}
{{ end }}
```

#### toJsonRawValue

Cast any type to raw json value.

Example:

```gotemplate
{{ 1 | toJsonRawValue }}
```

### Testing functions

#### isInt

Check that the value is integer type

```gotemplate
{{- if isInt "string" -}}
it is an int value
{{- else -}}
it is not an int value
{{- end -}}
```

#### isFloat

Check that the value is float type

```gotemplate
{{- if isFloat "string" -}}
it is a float value
{{- else -}}
it is not a float value
{{- end -}}
```

#### isNil

Check that the value is nil

```gotemplate
{{- if isNil "string" -}}
it is a nil value
{{- else -}}
it is not a nil value
{{- end -}}
```

#### isString

Check that the value is string type

```gotemplate
{{- if isString "string" -}}
it is a string type value
{{- else -}}
it is not a string type value
{{- end -}}
```

#### isMap

Check that the value is map type

```gotemplate
{{- if isMap "string" -}}
it is a map type value
{{- else -}}
it is not a map type value
{{- end -}}
```

#### isSlice

Check that the value is slice type

```gotemplate
{{- if isSlice "string" -}}
it is a slice type value
{{- else -}}
it is not a slice type value
{{- end -}}
```

#### isBool

Check that the value is bool type

```gotemplate
{{- if isBool "string" -}}
it is a bool type value
{{- else -}}
it is not a bool type value
{{- end -}}
```

### Transformation and generators

#### masking

Replaces characters with asterisk `*` symbols depending on the provided data type. If the
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

Signature:

`masking(maskingRules string, value string) (res string, err error)`

Parameters:

* `dataType` - type of the masking rules
* `value` - original string value

Return Value:

* `res` - A masked string
* `err` - An error if there's an issue

Example:

```gotemplate
{{ "test1234" | masking "default" }} 
```

#### truncateDate

Truncate date and time (timestamp) up to provided part.

Signature:

`truncateDate(part string, original time.Time) (res time.Time, err error)`

Parameters:

* `part` - the truncation part. Must be one of `nano`, `second`, `minute`, `hour`, `day`, `month`, `year`.
* `original` - original time value

Return Value:

* `res` - truncated date
* `err` - An error if there's an issue

Example:

```gotemplate
{{ now | truncate "day" }} 
```

#### noiseDatePgInterval

Add or subtract a random duration in the provided `interval` to the original date value.

Signature:

`noiseDate(interval string, original time.Time) (res time.Time, err error)`

Parameters:

* `interval` - The maximum value of `ratio` that is added to the original value. The format is the same as in
  the [PostgreSQL interval format](https://www.postgresql.org/docs/current/datatype-datetime.html#DATATYPE-INTERVAL-INPUT).
* `original` - original time value

Return Value:

* `res` - noised date
* `err` - An error if there's an issue

Example:

```gotemplate
{{ now | noiseDatePgInterval "10 days" }} 
```

#### noiseFloat

Add or subtract a random fraction to the original float value. Multiplies the original float value by a provided random
value that is not higher than the `ratio` parameter and adds it to the original value with the option to specify the
precision via the `precision` parameter.

Signature:

`noiseFloat(ratio float, precision int, value float) (res float64, err error)`

Parameters:

* `ratio` - max multiplier value in interval (0:1). The value will be generated randomly up to `ratio`, multiplied on
  original value and then the summary will be added to original value
* `precision` - the precision of the resulted value
* `value` - the original value

Return Value:

* `res` - noised float value
* `err` - An error if there's an issue

Example:

```gotemplate
{{ 1.8 | noiseFloat 0.7 4 }} 
```

#### noiseInt

Add or subtract a random fraction to the original int value. Multiplies the original integer value by a provided random
value that is not higher than the
`ratio` parameter and adds it to the original value.

Signature:

`noiseInt(ratio float, value float) (res int, err error)`

Parameters:

* `ratio` - max multiplier value in interval (0:1). The value will be generated randomly up to `ratio`, multiplied on
  original value and then the summary will be added to original value
* `value` - the original value

Return Value:

* `res` - noised int value
* `err` - An error if there's an issue

Example:

```gotemplate
{{ 1234 | noiseInt 0.8 }} 
```

#### randomBool

Generate random int value in the provided interval

Signature:

`randomInt(min int, max int) (res int, err error)`

Parameters:

* `min` - minimum random value threshold
* `max` - maximum random value threshold

Return Value:

* `res` - randomly generated int value
* `err` - An error if there's an issue

Example:

```gotemplate
{{ randomInt 100 5000 }} 
```

#### randomDate

Generate random date in the provided interval

Signature:

`randomDate(min time.Time, max time.Time) (res time.Time, err error)`

Parameters:

* `min` - minimum random value threshold
* `max` - maximum random value threshold

Return Value:

* `res` - randomly generated date value
* `err` - An error if there's an issue

Example:

```gotemplate
{{ $minValue := .GetColumnValue "created_at" }}
{{ $maxValue := now }}
{{ randomDate $minValue $maxValue }} 
```

#### randomFloat

Generate random float in the provided interval

Signature:

`randomFloat(min any, max any, precision int) (res float, err error)`

Parameters:

* `min` - minimum random value threshold
* `max` - maximum random value threshold
* `precision` - the precision of the resulted value

Return Value:

* `res` - randomly generated float value
* `err` - An error if there's an issue

Example:

```gotemplate
{{ randomFloat 1.9 1000.849 4 }}
```

#### randomInt

Generate random int in the provided interval

Signature:

`randomInt(min int, max int) (res int, err error)`

Parameters:

* `min` - minimum random value threshold
* `max` - maximum random value threshold

Return Value:

* `res` - randomly generated int value
* `err` - An error if there's an issue

Example:

```gotemplate
{{ randomInt 1 1000 }}
```

#### randomString

Generates a random string using the provided characters within the specified length range.

Signature:

`randomString(minLength int, maxLength int, symbols string) (res string, err error)`

Parameters:

* `minLength` - minimum string length
* `maxLength` - maximum string length
* `symbols` - string with set of symbols which will be used. The default value is
  `abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ1234567890`

Return Value:

* `res` - randomly generated string value
* `err` - An error if there's an issue

Example:

```gotemplate
{{ randomString 10 16 "1234567abcd" }}
```

#### roundFloat

Round float value up to provided precision

Signature:

`roundFloat(precision int, original float) (res float, err error)`

Parameters:

* `precision` - the precision of the value
* `original` - the original float value

Return Value:

* `res` - rounded float value
* `err` - An error if there's an issue

```gotemplate
{{  1.2345678 | roundFloat 4 }}
```

### Faker functions

Greenmask uses [go-faker/faker](https://github.com/go-faker/faker) under the hood for generating of synthetic data

### Faker functions: Address

#### fakerRealAddress

Generate real world address randomly

Signature:

`fakerRealAddress() (res ReadAddress)`

Return Value:

* `res` - generated randomly real address

The returned value is RealAddress type that has the next structure:

```json
{
  "Address": "string",
  "City": "string",
  "State": "string",
  "PostalCode": "string",
  "Coordinates": {
    "Latitude": "float64",
    "Longitude": "float64"
  }
}
```

Example:

```gotemplate
{{- $res := fakerRealAddress -}}
{{ $res.Address }} {{ $res.City }} {{ $res.State }} {{ $res.PostalCode }} {{ $res.Coordinates.Latitude }}  {{ $res.Coordinates.Longitude }}
```

#### fakerLatitude

Generate fake latitude randomly

Signature:

`fakerLatitude() (res float64)`

Return Value:

* `res` - randomly generated latitude value

Example:

```gotemplate
{{ fakerLatitude }}
```

Result:

`81.12195`

#### fakerLongitude

Generate fake longitude randomly

Signature:

`fakerLongitude() (res float64)`

Return Value:

* `res` - randomly generated longitude value

Example:

```gotemplate
{{ fakerLongitude }}
```

Result:

`-84.38158`

### Faker functions: Datetime

#### fakerUnixTime

Generate random unix time in seconds

Signature:

`fakerLongitude() (res int64)`

Return Value:

* `res` - randomly generated unix time value

Example:

```gotemplate
{{ fakerUnixTime }}
```

Result:

`1197930901`

#### fakerDate

Generate random date in format `YYYY-MM-DD`

Signature:

`fakerDate() (res string)`

Return Value:

* `res` - randomly generated date value

Example:

```gotemplate
{{ fakerDate }}
```

Result:

`1982-02-27`

#### fakerTimeString

Generate time randomly in string format

Signature:

`fakerTimeString() (res string)`

Return Value:

* `res` - randomly generated time value

Example:

```gotemplate
{{ fakerTimeString }}
```

Result:

`03:10:25`

#### fakerMonthName

Generate month name randomly in string format

Signature:

`fakerMonthName() (res string)`

Return Value:

* `res` - randomly generated month name value

Example:

```gotemplate
{{ fakerMonthName }}
```

Result:

`February`

#### fakerYearString

Generate year randomly in string format

Signature:

`fakerYearString() (res string)`

Return Value:

* `res` - randomly generated year value

Example:

```gotemplate
{{ fakerYearString }}
```

Result:

`1994`

#### fakerDayOfWeek

Generate day of week randomly in string format

Signature:

`fakerDayOfWeek() (res string)`

Return Value:

* `res` - randomly generated day of week value

Example:

```gotemplate
{{ fakerDayOfWeek }}
```

Result:

`Sunday`

#### fakerDayOfMonth

Generate day of week randomly in string format

Signature:

`fakerDayOfMonth() (res string)`

Return Value:

* `res` - randomly generated day of month value

Example:

```gotemplate
{{ fakerDayOfMonth }}
```

Result:

`20`

#### fakerTimestamp

Generate timestamp randomly in string format: `YYYY-MM-DD HH:MM:SS`

Signature:

`fakerTimestamp() (res string)`

Return Value:

* `res` - randomly generated timestamp value

Example:

```gotemplate
{{ fakerTimestamp }}
```

Result:

`1973-06-21 14:50:46`

#### fakerCentury

Generate century randomly in

Signature:

`fakerCentury() (res string)`

Return Value:

* `res` - randomly generated century value

Example:

```gotemplate
{{ fakerCentury }}
```

Result:

`IV`

#### fakerTimezone

Generate timezone name randomly in string

Signature:

`fakerTimezone() (res string)`

Return Value:

* `res` - randomly generated timezone name value

Example:

```gotemplate
{{ fakerTimezone }}
```

Result:

`Asia/Jakarta`

#### fakerTimeperiod

Generate time period randomly in string `AM` or `PM`

Signature:

`fakerTimeperiod() (res string)`

Return Value:

* `res` - randomly generated time period value

Example:

```gotemplate
{{ fakerTimeperiod }}
```

Result:

`PM`

### Faker functions: Internet

#### fakerEmail

Generate email randomly in string

Signature:

`fakerEmail() (res string)`

Return Value:

* `res` - randomly generated email address value

Example:

```gotemplate
{{ fakerEmail }}
```

Result:

`mJBJtbv@OSAaT.com`

#### fakerMacAddress

Generate randomly Mac Address value in string

Signature:

`fakerMacAddress() (res string)`

Return Value:

* `res` - randomly generated MAC address value

Example:

```gotemplate
{{ fakerMacAddress }}
```

Result:

`cd:65:e1:d4:76:c6`

#### fakerDomainName

Generate random domain name in string

Signature:

`fakerDomainName() (res string)`

Return Value:

* `res` - randomly generated domain name value

Example:

```gotemplate
{{ fakerDomainName }}
```

Result:

`FWZcaRE.org`

#### fakerURL

Generate random URL in format `https://www.domainname.some/somepath`

Signature:

`fakerURL() (res string)`

Return Value:

* `res` - randomly generated URL value

Example:

```gotemplate
{{ fakerURL }}
```

Result:

`https://www.oEuqqAY.org/QgqfOhd`

#### fakerUsername

Generate random username

Signature:

`fakerUsername() (res string)`

Return Value:

* `res` - randomly generated username value

Example:

```gotemplate
{{ fakerUsername }}
```

Result:

`lVxELHS`

#### fakerIPv4

Generate random IP V4 address

Signature:

`fakerIPv4() (res string)`

Return Value:

* `res` - randomly generated IP V4 address value

Example:

```gotemplate
{{ fakerIPv4 }}
```

Result:

`99.23.42.63`

#### fakerIPv6

Generate random IP V6 address

Signature:

`fakerIPv6() (res string)`

Return Value:

* `res` - randomly generated IP V6 address value

Example:

```gotemplate
{{ fakerIPv6 }}
```

Result:

`975c:fb2c:2133:fbdd:beda:282e:1e0a:ec7d`

#### fakerPassword

Generate random password string

Signature:

`fakerPassword() (res string)`

Return Value:

* `res` - randomly generated password string

Example:

```gotemplate
{{ fakerPassword }}
```

Result:

`dfJdyHGuVkHBgnHLQQgpINApynzexnRpgIKBpiIjpTP`

### Faker functions: words and sentences

#### fakerWord

Generates a word randomly in string

Signature:

`fakerWord() (res string)`

Return Value:

* `res` - randomly generated word

Example:

```gotemplate
{{ fakerWord }}
```

Result:

`nesciunt`

#### fakerSentence

Generates sentence randomly in string

Signature:

`fakerSentence() (res string)`

Return Value:

* `res` - randomly generated sentence

Example:

```gotemplate
{{ fakerSentence }}
```

Result:

`Consequatur perferendis voluptatem accusantium.`

#### fakerParagraph

Generates series of sentences as a paragraph

Signature:

`fakerParagraph() (res string)`

Return Value:

* `res` - randomly generated paragraph

Example:

```gotemplate
{{ fakerParagraph }}
```

Result:

`Aut consequatur sit perferendis accusantium voluptatem. Accusantium perferendis consequatur voluptatem sit aut. Aut sit accusantium consequatur voluptatem perferendis. Perferendis voluptatem aut accusantium consequatur sit.`

### Faker functions: Payment

#### fakerCCType

Generate a credit card type randomly in string (VISA, MasterCard, etc)

Signature:

`fakerCCType() (res string)`

Return Value:

* `res` - randomly generated credit card type

Example:

```gotemplate
{{ fakerCCType }}
```

Result:

`American Express`

#### fakerCCNumber

Generate credit card number randomly

Signature:

`fakerCCNumber() (res string)`

Return Value:

* `res` - randomly generated credit card number

Example:

```gotemplate
{{ fakerCCNumber }}
```

Result:

`373641309057568`

#### fakerCurrency

Generate currency name randomly

Signature:

`fakerCurrency() (res string)`

Return Value:

* `res` - randomly generated currency name

Example:

```gotemplate
{{ fakerCurrency }}
```

Result:

`USD`

#### fakerAmountWithCurrency

Generate randomly amount with currency

Signature:

`fakerAmountWithCurrency() (res string)`

Return Value:

* `res` - randomly generated amount with currency

Example:

```gotemplate
{{ fakerAmountWithCurrency }}
```

Result:

`USD 49257.100`

### Faker functions: Person

#### fakerTitleMale

Generate a title male randomly in string `("Mr.", "Dr.", "Prof.", "Lord", "King", "Prince")`

Signature:

`fakerTitleMale() (res string)`

Return Value:

* `res` - randomly generated male title

Example:

```gotemplate
{{ fakerTitleMale }}
```

Result:

`Mr.`

#### fakerTitleFemale

Generate a title female randomly in string `("Mrs.", "Ms.", "Miss", "Dr.", "Prof.", "Lady", "Queen", "Princess")`

Signature:

`fakerTitleFemale() (res string)`

Return Value:

* `res` - randomly generated female title

Example:

```gotemplate
{{ fakerTitleFemale }}
```

Result:

`Mrs.`

#### fakerFirstName

Generate fake firstname

Signature:

`fakerFirstName() (res string)`

Return Value:

* `res` - randomly generated first name

Example:

```gotemplate
{{ fakerFirstName }}
```

Result:

`Whitney`

#### fakerFirstNameMale

Generate a fake firstname for male

Signature:

`fakerFirstNameMale() (res string)`

Return Value:

* `res` - randomly generated male first name

Example:

```gotemplate
{{ fakerFirstNameMale }}
```

Result:

`Kenny`

#### fakerFirstNameFemale

Generate a fake firstname for female

Signature:

`fakerFirstNameFemale() (res string)`

Return Value:

* `res` - randomly generated female first name

Example:

```gotemplate
{{ fakerFirstNameFemale }}
```

Result:

`Jana`

#### fakerFirstLastName

Generate a fake lastname

Signature:

`fakerFirstLastName() (res string)`

Return Value:

* `res` - randomly generated lastname

Example:

```gotemplate
{{ fakerFirstLastName }}
```

Result:

`Rohan`

#### fakerName

Generate a fake name

Signature:

`fakerName() (res string)`

Return Value:

* `res` - randomly generated name

Example:

```gotemplate
{{ fakerName }}
```

Result:

`Mrs. Casandra Kiehn`

### Faker functions: Phone

#### fakerPhoneNumber

Generate a fake phone number

Signature:

`fakerPhoneNumber() (res string)`

Return Value:

* `res` - randomly generated phone number

Example:

```gotemplate
{{ fakerPhoneNumber }}
```

Result:

`201-886-0269`

#### fakerTollFreePhoneNumber

Generate a phone number of type: "(888) 937-7238"

Signature:

`fakerTollFreePhoneNumber() (res string)`

Return Value:

* `res` - randomly generated toll free phone number

Example:

```gotemplate
{{ fakerTollFreePhoneNumber }}
```

Result:

`(777) 831-964572`

#### fakerE164PhoneNumber

Generate phone numbers of type: `+27113456789`

Signature:

`fakerE164PhoneNumber() (res string)`

Return Value:

* `res` - randomly generated E164 phone number format

Example:

```gotemplate
{{ fakerE164PhoneNumber }}
```

Result:

`+724891571063`

### Faker functions: UUID

#### fakerUUIDHyphenated

Generate a random UUID

Signature:

`fakerUUID() (res string)`

Return Value:

* `res` - randomly generated hyphenated UUID value

Example:

```gotemplate
{{ fakerUUID }}
```

Result:

`8f8e4463-9560-4a38-9b0c-ef24481e4e27`

#### fakerUUIDDigit

Generate a random in digit (HEX) format

Signature:

`fakerUUIDDigit() (res string)`

Return Value:

* `res` - randomly generated UUID value in HEX string

Example:

```gotemplate
{{ fakerUUIDDigit }}
```

Result:

`90ea6479fd0e4940af741f0a87596b73`

