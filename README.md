# Siren Extensions

- [Introduction](#introduction)
  - [Conventions](#conventions)
- [Action Submission](#action-submission)
  - [Overview](#overview)
  - [Action Submission Algorithm](#action-submission-algorithm)
  - [Constructing the Entry List](#constructing-the-entry-list)
- [Constraints](#constraints)
  - [Definitions](#definitions)
    - [Validity States](#validity-states)
  - [Constraint Validation](#constraint-validation)
- [Field Extensions](#field-extensions)
  - [Unrecognized `type`](#unrecognized-type)
  - [`value` Type](#value-type)
  - [Algorithm to Convert a String to a Number](#algorithm-to-convert-a-string-to-a-number)
  - [`value` Coercion](#value-coercion)
  - [`checkbox` Fields](#checkbox-fields)
    - [`checked`](#checked)
    - [`value`](#value)
  - [`color` Fields](#color-fields)
  - [`date` Fields](#date-fields)
  - [`datetime-local` Fields](#datetime-local-fields)
  - [`email` Fields](#email-fields)
  - [`file` Fields](#file-fields)
    - [`accept`](#accept)
    - [`files`](#files)
  - [`hidden` Fields](#hidden-fields)
  - [`month` Fields](#month-fields)
  - [`number` and `range` Fields](#number-and-range-fields)
  - [`radio` Fields](#radio-fields)
    - [`group`](#group)
    - [Radio Object](#radio-object)
      - [`checked`](#checked-1)
      - [`disabled`](#disabled)
      - [`title`](#title)
      - [`value`](#value-1)
  - [`select` Fields](#select-fields)
    - [`options`](#options)
    - [`size`](#size)
    - [Placeholder Label Option](#placeholder-label-option)
    - [Option Object](#option-object)
      - [`disabled`](#disabled-1)
      - [`optgroup`](#optgroup)
      - [`selected`](#selected)
      - [`title`](#title-1)
      - [`value`](#value-2)
  - [`textarea` Fields](#textarea-fields)
    - [`cols`](#cols)
    - [`rows`](#rows)
    - [`wrap`](#wrap)
  - [`time` Fields](#time-fields)
  - [`url` Fields](#url-fields)
  - [`week` Fields](#week-fields)
  - [Common Properties](#common-properties)
    - [`dirname`](#dirname)
    - [`disabled`](#disabled-2)
    - [`maxlength` and `minlength`](#maxlength-and-minlength)
    - [`min` and `max`](#min-and-max)
    - [`multiple`](#multiple)
    - [`pattern`](#pattern)
    - [`placeholder`](#placeholder)
    - [`readonly`](#readonly)
    - [`required`](#required)
    - [`step`](#step)
- [Link Extensions](#link-extensions)
  - [`hreflang`](#hreflang)
  - [`media`](#media)

## Introduction

This document defines various extensions to the core [Siren] specification.

[siren]: https://github.com/kevinswiber/siren

### Conventions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this
document are to be interpreted as described in [BCP 14] [[RFC2119]] [[RFC8174]]
when, and only when, they appear in all capitals, as shown here.

[bcp 14]: https://tools.ietf.org/html/bcp14
[rfc2119]: https://tools.ietf.org/html/rfc2119
[rfc8174]: https://tools.ietf.org/html/rfc8174

## Action Submission

The extension for action submission aims to clarify how user agents should
translate an [action] into an HTTP request. The content in this section is
adapted from [HTML's form submission][html-fs] into the context of Siren.

[html-fs]: https://html.spec.whatwg.org/multipage/form-control-infrastructure.html#form-submission-2

### Overview

When an action is submitted, its `fields` are serialized into the format
specified by the action's `type`, and then sent to the destination specified by
the `href` using the given `method`.

For example, take the following action:

```json
{
  "name": "find",
  "href": "/find.cgi",
  "fields": [
    { "name": "t" }
    { "name": "q", "type": "search" }
  ]
}
```

If the value `"cats"` is provided for the first field and `"fur"` for the
second, and then the action is submitted, the user agent will make the following
HTTP request:

```http
GET /find.cgi?t=cats&q=fur HTTP/1.1
Host: example.com
```

Since the `method` member was omitted, `GET` is used. The `type` member was also
omitted, but `application/x-www-form-urlencoded` is always used for `GET`
actions.

If the `method` were explicityly `POST`, then the request will instead look as
follows:

```http
POST /find.cgi HTTP/1.1
Host: example.com
Content-Type: application/x-www-form-urlencoded
Content-Length:  12

t=cats&q=fur
```

The default value for `type` is used: `application/x-www-form-urlencoded`.
However, if the `type` were `multipart/form-data`, then the user agent makes a
request similar to the following:

```http
POST /find.cgi HTTP/1.1
Host: example.com
Content-Type: multipart/form-data;boundary=----kYFrd4jNJEgCervE
Content-Length: 171

------kYFrd4jNJEgCervE
Content-Disposition: form-data; name="t"

cats
------kYFrd4jNJEgCervE
Content-Disposition: form-data; name="q"

fur
------kYFrd4jNJEgCervE--
```

### Action Submission Algorithm

The following algorithm describes the steps for submitting a Siren action. It is
adapted from HTML's [form submission algorithm][fsa].

[fsa]: https://html.spec.whatwg.org/multipage/form-control-infrastructure.html#form-submission-algorithm

When an [action] `action` is submitted, the user agent must run the following
steps:

1. [Validate the constraints](#constraint-validation) of `action` and examine
   the result. If the result is negative (i.e., the constraint validation
   concluded that there were invalid fields), then return.
1. Let `entryList` be the result of
   [constructing the entry list](#constructing-the-entry-list) with `action`.
1. [Parse a URL][pau] given `action.href` and let `url` be the
   [resulting URL record][rur].
1. Let `method` be the value of `action.method`.
1. If `method` is a case-insensitive match for either of the strings `"GET"` or
   `"DELETE"`, then:
   1. Let `query` be the result of running the
      [`application/x-www-form-urlencoded` serializer][axwfus] with `entryList`.
   1. Set `url`'s query component to `query`.
   1. Make an HTTP request where `url` is the request target and `method` is the
      request method.
1. Otherwise:
   1. Switch on `action.type` (case-insensitive):
      - `"application/x-www-form-urlencoded"`, `null`, or undefined
        1. Let `body` be the result of running the
           [`application/x-www-form-urlencoded` serializer][axwfus] with
           `entryList`.
        1. Let `mimeType` be `"application/x-www-form-urlencoded"`.
      - `"multipart/form-data"`
        1. Let `body` be the result of running the
           [`multipart/form-data` encoding algorithm][mfdea] with `entryList`.
        1. Let `mimeType` be the concatenation of the string
           `"multipart/form-data;"`, a U+0020 SPACE character, the string
           `"boundary="`, and the `multipart/form-data` boundary string
           generated by the [`multipart/form-data` encoding algorithm][mfdea].
      - `"text/plain"`
        1. Let `body` be the result of running the
           [`text/plain` encoding algorithm][tpea] with `entryList`.
        1. Let `mimeType` be `"text/plain"`.
      - Otherwise, fail.
   1. Make an HTTP request where `url` is the request target, `method` is the
      request method, `mimeType` is the `Content-Type` header, and `body` is the
      message payload.

[rur]: https://html.spec.whatwg.org/multipage/urls-and-fetching.html#resulting-url-record
[pau]: https://html.spec.whatwg.org/multipage/urls-and-fetching.html#parse-a-url
[axwfus]: https://url.spec.whatwg.org/#concept-urlencoded-serializer
[mfdea]: https://html.spec.whatwg.org/multipage/form-control-infrastructure.html#multipart/form-data-encoding-algorithm
[tpea]: https://html.spec.whatwg.org/multipage/form-control-infrastructure.html#text/plain-encoding-algorithm

### Constructing the Entry List

The algorithm for constructing an entry list from an [action] `action` is as
follows. It is adapted from HTML's algorithm for
[constructing the entry list][ctel].

[ctel]: https://html.spec.whatwg.org/multipage/form-control-infrastructure.html#constructing-form-data-set

1. If `action.fields` is not an array, then return an empty [list].
1. Let `entryList` be a new empty [list] of [entries][entry].
1. For each value `field` in `action.fields`, in order:
   1. Let `name` be the value of `field.name`.
   1. If any of the following is true, then continue:
      - `name` is not a non-empty string.
      - `field.disabled` is [truthy].
      - `type` is `"image"`.
   1. Switch on `field.type` (case-insentive)
      - `"select"` (see [`select` Fields](#select-fields))
        1. If `field.options` is not an array, then continue.
        1. For each value `option` in `field.options`:
           1. If `option.selected` is [falsy] or `option.disabled` is [truthy],
              then continue.
           1. If `option.value` is present, then let `value` be the string value
              of `option.value`; otherwise, if `option.title` is present, then
              let `value` be the string value of `option.title`; otherwise,
              continue.
           1. [Append an entry][aae] to `entryList` with `name` and `value`.
      - `"checkbox"` (see [`checkbox` Fields](#checkbox-fields))
        1. If `field.checked` is [falsy], then continue.
        1. If `field.value` is present, then let `value` be the string value of
           `field.value`; otherwise let `value` be the string `"on"`.
        1. [Append an entry][aae] to `entryList` with `name` and `value`.
      - `"radio"` (see [`radio` Fields](#radio-fields))
        1. If `field.group` is not an array, then continue.
        1. Let `radio` be the first value of `field.group` whose `checked` member
           is [truthy].
        1. If `radio.value` is present, then let `value` be the string value of
           `radio.value`; otherwise, let `value` be the string `"on"`.
        1. [Append an entry][aae] to `entryList` with `name` and `value`.
      - `"file"` (see [`file` Fields](#file-fields))
        1. If `field.files` is either not an array or empty, then
           [append an entry][aae] to `entryList` with `name` and a new
           [`File`][file] object with an empty name, `application/octet-stream`
           as type, and an empty body.
        1. Otherwise, for each [file] `file` in `field.files`,
           [append an entry][aae] to `entryList` with `name` and `file`.
      - `"textarea"` (see [`textarea` Fields](#textarea-fields))
        1. [Append an entry][aae] to `entryList` with `name` and `field.value`,
           and the _prevent line break normalization flag_ set.
      - Otherwise
        1. [append an entry][aae] to `entryList` with `name` and `field.value`.
1. Return the `entryList`.

[aae]: https://html.spec.whatwg.org/multipage/form-control-infrastructure.html#append-an-entry
[entry]: https://xhr.spec.whatwg.org/#concept-formdata-entry
[falsy]: https://developer.mozilla.org/en-US/docs/Glossary/Falsy
[list]: https://infra.spec.whatwg.org/#list
[truthy]: https://developer.mozilla.org/en-US/docs/Glossary/Truthy

## Constraints

### Definitions

A [field][fields] is a **candidate for constraint validation** except when a
condition has **barred the field from constraint validation**. (For example, a
field is barred from constraint validation if it is disabled.)

A [field][fields] **satisfies its constraints** if it is not suffering from any
of the below [validity states](#validity-states).

#### Validity States

A field can be constrained in various ways. The following is the list of
**validity states** that a field can be in, making the field invalid for the
purposes of [constraint validation](#constraint-validation). (The definitions
below are non-normative; other parts of this specification define more precisely
when each state applies or does not.)

- **Suffering from being missing**
  - When a field is [`required`](#required) but has no `value`; or, more
    complicated rules for [`checkbox` fields](#radio-fields),
    [`file` fields](#radio-fields), [`radio` fields](#radio-fields), and
    [`select` fields](#select-fields).
- **Suffering from a type mismatch**
  - When a field that allows arbitrary user input has a value that is not in the
    correct syntax.
- **Suffering from a pattern mismatch**
  - When a field has a value that doesn't satisfy the [`pattern`](#pattern)
    member.
- **Suffering from being too long**
  - When a field has a value that is too long for the field's
    [`maxlength`](#maxlength-and-minlength) member.
- **Suffering from being too short**
  - When a field has a value that is too short for the field's
    [`minlength`](#maxlength-and-minlength) member.
- **Suffering from an underflow**
  - When a field has a value that is not the empty string and is too low for the
    [`min`](#min-and-max) member.
- **Suffering from an overflow**
  - When a field has a value that is not the empty string and is too high for
    the [`max`](#min-and-max) member.
- **Suffering from a step mismatch**
  - When a field has a value that doesn't fit the rules given by the
    [`step`](#step) member.
- **Suffering from bad input**
  - When a field has incomplete input and the user agent does not think the user
    ought to be able to [submit the action](#action-submission) in its current
    state.
- **Suffering from a custom error**
  - When a field's custom validity error message is not the empty string.

### Constraint Validation

The algorithm for validating an [action]'s constraints is as follows. It is
adapted from HTML's algorithm for [statically validating constraints][svtc].

[svtc]: https://html.spec.whatwg.org/multipage/form-control-infrastructure.html#statically-validate-the-constraints

When the user agent is required to validate the constraints of an [action]
`action`, it must run the following steps, which return either a positive result
(all the [fields] in the action are valid) or a negative result (there are invalid
fields) along with a (possibly empty) list of fields that are invalid:

1. Let `invalidControls` be an initially empty list of fields.
1. For each element `field` in `action.fields`, in order:
   1. If field is not a [candidate for constraint validation](#definitions),
      then continue.
   1. Otherwise, if `field` [satisfies its constraints](#definitions), then
      continue.
   1. Otherwise, add `field` to `invalidControls`.
1. If `invalidControls` is empty, then return a positive result.
1. Otherwise, return a negative result with the list of fields in the
   `invalidControls` list.

## Field Extensions

This section defines several, OPTIONAL extension members to [fields].

[fields]: https://github.com/kevinswiber/siren#fields-1

### Unrecognized `type`

Clients that do not recognize a field's `type` SHOULD treat the field as though
its `type` were `"text"`.

### `value` Type

The following table summarizes the RECOMMENDED [data type(s)][rfc8259-1] of a
field's `value` property based on the field's `type`:

[rfc8259-1]: https://tools.ietf.org/html/rfc8259#section-1

| Field `type`     | Allowed `value` Types   |
| ---------------- | ----------------------- |
| `hidden`         | Boolean, Number, String |
| `text`           | String                  |
| `search`         | String                  |
| `url`            | String                  |
| `tel`            | String                  |
| `email`          | String                  |
| `password`       | String                  |
| `date`           | String                  |
| `month`          | String                  |
| `week`           | String                  |
| `time`           | String                  |
| `datetime-local` | String                  |
| `number`         | Number, String          |
| `range`          | Number, String          |
| `color`          | String                  |
| `checkbox`       | Boolean, Number, String |

### Algorithm to Convert a String to a Number

The following field `type`s define an algorithm to convert a string to a number
that match's HTML's [algorithm to convert a string to a number][atcastan]:

[atcastan]: https://html.spec.whatwg.org/multipage/input.html#concept-input-value-string-number

- [`date`][atcastan-date]
- [`month`][atcastan-month]
- [`week`][atcastan-week]
- [`time`][atcastan-time]
- [`datetime-local`][atcastan-datetime-local]
- [`number`][atcastan-number], if `value` is a string
- [`range`][atcastan-range], if `value` is a string

[atcastan-date]: https://html.spec.whatwg.org/multipage/input.html#date-state-(type=date):concept-input-value-string-number
[atcastan-month]: https://html.spec.whatwg.org/multipage/input.html#date-state-(type=date):concept-input-value-string-number
[atcastan-week]: https://html.spec.whatwg.org/multipage/input.html#date-state-(type=date):concept-input-value-string-number
[atcastan-time]: https://html.spec.whatwg.org/multipage/input.html#date-state-(type=date):concept-input-value-string-number
[atcastan-datetime-local]: https://html.spec.whatwg.org/multipage/input.html#date-state-(type=date):concept-input-value-string-number
[atcastan-number]: https://html.spec.whatwg.org/multipage/input.html#date-state-(type=date):concept-input-value-string-number
[atcastan-range]: https://html.spec.whatwg.org/multipage/input.html#date-state-(type=date):concept-input-value-string-number

### `value` Coercion

If a field's `value` is undefined (i.e., absent in the representation) or
`null`, then clients SHOULD treat this as the empty string (`""`).

### `checkbox` Fields

This section defines additional semantics for `checkbox` fields.

#### `checked`

Indicates whether the field is checked. When it is checked, the field is
included during [action submission](#action-submission). This property is
OPTIONAL, it MUST be a boolean, and it defaults to `false`.

**Constraint validation:** If the field is [required](#required) and its
`checked` property is [falsy], then the field is
[suffering from being missing](#validity-states).

#### `value`

The value sent to the server on [action submission](#action-submission). The
default value is `"on"`.

### `color` Fields

**Constraint validation:** While the `value` of the field is neither the empty
string nor a [valid lowercase simple color][valid-color], the field is
[suffering from a type mismatch](#validity-states).

[valid-color]: https://html.spec.whatwg.org/multipage/common-microsyntaxes.html#valid-simple-colour

### `date` Fields

**Constraint validation:** While the `value` of the field is neither the empty
string nor a [valid date string][valid-date], the field is
[suffering from a type mismatch](#validity-states).

[valid-date]: https://html.spec.whatwg.org/multipage/common-microsyntaxes.html#valid-date-string

### `datetime-local` Fields

**Constraint validation:** While the `value` of the field is neither the empty
string nor a [valid normalized local date and time string][valid-datetime], the
field is [suffering from a type mismatch](#validity-states).

[valid-datetime]: https://html.spec.whatwg.org/multipage/common-microsyntaxes.html#valid-local-date-and-time-string

### `email` Fields

- When the field's `multiple` property is [falsy]
  - **Constraint validation:** While the `value` of the field is neither the
    empty string nor a single [valid email address][valid-email], the field is
    [suffering from a type mismatch](#validity-states).
- When the field's `multiple` property is [truthy]
  - **Constraint validation:** While the `value` of the field is not a
    [valid email address list][valid-emails], the field is
    [suffering from a type mismatch](#validity-states).

[valid-email]: https://html.spec.whatwg.org/multipage/input.html#valid-e-mail-address
[valid-emails]: https://html.spec.whatwg.org/multipage/input.html#valid-e-mail-address-list

### `file` Fields

This section defines additional semantics for `file` fields.

#### `accept`

The `accept` property provides user agents with a hint of what file types will
be accepted.

This property is OPTIONAL and its value MUST be an array whose elements MUST be
an ASCII case-insensitive match for one of the following:

- The string `"audio/*"`
  - Indicates that sound files are accepted
- The string `"video/*"`
  - Indicates that video files are accepted
- The string `"image/*"`
  - Indicates that image files are accepted
- A [valid MIME type string with no parameters][vmtswnp]
  - Indicates that files of the specified type are accepted
- A string whose first character is a U+002E FULL STOP character (.)
  - Indicates that files with the specified file extension are accepted

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

The list of selected files, each file consisting of a filename, a file type, and
a file body (the contents of the file). Intended for use by clients to keep
track of selected files. This property is OPTIONAL and MUST be an array of
[`File`][file] objects.

[file]: https://w3c.github.io/FileAPI/#dfn-file

**Constraint validation:** If the field is [required](#required) and its `files`
property is not a non-empty array of [`File`][file] objects, then the field is
[suffering from being missing](#validity-states).

### `hidden` Fields

**Constraint validation:** The field is
[barred from constraint validation](#definitions).

### `month` Fields

**Constraint validation:** While the `value` of the field is neither the empty
string nor a [valid month string][valid-month], the field is
[suffering from a type mismatch](#validity-states).

[valid-month]: https://html.spec.whatwg.org/multipage/common-microsyntaxes.html#valid-month-string

### `number` and `range` Fields

**Constraint validation:** While the `value` of the field is not the empty
string, a [valid floating-point number][valid-number], nor a
[JSON number][rfc8259-6], the field is
[suffering from a type mismatch](#validity-states).

[rfc8259-6]: https://tools.ietf.org/html/rfc8259#section-6
[valid-number]: https://html.spec.whatwg.org/multipage/common-microsyntaxes.html#valid-floating-point-number

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
Two or more radio objects in the same group MUST NOT be [checked](#checked-1) at
a given time.

**Constraint validation:** If the field is [required](#required) and its
[`group`](#group) property does not contain an element whose
[`checked`](#checked-1) property is [truthy], then the field is
[suffering from being missing](#validity-states).

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

If the field is [required](#required) and its [`options`](#options) member does
not contain an element whose [`selected`](#selected) member is [truthy], then
the field is [suffering from being missing](#validity-states).

Multiple options can be selected if the field allows [multiple](#multiple)
values.

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
`select` field MUST have a placeholder label option.

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

### `time` Fields

**Constraint validation:** While the `value` of the field is neither the empty
string nor a [valid time string][valid-time], the field is
[suffering from a type mismatch](#validity-states).

[valid-time]: https://html.spec.whatwg.org/multipage/common-microsyntaxes.html#valid-time-string

### `url` Fields

**Constraint validation:** While the `value` of the field is neither the empty
string nor a [valid][valid-url] [absolute URL][abs-url], the field is
[suffering from a type mismatch](#validity-states).

[abs-url]: https://url.spec.whatwg.org/#syntax-url-absolute
[valid-url]: https://url.spec.whatwg.org/#valid-url-string

### `week` Fields

**Constraint validation:** While the `value` of the field is neither the empty
string nor a [valid week string][valid-week], the field is
[suffering from a type mismatch](#validity-states).

[valid-week]: https://html.spec.whatwg.org/multipage/common-microsyntaxes.html#valid-week-string

### Common Properties

This section defines properties common to several types of fields.
These properties are adapted from HTML's [common input attributes][input-attrs].

[input-attrs]: https://html.spec.whatwg.org/multipage/input.html#input-type-attr-summary

The following table summarizes where a property is applicable based on a field's
`type`:

|               | `text`, `search` | `url`, `tel` | `email` | `password` | `date`, `month`, `week`, `time`, `datetime-local` | `number` | `range` | `checkbox`, `radio` | `file`  | `select` | `textarea` |
| ------------- | :--------------: | :----------: | :-----: | :--------: | :-----------------------------------------------: | :------: | :-----: | :-----------------: | :-----: | :------: | :--------: |
| `dirname`     |     &check;      |              |         |            |                                                   |          |         |                     |         |          |  &check;   |
| `disabled`    |     &check;      |   &check;    | &check; |  &check;   |                      &check;                      | &check;  | &check; |       &check;       | &check; | &check;  |  &check;   |
| `max`         |                  |              |         |            |                      &check;                      | &check;  | &check; |                     |         |          |            |
| `maxlength`   |     &check;      |   &check;    | &check; |  &check;   |                                                   |          |         |                     |         |          |  &check;   |
| `min`         |                  |              |         |            |                      &check;                      | &check;  | &check; |                     |         |          |            |
| `minlength`   |     &check;      |   &check;    | &check; |  &check;   |                                                   |          |         |                     |         |          |  &check;   |
| `multiple`    |                  |              | &check; |            |                                                   |          |         |                     | &check; | &check;  |            |
| `pattern`     |     &check;      |   &check;    | &check; |  &check;   |                                                   |          |         |                     |         |          |            |
| `placeholder` |     &check;      |   &check;    | &check; |  &check;   |                                                   | &check;  |         |                     |         |          |  &check;   |
| `readonly`    |     &check;      |   &check;    | &check; |  &check;   |                      &check;                      | &check;  |         |                     |         |          |  &check;   |
| `required`    |     &check;      |   &check;    | &check; |  &check;   |                      &check;                      | &check;  |         |       &check;       | &check; | &check;  |  &check;   |
| `step`        |                  |              |         |            |                      &check;                      | &check;  | &check; |                     |         |          |            |

#### `dirname`

Enables the submission of the field `value`'s [directionality], giving the name
of the entry that contains this value during action submission (see
[Submitting element directionality][submit-directionality] and step 5.13 of the
[constructing the entry list][ctel] algorithm).
This property MUST be a non-empty string.

[directionality]: https://html.spec.whatwg.org/multipage/dom.html#the-directionality
[submit-directionality]: https://html.spec.whatwg.org/multipage/form-control-infrastructure.html#submitting-element-directionality:-the-dirname-attribute

#### `disabled`

Indicates whether the field is **disabled**, meaning it is not included during
[action submission](#action-submission) and its `value` cannot be mutated.
This member MUST be a boolean and it defaults to `false`.
If this member is [truthy], the field is considered disabled.

> Note that the [`disabled`](#disabled) member for
> [radio objects](#radio-object) and the [`disabled`](#disabled-1) member for
> [option objects](#option-object) are defined separately.

**Constraint validation:** If a field is disabled, it is
[barred from constraint validation](#definitions).

#### `maxlength` and `minlength`

Declares inclusive upper (`maxlength`) or lower (`minlength`) bounds on the
number of characters in a field's `value`.
These properties MUST be either a number or a string representing a
[valid non-negative ingeger][valid-non-negative-int].

[valid-non-negative-int]: https://html.spec.whatwg.org/multipage/common-microsyntaxes.html#valid-non-negative-integer

**Constraint validation:** If a field's `maxlength` is specified and the length
of the field's `value` is greater than `maxlength`, then the field is
[suffering from being too long](#validity-states).

**Constraint validation:** If a field's `minlength` is specified and the length
of the field's `value` is less than `minlength`, then the field is
[suffering from being too short](#validity-states).

#### `min` and `max`

Indicate the allowed range of values for a field. The data type of the `min` and
`max` properties' value MUST be an allowed
[data type of the field's `value`](#value-type).

For `range` fields, the default `min` is `0` and the default `max` is `100`.

If the field has a `min` property, and the result of applying the
[algorithm to convert a string to a number](#algorithm-to-convert-a-string-to-a-number)
to the value of the `min` property is a number, then that number is the field's
**minimum**; otherwise, if the field defines a **default minimum** for its
`type`, then that is the minimum; otherwise, the field has no minimum.

If the field has a `max` property, and the result of applying the
[algorithm to convert a string to a number](#algorithm-to-convert-a-string-to-a-number)
to the value of the `max` property is a number, then that number is the field's
**maximum**; otherwise, if the field defines a **default maximum** for its
`type`, then that is the maximum; otherwise, the field has no minimum.

A field **has a reversed range** if its maximum is less than its minimum.

**Constraint validation:** When the field has a minimum and does not have a
reversed range, and the result of applying the
[algorithm to convert a string to a number](#algorithm-to-convert-a-string-to-a-number)
to the string given by the field's `value` is a number, and the number obtained
from that algorithm is less than the minimum, the field is
[suffering from an underflow](#validity-states).

**Constraint validation:** When the field has a maximum and does not have a
reversed range, and the result of applying the
[algorithm to convert a string to a number](#algorithm-to-convert-a-string-to-a-number)
to the string given by the field's `value` is a number, and the number obtained
from that algorithm is more than the maximum, the field is
[suffering from an overflow](#validity-states).

**Constraint validation:** When a field has a reversed range, and the result of
applying the
[algorithm to convert a string to a number](#algorithm-to-convert-a-string-to-a-number)
to the string given by the fields's `value` is a number, and the number obtained
from that algorithm is more than the maximum and less than the minimum, the
field is simultaneously [suffering from an underflow](#validity-states) and
[suffering from an overflow](#validity-states).

#### `multiple`

Indicates whether the client is allowed to specify more than one value.
This property MUST be a boolean and it defaults to `false`.

For `email` fields, when `multiple` is `true`, the field's `value` is a string
containing a list of comma-separated email addresses.

#### `pattern`

Specifies a regular expression against which the field's `value` is to be
checked.
This property MUST be a string representing a valid
[regular expression][regexp-syntax].

[regexp-syntax]: https://tc39.es/ecma262/#prod-Pattern

The **compiled pattern regular expression** of a field, if it exists, is a
JavaScript [RegExp] (or equivalent) object. It is determined as follows:

[regexp]: https://tc39.es/ecma262/#sec-regexp-regular-expression-objects

1. If the field does not have a `pattern` specified, then return nothing.
   The field has no compiled pattern regular expression.
1. Let `pattern` be the value of the `pattern` property of the field.
1. Let `regexpCompletion` be [RegExpCreate](`pattern`, `"u"`).
1. If `regexpCompletion` is an [abrupt completion][regexp-abrubt], then return
   nothing. The field has no compiled pattern regular expression.
   > User agents are encouraged to log this error in a developer console, to aid
   > debugging.
1. Let `anchoredPattern` be the string `"^(?:"`, followed by `pattern`, followed
   by `")$"`.
1. Return ! [RegExpCreate](`anchoredPattern`, `"u"`).

[regexpcreate]: https://tc39.es/ecma262/#sec-regexpcreate
[regexp-abrubt]: https://tc39.es/ecma262/#sec-completion-record-specification-type

> The reasoning behind these steps, instead of just using the value of the
> pattern attribute directly, is twofold. First, we want to ensure that when
> matched against a string, the regular expression's start is anchored to the
> start of the string and its end to the end of the string. Second, we want to
> ensure that the regular expression is valid in standalone form, instead of
> only becoming valid after being surrounded by the `"^(?:"` and `")$"` anchors.

**Constraint validation:** If the field's `value` is not the empty string, and
either the field's [`multiple`](#multiple) property is not specified or it does
not apply to the field given its `type`, and the field has a compiled pattern
regular expression but that regular expression does not match the field's
`value`, then the field is
[suffering from a pattern mismatch](#validity-states).

**Constraint validation:** If the field's `value` is not the empty string, and
the field's [multiple](#multiple) property is specified and applies to the
field, and the field has a compiled pattern regular expression but that
regular expression does not match each of the field's values, then the field
is [suffering from a pattern mismatch](#validity-states).

#### `placeholder`

Intended for use as an `input` element's [placeholder attribute][placeholder]
when representing the field in HTML.
This differs from the field's `title` in that it is not meant as a label when
displaying the field in a UI.
This property MUST be a string that does not contain U+000A LINE FEED (LF) or
U+000D CARRIAGE RETURN (CR) characters.

[placeholder]: https://html.spec.whatwg.org/multipage/input.html#attr-input-placeholder

#### `readonly`

Indicates whether the field is **readonly**, meaning its `value` cannot be
mutated. This member MUST be a boolean and it defaults to `false`. If this
member is [truthy], the field is considered read-only.

**Constraint validation:** If a field is read-only, it is
[barred from constraint validation](#definitions).

#### `required`

Indicates whether the field is **required**, meaning is must be included during
[action submission](#action-submission). This member MUST be a boolean and
it defaults to `false`. If this member is [truthy], the field is considered
required.

**Constraint validation:** If the field is required, the `type` member is none
of `checkbox`, `file`, `radio`, or `select`, and the `value` member is undefined,
`null`, or the empty string, then the field is
[suffering from being missing](#validity-states).

#### `step`

Indicates the granularity that is expected (and required) of the field's
`value`, by limiting the allowed values. This property MUST be a
[valid floating-point number][valid-number].

Based on a field's `type`, it will have a **default step**, a
**step scale factor**, and in some cases a **default step base**, which are used
in processing the property as described below. The values for each of these
attributes match those defined in HTML; however, the following table summarizes
these for your convenience:

| Field `type`     | Default Step | Step Scale Factor | Default Step Base |
| ---------------- | ------------ | ----------------- | ----------------- |
| `date`           | 1            | 86,400,000        |                   |
| `month`          | 1            | 1                 |                   |
| `week`           | 1            | 604,800,000       | -259,200,000      |
| `time`           | 60           | 1,000             |                   |
| `datetime-local` | 60           | 1,000             |                   |
| `number`         | 1            | 1                 |                   |
| `range`          | 1            | 1                 |                   |

The `step` property provides the **allowed value step** for the field, as
follows:

1. If the property does not apply, then there is no allowed value step.
1. Otherwise, if the property is absent, then the allowed value step is the
   default step multiplied by the step scale factor.
1. Otherwise, if the property's value is not a
   [valid floating-point number][valid-number], or is a number less than zero,
   then the allowed value step is the default step multiplied by the step scale
   factor.
1. Otherwise, the allowed value step is the property's value, multiplied by the
   step scale factor.

The **step base** is the value returned by the following algorithm:

1. If the field has a [`min`](#min-and-max) member, and the result of applying
   the [algorithm to convert a string to a number](#algorithm-to-convert-a-string-to-a-number)
   to the value of `min` is not an error, then return that result.
1. If the field has a `value` member, and the result of applying the
   [algorithm to convert a string to a number](#algorithm-to-convert-a-string-to-a-number)
   to the value of `value` is not an error, then return that result.
1. If a default step base is defined for this field given its `type`, then
   return it.
1. Return zero.

**Constraint validation:** When the field has an allowed value step, and the
result of applying the [algorithm to convert a string to a number](#algorithm-to-convert-a-string-to-a-number)
to the string given by the field's `value` is a number, and that number
subtracted from the step base is not an integral multiple of the allowed value
step, the element is [suffering from a step mismatch](#validity-states).

## Link Extensions

This section defines OPTIONAL extension members for [links] that correlate to
[Section 3.4 of RFC 8288][rfc8288-3.4].

[links]: https://github.com/kevinswiber/siren#links-1
[rfc8288-3.4]: https://tools.ietf.org/html/rfc8288#section-3.4

```json
{
  "rel": ["profile"],
  "href": "https://example.com/profile",
  "type": "application/xhtml+xml",
  "hreflang": "en",
  "media": "screen"
}
```

### `hreflang`

A hint indicating the language of the linked resource, per the `hreflang` target
attribute defined in [Section 3.4.1 or RFC 8288][rfc8288-3.4.1].
The value of this member MUST be a string or an array of strings, each
representing the `Language-Tag`.

[rfc8288-3.4.1]: https://tools.ietf.org/html/rfc8288#section-3.4.1

### `media`

Indicates the intended destination medium or media for style information, per
the `media` target attribute defined in
[Section 3.4.1 or RFC 8288][rfc8288-3.4.1].
The value of this member MUST be a string representing the `media-query-list`.
