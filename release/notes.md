NGPC Catalog, storage v4 (128 KiB blocks, 2 solid LZMA2 groups of up to 256 MiB). Metadata only: **no ROM payloads are published**; `compression_groups`, `chunks` and `object_chunks` are empty.

- One schema for all twenty-three platforms: the header tables of the eight new platforms (Game Gear, PC Engine, SuperGrafx, MSX, MSX2, Virtual Boy, Game & Watch, Super A'Can: `pce_hardware`, `msx_hardware`, `vb_hardware`) and `rom_annotations` exist in every Catalog; tables of other platforms and provider tables have no rows.
- RetroAchievements reports look up sibling databases (NES<->FDS, SNES<->Satellaview, WonderSwan<->WonderSwan Color, NeoGeo Pocket<->NeoGeo Pocket Color, PC Engine<->SuperGrafx, MSX<->MSX2): a game whose ROM is stored there is `local_other_platform`, not a gap.
- Source: 183 ZIPs (nointro 130, retroachievements 53), 93.0 MiB (183 ROM files, 261.8 MiB uncompressed). Populated database: 38.2 MiB (41.1% of the ZIPs). All source ZIPs are reproduced byte-for-byte.
- Contents: 160 ROM records, 76 games, 128 releases; DAT versions: 20240506-123728, 20260626-085623, 20260919-122044.
- RetroAchievements: 41 of 41 games with achievements have a local ROM.
- Export (Intel(R) Core(TM) i7-8650U CPU @ 1.90GHz, idle, all checks): whole newest-DAT set with export_set.py 52.5 MiB/s (128 files); single file with a cold cache 0.943 s (ROM) / 1.184 s (TorrentZip) on average.
- Full audit of the populated database: 165 objects, 2 groups, 171 archive plans, no errors.

The release workflow starts from the base Catalog pinned by SHA256 in `release/catalog-release.json`, injects the engine and documents of the tagged commit, checks every data-table digest, SQLite integrity and foreign keys, runs the Catalog audit and the repository tests. Verify the download with `SHA256SUMS`.

[中文说明](https://github.com/rshi0212/RetroBoxDB-NGPC/blob/main/README.zh-CN.md)
