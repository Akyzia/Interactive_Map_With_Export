# Security Audit — Interactive_Map_With_Export

This document summarizes the results of a security review of this repository.

## Scope

The repository is a **purely static, client-side project** with three content files:

- `index.html` — a [Folium](https://python-visualization.github.io/folium/)-generated interactive Leaflet map.
- `cod.ipynb` — the Jupyter notebook that generates `index.html` via Folium.
- `Norte.png` — a compass image embedded as a floating overlay.

There is no backend, no database, no server-side code, no authentication layer,
no API, and no user-session state. This means several classes of vulnerabilities
commonly requested in security reviews are **not applicable** to this codebase.

## Findings

### Not Applicable (no attack surface)

| Requested check                      | Result | Notes                                          |
| ------------------------------------ | ------ | ---------------------------------------------- |
| Hardcoded API keys / secrets         | None   | No API keys or credentials found in any file.  |
| SQL injection                        | N/A    | No database, no SQL, no query strings.         |
| Overly permissive CORS               | N/A    | No backend / no CORS headers are set.          |
| Exposed debug endpoints              | N/A    | No server endpoints exist.                     |
| Missing authentication checks        | N/A    | No authentication system exists.               |

### Findings & Fixes

#### 1. Unpinned third-party CDN script references — **High (supply-chain)** — *fixed*

`index.html` referenced several third-party scripts/stylesheets at mutable paths.
Any compromise of the CDN, DNS hijack, or (for `gh/` references) a force-push
by the upstream maintainer would have let an attacker run arbitrary JavaScript
in the user's browser with the full privileges of the page.

The unpinned references were:

- `https://unpkg.com/leaflet-control-geocoder/dist/Control.Geocoder.{js,css}` — resolves to whatever unpkg decides is the latest version.
- `https://cdn.jsdelivr.net/gh/python-visualization/folium/folium/templates/leaflet.awesome.rotate.min.css` — tracks the `HEAD` of the `python-visualization/folium` GitHub repo.
- `https://cdn.jsdelivr.net/gh/ardhi/Leaflet.MousePosition/src/L.Control.MousePosition.min.{js,css}` — tracks `HEAD` of a third-party user's GitHub repo (`ardhi`), which is especially risky since individual user accounts can be compromised independently of any CDN.

**Fix:** All three were pinned to an immutable reference:

- `leaflet-control-geocoder` → `@3.1.0` on jsdelivr npm.
- `folium` template → `@v0.17.0` tag on jsdelivr/gh.
- `ardhi/Leaflet.MousePosition` → commit SHA `c32f1c84ec49dbf7ad599c51c8659d5e08af0f97` on jsdelivr/gh.

#### 2. Missing Subresource Integrity (SRI) on all external resources — **High (supply-chain)** — *fixed*

Even pinned CDN URLs are trusted blindly by the browser. If a CDN is compromised
(e.g. `cdnjs.cloudflare.com`, `cdn.jsdelivr.net`, `code.jquery.com`), the attacker
can serve modified contents at the same URL. SRI attributes (`integrity="sha384-..."`)
make the browser refuse to execute a script whose hash does not match.

**Fix:** Added `integrity`, `crossorigin="anonymous"`, and `referrerpolicy="no-referrer"` attributes to every `<script>` and `<link>` pointing at a third-party CDN.

#### 3. Reference to a deprecated CDN (`netdna.bootstrapcdn.com`) — **Medium** — *mitigated*

The `bootstrap-glyphicons.css` stylesheet was served from `netdna.bootstrapcdn.com`,
the domain of the discontinued BootstrapCDN project. While the domain still responds
today, it is unmaintained and represents a supply-chain risk.

**Fix (interim):** Pinned with SRI so that if the domain's contents ever change
(or it starts serving malicious content), the browser refuses to apply it.
**Recommended follow-up:** Folium's `AwesomeMarkers` plugin is the only consumer
of this stylesheet. Since Font Awesome 6 is already loaded, migrating the markers
to `prefix='fa'` (Font Awesome) in the notebook and dropping this `<link>` entirely
is the clean long-term fix.

#### 4. No Content Security Policy (CSP) — **Medium** — *fixed*

The page had no CSP, meaning any JavaScript injected via a future bug would
have had unrestricted network access (exfiltration) and unrestricted ability
to load additional scripts.

**Fix:** Added a `<meta http-equiv="Content-Security-Policy">` header that
restricts scripts, styles, images, fonts, and connections to the exact CDN /
tile-server / geocoder origins this map actually needs. The policy keeps
`'unsafe-inline'` for scripts and styles because Folium emits inline
`<script>` and `<style>` blocks; a stricter nonce-based CSP would require
changes to Folium itself.

#### 5. Unvalidated user input via `prompt()` — **Informational** — *no action required*

The draw handler calls `prompt("Insira o nome da camada", "nome")` and
`prompt("Insira um valor", "valor")` and stores the strings in GeoJSON
properties (`props.Camada`, `props.Valor`).

Inside *this* page the values are never rendered as HTML — they only appear
in the exported GeoJSON file (via `encodeURIComponent` + `data:` URI, which
is safe). There is therefore no XSS risk in this project.

**Note for downstream consumers:** If you import the exported GeoJSON into
another tool that renders `Camada`/`Valor` as HTML (e.g. a popup on another
Leaflet map built with `L.popup().setContent(feature.properties.Camada)`),
that downstream tool must HTML-escape the values. This is out of scope for
this repository but worth noting.

#### 6. `cod.ipynb` runs unpinned `pip install` — **Low (supply-chain)** — *not fixed*

The notebook contains `!pip install geopandas folium numpy ipywidgets` with
no version pins. Anyone running the notebook today will get whatever the
latest versions happen to be, which exposes them to dependency-confusion or
malicious-release attacks on any of those packages (or their transitive deps).

**Recommendation (not applied to avoid breaking the user's workflow):**
add a `requirements.txt` with pinned versions and run `pip install -r requirements.txt` in the notebook instead. Suggested starting point:

```
folium==0.17.0
geopandas==1.0.1
numpy==2.1.3
ipywidgets==8.1.5
```

## Generation pipeline note

`index.html` is Folium output. If you re-run `cod.ipynb`, Folium will
regenerate `index.html` and **erase the SRI / CSP / pinning fixes** in this PR.
Upstreaming these fixes to Folium itself is out of scope. Until then, re-apply
this PR's changes to any regenerated `index.html` before committing.

## Summary

| # | Finding                                              | Severity | Status        |
| - | ---------------------------------------------------- | -------- | ------------- |
| 1 | Unpinned third-party CDN references                  | High     | Fixed         |
| 2 | No Subresource Integrity on external resources       | High     | Fixed         |
| 3 | Reference to deprecated BootstrapCDN domain          | Medium   | Mitigated     |
| 4 | No Content Security Policy                           | Medium   | Fixed         |
| 5 | `prompt()` input into GeoJSON properties             | Info     | No action     |
| 6 | Unpinned `pip install` in notebook                   | Low      | Recommendation |
