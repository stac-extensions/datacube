# Datacube Extension Specification

- **Title:** Datacube
- **Identifier:** <https://stac-extensions.github.io/datacube/v2.3.0/schema.json>
- **Field Name Prefix:** cube
- **Scope:** Item, Collection
- **Extension [Maturity Classification](https://github.com/radiantearth/stac-spec/tree/master/extensions/README.md#extension-maturity):** Candidate
- **Owner**: @m-mohr

This document explains the Datacube Extension to the [SpatioTemporal Asset Catalog](https://github.com/radiantearth/stac-spec) (STAC) specification.
It specifies datacube related metadata, especially their dimensions and potentially more in the future.

- Examples:
  - [Item example](examples/item.json): Shows the basic usage of the extension in a STAC Item
  - [Collection example](examples/collection.json): Shows the basic usage of the extension in a STAC Collection
  - [Item asset example](examples/item_asset.json): Shows usage of the extension in Item Assets
  - [Vector example](examples/vector.json): Shows a vector dimension with geometry metadata
  - [Daymet HI annual example](examples/daymet-hi-annual.json): Shows a real-world Collection with datacube metadata
  - [DGGS item example](examples/dggs.json): Shows usage of a DGGS-based spatial dimension in a STAC Item
- [JSON Schema](json-schema/schema.json)
- [Changelog](./CHANGELOG.md)

## Terminology

This document uses the following terms:

- **Dimension**: A named axis of the datacube, described by a [Dimension Object](#dimension-object).
  Not to be confused with the `axis` field, which specifies the direction of a spatial dimension.
- **Dimension label** (short: *label*): A single position along a dimension,
  e.g. a timestamp, a band name, a coordinate, a geometry or a zone identifier.
  Dimension labels are listed in `values`, bounded by `extent`, spaced by `step` and counted by `count` of a Dimension Object.
- **Cell**: A single element of the datacube, addressed by one dimension label per dimension.
- **Variable**: A named multi-dimensional array of the datacube, described by a [Variable Object](#variable-object).
- **Data value** (short: *value*): The content of a cell for a variable.
  Data values are listed in `values` and bounded by `extent` of a Variable Object. Cells without data contain the `nodata` value.

Please note that the field `values` contains dimension labels in Dimension Objects, but data values in Variable Objects.

For a mapping of these terms to openEO, OGC CIS, CF Conventions, xarray and OLAP, see [Mapping to other terminologies](#mapping-to-other-terminologies).

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
but they differ slightly depending on their use case. All objects share the fields `type`, `description`, and `count` with the same definition,
but `type` may be restricted to certain values. The definition of `axis` is shared between the spatial dimensions, but restricted to
certain values, too. `extent`, `values` and `step` share the same definition, but differ in the supported data types (number or string)
depending on the type of dimension. Whenever it's useful to specify these fields, the objects add the additional fields `reference_system`
and `unit` with very similar definitions across the objects.

The optional `count` field specifies the total number of dimension labels along the dimension.
When `values` and `count` are provided, `count` MUST equal the length of `values`.
`count` is especially useful if `values` is not provided, e.g. for large or irregularly spaced dimensions.
It can't always be derived reliably from `extent` and `step` as it depends on whether the dimension labels represent points or intervals
(e.g. an extent of `[0, 10]` with a step of `1` can have 10 or 11 dimension labels).

`values` is meant for dimensions with a small number of dimension labels, e.g. bands.
For large dimensions, it is RECOMMENDED to provide `extent` (and `step` and `count` if applicable) instead of listing all dimension labels.

### Horizontal Spatial Raster Dimension Object

A spatial raster dimension in one of the horizontal (x or y) directions.

| Field Name       | Type           | Description                                                  |
| ---------------- | -------------- | ------------------------------------------------------------ |
| type             | string         | **REQUIRED.** Type of the dimension, always `spatial`.       |
| axis             | string         | **REQUIRED.** Axis of the spatial raster dimension (`x`, `y`).      |
| description      | string         | Detailed multi-line description to explain the dimension. [CommonMark 0.29](http://commonmark.org/) syntax MAY be used for rich text representation. |
| extent           | \[number]      | **REQUIRED.** Extent (lower and upper bounds) of the dimension as two-element array. Open intervals with `null` are not allowed. |
| values           | \[number]      | Optionally, an ordered list of all dimension labels.         |
| count            | integer        | Total number of dimension labels along the dimension (`>= 0`). |
| step             | number\|null   | The space between consecutive dimension labels. Use `null` for irregularly spaced dimension labels. |
| unit             | string         | The unit of measurement for the dimension labels, preferably compliant to [UCUM](https://ucum.org) (unit code) or [UDUNITS-2](https://ncics.org/portfolio/other-resources/udunits2/) (unit symbol or alternatively singular unit name). |
| reference_system | string\|number\|object | The spatial reference system for the data and the dimension labels (`extent`, `values` and `step`), specified as [numerical EPSG code](http://www.epsg-registry.org/), [WKT2 (ISO 19162) string](http://docs.opengeospatial.org/is/18-010r7/18-010r7.html) or [PROJJSON object](https://proj.org/specifications/projjson.html). Defaults to EPSG code 4326. |

### Vertical Spatial Dimension Object

A spatial dimension in vertical (z) direction.

| Field Name       | Type             | Description                                                  |
| ---------------- | ---------------- | ------------------------------------------------------------ |
| type             | string           | **REQUIRED.** Type of the dimension, always `spatial`.       |
| axis             | string           | **REQUIRED.** Axis of the spatial dimension, always `z`.     |
| description      | string           | Detailed multi-line description to explain the dimension. [CommonMark 0.29](http://commonmark.org/) syntax MAY be used for rich text representation. |
| extent           | \[number\|null\]   | If the dimension labels are [ordinal](https://en.wikipedia.org/wiki/Level_of_measurement#Ordinal_scale), the extent (lower and upper bounds) of the dimension labels as two-element array. Use `null` for open intervals. |
| values           | \[number\|string\] | An ordered list of all dimension labels, especially useful for [nominal](https://en.wikipedia.org/wiki/Level_of_measurement#Nominal_level) dimension labels. |
| count            | integer          | Total number of dimension labels along the dimension (`>= 0`). |
| step             | number\|null     | If the dimension labels are [interval](https://en.wikipedia.org/wiki/Level_of_measurement#Interval_scale) values, the space between consecutive dimension labels. Use `null` for irregularly spaced dimension labels. |
| unit             | string           | The unit of measurement for the dimension labels, preferably compliant to [UCUM](https://ucum.org) (unit code) or [UDUNITS-2](https://ncics.org/portfolio/other-resources/udunits2/) (unit symbol or alternatively singular unit name). |
| reference_system | string\|number\|object | The spatial reference system for the data and the dimension labels (`extent`, `values` and `step`), specified as [numerical EPSG code](http://www.epsg-registry.org/), [WKT2 (ISO 19162) string](http://docs.opengeospatial.org/is/18-010r7/18-010r7.html) or [PROJJSON object](https://proj.org/specifications/projjson.html). Defaults to EPSG code 4326. |

A Vertical Spatial Dimension Object MUST specify an `extent` or `values`. It MAY specify both.

### Spatial DGGS Dimension Object

A spatial dimension based on a Discrete Global Grid System (DGGS).

| Field Name       | Type           | Description                                                  |
| ---------------- | -------------- | ------------------------------------------------------------ |
| type             | string         | **REQUIRED.** Type of the dimension, always `dggs`. |
| description      | string         | Detailed multi-line description to explain the dimension. [CommonMark 0.29](http://commonmark.org/) syntax MAY be used for rich text representation. |
| extent           | \[number\]     | The spatial extent of the DGGS zones as a GeoJSON-style bbox: `[west, south, east, north]` or `[west, south, minz, east, north, maxz]`. |
| values           | \[string\]     | An ordered list of dimension labels, i.e. DGGS zone identifiers. Useful when enumerating a known set of zones (for example, a tile index or explicit zone subset). |
| count            | integer        | Total number of dimension labels (i.e. zones) along the dimension (`>= 0`). |
| reference_system | string         | **REQUIRED.** Identifier of the DGGS reference system (DGGRS), preferably a URI. |

It is RECOMMENDED that a Spatial DGGS Dimension Object specifies at least one of `extent` or `values`
unless it would be equivalent to the bbox/extent in the containing Item/Collection.

For DGGS dimensions, `reference_system` identifies the DGGS reference system (DGGRS), not a coordinate reference system (CRS).
Identifiers such as `https://www.opengis.net/def/dggrs/OGC/1.0/HEALPix` from the
[OGC DGGRS Register](https://defs.opengis.net/prez/catalogs/ogc-cat:register/col/def:dggrs) are valid examples.
Additional DGGS metadata, e.g. the refinement level (`dggs:refinement_level`) or the indexing scheme of the zone identifiers,
can be provided through the [DGGS Extension](https://github.com/stac-extensions/dggs).

Guidance for vertical information with DGGS:

- Most DGGS implementations used in practice are surface-based (2D).
- For elevation, depth, pressure level, or altitude, prefer a separate
  [Vertical Spatial Dimension Object](#vertical-spatial-dimension-object) with `axis` set to `z`.
- Use DGGS-only encoding of `z` only if the selected DGGRS natively defines volumetric zones and the zone identifiers represent those volumes.
- In most interoperable STAC/datacube use cases, model horizontal partitioning as `dggs`
  and vertical structure as a separate `z` dimension.

### Temporal Dimension Object

A temporal dimension based on the ISO 8601 standard. The temporal reference system for the data is expected to be ISO 8601 compliant
(Gregorian calendar / UTC). Data not compliant with ISO 8601 can be represented as an *Additional Dimension Object* with `type` set to `temporal`.

| Field Name | Type            | Description                                                  |
| ---------- | --------------- | ------------------------------------------------------------ |
| type       | string          | **REQUIRED.** Type of the dimension, always `temporal`.      |
| description | string         | Detailed multi-line description to explain the dimension. [CommonMark 0.29](http://commonmark.org/) syntax MAY be used for rich text representation. |
| extent     | \[string\|null] | **REQUIRED.** Extent (lower and upper bounds) of the dimension as two-element array. The dates and/or times must be strings compliant to [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601). `null` is allowed for open date ranges. |
| values     | \[string]       | An ordered list of all dimension labels. The dates and/or times must be strings compliant to [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601). |
| count      | integer         | Total number of dimension labels along the dimension (`>= 0`). |
| step       | string\|null    | The space between consecutive dimension labels as [ISO 8601 duration](https://en.wikipedia.org/wiki/ISO_8601#Durations), e.g. `P1D`. Use `null` for irregularly spaced dimension labels. |
| unit       | string          | The unit of measurement for the dimension labels, preferably compliant to [UCUM](https://ucum.org) (unit code) or [UDUNITS-2](https://ncics.org/portfolio/other-resources/udunits2/) (unit symbol or alternatively singular unit name). |

### Spatial Vector Dimension Object

A vector dimension that defines a spatial dimension based on geometries.

| Field Name       | Type           | Description                                                  |
| ---------------- | -------------- | ------------------------------------------------------------ |
| type             | string         | **REQUIRED.** Type of the dimension, always `geometry`.    |
| axes             | \[string]      | Axes of the vector dimension as an ordered set of `x`, `y` and `z`. Defaults to `x` and `y`. |
| description      | string         | Detailed multi-line description to explain the dimension. [CommonMark 0.29](http://commonmark.org/) syntax MAY be used for rich text representation. |
| bbox             | \[number]      | **REQUIRED.** A single bounding box of the geometries as defined for [STAC Collections](https://github.com/radiantearth/stac-spec/blob/master/collection-spec/collection-spec.md#spatial-extent-object), but not nested. |
| values           | \[string\]     | Optionally, an ordered list of dimension labels representing the geometries. This could be a list of WKT strings or other identifiers. |
| count            | integer        | Total number of dimension labels (i.e. geometries) along the dimension (`>= 0`). |
| geometry_types   | \[[GeoJSON Types](https://www.rfc-editor.org/rfc/rfc7946#section-1.4)] | A set of geometry types. If not present, mixed geometry types must be assumed. |
| reference_system | string\|number\|object | The spatial reference system for the data, the `bbox` and the dimension labels (`values`), specified as [numerical EPSG code](http://www.epsg-registry.org/), [WKT2 (ISO 19162) string](http://docs.opengeospatial.org/is/18-010r7/18-010r7.html) or [PROJJSON object](https://proj.org/specifications/projjson.html). Defaults to EPSG code 4326. |

For a general explanation what a vector datacube and a vector dimension is, please read the article "[Vector Data Cubes](https://r-spatial.org/r/2022/09/12/vdc.html)".

### Additional Dimension Object

An additional dimension that is not `spatial`, but may be `temporal` if the data is not compliant with ISO 8601 (see below).

| Field Name       | Type              | Description                                                  |
| ---------------- | ----------------- | ------------------------------------------------------------ |
| type             | string            | **REQUIRED.** Custom type of the dimension, never `spatial` or `geometry`. |
| description      | string            | Detailed multi-line description to explain the dimension. [CommonMark 0.29](http://commonmark.org/) syntax MAY be used for rich text representation. |
| extent           | \[number\|null]   | If the dimension labels are [ordinal](https://en.wikipedia.org/wiki/Level_of_measurement#Ordinal_scale), the extent (lower and upper bounds) of the dimension labels as two-element array. Use `null` for open intervals. |
| values           | \[number\|string] | An ordered list of all dimension labels, especially useful for [nominal](https://en.wikipedia.org/wiki/Level_of_measurement#Nominal_level) dimension labels. |
| count            | integer           | Total number of dimension labels along the dimension (`>= 0`). |
| step             | number\|null      | If the dimension labels are [interval](https://en.wikipedia.org/wiki/Level_of_measurement#Interval_scale) values, the space between consecutive dimension labels. Use `null` for irregularly spaced dimension labels. |
| unit             | string            | The unit of measurement for the dimension labels, preferably compliant to [UCUM](https://ucum.org) (unit code) or [UDUNITS-2](https://ncics.org/portfolio/other-resources/udunits2/) (unit symbol or alternatively singular unit name). |
| reference_system | string            | The reference system for the data and the dimension labels (`extent`, `values` and `step`). |

An Additional Dimension Object MUST specify an `extent` or `values`. It MAY specify both.

Note on "Additional Dimension" with type `temporal`:
You can distinguish the "Temporal Dimension" from an "Additional Dimension" by checking whether the extent exists and contains strings.
So if the `type` equals `temporal` and `extent` is an array of strings/null, then you have a "Temporal Dimension",
otherwise you have an "Additional Dimension".

### Variable Object

A *Variable Object* defines a variable (or a multi-dimensional array). The variable may have dimensions, which are described by [Dimension Objects](#dimension-object).

| Field Name       | Type                     | Description |
| ---------------- | -------------------------| ----------- |
| dimensions       | \[string]                | **REQUIRED.** The dimensions of the variable. This should refer to keys in the ``cube:dimensions`` object or be an empty list if the variable has no dimensions. |
| type             | string                   | **REQUIRED.** Type of the variable, either `data` or `auxiliary`. | 
| description      | string                   | Detailed multi-line description to explain the variable. [CommonMark 0.29](http://commonmark.org/) syntax MAY be used for rich text representation. |
| extent           | \[number\|string\|null\] | If the data values are [ordinal](https://en.wikipedia.org/wiki/Level_of_measurement#Ordinal_scale), the extent (lower and upper bounds) of the data values as two-element array. Use `null` for open intervals. |
| values           | \[number\|string]        | An (ordered) list of all data values, especially useful for [nominal](https://en.wikipedia.org/wiki/Level_of_measurement#Nominal_level) data values. |
| unit             | string                   | The unit of measurement for the data values, preferably compliant to [UCUM](https://ucum.org) (unit code) or [UDUNITS-2](https://ncics.org/portfolio/other-resources/udunits2/) (unit symbol or alternatively singular unit name). |
| nodata           | number\|string           | Data value used to identify cells without data, see [common metadata](https://github.com/radiantearth/stac-spec/blob/v1.1.0/commons/common-metadata.md#no-data) for more details. |
| data_type        | string                   | The data type of the data values, see [common metadata](https://github.com/radiantearth/stac-spec/blob/v1.1.0/commons/common-metadata.md#data-types) for more details. |

**type**: The Variable `type` indicates whether what kind of variable is being described. It has two allowed values:

1. `data`: a variable containing measured or derived data values, for example "precipitation", "temperature", etc.
2. `auxiliary`: a variable that contains coordinate data, but isn't a dimension in `cube:dimensions`.
  For example, the dimension labels of the spatial dimensions might be provided in a projected coordinate reference system, but
  the datacube could have a variable `lon` with dimensions `(y, x)`, giving the longitude for each cell.

One-dimensional variables that only contain the dimension labels of a dimension (e.g. `time(time)`, called coordinate variables in CF)
are described by the corresponding Dimension Object and SHOULD NOT be listed in `cube:variables`.

See the [CF Conventions](http://cfconventions.org/Data/cf-conventions/cf-conventions-1.8/cf-conventions.html#terminology)
for more on auxiliary coordinates.

## Mapping to other terminologies

The following table maps the terms to related standards and tools.
The concepts are similar, but don't always match exactly.

| This extension      | [openEO](https://openeo.org/documentation/1.0/datacubes.html) | [OGC CIS 1.1](https://docs.ogc.org/is/09-146r8/09-146r8.html) / ISO 19123 | [CF Conventions](https://cfconventions.org/) | [xarray](https://docs.xarray.dev/en/stable/user-guide/terminology.html) | OLAP         |
| ------------------- | --------------------- | ------------------------------ | ------------------- | ---------------------- | ------------ |
| Dimension           | dimension             | axis                           | dimension           | dimension              | dimension    |
| Dimension label     | dimension label       | coordinate                     | coordinate value    | coordinate label       | member       |
| Cell                | -                     | grid point / grid cell         | cell                | element                | cell         |
| Dimension labels of a dimension | dimension labels | coordinates of an axis | coordinate variable | dimension coordinate | members of a dimension |
| Variable (`data`)   | label of a bands dimension | range component           | data variable       | data variable          | measure      |
| Variable (`auxiliary`) | -                  | -                              | auxiliary coordinate variable | non-dimension coordinate | -  |
| Data value          | value                 | range value                    | data value          | value (`.values`)      | cell value   |
| `step`              | resolution            | resolution                     | -                   | -                      | -            |
| `count`             | -                     | -                              | dimension length    | size (`.sizes`)        | -            |

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
