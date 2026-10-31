# kasseigo-packs — city packs workflow

Public repository whose only job is to build the KasseiGo city packs and publish them in its release
`packs`, where the app downloads them:
`https://github.com/clementmtz/kasseigo-packs/releases/download/packs/` (`manifest.json`, and the
packs it names: `<city>-<hash>.kgpack.gz`). Workspace-wide rules are in `../CLAUDE.md`: read it too.
The pack and manifest format is contract 1 of `../kasseigo/docs/workspace/contracts.md`.

## How it works

`.github/workflows/packs.yml` runs every Monday at 03:00 UTC and on demand. It checks out the private
`kasseigo` repository with a read-only deploy key (secret `SOURCE_DEPLOY_KEY`, set by the owner), runs
the engine tests, builds each city with `backend/packs/build.sh`, and publishes them with
`backend/packs/publish.py`: each pack under a name of its own content, then `manifest.json` last,
then the files that neither it nor the previous manifest names are deleted. An app therefore never
reads a manifest whose files are missing or not yet replaced. The app compares the published SHA-256
with its own and downloads a newer pack in the background. Third-party actions are pinned to a
commit (this job holds the deploy key): update a pin on purpose, with the version in its comment.

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
