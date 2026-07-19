# wingflight-artifacts

Mirrors built binary artifacts from other WingFlight repos into plain git-tracked
files, so they can be fetched directly from a browser.

## Why this repo exists

GitHub Release assets are served from `release-assets.githubusercontent.com`,
which does not send `Access-Control-Allow-Origin` — any in-browser `fetch()`
of a release asset is blocked by CORS, regardless of the requesting origin.
Files committed to a public repo don't have this problem: `raw.githubusercontent.com`
sends permissive CORS headers for any public repo's tracked files, and
[jsDelivr](https://www.jsdelivr.com/) mirrors public GitHub repos automatically
with proper CDN caching on top.

So instead of the browser downloading straight from a GitHub Release, the
originating repo's CI also commits the built file here, and consumers (e.g.
wingflight-configurator's firmware flasher) fetch it from here instead.

## Layout

Each producing repo/workflow gets its own top-level folder, then a
version/tag-scoped subfolder so nothing is ever overwritten:

```
firmware/<tag>/<filename>
```

`<tag>` is the exact git tag from `wingflight-firmware` that produced the
build (e.g. `release/1.2.3` or `snapshot/0.0.7`).

Future binary-artifact needs (other repos, other kinds of build output) can
add their own top-level folder following the same `<name>/<tag>/<file>` shape.

## Fetching a file

Prefer jsDelivr over raw.githubusercontent.com — same CORS guarantees, plus
real CDN caching and much higher rate limits:

```
https://cdn.jsdelivr.net/gh/WingFlight/wingflight-artifacts@master/firmware/<tag>/<filename>
```

Falls back cleanly to the raw form if ever needed:

```
https://raw.githubusercontent.com/WingFlight/wingflight-artifacts/master/firmware/<tag>/<filename>
```

## Adding a new producer

1. Add a step to the producing workflow that clones this repo (using a PAT
   with `Contents: Read and write` on this repo, stored as a secret in the
   *producing* repo), copies the built file(s) into
   `<top-level-folder>/<tag>/`, commits, and pushes.
2. Nothing needs to change here — the folder just starts existing on next push.
