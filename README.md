# Panoramax API - Upload Extension Specification

- [Overview](#overview)
- [Methods](#methods)
  - [Create a collection](#create-a-collection)
  - [Get upload status](#get-upload-status)
  - [Upload a picture](#upload-a-picture)

## Overview

- **Title:** Upload
- **OpenAPI specification:** [openapi.yaml](openapi.yaml)
- **Conformance Classes:**
  - **Panoramax - Upload** binding: <https://specs.panoramax.fr/upload/v1.0.0>
- **Scope:** STAC API - Collections, STAC API - Features
- **[Extension Maturity Classification](https://github.com/radiantearth/stac-api-spec/tree/main/README.md#maturity-classification):** Stable
- **Dependencies:**
  - [Features](https://github.com/radiantearth/stac-api-spec/tree/v1.0.0/ogcapi-features)
  - [Collections](https://github.com/radiantearth/stac-api-spec/tree/v1.0.0/collections)
- **Owner**: none

The Panoramax API Upload Extension adds support for the creation of new collections and items in collections through POST method requests.
The behavior described here is based on the [OGC API - Features](https://ogcapi.ogc.org/features/) transactions
as specified in [OGC API - Features - Part 4: Create, Replace, Update and Delete](http://docs.opengeospatial.org/DRAFTS/20-002.html).
The core OGC standard lays out the end points for transactions, without specifying any content types.
For Panoramax we use a reduced list of input parameters for collections, and JPEG file + optional body parameters for items.

## Methods

| Path                                              | Content-Type Header                                                            | Body                     | Success Status | Description                                    |
|---------------------------------------------------|--------------------------------------------------------------------------------|--------------------------|----------------|------------------------------------------------|
| `POST /collections/`                              | `application/json`, `application/x-www-form-urlencoded`, `multipart/form-data` | post Collection          | 200            | Adds a new collection.                         |
| `GET /collections/{collectionId}/geovisio_status` |                                                                                |                          | 200            | Retrieve upload status of items in collection. |
| `POST /collections/{collectionId}/items`          | `multipart/form-data`                                                          | JPEG file and parameters | 202            | Adds a new item in a collection.               |

### Create a collection

Collection properties can be passed through one of these content types: `application/json`, `application/x-www-form-urlencoded`, `multipart/form-data`.

Available properties are:
- `title`: the collection title

Response sent with the 200 code is a STAC Collection.

### Get upload status

Response sent with the 200 code is a JSON with upload status of pictures, looking like:

```json
{
  "items": [
    {
      "id": "item_uuid",
      "nb_errors": 0,
      "process_error": "eventual error",
      "processed_at": "2024-03-27T14:32:25.282Z",
      "processing_in_progress": boolean,
      "rank": 1,
      "status": "ready" / "broken" / "waiting-for-process"
    }
  ],
  "status": "ready" / "broken" / "preparing" / "waiting-for-process"
}
```

### Upload a picture

Picture upload is using `multipart/form-data` content type.

Query **should** contain following properties:
- `picture`: the picture JPEG file
- `position`: the item position in collection (starting from 1)

Query **may** contain following properties:
- `isBlurred`: true if picture is already blurred
- `override_capture_time`: datetime when the picture was taken.
It will change the picture's metadata with this datetime.
It should be an iso 3339 formated datetime (like '2017-07-21T17:32:28Z')
- `override_latitude`: latitude of the picture in decimal degrees (WGS84 / EPSG:4326). It will change the picture's metadata with this latitude.
- `override_longitude`: longitude of the picture in decimal degrees (WGS84 / EPSG:4326). It will change the picture's metadata with this longitude.

Response sent with the 202 code is a STAC item, with additionnal properties to know the processing status.
