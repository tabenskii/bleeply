# Local storage and cache recovery

The repository's `.cache/` contains only disposable downloads, build products and
temporary intermediates. It can be removed in full while the development app,
workers, downloads and builds are stopped. The next setup/build may need network
access and take longer; model downloads are required before offline recognition
works again. Removing a symlink removes the link, not its external target.

## Persistent local files

`local-artifacts/` is ignored by Git and must be backed up separately. It holds:

- `release-*`, `testflight-*`, `appstore-*`, `app-review-*` and `rebrand/`:
  release/submission responses, notarization records, screenshots, release notes,
  audit evidence and historical artifacts. Signed releases cannot be recreated
  with the same signatures and submission history by rebuilding source.
- `diagnostics/`, `icon-diagnosis/` and `macos-previews/`: diagnostic recordings,
  manual reviews, recognition snapshots and visual comparison evidence.
- `speedtest/`: raw measurements from terminal benchmark runs. The double-click
  launcher keeps raw measurements under `raw/` inside its selected report folder.
- Historical logs, one-off scripts and other retained working materials.

The local `cache-relocation-2026-10-19.json` inventory records old and new paths,
file checksums and symlink targets for the cache migration. Historical reports
retain their original contents, including old paths; use the inventory to locate
the corresponding files. Relocated helper scripts have their moved input/output
paths updated; their original bytes remain in `migration-originals/`. New Swift
visual checks write to `macos-previews/current/`, preserving the migrated
snapshots. The relocated scripts are historical tools, not commands to rerun
a publication.

Keep original recordings, selected cue sounds, transcripts, saved reviews and
credentials outside `.cache/`. Existing project-root media and
`тесты bleeply/` remain in place. The development Python environments are in
`.venv/` and `.runtime/`. Installed app data lives in Application Support or the
app's sandbox container and is separate from the repository cache.

## Recreate a deleted cache

Run commands from the repository root, with the prerequisites in the project
README installed. These recreate the cache directories they need:

| Cache content | Recovery |
| --- | --- |
| `clang/`, `swift-build/`, `whisperkit-build/` | `bash scripts/build-macos.sh` |
| `huggingface/`, `gigaam/` | Allow model downloads in the app and select the model again |
| `whisperkit/model/` | `.venv/bin/python -m pip install -r requirements-whisperkit.txt`, then `.venv/bin/python scripts/prepare-whisperkit.py` |
| `speedtest/` environment and model fingerprints | `make speedtest-setup` |
| `speaker-models/` | Repeat the model downloads in [CLI setup](cli.md) |
| `release-ffmpeg/`, `ffmpeg-8.1.2*` | `bash scripts/release/build-ffmpeg.sh` |
| `pyav-lgpl/` | `bash scripts/release/build-pyav.sh` |
| `app-store-runtime/` | `python3 scripts/release/app_store_runtime.py fetch` |
| `release-python/`, `dmg-tools/`, `uv/` | Repeat the runtime/tool setup in [release prerequisites](../scripts/release/README.md#prerequisites) |

Release packaging requires its documented preparation steps after a cache reset;
running the bundler alone does not download or rebuild missing prerequisites.
Store release logs and submission responses in `local-artifacts/release-VERSION/`.
Keep final signed apps, DMGs, PKGs and source archives in the release directories
under `build/` and `dist/`, or in a persistent release archive, never in `.cache/`.

New code must create missing cache directories. Use temporary directories for
scratch tests, and persistent artifact directories for measurements, manual
decisions and evidence that cannot be recovered by rerunning a build.
