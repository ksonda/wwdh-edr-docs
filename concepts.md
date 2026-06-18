---
title: "EDR concepts for water data"
hero_title: "EDR concepts"
lede: "A high-level map from OGC API - Environmental Data Retrieval concepts to familiar sites, time series, feature layers, and gridded data."
permalink: /concepts/
---

OGC API - Environmental Data Retrieval, usually shortened to EDR, is a
standard way to ask environmental datasets for values by place, time,
variable, and shape. The Western Water Datahub uses EDR alongside OGC
API Features, so the same service can expose both data values and
feature layers.

This page is the conceptual bridge. It explains how EDR terms such as
collections, items, locations, parameters, coverages, axes, and ranges
map to traditional water-data ideas like sites, reservoirs, station
time series, basin polygons, and forecast layers.

## 1. Discovery versus sampling

EDR is easiest to understand if you separate two jobs:

| Job | Question | Typical routes |
|---|---|---|
| Discovery | What datasets, sites, variables, features, and filters exist? | `/collections`, `/collections/{id}`, `/locations`, `/items`, `/queryables` |
| Sampling | What values do I get for this place, time, variable, or shape? | `/locations/{locId}`, `/cube`, `/area`, `/position` when advertised |

Discovery responses are often JSON metadata or GeoJSON features.
Sampling responses are usually CoverageJSON, a compact format for
values arranged along coordinate axes.

## 2. The core objects

| EDR concept | Plain-language meaning | Traditional water-data analogy |
|---|---|---|
| Service | The whole API at `https://api.wwdh.internetofwater.app` | A data portal or shared catalog |
| Collection | One dataset or provider-backed layer | A station network, reservoir feed, forecast layer, drought layer, or boundary dataset |
| Item | A GeoJSON feature in a collection | A reservoir feature, basin polygon, forecast polygon, boundary, or station-like catalog record |
| Location | A named sampling location advertised by an EDR collection | A SNOTEL station, reservoir, gage, or other site id |
| Parameter | A data variable that can be requested | SWE, snow depth, storage, stage, outflow, precipitation, temperature |
| Coverage | The response object that packages values over space/time | A time series, a set of site series, a gridded slice, or a sampled polygon result |
| Axis | A coordinate dimension inside a coverage domain | `x` longitude, `y` latitude, `z` elevation/depth, `t` time |
| Range | The measured or modeled values for a parameter | The observation/forecast vector or array aligned to the axes |

Collections are the main organizing unit. A collection might be a
station network such as `snotel-edr`, a reservoir data source such as
`rise-edr`, a basin summary layer such as `snotel-huc06-means`, or a
reference layer such as `doi-regions`.

## 3. Items and locations are related, but not identical

`/items` is the OGC API Features view of a collection. It returns
GeoJSON features: points, lines, polygons, or multi-polygons with
properties. Use it when you want catalog records, boundaries, forecast
polygons, basin summaries, or filterable feature attributes.

`/locations` is the EDR view of named sampling locations. It is most
useful for station-style or reservoir-style collections where the next
question is "give me values for this site."

Many EDR collections expose both. A reservoir may appear as a feature in
`/items` and as a sampling location in `/locations`. A feature-only
collection may have `/items` but no EDR sampling routes.

## 4. Parameters are variables

Parameters are the data variables you can request. In raw URLs, the
query parameter is named `parameter-name`. In `edr4r`, the equivalent
argument is `parameter_name`.

Examples:

| Collection | Parameter id | Meaning |
|---|---|---|
| `snotel-edr` | `WTEQ` | Snow water equivalent |
| `snotel-edr` | `SNWD` | Snow depth |
| `rise-edr` | `3` | Daily lake/reservoir storage |
| `rise-edr` | `1830` | Lake/reservoir release - total |
| `usace-edr` | `Stage` | Stage |
| `usace-edr` | `Outflow` | Outflow |
| `resops-edr` | `avg`, `p10`, `p90` | Long-term storage summary statistics |

Always use the exact parameter id from collection metadata. Labels are
for humans; ids are for requests.

## 5. Coverages, axes, and ranges

In a conventional site time-series database, you might think in rows:

| site_id | variable | time | value |
|---|---|---|---|
| `303:CO:SNTL` | `WTEQ` | `2024-02-01` | `11.6` |

CoverageJSON stores the same idea in a more array-oriented form:

``` json
{
  "type": "Coverage",
  "domain": {
    "domainType": "PointSeries",
    "axes": {
      "x": {"values": [-105.0]},
      "y": {"values": [37.3]},
      "t": {"values": ["2024-02-01T00:00:00Z"]}
    }
  },
  "ranges": {
    "WTEQ": {
      "axisNames": ["t"],
      "values": [11.6]
    }
  }
}
```

The `domain.axes` block tells you the coordinates. The `ranges` block
holds values, keyed by parameter id. When `axisNames` is `["t"]`, the
first range value belongs to the first time value. For a grid, the
range may be aligned to multiple axes such as `["t", "y", "x"]`.

A CoverageCollection is simply a bundle of coverages. For station data,
that often means one coverage per site. For gridded data, it may mean
one coverage representing a grid.

## 6. Mapping routes to traditional tasks

| Familiar task | EDR or Features route | Typical response |
|---|---|---|
| List datasets | `/collections` | JSON collection list |
| Inspect one dataset | `/collections/{collectionId}` | JSON collection metadata |
| Find stations or reservoirs | `/collections/{collectionId}/locations` | GeoJSON FeatureCollection |
| Fetch values for one known site | `/collections/{collectionId}/locations/{locId}` | CoverageJSON |
| Fetch values in a bounding box | `/collections/{collectionId}/cube` | CoverageJSON or CoverageCollection |
| Fetch values inside a polygon | `/collections/{collectionId}/area` | CoverageJSON |
| Inspect feature filters | `/collections/{collectionId}/queryables` | JSON Schema |
| Download features | `/collections/{collectionId}/items` | GeoJSON FeatureCollection |

Do not assume every collection supports every route. The collection
metadata is the contract. If `data_queries` contains `cube`, try
`/cube`. If a collection has no `data_queries`, start with
`/queryables` and `/items`.

## 7. The mental model in one pass

For a traditional site time series:

1. Choose the collection, such as `snotel-edr`.
2. Find sites with `/locations`.
3. Choose a parameter, such as `WTEQ`.
4. Request `/locations/{locId}` or `/cube` with `datetime` and
   `parameter-name`.
5. Flatten CoverageJSON axes plus ranges into rows.

For a feature layer:

1. Choose the collection, such as `snotel-huc06-means`.
2. Inspect `/queryables` to learn filter fields.
3. Request `/items` with `bbox`, `limit`, or attribute filters.
4. Treat the response as GeoJSON features with properties.

For code examples in HTTP, Python, raw R, and `edr4r`, continue to the
[API workflow guide]({{ '/raw-http/' | relative_url }}).
