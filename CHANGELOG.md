# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog][kac], and this project adheres to
[Semantic Versioning][semver].

[kac]: https://keepachangelog.com/en/1.0.0
[semver]: https://semver.org/spec/v2.0.0.html

## Unreleased

### Added

* Defined more HTML5 input attributes for fields:
  * `accept`
  * `alt`?
  * `autocomplete`?
  * `checked`
  * `dirname`
  * `list`?
  * `max`
  * `maxlength`
  * `min`
  * `minlength`
  * `placeholder`
  * `readonly`
  * `size`
  * `src`?
  * `step`?

## 0.2.0

### Added

* Defined the `size` property for `select` fields.
### Changed

* Changed the type requirement for `multiple` `email` fields' `value` property
  ([#2](https://github.com/dillonredding/siren-extensions/issues/2)).

## 0.1.0 - 2020-12-04

### Added

* Defined the `group` extension for `radio` fields
* Defined semantics for `select` fields
* Defined additional HTML5 input attributes for fields:
  * `disabled`
  * `files`
  * `multiple`
  * `pattern`
  * `required`
