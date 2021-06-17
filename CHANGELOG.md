# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog][kac], and this project adheres to
[Semantic Versioning][semver].

[kac]: https://keepachangelog.com/en/1.0.0
[semver]: https://semver.org/spec/v2.0.0.html

## Unreleased

## 0.3.0 - 2021-06-17

### Added

* Action submission specification
* Action constraint validation specification
* How to treat unrecognized field `type`s
* Constraints for fields
* How to apply the `pattern` property
* Common `step` property for fields
* Extensions for link objects: `hreflang` and `media`
* `FileList` as an acceptable type for a `file` field's `files` property

### Changed

* Clarified "type" as "data type" in recommendation of fields' `value`'s type
* Several field extension and common property descriptions to align with
  constraint validation
* Clarified adaptation of the HTML specification

### Fixed

* Typo in "`checkbox` Fields" section

### Removed

* Column for specifying valid `value` formats; this is covered by constraint
  validation
* Mentions of HTML's checkedness concept
* Unnecessary use of the term "element" in the Placeholder Label Option section

## 0.2.0 - 2021-01-04

### Added

* Recommendation for field `value`'s types and format based on field's `type`
* Defined how `null` or undefined field `value`s should be treated
* Defined the `checked` property for `checkbox` fields
* Defined a default value for `checkbox` fields' `value` property
* Defined the `size` property for `select` fields
* Defined semantics for placeholder label options in `select` fields
* Defined the `accept` property for `files` fields
* Defined semantics for `textarea` fields
* Defined more common properties based on common HTML input attributes
  * `dirname`
  * `max`
  * `maxlength`
  * `min`
  * `minlength`
  * `placeholder`
  * `readonly`
* Added table summarizing when common properties apply to a field

### Changed

* Moved mention of `files` property for consistency
* Renamed the section "HTML Input Attributes" to "Common Properties"
* Aligned type requirement for `multiple` `email` fields' `value` property with
  the HTML specification ([#2])
* Clarified when `disabled` and `required` properties apply to fields
* Clarified what `disabled` means for the field's `value`
* Clarified requirements for `required` fields

[#2]: https://github.com/dillonredding/siren-extensions/issues/2

### Removed

* Removed mentions of the HTML version

## 0.1.0 - 2020-12-04

### Added

* Defined the `group` extension for `radio` fields
* Defined semantics for `select` fields
* Defined field properties corresponding to common HTML input attributes:
  * `disabled`
  * `files`
  * `multiple`
  * `pattern`
  * `required`
