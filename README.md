# RetroBoxDB PCEngine

English | [中文说明](README.zh-CN.md)

Single-file SQLite preservation database for NEC PC Engine / TurboGrafx-16. The public Catalog holds metadata only (checksums, DAT and provenance records, header fields, archive recipes and the processing code); it contains no ROM data and cannot restore files. The populated database stays local.

| Item | Value |
| --- | --- |
| Original size | 825 source ZIPs, 172.7 MiB (No-Intro 651, RetroAchievements sets 174); 825 ROM files, 334.9 MiB uncompressed |
| Stored size | populated database 71.7 MiB; public Catalog 6.5 MiB (no ROM data) |
| Ratio | 41.5% of the source ZIPs, 21.4% of the uncompressed ROM files |
| Technology | storage v4: SHA256-deduplicated 128 KiB blocks packed in No-Intro family order into solid LZMA2 groups of up to 256 MiB (256 MiB dictionary); per-block SHA256 and per-object CRC32/MD5/SHA1/SHA256 verification; source ZIPs reproduced byte-for-byte from TorrentZip plans |
| Export performance | Intel(R) Core(TM) i7-8650U CPU @ 1.90GHz, idle, Python 3.14.4, all checks included. whole newest-DAT set with `export_set.py` (509 files, each checked against the DAT hashes): 39.3 MiB/s, 10 ms per file on average; single file with a cold cache (the group is decoded up to the file): ROM 1.798 s, TorrentZip 1.8 s on average |

## Downloads and documents

| File / document | Content |
| --- | --- |
| [RetroBoxDB.PCEngine.Catalog.sqlite](https://github.com/rshi0212/RetroBoxDB-PCEngine/releases/latest/download/RetroBoxDB.PCEngine.Catalog.sqlite) | Public Catalog (Release asset with `SHA256SUMS`) |
| [Storage v4 guide](RetroBoxDB.Storage-v4.en.md) / [中文](RetroBoxDB.Storage-v4.zh-CN.md) | Storage evaluation, contents, RA, names and maintenance for every platform |
| [Technical design](RetroBoxDB.Storage-v4.Technical-Design.en.md) | Storage format, platform adapters, incremental updates, verification |
| [RA list](reports/ra-pcengine-games.csv) / [summary](reports/ra-pcengine.json), [build report](reports/pcengine-build-report.json), [audit resolution](reports/audit-resolution-20261004.md) | Detailed data |

## Storage choice and platform specifics

26 block/group combinations measured on the whole local collection (`assessment/data/storage-experiment-pcengine.json`): smallest 1024 KiB / 256 MiB at 62.52 MiB; by the rule (within 0.5% of the smallest, the smallest block, then the smallest group) 128 KiB / 256 MiB at 62.67 MiB. ZIPs 172.69 MiB, per-file LZMA 100.74 MiB.

- HuCards have no internal header. `pce_hardware` records image facts only: a 512-byte copier header (file size % 8192 = 512; cut into its own block so the body deduplicates with headerless dumps), ROM size, 8 KiB banks, bytes of a partial bank and the reset vector at the end of the first bank. Mapper and region need external evidence and are not guessed.
- RetroAchievements lists PC Engine and SuperGrafx games under one console (8) and one folder (TurboGrafx-16). Both databases import that folder; this one keeps `.pce` files and skips `.sgx` files, which [RetroBoxDB-SuperGrafx](https://github.com/rshi0212/RetroBoxDB-SuperGrafx) holds. RA hashes drop a 512-byte copier header when the size % 131072 = 512 (rcheevos). The RA report covers only games tied to this database.

## Contents

| Item | Value |
| --- | --- |
| ROM records / games / releases | 545 / 363 / 509 |
| DAT coverage per version | 20260124-120557: 509/509 |
| Local ROMs in no DAT | 36 |
| ROM files of the RetroAchievements set | in a No-Intro DAT 141, RA only 32, hash not in the latest RA snapshot 1 ([list](reports/ra-pcengine-collection-unknown.csv)); RA games still without a local ROM: [gap list](reports/ra-pcengine-missing.csv) |
| No-Intro DB Export + Dump Log 20260124-120557 | 510 archives, 714 file identities, 208 documented hardware assertions; Dump Log Verified 292 |
| RetroAchievements (console 8) | 138 games with achievements: 138 with a local ROM (180 ROMs), 0 with the ROM in a sibling database, 0 DAT only, 0 DB file only, 0 without a No-Intro counterpart |
| Chinese names | 468 of 468 rows translated (339 unique); 460 local ROMs have a Chinese name |
| Populated-database audit | 548 objects, 2 groups, 684 archive plans, all passed |

Every source ZIP is reproduced byte-for-byte from its TorrentZip plan (`v_file_checksums.exported_bytes_equal_source`).

## Usage

```bash
# Query-only audit with the Catalog's embedded engine (also: stats, checksums FILE_ID, help)
python3 -B -c 'import sqlite3,sys; c=sqlite3.connect(sys.argv[1]); s=c.execute("SELECT content FROM resources WHERE name=?",("engine.py",)).fetchone()[0]; c.close(); exec(compile(s,"RetroBoxDB:engine.py","exec"))' ./RetroBoxDB.PCEngine.Catalog.sqlite audit
# Populated database: export by DAT version, 1G1R, RA achievements, TorrentZip or plain ROMs
python3 -B tools/export_set.py RetroBoxDB.PCEngine.sqlite OUT --set 1g1r --ra achievements --container torrentzip --layout ra-category
# Add new DATs, DB Export / Dump Log snapshots, ROMs and an RA snapshot incrementally
python3 -B tools/update_db.py RetroBoxDB.PCEngine.sqlite --discover --ra --catalog RetroBoxDB.PCEngine.Catalog.sqlite
```

Python 3.10+ standard library only. `engine.py` and the other `resources` entries are executable code; run them only from a database you built or a Release asset whose SHA256 you verified. Releases are produced by `.github/workflows/publish-catalog.yml`: it starts from the base Catalog pinned in `release/catalog-release.json`, injects the engine and documents of this commit, checks every data-table digest, runs the tests and the audit, then publishes.
