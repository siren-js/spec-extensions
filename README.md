# Siren Extensions

* [Introduction](#introduction)
  * [Conventions](#conventions)
* [Field Extensions](#field-extensions)
  * [`radio` Fields](#radio-fields)
    * [`group`](#group)
  * [Radio Object](#radio-object)
    * [`checked`](#checked)
    * [`disabled`](#disabled)
    * [`title`](#title)
    * [`value`](#value)
  * [`select` Fields](#select-fields)
    * [`options`](#options)
  * [Option Object](#option-object)
    * [`disabled`](#disabled-1)
    * [`optgroup`](#optgroup)
    * [`selected`](#selected)
    * [`title`](#title-1)
    * [`value`](#value-1)
  * [HTML5 Input Attributes](#html5-input-attributes)
    * [`disabled`](#disabled-2)
    * [`files`](#files)
    * [`multiple`](#multiple)
    * [`pattern`](#pattern)
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

The above example is one way to represent the [radio buttons][rb] from the
example in the HTML5 specification.
Rather than having `required` on the radio button directly, we utilize the
[`required` extension](#required) on the field object.

[rb]: https://html.spec.whatwg.org/multipage/input.html#radio-button-state-(type=radio)

### Radio Object

Represents a radio button as part of a radio button group (i.e., a `radio`
field).

#### `checked`

The [checkedness] of the radio button.
This property is OPTIONAL, it MUST be a boolean, and it defaults to `false`.

[checkedness]: https://html.spec.whatwg.org/multipage/form-control-infrastructure.html#concept-fe-checked

#### `disabled`

Indicates that the radio button cannot be selected (i.e., `checked` cannot be
`true`).
This property is OPTIONAL, it MUST be a boolean, and it defaults to `false`.

#### `title`

Textual annotation of the radio button.
Clients can use this as a label for the radio button.
This property MUST be a string.
This property is OPTIONAL, but including it is RECOMMENDED to avoid "blank"
radio buttons in a user interface.

#### `value`

The value assigned to the radio button.
This property is OPTIONAL and its default value is `"on"`, which is in
accordance with the HTML5 specification (see step 5.7 of the
[constructing the entry list][ctel] algorithm).

[ctel]: https://html.spec.whatwg.org/multipage/form-control-infrastructure.html#constructing-form-data-set

### `select` Fields

The core Siren specification doesn't explicitly offer an equivalent of
[HTML5's `select` element][select]. There is a [proposed update][pr69] that
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

The above example is one way to represent the [`select` element][select] from
the first example in the HTML5 specification.

### Option Object

Represents an [option] in a drop-down (i.e., a `select` field).

[option]: https://html.spec.whatwg.org/multipage/form-elements.html#the-option-element

#### `disabled`

Indicates that the option is [disabled][opt-disabled], meaning it cannot be
selected (i.e., `selected` cannot be `true`) for submission.
This property is OPTIONAL, it MUST be a boolean, and it defaults to `false`.

[opt-disabled]: https://html.spec.whatwg.org/multipage/form-elements.html#concept-option-disabled

#### `optgroup`

Specifies the name of the [group][optgroup] in which the option belongs.
The group is primarily intended for the purposes of a user interface.
This property is OPTIONAL and it MUST be a string.

[optgroup]: https://html.spec.whatwg.org/multipage/form-elements.html#the-optgroup-element

#### `selected`

The [selectedness] of the option.
This property is OPTIONAL, it MUST be a boolean, and it defaults to `false`.

[selectedness]: https://html.spec.whatwg.org/multipage/form-elements.html#concept-option-selectedness

#### `title`

Textual annotation corresponding to the option's [label][opt-label].
This property is REQUIRED and its value MUST be a non-empty string.

[opt-label]: https://html.spec.whatwg.org/multipage/form-elements.html#concept-option-label

#### `value`

The value assigned to the option.
This property is OPTIONAL and defaults to the value of the option object's
`title` property in accordance with the HTML5 specification (see `option`'s
[value][opt-value]).

[opt-value]: https://html.spec.whatwg.org/multipage/form-elements.html#concept-option-value

### HTML5 Input Attributes

The following properties correspond to HTML5's [input attributes][input-attrs].

[input-attrs]: https://html.spec.whatwg.org/multipage/input.html#input-type-attr-summary

#### `disabled`

Indicates that the field is [disabled], meaning its value cannot be mutated and
it is barred from constraint validation.
This property MUST be a boolean and it defaults to `false`.

[disabled]: https://html.spec.whatwg.org/multipage/form-control-infrastructure.html#concept-fe-disabled

#### `files`

Intended for use by clients to keep track of [selected files][selected-files].
This property is only application to fields whose `type` is `file`.
This property MUST be an array of [`File` objects][file].

[file]: https://w3c.github.io/FileAPI/#dfn-file
[selected-files]: https://html.spec.whatwg.org/multipage/input.html#concept-input-type-file-selected

#### `multiple`

Indicates whether the clients is allowed to specify more than one value.
This property is only applicable to fields whose `type` is `email`, `file`, or
[`select`](#select-fields).
This property MUST be a boolean and it defaults to `false`.

When `multiple` is `true` and `type` is `email`, the field's `value` MUST be an
array of strings.

#### `pattern`

Specifies a regular expression against which the field's `value` is to be
checked.
When [`multiple`](#multiple) is `true`, each individual value is to be checked
against the pattern.
This property is only applicable to fields whose `type` is `text`, `search`,
`url`, `tel`, `email`, or `password`.
This property MUST be a string representing a valid
[regular expression][regexp].

[regexp]: https://tc39.es/ecma262/#prod-Pattern

#### `required`

Indicates that the field is required, meaning it must be included when
submitting the corresponding action to the server.
This property MUST be a boolean and it defaults to `false`.

<!--
#### `accept`

#### `alt`?

#### `autocomplete`?

#### `checked`

#### `dirname`

#### `list`?

#### `max`

#### `maxlength`

#### `min`

#### `minlength`

#### `placeholder`

#### `readonly`

#### `size`

#### `src`?

#### `step`?
-->
