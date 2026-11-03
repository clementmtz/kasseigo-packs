# kasseigo-packs — city packs workflow

Public repository whose only job is to build the KasseiGo city packs and publish them in its release
`packs`, where the app downloads them:
`https://github.com/clementmtz/kasseigo-packs/releases/download/packs/` (`manifest.json`, and the
packs it names: `<city>-<hash>.kgpack.gz`). Workspace-wide rules are in `../CLAUDE.md`: read it too.
The pack and manifest format is contract 1 of `../kasseigo/docs/workspace/contracts.md`.

## How it works

`.github/workflows/packs.yml` runs every Monday at 03:00 UTC and on demand. One run at a time, in two
jobs, both reading the private `kasseigo` repository with a read-only deploy key (secret
`SOURCE_DEPLOY_KEY`, set by the owner), at the same commit: `build` runs the engine tests and builds
each city with `backend/packs/build.sh` (the engine's dependencies run there, without a shared cache,
nor the signing key); its only artifact is the packs: **never upload the sources as an artifact, those
of a public repository can be downloaded by anyone**. `publish`, which runs none of that code, publishes them with
`backend/packs/publish.py`, the one script of the source it runs (its SHA-256 pinned in the workflow: a new version of the script needs the new hash here, `contracts_test.py` says so; run with `python3 -I`, the cities read from `build.sh`, not by running it), after checking the key is the one the app holds, and keeps the schedule
alive (GitHub stops it after 60 days without activity); a failure opens an issue here, or adds to the one still open. `publish.py` refuses packs that look wrong (a city of `build.sh --list` missing, a date to come, a file not the manifest's, a pack under 70 % of the one before it; on purpose: run by hand with `allow_smaller`). The runners are `ubuntu-24.04`, GitHub's host key is pinned, cities are never listed here. Each pack under a name of its own content, read back from the release against what was sent (sent once more, then nothing published), the list signed with the
secret `PACKS_SIGNING_KEY` (set by `kasseigo/backend/packs/keygen.sh`), then `manifest.json` last,
then the files that neither it nor the previous manifest names are deleted. An app therefore never
reads a manifest whose files are missing or not yet replaced. Its logs are public and the engine's
sources are not: the source is fetched quietly, build and test outputs stay out of the log, and a
failure is reproduced locally (`./kg test`, `./kg packs`). The app compares the published SHA-256
with its own and downloads a newer pack in the background. Third-party actions are pinned to a
commit (these jobs hold the keys): update a pin on purpose, with the version in its comment.

The pack builder itself (OSM reading, addresses, streets, places) lives in
`kasseigo/core/src/build.rs` and `kasseigo/backend/packs/`: change it there, not here.

## Common tasks

- Publish packs now (after a builder fix pushed to `kasseigo`): ask the owner first (the release is
  public and every user downloads it), then
  `gh workflow run packs.yml -R clementmtz/kasseigo-packs` and watch it with `gh run watch`.
- A bad pack that passed the checks: phones refuse older lists and packs, so there is no going back
  to an old file. Fix or revert the builder on `main` of `kasseigo`, ask the owner, then run the
  workflow (with `allow_smaller` if the good pack is under 70 % of the bad one): a new pack, newer than
  the bad one, reaches phones at their next launch.
- Check what is published: `gh release view packs -R clementmtz/kasseigo-packs`, then download a pack
  and query it with the engine's command-line tool (`kasseigo search …`, see `kasseigo/CLAUDE.md`).
- Add a city: its bounding box and Geofabrik extract in `kasseigo/backend/packs/build.sh`, the city in
  `kasseigo/app/lib/src/cities.dart`, `CITIES` in `kasseigo/backend/feedback/worker.js` and the form of
  `kasseigo-site` (`contracts_test.py` checks all four). Nothing here: the workflow reads the list from `build.sh --list`.

Pack data © OpenStreetMap contributors, ODbL (see `README.md`).
