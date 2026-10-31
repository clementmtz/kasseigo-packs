# KasseiGo city packs

Data files downloaded by the KasseiGo app (shared bikes and scooters guidance, Brussels and Paris),
published in the release [`packs`](../../releases/tag/packs) and rebuilt every Monday by
[`.github/workflows/packs.yml`](.github/workflows/packs.yml).

| File | What it is |
|---|---|
| `manifest.json` | List of the files below, with their sizes and SHA-256 hashes (checked by the app) |
| `<city>-<hash>.kgpack.gz` | Streets graph, places index and bicycle parkings of the city (the name changes with its content) |

Data © OpenStreetMap contributors, available under the
[Open Database License (ODbL)](https://opendatacommons.org/licenses/odbl/). Extracts from Geofabrik.
