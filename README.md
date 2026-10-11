# Sycamore Medicare Guide annual figures

This folder is the source package for the app's remotely refreshable annual Medicare figures.

Planned production URL:

`https://config.restingsycamore.com/medicare-guide/annual-values.json`

Planned repository:

`https://github.com/papiofficial/sycamore-medicare-guide-config`

The reviewed JSON will be served by GitHub Pages from that dedicated public
repository. Do not reuse `papiofficial/resting-sycamore-pages`; it already
serves Webflow embed scripts whose URLs must remain unchanged.

The app must not point at the production endpoint in an uploaded binary until
the GitHub Pages URL returns the reviewed file over HTTPS at the custom domain.

## Contact config (added 2026-10-11)

`contact-config.json` is served at
`https://config.restingsycamore.com/medicare-guide/contact-config.json` for the
sign-up app (App 1). It holds the states where Peter is licensed, the booking
link, and the SHIP locator link. The app validates it (known state codes, HTTPS,
allow-listed booking and SHIP hosts) and falls back to its bundled copy, so a
bad file can't redirect anyone. Peter's phone and email are compiled into the
app on purpose and are not in this file. Keep the bundled copy in
`Apps/SignUpHelp/Resources/contact-config.json` (rsa-medicare-apps) identical.

## Cache policy

GitHub Pages controls its CDN response headers and does not provide per-file
cache-header configuration. The app therefore does not rely on a promised CDN
time-to-live:

- the app asks URLSession to revalidate its cached response;
- every document must pass schema, date, source, range, and IRMAA-tier checks;
- last year's values stop working on January 1 even if a CDN returns them;
- a deployment is not considered live until repeated production checks return
  the new file's exact checksum.

The production verification receipt must record the observed `Cache-Control`,
`ETag`, `Last-Modified`, `Age`, and `Content-Type` headers. GitHub's observed
cache behavior is not a substitute for the checksum check.

`netlify.toml` remains in the parent working folder only as superseded local
history. It is excluded from the dedicated GitHub repository, is not copied
into the GitHub Pages artifact, and must not be deployed.

## GitHub Pages publication contract

The Pages workflow is intentionally manual:

`.github/workflows/deploy-pages.yml`

It validates the JSON shape and 64 KB limit, copies only these files into the
Pages artifact, and deploys that artifact:

- `annual-values.json` as `medicare-guide/annual-values.json`;
- `.nojekyll`.

A custom GitHub Actions Pages workflow ignores a `CNAME` file. The repository's
`CNAME` records the intended hostname for operators, while the actual custom
domain is configured through the GitHub Pages setting or API.

A merge or push does not publish by itself. An authorized operator must run the
`Deploy annual values to GitHub Pages` workflow after the reviewed commit is on
`main`.

## Annual update sequence

1. Verify each value against the current CMS, SSA, or Medicare.gov source.
2. Add the new year beside the existing year in `annual-values.json`.
3. Set `validFrom` and `validThrough` to the new calendar year.
4. Update `publishedAt`, `lastVerified`, and the source URLs.
5. Run the app's config validation tests.
6. Compare the deployable and bundled file checksums.
7. Commit the reviewed file to the dedicated GitHub repository.
8. Run the manual Pages deployment workflow after publication approval.
9. Confirm the production URL returns HTTP 200, the exact reviewed JSON, and
   an appropriate content type. Record the observed cache and validation
   headers.
10. Repeat the checksum check until the custom-domain response matches the
   reviewed file. Do not treat the update as live before it matches.
11. If an urgent correction is needed, fix the reviewed JSON and make a new
   reviewed commit plus manual Pages deployment. Verify the production checksum
   again.

Suggested verification:

```text
# From the iOS workspace root:
swift test
shasum -a 256 remote-config/annual-values.json Sources/MedicareNavigatorApp/Resources/medicare-annual-values.json

# After deployment:
curl --fail --silent --show-error --dump-header - --output /tmp/sycamore-annual-values.json https://config.restingsycamore.com/medicare-guide/annual-values.json
shasum -a 256 /tmp/sycamore-annual-values.json remote-config/annual-values.json
```

The two final checksums must match. Never print secrets while deploying.

The app validates the schema, dates, source URLs, dollar ranges, and IRMAA tier ordering before accepting an update. It caches only a validated document. If no valid values exist for the current calendar year, annual-figure calculators stop and ask the user to refresh instead of using an expired year.
