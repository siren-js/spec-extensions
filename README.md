# Siren Extensions

* [Introduction](#introduction)
  * [Conventions](#conventions)
* [Field Extensions](#field-extensions)
  * [`value` Type and Format](#value-type-and-format)
  * [`value` Coercion](#value-coercion)
  * [`checkbox` Fields](#checkbox-fields)
    * [`checked`](#checked)
    * [`value`](#value)
  * [`file` Fields](#file-fields)
    * [`accept`](#accept)
    * [`files`](#files)
  * [`radio` Fields](#radio-fields)
    * [`group`](#group)
    * [Radio Object](#radio-object)
      * [`checked`](#checked-1)
      * [`disabled`](#disabled)
      * [`title`](#title)
      * [`value`](#value-1)
  * [`select` Fields](#select-fields)
    * [`options`](#options)
    * [`size`](#size)
    * [Placeholder Label Option](#placeholder-label-option)
    * [Option Object](#option-object)
      * [`disabled`](#disabled-1)
      * [`optgroup`](#optgroup)
      * [`selected`](#selected)
      * [`title`](#title-1)
      * [`value`](#value-2)
  * [`textarea` Fields](#textarea-fields)
    * [`cols`](#cols)
    * [`rows`](#rows)
    * [`wrap`](#wrap)
  * [Common Properties](#common-properties)
    * [`dirname`](#dirname)
    * [`disabled`](#disabled-2)
    * [`min` and `max`](#min-and-max)
    * [`maxlength` and `minlength`](#maxlength-and-minlength)
    * [`multiple`](#multiple)
    * [`pattern`](#pattern)
    * [`placeholder`](#placeholder)
    * [`readonly`](#readonly)
    * [`required`](#required)

## Introduction

This document defines various extensions to the core [Siren] specification.

[Siren]: https://github.com/kevinswiber/siren

### Conventions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this
document are to be interpreted as described in [BCP 14] [[RFC2119]] [[RFC8174]]
when, and only when, they appear in all capitals, as shown here.

[BCP 14]: https://tools.ietf.org/html/bcp14
[RFC2119]: https://tools.ietf.org/html/rfc2119
[RFC8174]: https://tools.ietf.org/html/rfc8174

## Field Extensions

This section defines several extension properties to [fields]. Since these are
extensions, their presence is OPTIONAL in Siren representations.

[fields]: https://github.com/kevinswiber/siren#fields-1

### `value` Type and Format

The following table summarizes the RECOMMENDED [types][rfc8259-1] and format of
a field's `value` property based on the field's `type`:

[rfc8259-1]: https://tools.ietf.org/html/rfc8259#section-1

<!-- Note: datetime-local below uses a non-breaking hyphen  -->

| Field&nbsp;`type` | Allowed&nbsp;`value`&nbsp;Types   | `value` Format
|-------------------|-----------------------------------|--------
| `hidden`          | Boolean,&nbsp;Number,&nbsp;String |
| `text`            | String                            |
| `search`          | String                            |
| `url`             | String                            | The empty string or a [valid][valid-url] [absolute URL][abs-url] |
| `tel`             | String                            |
| `email`           | String                            | The empty string, or a [valid email address list][valid-emails], if [`multiple`](#multiple) is `true`; otherwise, a [valid email address][valid-email]
| `password`        | String                            |
| `date`            | String                            | The empty string or a [valid date string][valid-date]
| `month`           | String                            | The empty string or a [valid month string][valid-month]
| `week`            | String                            | The empty string or a [valid week string][valid-week]
| `time`            | String                            | The empty string or a [valid time string][valid-time]
| `datetime‑local`  | String                            | The empty string or a [valid local date and time string][valid-datetime]
| `number`          | Number, String                    | A [valid floating-point number][valid-number], if `value` is a string
| `range`           | Number, String                    | A [valid floating-point number][valid-number], if `value` is a string
| `color`           | String                            | A [valid simple color][valid-color]
| `checkbox`        | Boolean, Number, String           |

[abs-url]: https://url.spec.whatwg.org/#syntax-url-absolute
[valid-color]: https://html.spec.whatwg.org/multipage/common-microsyntaxes.html#valid-simple-colour
[valid-date]: https://html.spec.whatwg.org/multipage/common-microsyntaxes.html#valid-date-string
[valid-datetime]: https://html.spec.whatwg.org/multipage/common-microsyntaxes.html#valid-local-date-and-time-string
[valid-email]: https://html.spec.whatwg.org/multipage/input.html#valid-e-mail-address
[valid-emails]: https://html.spec.whatwg.org/multipage/input.html#valid-e-mail-address-list
[valid-month]: https://html.spec.whatwg.org/multipage/common-microsyntaxes.html#valid-month-string
[valid-number]: https://html.spec.whatwg.org/multipage/common-microsyntaxes.html#valid-floating-point-number
[valid-time]: https://html.spec.whatwg.org/multipage/common-microsyntaxes.html#valid-time-string
[valid-url]: https://url.spec.whatwg.org/#valid-url-string
[valid-week]: https://html.spec.whatwg.org/multipage/common-microsyntaxes.html#valid-week-string

### `value` Coercion

If a field's `value` is undefined (i.e., absent in the representation) or
`null`, then clients SHOULD treat this as the empty string (`""`).

### `checkbox` Fields

This sections defines additional semantics for `checkbox` fields.

#### `checked`

The `checked` property of a `checkbox` field refers to the [checkedness] of the
checkbox.
This property is OPTIONAL, it MUST be a boolean, and it defaults to `false`.

[checkedness]: https://html.spec.whatwg.org/multipage/form-control-infrastructure.html#concept-fe-checked

#### `value`

The default value for a `checkbox` field's `value` property is `"on"`, which is
in accordance with the HTML specification (see step 5.7 of the algorithm for
[constructing the entry list][ctel] during form submission).

[ctel]: https://html.spec.whatwg.org/multipage/form-control-infrastructure.html#constructing-form-data-set

### `file` Fields

This section defines additional semantics for `file` fields.

#### `accept`

The `accept` property provides user agents with a hint of what file types will
be accepted.

This property is OPTIONAL and its value MUST be an array whose elements MUST be
an ASCII case-insensitive match for one of the following:

* The string `"audio/*"`
  * Indicates that sound files are accepted
* The string `"video/*"`
  * Indicates that video files are accepted
* The string `"image/*"`
  * Indicates that image files are accepted
* A [valid MIME type string with no parameters][vmtswnp]
  * Indicates that files of the specified type are accepted
* A string whose first character is a U+002E FULL STOP character (.)
  * Indicates that files with the specified file extension are accepted

[vmtswnp]: https://mimesniff.spec.whatwg.org/#valid-mime-type-with-no-parameters

The elements of the `accept` array MUST NOT be ASCII case-insensitive matches
for any of the other elements (i.e., duplicates are not allowed).

User agents MAY use the value of this property to display a more appropriate
user interface than a generic file picker.
For instance, given the value `["image/*"]`, a user agent could offer the user
the option of using a local camera or selecting a photograph from their photo
collection; given the value `["audio/*"]`, a user agent could offer the user the
option of recording a clip using a headset microphone.

User agents SHOULD prevent the user from selecting files that are not accepted
by one (or more) of these elements.

Servers are encouraged to specify both any MIME types and any corresponding
extensions when looking for data in a specific format.

The following example shows how to represent the example
[File Upload `input`][file-upload] from the HTML specification.

[file-upload]: https://html.spec.whatwg.org/multipage/input.html#file-upload-state-(type=file)

```json
{
  "name": "doc",
  "type": "file",
  "accept": [
    ".doc",
    ".docx",
    ".xml",
    "application/msword",
    "application/vnd.openxmlformats-officedocument.wordprocessingml.document"
  ]
}
```

#### `files`

Intended for use by clients to keep track of [selected files][selected-files].
This property MUST be an array of [`File` objects][file].

[file]: https://w3c.github.io/FileAPI/#dfn-file
[selected-files]: https://html.spec.whatwg.org/multipage/input.html#concept-input-type-file-selected

### `radio` Fields

Since fields are unique by `name`, there isn't a way to specify a
[radio button group][rbg] within an [action]. While there is a
[proposed update][pr69] to the core specification that would support this, the
update would introduce some oddities. Specifically, a `checkbox` doesn't
typically provide multiple options (at least in HTML). Furthermore, the update
does not provide a way for clients to distinguish between a radio button group
and a [drop-down](#select-fields).

[action]: https://github.com/kevinswiber/siren#actions-1
[pr69]: https://github.com/kevinswiber/siren/pull/69
[rbg]: https://html.spec.whatwg.org/multipage/input.html#radio-button-group

#### `group`

The `group` property allows for specifying several radio buttons with the same
`name`.
This property MUST be an array of [radio object](#radio-object)s.
This property applies only to fields whose `type` is `radio`.
Clients SHOULD ignore this property when present on any other type of field.

When the `radio` field is [required](#required), one of the radio objects MUST
be [checked](#checked-1) in the `group` array.
Two or more radio objects in the same group MUST NOT be checked at a given time.

The following example shows how to represent the [radio buttons][rb] from the
example in the HTML specification.

[rb]: https://html.spec.whatwg.org/multipage/input.html#radio-button-state-(type=radio)

```json
{
  "name": "dog-type",
  "type": "radio",
  "title": "Make your choice",
  "required": true,
  "group": [
    { "title": "Pupper", "value": "pupper", "disabled": true },
    { "title": "Doggo", "value": "doggo" }
  ]
}
```

Rather than having `required` on the radio button directly (i.e., the
[radio object](#radio-object)), we utilize the
[`required` extension](#required) on the field object.

#### Radio Object

Represents a radio button as part of a radio button group (i.e., a `radio`
field).

##### `checked`

The [checkedness] of the radio button.
This property is OPTIONAL, it MUST be a boolean, and it defaults to `false`.

##### `disabled`

Indicates that the radio button cannot be selected (i.e., `checked` cannot be
`true`).
This property is OPTIONAL, it MUST be a boolean, and it defaults to `false`.

##### `title`

Textual annotation of the radio button.
Clients can use this as a label for the radio button.
This property MUST be a string.
This property is OPTIONAL, but including it is RECOMMENDED to avoid "blank"
radio buttons in a user interface.

##### `value`

The value assigned to the radio button.
This property is OPTIONAL and its default value is `"on"`, which is in
accordance with the HTML specification (see step 5.7 of the algorithm for
[constructing the entry list][ctel] during form submission).

### `select` Fields

The core Siren specification doesn't explicitly offer an equivalent of
[HTML's `select` element][select]. There is a [proposed update][pr69] that
would add support for this, but it doesn't provide a way for clients to
distinguish between a [radio button group](#radio-fields) and a drop-down.
To resolve this issue, we provide additional semantics for fields whose `type`
is `select`.

[select]: https://html.spec.whatwg.org/multipage/form-elements.html#the-select-element

#### `options`

The `options` property is used to specify the field's
[list of options][opt-list].
This property MUST be an array of [option object](#option-object)s.
This property applies only to fields whose `type` is `select`.
Clients SHOULD ignore this property when present on any other type of field.

[opt-list]: https://html.spec.whatwg.org/multipage/form-elements.html#concept-select-option-list

When the `select` field is [required](#required), at least one of the option
objects MUST be [selected](#selected) in the `options` array.
Multiple options can be selected if the field allows
[multiple](#multiple) values.

Here's an example representing the first example [`select` element][select] from
the HTML specification.

```json
{
  "name": "unitType",
  "type": "select",
  "title": "Select unit type",
  "options": [
    { "title": "Miner", "value": 1 },
    { "title": "Puffer", "value": 2 },
    { "title": "Snipey", "value": 3, "selected": true },
    { "title": "Max", "value": 4 },
    { "title": "Firebot", "value": 5 }
  ]
}
```

#### `size`

The `size` property of a `select` field refers to the [display size][select-size]
This property is OPTIONAL and it MUST be a positive integer (i.e., greater than
zero).
If this property is absent or not a positive integer, then the default value is
`4` when [`multiple`](#multiple) is `true`, and `1` otherwise.

[select-size]: https://html.spec.whatwg.org/multipage/form-elements.html#concept-select-size

#### Placeholder Label Option

If a `select` field is [required](#required), does not allow
[multiple](#multiple) selections, and has a [`size`](#size) of 1; and if the
[`value`](#value-1) of the first [option object](#option-object) in the `select`
field's [`options`](#options) (if any) is absent or the empty string, and that
option object's `optgroup` property is undefined, then that option is the
`select` field's **placeholder label option**.

If a `select` field is [required](#required), does not allow
[multiple](#multiple) selections, and has a [`size`](#size) of 1, then the
`select` field element MUST have a placeholder label option.

The following example represents the second example [`select` element][select]
from the HTML specification, which contains a placeholder label option.

```json
{
  "name": "unitType",
  "type": "select",
  "required": true,
  "options": [
    { "title": "Select unit type" },
    { "title": "Miner", "value": 1 },
    { "title": "Puffer", "value": 2 },
    { "title": "Snipey", "value": 3 },
    { "title": "Max", "value": 4 },
    { "title": "Firebot", "value": 5 }
  ]
}
```

#### Option Object

Represents an [option] in a drop-down (i.e., a `select` field).

[option]: https://html.spec.whatwg.org/multipage/form-elements.html#the-option-element

##### `disabled`

Indicates that the option is [disabled][opt-disabled], meaning it cannot be
selected (i.e., `selected` cannot be `true`) for submission.
This property is OPTIONAL, it MUST be a boolean, and it defaults to `false`.

[opt-disabled]: https://html.spec.whatwg.org/multipage/form-elements.html#concept-option-disabled

##### `optgroup`

Specifies the name of the [group][optgroup] in which the option belongs.
The group is primarily intended for the purposes of a user interface.
This property is OPTIONAL and it MUST be a string.

[optgroup]: https://html.spec.whatwg.org/multipage/form-elements.html#the-optgroup-element

##### `selected`

The [selectedness] of the option.
This property is OPTIONAL, it MUST be a boolean, and it defaults to `false`.

[selectedness]: https://html.spec.whatwg.org/multipage/form-elements.html#concept-option-selectedness

##### `title`

Textual annotation corresponding to the option's [label][opt-label].
This property is REQUIRED and its value MUST be a non-empty string.

[opt-label]: https://html.spec.whatwg.org/multipage/form-elements.html#concept-option-label

##### `value`

The value assigned to the option.
This property is OPTIONAL and defaults to the value of the option object's
`title` property in accordance with the HTML specification (see `option`'s
[value][opt-value]).

[opt-value]: https://html.spec.whatwg.org/multipage/form-elements.html#concept-option-value

### `textarea` Fields

This section adds semantics for fields whose `type` is `textarea`, which
corresponds to [HTML's `textarea` element][textarea], a multiline plain text
control.
A `textarea` field's `value` property coincides with a `textarea` element's text
content.
There the `value` of a `textarea` field MUST be a string.

[textarea]: https://html.spec.whatwg.org/multipage/form-elements.html#the-textarea-element

#### `cols`

The `cols` property specifies the field's **character width**: the expected
maximum number of characters per line.
This property is OPTIONAL and it MUST be a positive integer (i.e., greater than
zero).

If the `cols` property is missing or not a positive integer, the default value
is `20`.

#### `rows`

The `rows` property specifies the field's **character height**: the number of
lines to show.
This property is OPTIONAL and it MUST be a positive integer (i.e., greater than
zero).

If the `rows` property is missing or not a positive integer, the default value
is `2`.

#### `wrap`

The `wrap` property specifies how the field's `value` is wrapped when submitting
the corresponding action.
This property is OPTIONAL and its value MUST be one of two strings: `soft` or
`hard`.

When `wrap` is set to `soft`, the field's `value` MUST NOT be wrapped before
being submitted.

When `wrap` is set to `hard`, the field's `value` MUST be wrapped by the user
agent as follows before being submitted:
Insert U+000D CARRIAGE RETURN U+000A LINE FEED (CRLF) character pairs into the
string using an implementation-defined algorithm so that each line has no more
than [`cols`](#cols) characters. For the purposes of this requirement, lines are
delimited by the start of the string, the end of the string, and U+000D CARRIAGE
RETURN U+000A LINE FEED (CRLF) character pairs.

If the `wrap` property is missing or is neither of the string `soft` nor `hard`,
then the default value is `soft`.

### Common Properties

This section defines properties common to several types of fields.
These properties are based on HTML's [common input attributes][input-attrs].

[input-attrs]: https://html.spec.whatwg.org/multipage/input.html#input-type-attr-summary

#### `dirname`

Enables the submission of the field `value`'s [directionality], giving the name
of the entry that contains this value during action submission (see
[Submitting element directionality][submit-directionality] and step 5.13 of the
[constructing the entry list][ctel] algorithm).
This property MUST be a non-empty string.
This property applies to fields whose `type` is `text`, `search`, or `textarea`.

[directionality]: https://html.spec.whatwg.org/multipage/dom.html#the-directionality
[submit-directionality]: https://html.spec.whatwg.org/multipage/form-control-infrastructure.html#submitting-element-directionality:-the-dirname-attribute

#### `disabled`

Indicates that the field is [disabled], meaning its value cannot be mutated and
it is barred from constraint validation.
This property MUST be a boolean and it defaults to `false`.
When `true`, the field's `value` is immutable and barred from constraint
validation.
This property applies to all field types.

[disabled]: https://html.spec.whatwg.org/multipage/form-control-infrastructure.html#concept-fe-disabled

#### `min` and `max`

Indicate the allowed range of values for a field.
These properties SHOULD align with the type and format of the field's `value`
(see [`value` Type and Format](#value-type-and-format)).
These properties apply to fields whose `type` is `date`, `month`, `week`,
`time`, `datetime-local`, `number`, or `range`.
`range` fields have a default `min` value of `0` and a default `max` value of
`100`.

#### `maxlength` and `minlength`

Declare inclusive upper (`maxlength`) or lower (`minlength`) bounds on the
number of characters in a field's `value`.
These properties MUST be either a number or a string representing a
[valid floating-point number][valid-number].

#### `multiple`

Indicates whether the client is allowed to specify more than one value.
This property MUST be a boolean and it defaults to `false`.
This property is only applicable to fields whose `type` is `email`, `file`, or
[`select`](#select-fields).

When `multiple` is `true` and `type` is `email`, the field's `value` is a string
containing a list of comma-separated email addresses.

#### `pattern`

Specifies a regular expression against which the field's `value` is to be
checked.
When [`multiple`](#multiple) is `true`, each individual value is to be checked
against the pattern.
This property MUST be a string representing a valid
[regular expression][regexp].
This property is only applicable to fields whose `type` is `text`, `search`,
`url`, `tel`, `email`, or `password`.

[regexp]: https://tc39.es/ecma262/#prod-Pattern

#### `placeholder`

Intended for use as an `input` element's [placeholder attribute][placeholder]
when representing the field in HTML.
This differs from the field's `title` in that it is not meant as a label when
displaying the field in a UI.
This property MUST be a string that does not contain U+000A LINE FEED (LF) or
U+000D CARRIAGE RETURN (CR) characters.

[placeholder]: https://html.spec.whatwg.org/multipage/input.html#attr-input-placeholder

#### `readonly`

Indicates whether or not the field can be edited.
This property MUST be a boolean and it defaults to `false`.
When `true`, the field's `value` is immutable and barred from constraint
validation.

#### `required`

Indicates that the field is required, meaning the value submitted to the server
MUST NOT be the empty string (see [`value` Coercion](#value-coercion)).
With the exception of [`radio` fields](#radio-fields) and
[`select` fields](#select-fields), the value submitted to the server is the
value of the `value` property.
This property MUST be a boolean and it defaults to `false`.
This property applies to all field types.
