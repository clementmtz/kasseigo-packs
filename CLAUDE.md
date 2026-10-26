# kasseigo-packs — city packs workflow

Public repository whose only job is to build the KasseiGo city packs and publish them in its release
`packs`, where the app downloads them:
`https://github.com/clementmtz/kasseigo-packs/releases/download/packs/` (`manifest.json`,
`brussels.kgpack.gz`, `paris.kgpack.gz`). Workspace-wide rules are in `../CLAUDE.md`: read it too.

## How it works

`.github/workflows/packs.yml` runs every Monday at 03:00 UTC and on demand. It checks out the private
`kasseigo` repository with a read-only deploy key (secret `SOURCE_DEPLOY_KEY`, set by the owner), runs
the engine tests, builds each city with `backend/packs/build.sh`, and uploads `backend/packs/dist/*`
to the release, replacing the previous files. The app compares the published SHA-256 with its own
and downloads a newer pack in the background.

The pack builder itself (OSM reading, addresses, streets, places) lives in
`kasseigo/core/src/build.rs` and `kasseigo/backend/packs/`: change it there, not here.

## Common tasks

- Publish packs now (after a builder fix pushed to `kasseigo`): ask the owner first (the release is
  public and every user downloads it), then
  `gh workflow run packs.yml -R clementmtz/kasseigo-packs` and watch it with `gh run watch`.
- Check what is published: `gh release view packs -R clementmtz/kasseigo-packs`, then download a pack
  and query it with the engine's command-line tool (`kasseigo search …`, see `kasseigo/CLAUDE.md`).
- Add a city: its bounding box and Geofabrik extract in `kasseigo/backend/packs/build.sh`, the city in
  `kasseigo/app/lib/src/cities.dart`, and the city in the loop of `packs.yml` here.

Pack data © OpenStreetMap contributors, ODbL (see `README.md`).
