# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog][kac], and this project adheres to
[Semantic Versioning][semver].

[kac]: https://keepachangelog.com/en/1.0.0
[semver]: https://semver.org/spec/v2.0.0.html

## Unreleased

### Added

* Defined more HTML5 input attributes for fields:
  * `readonly`
  * `step`

## 0.2.0

### Added

* Added recommendation for field `value`'s types and format based on field's
  `type`.
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

### Changed

* Moved mention of `files` property for consistency
* Renamed the section "HTML Input Attributes" to "Common Properties"

### Fixed

* Aligned type requirement for `multiple` `email` fields' `value` property with
  the HTML specification ([#2])
* Clarified requirements for `required` fields

[#2]: https://github.com/dillonredding/siren-extensions/issues/2

### Removed

* Removed mentions of the HTML version

## 0.1.0 - 2020-12-04

### Added

* Defined the `group` extension for `radio` fields
* Defined semantics for `select` fields
* Defined properties corresponding to common HTML input attributes for fields:
  * `disabled`
  * `files`
  * `multiple`
  * `pattern`
  * `required`
