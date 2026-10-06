# RetroBoxDB NGPC

English | [中文说明](README.zh-CN.md)

Single-file SQLite preservation database for SNK NeoGeo Pocket Color. The public Catalog holds metadata only (checksums, DAT and provenance records, header fields, archive recipes and the processing code); it contains no ROM data and cannot restore files. The populated database stays local.

| Item | Value |
| --- | --- |
| Original size | 183 source ZIPs, 93.0 MiB (No-Intro 130, RetroAchievements sets 53); 183 ROM files, 261.8 MiB uncompressed |
| Stored size | populated database 38.2 MiB; public Catalog 3.7 MiB (no ROM data) |
| Ratio | 41.1% of the source ZIPs, 14.6% of the uncompressed ROM files |
| Technology | storage v4: SHA256-deduplicated 128 KiB blocks packed in No-Intro family order into solid LZMA2 groups of up to 256 MiB (256 MiB dictionary); per-block SHA256 and per-object CRC32/MD5/SHA1/SHA256 verification; source ZIPs reproduced byte-for-byte from TorrentZip plans |
| Export performance | Intel(R) Core(TM) i7-8650U CPU @ 1.90GHz, idle, Python 3.14.4, all checks included. whole newest-DAT set with `export_set.py` (128 files, each checked against the DAT hashes): 52.5 MiB/s, 27 ms per file on average; single file with a cold cache (the group is decoded up to the file): ROM 0.943 s, TorrentZip 1.184 s on average |

## Downloads and documents

| File / document | Content |
| --- | --- |
| [RetroBoxDB.NGPC.Catalog.sqlite](https://github.com/rshi0212/RetroBoxDB-NGPC/releases/latest/download/RetroBoxDB.NGPC.Catalog.sqlite) | Public Catalog (Release asset with `SHA256SUMS`) |
| [Storage v4 guide](RetroBoxDB.Storage-v4.en.md) / [中文](RetroBoxDB.Storage-v4.zh-CN.md) | Storage evaluation, contents, RA, names and maintenance for every platform |
| [Technical design](RetroBoxDB.Storage-v4.Technical-Design.en.md) | Storage format, platform adapters, incremental updates, verification |
| [RA list](reports/ra-ngpc-games.csv) / [summary](reports/ra-ngpc.json), [build report](reports/ngpc-build-report.json), [audit resolution](reports/audit-resolution-20261004.md) | Detailed data |

## Storage choice and platform specifics

18 block/group combinations measured on the whole local collection (`assessment/data/storage-experiment-ngpc.json`): smallest 256 KiB / 256 MiB at 32.94 MiB; by the rule (within 0.5% of the smallest, the smallest block, then the smallest group) 128 KiB / 256 MiB at 33.02 MiB. ZIPs 92.97 MiB, per-file LZMA 64.83 MiB.

- Header: 64 bytes at 0 (`COPYRIGHT BY SNK CORPORATION` or ` LICENSED BY SNK CORPORATION`, start address, software id, sub code, colour mode, title) stored in `ngp_hardware`; BIOS images have no header and stay `unclassified`.
- RetroAchievements lists NeoGeo Pocket and NeoGeo Pocket Color under one console (14) and one folder. Both databases import that folder; this one keeps `.ngc` files and skips `.ngp` files, which [RetroBoxDB-NGP](https://github.com/rshi0212/RetroBoxDB-NGP) holds. The RA report covers only games tied to this database.

## Contents

| Item | Value |
| --- | --- |
| ROM records / games / releases | 160 / 76 / 128 |
| DAT coverage per version | 20240506-123728: 128/128; 20260626-085623: 128/128; 20260919-122044: 128/128 |
| Local ROMs in no DAT | 32 |
| ROM files of the RetroAchievements set | in a No-Intro DAT 22, RA only 30, hash not in the latest RA snapshot 1 ([list](reports/ra-ngpc-collection-unknown.csv)); RA games still without a local ROM: [gap list](reports/ra-ngpc-missing.csv) |
| No-Intro DB Export + Dump Log 20260919-122044 | 128 archives, 215 file identities, 215 documented hardware assertions; Dump Log Verified 81 |
| RetroAchievements (console 14) | 41 games with achievements: 41 with a local ROM (54 ROMs), 0 with the ROM in a sibling database, 0 DAT only, 0 DB file only, 0 without a No-Intro counterpart |
| Chinese names | 132 of 132 rows translated (83 unique); 128 local ROMs have a Chinese name |
| Populated-database audit | 165 objects, 2 groups, 171 archive plans, all passed |

Every source ZIP is reproduced byte-for-byte from its TorrentZip plan (`v_file_checksums.exported_bytes_equal_source`).

## Usage

```bash
# Query-only audit with the Catalog's embedded engine (also: stats, checksums FILE_ID, help)
python3 -B -c 'import sqlite3,sys; c=sqlite3.connect(sys.argv[1]); s=c.execute("SELECT content FROM resources WHERE name=?",("engine.py",)).fetchone()[0]; c.close(); exec(compile(s,"RetroBoxDB:engine.py","exec"))' ./RetroBoxDB.NGPC.Catalog.sqlite audit
# Populated database: export by DAT version, 1G1R, RA achievements, TorrentZip or plain ROMs
python3 -B tools/export_set.py RetroBoxDB.NGPC.sqlite OUT --set 1g1r --ra achievements --container torrentzip --layout ra-category
# Add new DATs, DB Export / Dump Log snapshots, ROMs and an RA snapshot incrementally
python3 -B tools/update_db.py RetroBoxDB.NGPC.sqlite --discover --ra --catalog RetroBoxDB.NGPC.Catalog.sqlite
```

Python 3.10+ standard library only. `engine.py` and the other `resources` entries are executable code; run them only from a database you built or a Release asset whose SHA256 you verified. Releases are produced by `.github/workflows/publish-catalog.yml`: it starts from the base Catalog pinned in `release/catalog-release.json`, injects the engine and documents of this commit, checks every data-table digest, runs the tests and the audit, then publishes.
