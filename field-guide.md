---
title: "Western Water Datahub dataset guide"
hero_title: "Dataset field guide"
lede: "Understand what kinds of water datasets are in the API and which collection family fits a traditional water-data task."
permalink: /field-guide/
---

The [Western Water Datahub](https://api.wwdh.internetofwater.app)
publishes water-relevant datasets through an
[OGC API - Environmental Data Retrieval](https://ogcapi.ogc.org/edr/)
and OGC API Features interface. The
[Swagger/OpenAPI page](https://api.wwdh.internetofwater.app/openapi)
is the authoritative endpoint reference. This page is the dataset guide:
it explains what kinds of datasets are present and how to decide which
collection family to open first.

The Datahub is provider-backed and changes over time, so always re-run
discovery before starting production analysis.

## 1. What kinds of datasets are in the API

The API currently advertises 35 collections. They are not all the same
kind of thing. Some are EDR time-series services, some are feature
layers, and some are forecast or reference layers exposed as OGC API
Features.

| Dataset family | Collections | What they contain | Usual first move |
|---|---|---|---|
| Reservoir, lake, and operations time series | `rise-edr`, `usace-edr`, `resviz-edr`, `teacup-edr`, `resops-edr` | Reservoir storage, stage, elevation, inflow, outflow, release, and storage-summary statistics | Use `/locations` or `/items` to find reservoirs, then `/cube` or `/locations/{locId}` for values |
| Snow station observations and forecasts | `snotel-edr`, `awdb-forecasts-edr` | SNOTEL observations, snow water equivalent, snow depth, precipitation, air temperature, and forecast products | Use `/locations` to find stations, then `/cube` for a bbox/time window |
| Basin snow summaries | `snotel-huc06-means` | HUC6 polygons with SWE summaries, basin index attributes, station lists, latest values, and median values | Treat as a feature layer: inspect `/queryables`, then request `/items` |
| NOAA precipitation, snow, icing, temperature, and river forecasts | `noaa-qpf-day-*`, `noaa-cqpf-*`, `noaa-4-inch-snow`, `noaa-8-inch-snow`, `noaa-12-inch-snow`, `noaa-0.25-inch-icing`, `noaa-temp-6-10-day`, `noaa-precip-6-10-day`, `noaa-rfc`, `noaa-river-stage-forecast-day-*` | Forecast features and rasters represented as layers, polygons, points, contours, or provider attributes | Start with collection metadata and `/items`; use `bbox`, `datetime`, and queryable filters when advertised |
| NOAA snow model layers | `nohrsc-swe`, `nohrsc-sd` | Snow water equivalent and snow depth forecast layers from NOHRSC | Treat as feature/coverage-like forecast layers and keep spatial requests small |
| Drought monitor layers | `us-current-drought-monitor`, `us-historical-drought-monitor` | Current and historical drought category polygons | Use `/items` and queryable properties such as date or category when advertised |
| Reference boundaries | `doi-regions`, `area-office-regions` | DOI Unified Interior Regions and USBR Area Office regions | Use `/items` as GIS boundary downloads |

Collections whose ids end in `-edr` usually advertise EDR data queries
such as `locations`, `cube`, `area`, and `items`. Collections without
advertised EDR data queries are often still useful, but they are usually
feature layers rather than time-series sampling services.

If you have seen older examples that use `usgs-prism`, re-run discovery
before relying on that collection because available collections can
change.

For the conceptual model behind collections, items, locations,
parameters, coverages, axes, and ranges, see the
[EDR concepts guide]({{ '/concepts/' | relative_url }}).

## 2. Dataset families in more detail

### Reservoir and operations data

Reservoir collections are the best match for traditional water-ops
questions:

- `rise-edr`: USBR RISE telemetry, including storage and release
  parameters.
- `usace-edr`: USACE Access2Water parameters such as stage, elevation,
  flood storage, conservation storage, inflow, and outflow.
- `resviz-edr` and `teacup-edr`: USBR reservoir storage products, with
  current or historical storage fields such as `raw`, `avg`, `p10`, and
  `p90`.
- `resops-edr`: 30-year USACE reservoir storage summaries with `avg`,
  `p10`, and `p90`.

For these collections, begin by finding reservoirs with `/locations` or
`/items`. Then use `/locations/{locId}` for one reservoir or `/cube` for
many reservoirs in a bbox.

### Snow data

Snow-related collections span both station time series and summarized
basin/forecast products:

- `snotel-edr`: SNOTEL station observations. Common variables include
  `WTEQ`, `SNWD`, `PREC`, and temperature parameters.
- `awdb-forecasts-edr`: AWDB forecast products with site-based EDR
  query support.
- `snotel-huc06-means`: HUC6 basin summary features, not a station
  sampling endpoint.
- `nohrsc-swe` and `nohrsc-sd`: NOAA snow water equivalent and snow
  depth forecast layers.

When you need station time series, use `snotel-edr` or
`awdb-forecasts-edr`. When you need basin status or map-ready snow
layers, use the feature-style collections.

### Forecast and condition layers

NOAA and drought-monitor collections are often map layers or feature
products rather than station time-series feeds. They include:

- Quantitative precipitation forecasts: `noaa-qpf-day-1`,
  `noaa-qpf-day-2`, `noaa-qpf-day-3`, `noaa-qpf-day-4-5`,
  `noaa-qpf-day-6-7`.
- Cumulative precipitation forecasts: `noaa-cqpf-6-hours`,
  `noaa-cqpf-48-hours`, `noaa-cqpf-72-hours`,
  `noaa-cqpf-120-hours`, `noaa-cqpf-168-hours`.
- Probability layers: `noaa-4-inch-snow`, `noaa-8-inch-snow`,
  `noaa-12-inch-snow`, `noaa-0.25-inch-icing`.
- Outlooks and hydrologic forecasts: `noaa-temp-6-10-day`,
  `noaa-precip-6-10-day`, `noaa-rfc`, and the
  `noaa-river-stage-forecast-day-*` collections.
- Drought layers: `us-current-drought-monitor` and
  `us-historical-drought-monitor`.

For these, start by opening the collection metadata and `/queryables`.
Then request `/items` with a small `bbox`, date filter, or low `limit`.

### Boundaries and reference layers

`doi-regions` and `area-office-regions` are reference features. Use them
as GIS context or as a way to join API results to management regions.
They are not time-series endpoints.

## 3. Discovery checklist

Start every session with discovery. This is the durable workflow even
when collection details change.

``` r
library(edr4r)

wwdh <- edr_client("https://api.wwdh.internetofwater.app")

collections <- edr_collections(wwdh)
collections[, c("id", "title", "data_queries")]
```

Then inspect one collection:

``` r
collection <- edr_collection(wwdh, "snotel-edr")
names(collection$data_queries)

params <- edr_parameters(wwdh, "snotel-edr")
params[params$id %in% c("WTEQ", "SNWD", "PREC"),
       c("id", "name", "unit_symbol")]
```

The same discovery is browsable without R:

- `https://api.wwdh.internetofwater.app/collections?f=html`
- `https://api.wwdh.internetofwater.app/collections/snotel-edr?f=html`
- `https://api.wwdh.internetofwater.app/collections/snotel-edr/queryables?f=html`

Look for these fields:

| Metadata field | Why it matters |
|---|---|
| `id` | The collection id used in URLs |
| `title` | Human-readable dataset name |
| `extent` | Rough spatial and temporal coverage |
| `parameter_names` | EDR variables you can request with `parameter-name` |
| `data_queries` | Supported EDR query types such as `locations`, `cube`, and `area` |
| `links` | HTML, JSON, schema, queryables, items, and source-documentation links |

## 4. Choosing a query shape

Choose the smallest query that answers the question.

| Question | Best first query | Why |
|---|---|---|
| What datasets are available? | `/collections?f=json` | Lists every collection and its advertised query types |
| What variables can I request? | `/collections/{id}?f=json` | Parameter ids live in collection metadata |
| Which stations or reservoirs are in my study area? | `/locations?bbox=...` | Returns site features with provider ids |
| I know one site id and want its time series | `/locations/{locId}` | Direct single-site EDR retrieval |
| I want many sites in a bbox | `/cube?bbox=...` | One bulk request across the spatial/time subset |
| I want features or polygons | `/items?bbox=...` | OGC API Features route, good for layers and catalogs |
| I want to filter feature attributes | `/queryables`, then `/items?...` | Queryables tells you valid filter fields |

Good defaults while exploring:

- Use `f=json`.
- Keep bbox order as `min_lon,min_lat,max_lon,max_lat`.
- Start with one parameter and a short time range.
- Prefer `/cube` for bulk EDR values when it is advertised.
- Prefer `/items` for collections with no `data_queries`.

## 5. Practical troubleshooting

| Symptom | Likely cause | What to try |
|---|---|---|
| HTTP 404 on a query route | The collection does not support that route | Re-check `data_queries` on the collection document |
| HTTP 400 or 500 on a large query | The request is too broad or an upstream wrapper failed | Reduce bbox, date range, and parameter count |
| Empty features | No data matched the bbox/date/filter combination | Try the HTML page, then a known active area |
| No parameters returned | The collection is probably feature-style | Use `/queryables` and `/items` |
| Confusing station ids | Provider display ids and URL ids differ | Use the GeoJSON feature `id` returned by `/locations` or `/items` |
| Values are hard to table | CoverageJSON is array-oriented | Flatten axes plus ranges, or use `covjson_to_tibble()` |

Once a small query works, enlarge one dimension at a time: wider bbox,
longer time range, or more parameters. That makes failures easier to
diagnose and keeps the shared API responsive.
