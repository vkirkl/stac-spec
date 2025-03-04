# STAC Collection Specification <!-- omit in toc -->

- [Overview](#overview)
- [Collection fields](#collection-fields)
  - [stac\_version](#stac_version)
  - [stac\_extensions](#stac_extensions)
  - [id](#id)
  - [license](#license)
  - [providers](#providers)
    - [Provider Object](#provider-object)
  - [extents](#extents)
    - [Extent Object](#extent-object)
    - [Spatial Extent Object](#spatial-extent-object)
    - [Temporal Extent Object](#temporal-extent-object)
  - [summaries](#summaries)
    - [Range Object](#range-object)
    - [JSON Schema Object](#json-schema-object)
  - [links](#links)
    - [Relation types](#relation-types)
  - [assets](#assets)
  - [item\_assets](#item_assets)
    - [Item Asset Definition Object](#item-asset-definition-object)
- [Media Type for STAC Collections](#media-type-for-stac-collections)
- [Standalone Collections](#standalone-collections)
- [Extensions](#extensions)

## Overview

The STAC Collection Specification defines a set of common fields to describe a group of Items that share properties and metadata. The
Collection Specification shares all fields with the STAC [Catalog Specification](../catalog-spec/catalog-spec.md) (with different allowed
values for `type` and `stac_extensions`) and adds fields to describe the whole dataset and the included set of Items. Collections
can have both parent Catalogs and Collections and child Items, Catalogs and Collections.

A STAC Collection is represented in JSON format.
Any JSON object that contains all the required fields is a valid STAC Collection and also a valid STAC Catalog.

STAC Collections are compatible with the [Collection](http://docs.opengeospatial.org/is/17-069r3/17-069r3.html#example_4) JSON
specified in [*OGC API - Features*](https://ogcapi.ogc.org/features/), but they are extended with additional fields.  

- [Examples](../examples/):
  - Sentinel 2: A basic standalone example of a [Collection](../examples/collection-only/collection.json) without Items.
  - Simple Example: A [Collection](../examples/collection.json) that links to 3 example Items.
  - Extension Collection: An additional [Collection](../examples/extensions-collection/collection.json), which is used to highlight
  various [extension](../extensions) functionality, but serves as another example.
- [JSON Schema](json-schema/collection.json)

## Collection fields

| Element         | Type                                                                                         | Description                                                                                                                                                               |
| --------------- | -------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| type            | string                                                                                       | **REQUIRED.** Must be set to `Collection` to be a valid Collection.                                                                                                       |
| stac_version    | string                                                                                       | **REQUIRED.** The STAC version the Collection implements.                                                                                                                 |
| stac_extensions | \[string]                                                                                    | A list of extension identifiers the Collection implements.                                                                                                                |
| id              | string                                                                                       | **REQUIRED.** Identifier for the Collection that is unique across all collections in the root catalog.                                                                    |
| title           | string                                                                                       | A short descriptive one-line title for the Collection.                                                                                                                    |
| description     | string                                                                                       | **REQUIRED.** Detailed multi-line description to fully explain the Collection. [CommonMark 0.29](http://commonmark.org/) syntax MAY be used for rich text representation. |
| keywords        | \[string]                                                                                    | List of keywords describing the Collection.                                                                                                                               |
| license         | string                                                                                       | **REQUIRED** License(s) of the data collection as SPDX License identifier, SPDX License expression, or `other` (see below).                                               |
| providers       | \[[Provider Object](#provider-object)]                                                       | A list of providers, which may include all organizations capturing or processing the data or the hosting provider.                                                        |
| extent          | [Extent Object](#extent-object)                                                              | **REQUIRED.** Spatial and temporal extents.                                                                                                                               |
| summaries       | Map<string, \[\*]\|[Range Object](#range-object)\|[JSON Schema Object](#json-schema-object)> | STRONGLY RECOMMENDED. A map of property summaries, either a set of values, a range of values or a [JSON Schema](https://json-schema.org).                                 |
| links           | \[[Link Object](#links)]                                                                     | **REQUIRED.** A list of references to other documents.                                                                                                                    |
| assets          | Map<string, [Asset Object](#assets)>                                                         | Dictionary of asset objects that can be downloaded, each with a unique key.                                                                                               |
| item_assets     | Map<string, [Item Asset Definition Object](#item-asset-definition-object)>                   | A dictionary of assets that can be found in member Items.                                                                                                                 |

### stac_version

In general, STAC versions can be mixed, but please keep the [recommended best practices](../best-practices.md#mixing-stac-versions) in mind.

### stac_extensions

A list of extensions the Collection implements.
The list consists of URLs to JSON Schema files that can be used for validation.
This list must only contain extensions that extend the Collection specification itself,
see the 'Scope' for each of the extensions.
This must **not** declare the extensions that are only implemented in child Collection objects or child Item objects.

### id

It is important that Collection identifiers are unique across all collections in the corresponding root catalog.
Providers should strive as much as possible to make their Collection ids 'globally' unique, prefixing any common information with a unique string.
This could be the provider's name if it is a fairly unique name, or their name combined with the domain they operate in.

### license

License(s) of the data that the STAC Collection and its children provides.
If possible, license information should be defined at the Collection level.

The license(s) can be provided as:

1. [SPDX License identifier](https://spdx.org/licenses/)
2. [SPDX License expression](https://spdx.github.io/spdx-spec/v2.3/SPDX-license-expressions/)
3. String with the value `other` if the license is not on the SPDX license list.
   The strings `various` and `proprietary` are **deprecated**.

If the license is **not** an SPDX license identifier, links to the license texts SHOULD be added.
The links MUST use the [`license` link relation type](#relation-types).
If there is no public license URL available,
it is RECOMMENDED to supplement the STAC Item with the license text in a separate file and link to this file.
If no link to a license is included and the `license` field is set to `other` (or one of the deprecated values),
the Collection is private, and consumers have not been granted any explicit right to use the data.

### providers

A list of providers, which may include all organizations capturing or processing the data or the hosting provider.
Providers should be listed in chronological order with the most recent provider being the last element of the list.

#### Provider Object

The object provides information about a provider.
A provider is any of the organizations that captures or processes the content of the Collection
and therefore influences the data offered by this Collection.
May also include information about the final storage provider hosting the data.

| Field Name  | Type      | Description                                                                                                                                                                                                                                                            |
| ----------- | --------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| name        | string    | **REQUIRED.** The name of the organization or the individual.                                                                                                                                                                                                          |
| description | string    | Multi-line description to add further provider information such as processing details for processors and producers, hosting details for hosts or basic contact information. [CommonMark 0.29](http://commonmark.org/) syntax MAY be used for rich text representation. |
| roles       | \[string] | Roles of the provider. Any of `licensor`, `producer`, `processor` or `host`.                                                                                                                                                                                           |
| url         | string    | Homepage on which the provider describes the dataset and publishes contact information.                                                                                                                                                                                |

**roles**: The provider's role(s) can be one or more of the following elements:

- *licensor*: The organization that is licensing the dataset under the license specified in the Collection's `license` field.
- *producer*: The producer of the data is the provider that initially captured and processed the source data, e.g. ESA for Sentinel-2 data.
- *processor*: A processor is any provider who processed data to a derived product.
- *host*: The host is the actual provider offering the data on their storage.
  There should be no more than one host, specified as last element of the list.

### extents

#### Extent Object

The object describes the spatio-temporal extents of the Collection. Both spatial and temporal extents are required to be specified.

| Element  | Type                                              | Description                                                           |
| -------- | ------------------------------------------------- | --------------------------------------------------------------------- |
| spatial  | [Spatial Extent Object](#spatial-extent-object)   | **REQUIRED.** Potential *spatial extents* covered by the Collection.  |
| temporal | [Temporal Extent Object](#temporal-extent-object) | **REQUIRED.** Potential *temporal extents* covered by the Collection. |

#### Spatial Extent Object

The object describes the spatial extents of the Collection.

| Element | Type         | Description                                                          |
| ------- | ------------ | -------------------------------------------------------------------- |
| bbox    | \[\[number]] | **REQUIRED.** Potential *spatial extents* covered by the Collection. |

**bbox**: Each outer array element can be a separate spatial extent describing the bounding boxes
of the assets represented by this Collection using either 2D or 3D geometries.

The first bounding box always describes the overall spatial extent of the data. All subsequent bounding boxes can be
used to provide a more precise description of the extent and identify clusters of data.
Clients only interested in the overall spatial extent will only need to access the first item in each array.
It is recommended to only use multiple bounding boxes if a union of them would then include
a large uncovered area (e.g. the union of Germany and Chile).
Thus, it doesn't make sense to provide two bounding boxes and the validation will fail in this case.

The length of the inner array must be 2*n where n is the number of dimensions.
The array contains all axes of the southwesterly most extent followed by all axes of the northeasterly most extent specified in
Longitude/Latitude or Longitude/Latitude/Elevation based on [WGS 84](http://www.opengis.net/def/crs/OGC/1.3/CRS84).
When using 3D geometries, the elevation of the southwesterly most extent is the minimum depth/height in meters
and the elevation of the northeasterly most extent is the maximum.

The coordinate reference system of the values is WGS 84 longitude/latitude.
Example that covers the whole Earth: `[[-180.0, -90.0, 180.0, 90.0]]`.
Example that covers the whole earth with a depth of 100 meters to a height of 150 meters: `[[-180.0, -90.0, -100.0, 180.0, 90.0, 150.0]]`.

#### Temporal Extent Object

The object describes the temporal extents of the Collection.

| Element  | Type               | Description                                                           |
| -------- | ------------------ | --------------------------------------------------------------------- |
| interval | \[\[string\|null]] | **REQUIRED.** Potential *temporal extents* covered by the Collection. |

**interval**: Each outer array element can be a separate temporal extent.
The first time interval always describes the overall temporal extent of the data. All subsequent time intervals
can be used to provide a more precise description of the extent and identify clusters of data.
Clients only interested in the overall extent will only need to access the first item in each array.
It is recommended to only use multiple temporal extents if a union of them would then include a large
uncovered time span (e.g. only having data for the years 2000, 2010 and 2020).

Each inner array consists of exactly two elements, either a timestamp or `null`.

Timestamps consist of a date and time in UTC and MUST be formatted according to
[RFC 3339, section 5.6](https://tools.ietf.org/html/rfc3339#section-5.6).
The temporal reference system is the Gregorian calendar.

Open date ranges are supported by setting the start and/or the end time to `null`.
Example for data from the beginning of 2019 until now: `[["2019-01-01T00:00:00Z", null]]`.
It is recommended to provide at least a rough guideline on the temporal extent and thus
it's not recommended to set both start and end time to `null`. Nevertheless, this is possible
if there's a strong use case for an open date range to both sides.

### summaries

Collections are *strongly recommended* to provide summaries of the values of fields that they can expect from the `properties`
of STAC Items contained in this Collection. This enables users to get a good sense of what the ranges and potential values of
different fields in the Collection are, without having to inspect a number of Items (or crawl them exhaustively to get a definitive answer).
Summaries are often used to give users a sense of the data in [Standalone Collections](#standalone-collections),
describing the potential values even when it can't be accessed as Items. They also give clients enough information to
build tailored user interfaces for querying the data, by presenting the potential values that are available.
 Fields selected to be included in summaries should capture all the potential values of the
 field that appear in every Item underneath the collection, including in any nested sub-Catalogs.

A summary for a field can be specified in three ways:

1. A set of all distinct values in an array: The set of values must contain at least one element and it is strongly recommended to list all values.
   If the field summarizes an array (e.g. [`instruments`](../commons/common-metadata.md#instrument)),
   the field's array elements of each Item must be merged to a single array with unique elements.
2. A Range in a [Range Object](#range-object): Statistics by default only specify the range (minimum and maximum values),
   but can optionally be accompanied by additional statistical values.
   The range specified by the `minimum` and `maximum` properties can specify the potential range of values,
   but it is recommended to be as precise as possible.
3. Extensible JSON Schema definitions for fine-grained information, see the [JSON Schema Object](#json-schema-object)
   section for more.

All values must follow the schema of the property field they summarize, unless the field is an array as described in (1) above.
So the values in the array or the values given for `minimum` and `maximum` must comply to the original data type
and any further restrictions that apply for the property they summarize.
For example, the `minimum` for `gsd` can't be lower than zero and the summaries for `platform` and `instruments`
must each be an array of strings (or alternatively minimum and maximum values, but that's not very meaningful).

It is recommended to list as many properties as reasonable so that consumers get a full overview about the properties included in the Items.
Nevertheless, it is not very useful to list all potential `title` values of the Items.
Also, a range for the `datetime` property may be better suited to be included in the STAC Collection's `extent` field.
In general, properties that are covered by the Collection specification should not be repeated in the summaries.

See the [examples folder](../examples) for Collections with summaries to get a sense of how to use them.

#### Range Object

For summaries that would normally consist of a lot of continuous values, statistics can be added instead.
By default, only ranges with a minimum and a maximum value can be specified.
Ranges can be specified for [ordinal](https://en.wikipedia.org/wiki/Level_of_measurement#Ordinal_scale) values only,
which means they need to have a rank order.
Therefore, ranges can only be specified for numbers and some special types of strings. Examples: grades (A to F), dates or times.
Implementors are free to add other derived statistical values to the object, for example `mean` or `stddev`.

| Field Name | Type           | Description                  |
| ---------- | -------------- | ---------------------------- |
| minimum    | number\|string | **REQUIRED.** Minimum value. |
| maximum    | number\|string | **REQUIRED.** Maximum value. |

#### JSON Schema Object

For a full understanding of the summarized field, a JSON Schema can be added for each summarized field.
This allows very fine-grained information for each field and each value as JSON Schema is also extensible.
Each schema must be valid against all corresponding values available for the property in the sub-Items.
Empty schemas are not allowed.

JSON Schema draft-07 is the default JSON Schema version, which aligns with the JSON Schemas provided by STAC.
It is allowed to use other versions of JSON Schema if the version is explicitly expressed in the JSON Schema `$schema` keyword,
but tooling may not support JSON Schema versions other than `draft-07`.

For an introduction to JSON Schema, see "[Learn JSON Schema](https://json-schema.org/learn/)".

### links

This object is described in the [Links](../commons/links.md) document.

#### Relation types

All the [common relation types](../commons/links.md#relation-types) can be used in Collection.
A `self` and a `root` links are STRONGLY RECOMMENDED.
Non-root Collections SHOULD include a `parent` link to their parent.

> \[!NOTE] The STAC Catalog specification requires a link to at least one `item` or `child` Catalog.
> This is *not* a requirement for Collections, but *recommended*. In contrast to Catalogs,
> it is **REQUIRED** that Items linked from a Collection MUST refer back to its Collection
> with a link with the [`collection` relation type](../item-spec/item-spec.md#relation-types).

### assets

Collection Assets provides an optional mechanism to expose assets that don't make sense at the Item level.
The property `assets` is a dictionary of [Asset Objects](../commons/assets.md#asset-object), each with a unique key.
Each asset refers to data associated with the Collection that can be downloaded or streamed.
This construct is further detailed in the [Assets](../commons/assets.md) document.

There are a few guidelines for using the asset construct at the Collection level:

- Collection-level assets SHOULD NOT list any files also available in Items.
- If possible, item-level assets are always the preferable way to expose assets.

Collection-level assets can be useful in some scenarios, for example:

1. Exposing additional data that applies Collection-wide and you don't want to expose it in each Item.
   This can be Collection-level metadata or a thumbnail for visualization purposes.
2. Individual Items can't properly be distinguished for some data structures,
   e.g. [Zarr](https://zarr.readthedocs.io/) as it's a data structure not contained in single files.
3. Exposing assets for
   "[Standalone Collections](https://github.com/radiantearth/stac-spec/blob/master/collection-spec/collection-spec.md#standalone-collections)".

Often, it is possible to model data and assets with either a Collection or an Item.
In those scenarios we *recommend* to use Items as much as possible, as they are designed for assets.
Using Collection-level assets should only be used if there is no other option.

### item_assets

This serves two purposes:

1. Provide a human-readable definition of assets available in **any** Items
   belonging to this Collection so that the user can determine the key(s)
   of assets they are interested in.
2. Provide a way to programmatically determine what assets are available
   in **any** member Item. Otherwise a random Item needs to be examined to
   determine assets available, but a random Item may not be representative of the set.

An Item Asset Object defined at the Collection level is nearly the same as the
[Asset Object in Items](../commons/assets.md#asset-object), except for two differences.
The `href` field is not required, because Item Asset Definitions don't point to any data by themselves, but at least two other fields must be present.

#### Item Asset Definition Object

An item asset is an object that contains details about the datafiles that will be included in member Items.
Assets included at the Collection level do not imply that all assets are available from all Items.
However, it is recommended that the Asset Definition is a complete set of **all** assets that may be available from **any** member Items.
So this should be the union of the available assets, not just the intersection of the available assets.

| Field Name  | Type      | Description                                                                                                                                                                                  |
| ----------- | --------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| title       | string    | The displayed title for clients and users.                                                                                                                                                   |
| description | string    | A description of the Asset providing additional details, such as how it was processed or created. [CommonMark 0.29](http://commonmark.org/) syntax MAY be used for rich text representation. |
| type        | string    | [Media type](../commons/assets.md#media-types) of the asset.                                                                                                                                 |
| roles       | \[string] | The [semantic roles](../commons/assets.md#roles) of the asset, similar to the use of `rel` in links.                                                                                         |

Other custom fields, or fields from other extensions may also be included in the Asset object.

Any property that exists for a Collection-level asset object must also exist in the corresponding assets object in
each Item. If a collection's asset object contains properties that are not explicitly stated in the Item's asset
object then that property does not apply to the item's asset. Item asset objects at the Collection-level can
describe any of the properties of an asset, but those assets properties and values must also reside in the item's
asset object. To consolidate item-level asset object properties in an API setting, consider storing the STAC Item
objects without the larger properties internally as 'invalid' STAC items, and merge in the desired properties at
serving time from the Collection-level.

At least two fields (e.g. `title` and `type`) are required to be provided, in order for it to adequately describe Item assets.
The two fields must not necessarily be taken from the list above and may include any custom field.

## Media Type for STAC Collections

A STAC Collection is a JSON file ([RFC 8259](https://tools.ietf.org/html/rfc8259)), and thus should use the
[`application/json`](https://tools.ietf.org/html/rfc8259#section-11) as the [Media Type](https://en.wikipedia.org/wiki/Media_type)
(previously known as the MIME Type).

## Standalone Collections

STAC Collections which don't link to any Item are called **standalone Collections**.
To describe them with more fields than the Collection fields has to offer,
it is allowed to re-use the metadata fields defined by extensions for Items in the `summaries` field.
This makes much sense for fields such as `platform` or `proj:code`, which are often the same for a whole Collection,
but doesn't make much sense for `eo:cloud_cover`, which usually varies heavily across a Collection.
The data provider is free to decide, which fields are reasonable to be used.

## Extensions

STAC Collections are [extensible](../extensions/README.md).
Please refer to the [extensions overview](https://stac-extensions.github.io) to find relevant extensions for STAC Collections.
