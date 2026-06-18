---
title: Western Water Datahub EDR Guide
hero_title: Western Water Datahub EDR Guide
lede: A standalone, Markdown-editable guide for exploring the Western Water Datahub OGC API - Environmental Data Retrieval service.
---

The Western Water Datahub publishes water-relevant datasets through a unified
OGC API interface supported by the United States Bureau of Reclamation. This
site packages the WWDH-focused `edr4r` vignette material as a standalone
GitHub Pages resource for analysts, water data SMEs, and developers who need
repeatable examples without digging through package documentation.

<div class="card-grid">
  <section class="card">
    <h2>Collection Field Guide</h2>
    <p>Start here when choosing which WWDH collection and query shape to use.</p>
    <p><a class="button-link" href="{{ '/field-guide/' | relative_url }}">Open the field guide</a></p>
  </section>
  <section class="card">
    <h2>Raw HTTP Guide</h2>
    <p>Use browser URLs, curl, or any HTTP client to call the WWDH API directly.</p>
    <p><a class="button-link" href="{{ '/raw-http/' | relative_url }}">Open the HTTP guide</a></p>
  </section>
</div>

## What this site is

- A standalone GitHub Pages site, independent from the `edr4r` package site.
- Plain Markdown source that can be edited directly in GitHub.
- Static documentation: no live API calls are made while building the site.
- Branded with WWDH/USBR color tokens and Bureau of Reclamation logo assets
  from the public Western Water Datahub repository.

## Key links

- [Western Water Datahub application](https://wwdh.internetofwater.app/)
- [WWDH EDR API](https://api.wwdh.internetofwater.app)
- [WWDH OpenAPI reference](https://api.wwdh.internetofwater.app/openapi)
- [Western Water Datahub GitHub repository](https://github.com/internetofwater/Western-Water-Datahub)
- [`edr4r` package site](https://ksonda.github.io/edr4r/)
