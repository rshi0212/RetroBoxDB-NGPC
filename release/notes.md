NeoGeo Pocket Color Catalog, storage v4 (128 KiB blocks, 2 solid LZMA2 groups of up to 256 MiB). Metadata only: **no ROM payloads are published**; `compression_groups`, `chunks` and `object_chunks` are empty.

- First release: NeoGeo Pocket Color cartridges from the No-Intro folder and the `.ngc` files of the shared RetroAchievements NeoGeo Pocket set; 3 Parent-Clone DAT versions, DB Export and Dump Log 20260919-122044, RetroAchievements snapshot and Chinese names.
- Storage measured on the whole collection: 128 KiB blocks, 256 MiB groups (`assessment/data/storage-experiment-ngpc.json`).
- RetroAchievements lists both platforms of the pair under one console and one folder; each database keeps its own files and looks up its sibling (NeoGeo Pocket<->NeoGeo Pocket Color).
- Source: 183 ZIPs (nointro 130, retroachievements 53), 93.0 MiB (183 ROM files, 261.8 MiB uncompressed). Populated database: 37.9 MiB (40.8% of the ZIPs). All source ZIPs are reproduced byte-for-byte.
- Contents: 160 ROM records, 76 games, 128 releases; DAT versions: 20240506-123728, 20260626-085623, 20260919-122044.
- RetroAchievements: 41 of 41 games with achievements have a local ROM.
- Chinese names: 132 of 132 CSV rows translated; 128 local ROMs with Chinese names.
- Export (Intel(R) Core(TM) i7-8650U CPU @ 1.90GHz, idle, all checks): whole newest-DAT set with export_set.py 52.5 MiB/s (128 files); single file with a cold cache 0.943 s (ROM) / 1.184 s (TorrentZip) on average.
- Full audit of the populated database: 165 objects, 2 groups, 171 archive plans, no errors.

The release workflow starts from the base Catalog pinned by SHA256 in `release/catalog-release.json`, injects the engine and documents of the tagged commit, checks every data-table digest, SQLite integrity and foreign keys, runs the Catalog audit and the repository tests. Verify the download with `SHA256SUMS`.

[中文说明](https://github.com/rshi0212/RetroBoxDB-NGPC/blob/main/README.zh-CN.md)
