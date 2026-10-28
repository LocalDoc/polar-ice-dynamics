# Roadmap

This document lists the phases for building polar-ice-dynamics. Each phase ends with something that works and can be checked. Each phase is done on its own branch and merged through a pull request.

The order puts a live site online early (Phase 3). Each later phase adds one feature to a working site.

## Phase 0: Project skeleton (done)

- Folder structure for `pipeline/` and `site/`.
- Placeholder GitHub Actions workflows.
- `.gitignore`, `pyproject.toml`, `vite.config.js`.

## Phase 1: Sea-ice extent pipeline

Build the simplest data source first, to test the pipeline end to end.

- `sources/sea_ice_index.py`: download NSIDC daily extent tables for both hemispheres, from 1978 to the present.
- `process/timeseries.py`: write `extent_north.json`, `extent_south.json`, and `climatology.json`. The climatology file holds the 1981-2010 daily median and range.
- `manifest.py` and `cli.py`: `python -m polarice update` runs every source and writes `manifest.json`.
- Basic `pytest` tests that check the structure of the output.

**Done when:** running the pipeline locally produces the JSON files in a local `data/` folder.

## Phase 2: Automated data updates

- Create an orphan `data` branch to hold generated files.
- Implement `update-data.yml`: a daily cron schedule that sets up Python, runs the pipeline, and commits to `data` only when something changed.
- Store any required credentials, such as an Earthdata login, as repository secrets.

**Done when:** the workflow runs on schedule and the `data` branch receives new commits automatically.

## Phase 3: First live site

- Set up Vite: `package.json`, `index.html`, `main.js`.
- `data/loader.js`: read `manifest.json` and fetch data files.
- `charts/seaIceExtent.js`: plot the current year against the 1981-2010 range and the record low.
- Implement `deploy.yml`: build the site, copy in the contents of the `data` branch, and publish to GitHub Pages.
- In repository settings, set the Pages source to GitHub Actions.

**Done when:** `https://<username>.github.io/polar-ice-dynamics/` shows a chart that updates daily. This is the first milestone.

## Phase 4: Polar map

- Add OpenLayers and proj4. Register EPSG:3413 (Arctic) and EPSG:3031 (Antarctic) in `map/projections.js`.
- Add Natural Earth coastlines, cut separately for each pole, to `public/basemap/`.
- `map/views.js`: switch between the North and South views.

**Done when:** each pole renders in the correct projection and can be panned and zoomed.

## Phase 5: Sea-ice concentration on the map

Pipeline:

- Download daily sea-ice concentration grids.
- Convert them to Cloud-Optimized GeoTIFFs in `process/rasters.py`.
- Apply retention rules: daily grids for the last 365 days, monthly grids for the full record.

Site:

- `map/layers.js`: a raster layer colored in the browser from the stored values.
- `ui/timeSlider.js`: date selection with play and pause.
- `ui/legend.js` and `ui/infoPanel.js`: color scale and value under the cursor.

**Done when:** you can scrub through dates and watch sea ice shrink and grow on the map. This is the core feature.

## Phase 6: Ice sheets

- `sources/gravis.py`: download GFZ GravIS basin time series and 50 km mass-change grids for Greenland and Antarctica.
- `charts/iceSheetMass.js`: cumulative mass change in gigatonnes since 2002. Show the 2017-2018 gap between GRACE and GRACE-FO as a break in the line rather than drawing across it.
- `process/basins.py`: basin outlines as simplified GeoJSON. Clicking a basin shows its mass history.
- Add the mass-change raster as a map layer for both poles.

**Done when:** sea ice and ice sheets are both available on the site, and the site covers everything we planned.

## Phase 7: Polish

- Visual design, mobile layout, and loading and error states.
- An "About the data" page and a footer crediting NSIDC and GFZ, as their terms require.
- `docs/data-sources.md`: units, update lag, and known quirks of each dataset.
- README with a screenshot, an explanation of how the site works, and instructions for running it locally.
- Check that the deployed site stays within a few tens of megabytes.

## Phase 8: Hardening

- A CI workflow that runs `pytest` and a linter on every pull request.
- If a data source is unavailable, the pipeline keeps the existing data instead of failing the whole run.
- Branch protection on `main`.
- Email notification when a scheduled workflow fails.

## Where the difficulty lies

- Phases 1 to 3 are mostly plumbing. Expect most debugging to come from data formats, Actions configuration, and deployment paths.
- Phases 4 and 5 are the hardest technically, because of polar projections and in-browser raster rendering.
- Phase 6 reuses patterns built earlier and should go faster.
