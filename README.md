# Western Water Datahub EDR Guide

Standalone GitHub Pages source for WWDH-focused EDR tutorials and workflow
guidance.

## Edit content

- `index.md` is the landing page.
- `field-guide.md` explains the available datasets and collection families.
- `raw-http.md` walks through the API workflow with HTTP, Python, raw R, and
  `edr4r` examples.
- `_layouts/default.html` controls the page frame.
- `assets/css/site.css` controls WWDH/USBR styling.

All tutorial pages are Markdown-editable and build as static HTML. The build
does not call the live WWDH API.

## Publish on GitHub Pages

1. Create a new GitHub repository.
2. Push this directory as the repository root.
3. In repository settings, enable GitHub Pages with **GitHub Actions** as the
   source.
4. Push to `main`; `.github/workflows/pages.yml` builds and deploys the site.

## Local preview

With Ruby 3.x available:

```sh
gem install jekyll
jekyll serve
```

Then open <http://127.0.0.1:4000/>.

No local build is required for publishing; the included GitHub Actions workflow
uses GitHub's hosted Pages/Jekyll builder.

## Branding sources

The color tokens and Bureau of Reclamation logo files are taken from the public
Western Water Datahub project:

- <https://github.com/internetofwater/Western-Water-Datahub>
- <https://wwdh.internetofwater.app/>
- <https://api.wwdh.internetofwater.app>
