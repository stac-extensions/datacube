# Changelog
All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [v2.2.0] - 2023-05-03

### Added

- A new Spatial Vector Cube Dimension for geometries

## [v2.1.0] - 2022-07-26

### Added

- Fields can also be used in assets. [#12](https://github.com/stac-extensions/datacube/issues/12)

### Changed

- Clarified that `values` must be an ordered list and not a set of unordered values. [#11](https://github.com/stac-extensions/datacube/issues/11)

## [v2.0.0] - 2021-07-26

### Added

- `cube:variable`: Object for describing NetCDF-style multi-dimensional arrays in a datacube [#6](https://github.com/stac-extensions/datacube/pull/6)

### Changed

- Updated examples to STAC 1.0.0

### Removed

- PROJ definitions have been removed as they have been deprecated by PROJ.

### Fixed

- Clarified that EPSG codes must be integers, PROJJSON must be objects and WKT2 must be a string
- Clarified how to distinguish a Temporal Dimension from an Additional Dimension with type 'temporal' [#5](https://github.com/stac-extensions/datacube/issues/5)
- The JSON Schema is more strict and should not have issues with missing required fields in Collections any longer [#4](https://github.com/stac-extensions/datacube/issues/4)

## [v1.0.0] - 2021-03-08

Initial independent release, see [previous history](https://github.com/radiantearth/stac-spec/commits/v1.0.0-rc.1/extensions/datacube)

### Changed
- Units for STAC dimensions should now be compliant to UDUNITS-2 units (singular) whenever available.

[Unreleased]: <https://github.com/stac-extensions/datacube/compare/v2.2.0...HEAD>
[v2.2.0]: <https://github.com/stac-extensions/datacube/compare/v2.1.0...v2.2.0>
[v2.1.0]: <https://github.com/stac-extensions/datacube/compare/v2.0.0...v2.1.0>
[v2.0.0]: <https://github.com/stac-extensions/datacube/compare/v1.0.0...v2.0.0>
[v1.0.0]: <https://github.com/stac-extensions/datacube/tree/v1.0.0>
