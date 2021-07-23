# Changelog
All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added

### Changed

### Deprecated

### Removed

- PROJ definitions have been removed as they have been deprecated by PROJ.

### Fixed

- Clarified that EPSG codes must be numbers, PROJJSON must be objects and WKT2 must be a string.
- Clarified how to distinguish a Temporal Dimension from an Additional Dimension with type 'temporal'. [#5](https://github.com/stac-extensions/datacube/issues/5)

## [v1.0.0] - 2021-03-08

Initial independent release, see [previous history](https://github.com/radiantearth/stac-spec/commits/v1.0.0-rc.1/extensions/datacube)

### Changed
- Units for STAC dimensions should now be compliant to UDUNITS-2 units (singular) whenever available.

[Unreleased]: <https://github.com/stac-extensions/datacube/compare/v1.0.0...HEAD>
[v1.0.0]: <https://github.com/stac-extensions/datacube/tree/v1.0.0>
