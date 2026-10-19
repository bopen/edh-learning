# edh-learning revision plan

Working notes, not to be committed. Status as of 2026-10-19, branch `api-key-zarr-v3-updates`.

Every notebook is linked directly from social posts. It has to work for a reader who
doesn't know EDH and has no DestinE account (so it is committed with outputs), and it
should still be a useful reference for existing users.

## What changed since the notebooks were written

- ERA5 and the test dataset moved from Zarr v2 to v3. The test dataset is now
  `test-dataset-v1.zarr`; `v0` is still served as Zarr v2.
- The ERA5 ECV dataset was dropped in favour of ERA5 monthly
  (`era5/era5-single-levels-atmosphere-monthly-v0.zarr`: 860 GB, 213 variables,
  chunks `(120, 240, 240)`).
- Personal access tokens were replaced by API keys from a different provider. There are now
  two data entry points:
  - `data.earthdatahub.destine.eu`: keys created on the DestinE platform get
    `426 Upgrade Required` here (see edh-docs troubleshooting).
  - `api.earthdatahub.destine.eu`
  - The edh-docs `.netrc` snippet lists both hosts.

## Phase 1: sync with the current platform

### Rules for all notebooks

- [ ] Call it an "API key" everywhere: no "personal access token" or "PAT". Use the
      environment variable `EDH_KEY`.
- [ ] Choose the host for dataset URLs (open question 1) and use it everywhere.
- [ ] The `.netrc` snippet lists both hosts, gives the Windows path (`%USERPROFILE%\_netrc`),
      and mentions that the file can be downloaded from the account page.
- [ ] Use `test-dataset-v1.zarr` (public and private).
- [ ] Call quota usage "requests", not "credits", matching the docs.
- [ ] Re-run every notebook top to bottom in the Python 3.14 environment so outputs are
      fresh and execution counts are sequential.

### 00-getting-started (sketch in progress)

Already done in the sketch: test dataset `v1` in the code, `EDH_KEY`/`edh_key_`, ERA5 monthly
URL.

Still to do:

- [ ] Cell 0: change "Zarr stores encoded in netCDF format" to plain wording; check the
      "Getting Started page" link is still the right target.
- [ ] Cell 7: "personal access token" → "API key" in the heading and text; the private URL
      in the text still says `v0`.
- [ ] Cell 8: the assert message still says "personal access token". Confirm that every new key
      starts with `edh_key_` (open question 2).
- [ ] Cell 12: `.netrc` with both hosts plus the Windows path; "token" → "API key".
- [ ] Cell 17: drop "homogeneous Zarr stores encoded as a netCDF".
- [ ] Cell 18: rewrite for ERA5 monthly. The "10 GB, smallest dataset" claim no longer
      holds. Fix the link and the typo ("yout").
- [ ] Cell 19: rename `ecv_dataset` (e.g. `era5_monthly`). Its HTML repr is 1.6 MB; consider
      displaying only `t2m`.
- [ ] Cells 20–29: **the result is wrong**. Monthly `t2m` is absolute temperature in K, but
      the plot is labelled "temperature change since 1981-2010 [°C]" and shows the seasonal
      cycle. Compute the anomaly against the 1981–2010 monthly climatology
      (open question 3).
- [ ] Cell 21: **the area of interest misses western Europe**. Longitudes are 0–360, so
      `slice(-10, 40)` selects only 0–40°E (open question 4). Already true before ERA5 monthly.
- [ ] Cell 26: typos ("Until new", "perform the both").
- [ ] Delete the empty trailing cell. Re-run (cells 8–9 have no execution count and stale
      output).

### 10-use-case-era5-small-area and 60-best-practice-local-cache

- [ ] Check the current hourly ERA5 single-levels URL in the catalogue
      (`reanalysis-era5-single-levels-v0.zarr`?) and its chunking after the Zarr v3 move.
- [ ] The chunk-alignment code hard-codes `64`. Read the chunk size from the data
      (`encoding["chunks"]` / `.chunksizes`), and re-check the text that depends on it
      ("64x64", "62 chunks", "11.5GB").
- [ ] 60: match the edh-docs cache snippet (add `"asyncwrapper": {"asynchronous": True}`,
      cache under a `./edh_cache/` path, not `/tmp`). Delete "the default expiry time is one
      week": simplecache never expires files, as the same notebook says a few cells earlier.

### 20-use-case-cmip6-rome

- [ ] Verify the 12 CMIP6 URLs and whether the numcodecs warning filter is still needed.

### 50-use-case-sentinel-1-small-area

- [ ] Verify the URL and the `(5000, 5000)` chunking.

## Phase 2: newcomers and reference users

From the first review; parts already covered in phase 1 are left out.

1. **Standard first cell.** Title that says what you get, the final figure (in
   `notebooks/images/`, because edh-docs only copies `notebooks/`), one-line lead
   (reuse `metadata.description`), and a table: data, runtime, requests used,
   "free DestinE account + API key", date of the outputs.
2. **"Before you run" block, identical in every notebook.** Three steps (register, copy the
   key, `.netrc` with both hosts), plus a check cell that opens the private test dataset and
   turns 401/426 into "register here / use this host" instead of a stack trace. A small
   script keeps the copies identical.
3. **Whole recipe in one cell** at the end of each use case: copy-paste code for returning
   users; shows newcomers how little code it takes.
4. **Show quota cost.** One line before `.compute()` (`.data.npartitions`) → "this uses ~N of
   your 500,000 monthly requests".
5. **README as an index.** Notebook → what you learn, dataset, runtime, requests, account
   needed; plus a lookup of techniques (auth, chunk alignment, caching, combining models,
   UTM/CRS, interactive maps).
6. **Less duplication.** Numbering: 0x onboarding, 1x–5x use cases, 6x+ best practices.
   Move chunk alignment (copied in 10, 50, 60) to its own `61-…` notebook and leave a short
   tip and link in the use cases. Replace the private `xr.core.indexing.map_index_queries`
   with a public API (check `indexes[dim].slice_indexer`).
7. **Upkeep.**
   - Fix text that disagrees with outputs (ERA5 "2GB/11.5GB" vs 3GB/12GB; S1
     "500GB/1.5GB/2.9GB" vs 564GB/1GB/3GB).
   - Fix about 20 typos and add codespell to pre-commit.
   - Trim the multi-MB HTML reprs for mobile readers.

## Open questions

1. **Host:** keep `data.` in URLs as the edh-docs examples do (`.netrc` covers both), or move
   everything to `api.` so DestinE-platform keys don't hit 426?
2. **Key prefix:** do all new API keys, including those created on the DestinE platform,
   start with `edh_key_`? If not, the assert in 00 rejects valid keys.
3. **00 result:** anomaly against 1981–2010 (keeps the story, about 2 more lines), or plain
   mean temperature in °C?
4. **00 area:** convert longitudes to -180..180 to cover all of Europe, or keep 0–40°E and
   describe it as such?
5. Do the social posts link to GitHub or to the docs site? This decides where the header and
   setup block live.
6. Could EDH expose a small public subset, so the use cases have a "try before
   registering" path?
7. Add "Run in Insula Code Lab" or Colab links?

Side note for edh-docs: `data-access.md` still hard-codes the public `test-dataset-v0.zarr`
while the private example uses `v1`.
