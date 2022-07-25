# Datacube Extension Specification

- **Title:** Datacube
- **Identifier:** <https://stac-extensions.github.io/datacube/v2.0.0/schema.json>
- **Field Name Prefix:** cube
- **Scope:** Item, Collection
- **Extension [Maturity Classification](https://github.com/radiantearth/stac-spec/tree/master/extensions/README.md#extension-maturity):** Proposal
- **Owner**: @m-mohr

This document explains the Datacube Extension to the [SpatioTemporal Asset Catalog](https://github.com/radiantearth/stac-spec) (STAC) specification.
It specifies datacube related metadata, especially their dimensions and potentially more in the future.

- Examples:
  - [Item example](examples/item.json): Shows the basic usage of the extension in a STAC Item
  - [Collection example](examples/collection.json): Shows the basic usage of the extension in a STAC Collection
- [JSON Schema](json-schema/schema.json)
- [Changelog](./CHANGELOG.md)

## Fields

The following fields can be used in the following parts of a STAC document:
- [ ] Catalogs
- [x] Collections
- [x] Item Properties (incl. Summaries in Collections)
- [x] Assets (both Collections and Items, incl. Item Asset Definitions in Collections)
- [ ] Links

| Field Name       | Type                                               | Description                                 |
| ---------------- | -------------------------------------------------- | ------------------------------------------- |
| cube:dimensions  | Map<string, [Dimension Object](#dimension-object)> | **REQUIRED.** Uniquely named dimensions of the datacube. |
| cube:variables   | Map<string, [Variable Object](#variable-object)>   | Uniquely named variables of the datacube. |

The keys of `cube:dimensions` and `cube:variables` should be unique together; a key like `lat` should not be both a dimension and a variable.

### Dimension Object

A *Dimension Object* comes in different flavors, each of them is defined below. The fields define mostly very similar fields, 
but they differ slightly depending on their use case. All objects share the fields `type` and `description` with the same definition, 
but `type` may be restricted to certain values. The definition of`axis` is shared between the spatial dimensions, but restricted to 
certain values, too. `extent`, `values` and `step` share the same definition, but differ in the supported data types (number or string) 
depending on the type of dimension. Whenever it's useful to specify these fields, the objects add the additional fields `reference_system` 
and `unit` with very similar definitions across the objects.

### Horizontal Spatial Dimension Object

A spatial dimension in one of the horizontal (x or y) directions.

| Field Name       | Type           | Description                                                  |
| ---------------- | -------------- | ------------------------------------------------------------ |
| type             | string         | **REQUIRED.** Type of the dimension, always `spatial`.       |
| axis             | string         | **REQUIRED.** Axis of the spatial dimension (`x`, `y`).      |
| description      | string         | Detailed multi-line description to explain the dimension. [CommonMark 0.29](http://commonmark.org/) syntax MAY be used for rich text representation. |
| extent           | \[number]      | **REQUIRED.** Extent (lower and upper bounds) of the dimension as two-element array. Open intervals with `null` are not allowed. |
| values           | \[number]      | Optionally, an ordered list of all values.                   |
| step             | number\|null   | The space between the values. Use `null` for irregularly spaced steps. |
| reference_system | string\|number\|object | The spatial reference system for the data, specified as [numerical EPSG code](http://www.epsg-registry.org/), [WKT2 (ISO 19162) string](http://docs.opengeospatial.org/is/18-010r7/18-010r7.html) or [PROJJSON object](https://proj.org/specifications/projjson.html). Defaults to EPSG code 4326. |

### Vertical Spatial Dimension Object

A spatial dimension in vertical (z) direction.

| Field Name       | Type             | Description                                                  |
| ---------------- | ---------------- | ------------------------------------------------------------ |
| type             | string           | **REQUIRED.** Type of the dimension, always `spatial`.       |
| axis             | string           | **REQUIRED.** Axis of the spatial dimension, always `z`.     |
| description      | string           | Detailed multi-line description to explain the dimension. [CommonMark 0.29](http://commonmark.org/) syntax MAY be used for rich text representation. |
| extent           | \[number\|null\]   | If the dimension consists of [ordinal](https://en.wikipedia.org/wiki/Level_of_measurement#Ordinal_scale) values, the extent (lower and upper bounds) of the values as two-element array. Use `null` for open intervals. |
| values           | \[number\|string\] | An ordered list of all values, especially useful for [nominal](https://en.wikipedia.org/wiki/Level_of_measurement#Nominal_level) values. |
| step             | number\|null     | If the dimension consists of [interval](https://en.wikipedia.org/wiki/Level_of_measurement#Interval_scale) values, the space between the values. Use `null` for irregularly spaced steps. |
| unit             | string           | The unit of measurement for the data, preferably compliant to [UDUNITS-2](https://ncics.org/portfolio/other-resources/udunits2/) units (singular). |
| reference_system | string\|number\|object | The spatial reference system for the data, specified as [numerical EPSG code](http://www.epsg-registry.org/), [WKT2 (ISO 19162) string](http://docs.opengeospatial.org/is/18-010r7/18-010r7.html) or [PROJJSON object](https://proj.org/specifications/projjson.html). Defaults to EPSG code 4326. |

A Vertical Spatial Dimension Object MUST specify an `extent` or `values`. It MAY specify both. 

### Temporal Dimension Object

A temporal dimension based on the ISO 8601 standard. The temporal reference system for the data is expected to be ISO 8601 compliant 
(Gregorian calendar / UTC). Data not compliant with ISO 8601 can be represented as an *Additional Dimension Object* with `type` set to `temporal`.

| Field Name | Type            | Description                                                  |
| ---------- | --------------- | ------------------------------------------------------------ |
| type       | string          | **REQUIRED.** Type of the dimension, always `temporal`.      |
| description | string         | Detailed multi-line description to explain the dimension. [CommonMark 0.29](http://commonmark.org/) syntax MAY be used for rich text representation. |
| extent     | \[string\|null] | **REQUIRED.** Extent (lower and upper bounds) of the dimension as two-element array. The dates and/or times must be strings compliant to [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601). `null` is allowed for open date ranges. |
| values     | \[string]       | If the dimension consists of an ordered list of specific values they can be listed here. The dates and/or times must be strings compliant to [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601). |
| step       | string\|null    | The space between the temporal instances as [ISO 8601 duration](https://en.wikipedia.org/wiki/ISO_8601#Durations), e.g. `P1D`. Use `null` for irregularly spaced steps. |

### Additional Dimension Object

An additional dimension that is not `spatial`, but may be `temporal` if the data is not compliant with ISO 8601 (see below).

| Field Name       | Type              | Description                                                  |
| ---------------- | ----------------- | ------------------------------------------------------------ |
| type             | string            | **REQUIRED.** Custom type of the dimension, never `spatial`. |
| description      | string            | Detailed multi-line description to explain the dimension. [CommonMark 0.29](http://commonmark.org/) syntax MAY be used for rich text representation. |
| extent           | \[number\|null]   | If the dimension consists of [ordinal](https://en.wikipedia.org/wiki/Level_of_measurement#Ordinal_scale) values, the extent (lower and upper bounds) of the values as two-element array. Use `null` for open intervals. |
| values           | \[number\|string] | An ordered list of all values, especially useful for [nominal](https://en.wikipedia.org/wiki/Level_of_measurement#Nominal_level) values. |
| step             | number\|null      | If the dimension consists of [interval](https://en.wikipedia.org/wiki/Level_of_measurement#Interval_scale) values, the space between the values. Use `null` for irregularly spaced steps. |
| unit             | string            | The unit of measurement for the data, preferably compliant to [UDUNITS-2](https://ncics.org/portfolio/other-resources/udunits2/) units (singular). |
| reference_system | string            | The reference system for the data.                           |

An Additional Dimension Object MUST specify an `extent` or `values`. It MAY specify both.

Note on "Additional Dimension" with type `temporal`:
You can distinguish the "Temporal Dimension" from an "Additional Dimension" by checking whether the extent exists and contains strings.
So if the `type` equals `temporal` and `extent` is an array of strings/null, then you have a "Temporal Dimension",
otherwise you have an "Additional Dimension".

### Variable Object

A *Variable Object* defines a variable (or a multi-dimensional array). The variable may have dimensions, which are described by [Dimension Objects](#dimension-object).

| Field Name       | Type                         | Description |
| ---------------- | -----------------------------| ----------- |
| dimensions       | \[string]                    | **REQUIRED.** The dimensions of the variable. This should refer to keys in the ``cube:dimensions`` object or be an empty list if the variable has no dimensions. |
| type             | string                       | **REQUIRED.** Type of the variable, either `data` or `auxiliary`. | 
| description      | string                       | Detailed multi-line description to explain the variable. [CommonMark 0.29](http://commonmark.org/) syntax MAY be used for rich text representation. |
| extent           | \[number\|string\|null]      | If the variable consists of [ordinal](https://en.wikipedia.org/wiki/Level_of_measurement#Ordinal_scale) values, the extent (lower and upper bounds) of the values as two-element array. Use `null` for open intervals. |
| values           | \[number\|string]            | An (ordered) list of all values, especially useful for [nominal](https://en.wikipedia.org/wiki/Level_of_measurement#Nominal_level) values. |
| unit             | string                       | The unit of measurement for the data, preferably compliant to [UDUNITS-2](https://ncics.org/portfolio/other-resources/udunits2/) units (singular). |

**type**: The Variable `type` indicates whether what kind of variable is being described. It has two allowed values:

1. `data`: a variable indicating some measured value, for example "precipitation", "temperature", etc.
2. `auxiliary`: a variable that contains coordinate data, but isn't a dimension in `cube:dimensions`.
  For example, the values of the datacube might be provided in the projected coordinate reference system, but
  the datacube could have a variable `lon` with dimensions `(y, x)`, giving the longitude at each point.

See the [CF Conventions](http://cfconventions.org/Data/cf-conventions/cf-conventions-1.8/cf-conventions.html#terminology)
for more on auxiliary coordinates.

## Contributing

All contributions are subject to the
[STAC Specification Code of Conduct](https://github.com/radiantearth/stac-spec/blob/master/CODE_OF_CONDUCT.md).
For contributions, please follow the
[STAC specification contributing guide](https://github.com/radiantearth/stac-spec/blob/master/CONTRIBUTING.md) Instructions
for running tests are copied here for convenience.

### Running tests

The same checks that run as checks on PR's are part of the repository and can be run locally to verify that changes are valid. 
To run tests locally, you'll need `npm`, which is a standard part of any [node.js installation](https://nodejs.org/en/download/).

First you'll need to install everything with npm once. Just navigate to the root of this repository and on 
your command line run:
```bash
npm install
```

Then to check markdown formatting and test the examples against the JSON schema, you can run:
```bash
npm test
```

This will spit out the same texts that you see online, and you can then go and fix your markdown or examples.

If the tests reveal formatting problems with the examples, you can fix them with:
```bash
npm run format-examples
```
