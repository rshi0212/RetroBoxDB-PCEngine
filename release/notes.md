PCEngine Catalog, storage v4 (128 KiB blocks, 2 solid LZMA2 groups of up to 256 MiB). Metadata only: **no ROM payloads are published**; `compression_groups`, `chunks` and `object_chunks` are empty.

- First release of this platform.
- The RetroAchievements TurboGrafx-16 folder holds both platforms; it is split by extension (`.pce` here and `.sgx` in SuperGrafx, or the reverse). RA hashes drop a 512-byte copier header when size % 131072 = 512 (rcheevos).
- One schema for all twenty-three platforms: the header tables of the eight new platforms (Game Gear, PC Engine, SuperGrafx, MSX, MSX2, Virtual Boy, Game & Watch, Super A'Can: `pce_hardware`, `msx_hardware`, `vb_hardware`) and `rom_annotations` exist in every Catalog; tables of other platforms and provider tables have no rows.
- RetroAchievements reports look up sibling databases (NES<->FDS, SNES<->Satellaview, WonderSwan<->WonderSwan Color, NeoGeo Pocket<->NeoGeo Pocket Color, PC Engine<->SuperGrafx, MSX<->MSX2): a game whose ROM is stored there is `local_other_platform`, not a gap.
- Source: 825 ZIPs (nointro 651, retroachievements 174), 172.7 MiB (825 ROM files, 334.9 MiB uncompressed). Populated database: 71.7 MiB (41.5% of the ZIPs). All source ZIPs are reproduced byte-for-byte.
- Contents: 545 ROM records, 363 games, 509 releases; DAT versions: 20260124-120557.
- RetroAchievements: 138 of 138 games with achievements have a local ROM.
- Export (Intel(R) Core(TM) i7-8650U CPU @ 1.90GHz, idle, all checks): whole newest-DAT set with export_set.py 39.3 MiB/s (509 files); single file with a cold cache 1.798 s (ROM) / 1.8 s (TorrentZip) on average.
- Full audit of the populated database: 548 objects, 2 groups, 684 archive plans, no errors.

The release workflow starts from the base Catalog pinned by SHA256 in `release/catalog-release.json`, injects the engine and documents of the tagged commit, checks every data-table digest, SQLite integrity and foreign keys, runs the Catalog audit and the repository tests. Verify the download with `SHA256SUMS`.

[中文说明](https://github.com/rshi0212/RetroBoxDB-PCEngine/blob/main/README.zh-CN.md)
