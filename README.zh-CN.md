# RetroBoxDB PCEngine

[English](README.md) | 中文

NEC PC Engine／TurboGrafx-16的单文件 SQLite 保存库。公开的 Catalog 只含元数据（校验值、DAT 与来源记录、头部字段、打包配方和程序），不含 ROM 数据，不能独立恢复文件；完整库保留在本地。

| 项目 | 数值 |
| --- | --- |
| 原始大小 | 源 ZIP 825 个，172.7 MiB（No-Intro 651 个，RetroAchievements 集合 174 个）；解压后 ROM 825 个，334.9 MiB |
| 入库后大小 | 完整库 71.7 MiB；公开 Catalog 6.5 MiB（不含 ROM 数据） |
| 比例 | 完整库为原 ZIP 的 41.5%，为解压后 ROM 总量的 21.4% |
| 使用的技术 | 存储 v4：128 KiB 块按 SHA256 去重，按 No-Intro 游戏族顺序装入最大 256 MiB 的 LZMA2 实体组（字典 256 MiB）；逐块 SHA256、逐对象 CRC32／MD5／SHA1／SHA256 校验；源 ZIP 由 TorrentZip 配方逐字节重建 |
| 导出性能 | Intel(R) Core(TM) i7-8650U CPU @ 1.90GHz，空闲负载，Python 3.14.4，含全部校验。按最新 DAT 整套导出（`export_set.py`，509 个文件，逐个按 DAT 哈希校验）：39.3 MiB/s，平均 10 毫秒／个；单个文件冷缓存（每次清空缓存，需解压所在组的前段）：ROM 平均 1.798 秒，TorrentZip 平均 1.8 秒 |

## 下载与说明

| 文件／文档 | 内容 |
| --- | --- |
| [RetroBoxDB.PCEngine.Catalog.sqlite](https://github.com/rshi0212/RetroBoxDB-PCEngine/releases/latest/download/RetroBoxDB.PCEngine.Catalog.sqlite) | 公开 Catalog（Release 附件，附 `SHA256SUMS`） |
| [存储 v4 说明](RetroBoxDB.Storage-v4.zh-CN.md)／[English](RetroBoxDB.Storage-v4.en.md) | 各平台的存储评估、内容、RA、中文名与维护 |
| [Technical design](RetroBoxDB.Storage-v4.Technical-Design.en.md) | 存储格式、平台适配、增量更新、校验 |
| [RA 清单](reports/ra-pcengine-games.csv)／[汇总](reports/ra-pcengine.json)、[构建报告](reports/pcengine-build-report.json)、[审计处理](reports/audit-resolution-20261004.md) | 逐项数据 |

## 本平台的存储选择与特殊情况

全部本地收藏实测 26 种块／组组合（`assessment/data/storage-experiment-pcengine.json`）：最小为 1024 KiB / 256 MiB 62.52 MiB；按规则（最小值 0.5% 以内选块最小、再选组最小）采用 128 KiB / 256 MiB 62.67 MiB。ZIP 172.69 MiB，逐文件 LZMA 100.74 MiB。

- HuCard 没有内部头部。`pce_hardware` 只记录镜像事实：512 字节 copier 头（文件大小 %8192 = 512；单独切块，使正文与无头版本去重）、容量、8 KiB bank 数、不足一个 bank 的字节数，以及第一个 bank 末尾的复位向量。mapper 和地区需要外部证据，不推测。
- RetroAchievements 把 PC Engine 和 SuperGrafx 放在同一个主机（8）和同一个目录（TurboGrafx-16）下。两个库都导入该目录：本库收 `.pce` 文件，`.sgx` 文件跳过，由 [RetroBoxDB-SuperGrafx](https://github.com/rshi0212/RetroBoxDB-SuperGrafx) 收录。RA 哈希在大小 %131072 = 512 时先去掉 512 字节 copier 头（rcheevos 规则）。RA 报告只统计与本库有关的游戏。

## 内容

| 项目 | 数值 |
| --- | --- |
| ROM 记录／游戏组／发行版本 | 545／363／509 |
| 各版 DAT 覆盖 | 20260124-120557：509/509 |
| 不在任何 DAT 的本地 ROM | 36 |
| RetroAchievements 集合中的 ROM 文件 | DAT 中有 141，仅 RA 收录 32，哈希不在最新 RA 快照 1（[清单](reports/ra-pcengine-collection-unknown.csv)）；仍缺本地 ROM 的 RA 游戏见 [缺口清单](reports/ra-pcengine-missing.csv) |
| No-Intro DB Export＋Dump Log 20260124-120557 | 510 个档案、714 个文件身份、208 条有文档的硬件声明；Dump Log Verified 292 |
| RetroAchievements（console 8） | 有成就的游戏 138 个：本地有 ROM 138（180 个 ROM），ROM 在兄弟库中 0，仅 DAT 有 0，仅 DB 文件 0，无 No-Intro 对应 0 |
| 中文名 | 468 条记录中 468 条有中文（339 个唯一名）；本地 ROM 460 个有中文名 |
| 完整库审计 | 548 个对象、2 个组、684 个 ZIP 配方，全部通过 |

源 ZIP 均可由 TorrentZip 配方逐字节重建（`v_file_checksums.exported_bytes_equal_source`）。

## 使用

```bash
# 用 Catalog 内嵌引擎做只读审计（stats、checksums FILE_ID、help 同理）
python3 -B -c 'import sqlite3,sys; c=sqlite3.connect(sys.argv[1]); s=c.execute("SELECT content FROM resources WHERE name=?",("engine.py",)).fetchone()[0]; c.close(); exec(compile(s,"RetroBoxDB:engine.py","exec"))' ./RetroBoxDB.PCEngine.Catalog.sqlite audit
# 完整库：按 DAT 版本、1G1R、RA 成就、TorrentZip／裸 ROM 组合导出
python3 -B tools/export_set.py RetroBoxDB.PCEngine.sqlite OUT --set 1g1r --ra achievements --container torrentzip --layout ra-category
# 增量加入新 DAT、DB Export／Dump Log、ROM 与 RA 快照
python3 -B tools/update_db.py RetroBoxDB.PCEngine.sqlite --discover --ra --catalog RetroBoxDB.PCEngine.Catalog.sqlite
```

只需 Python 3.10+ 标准库。`resources` 中的 `engine.py` 等是可执行代码，只应从自己构建或 SHA256 已核对的 Release 附件中执行。发布由 `.github/workflows/publish-catalog.yml` 完成：工作流从 `release/catalog-release.json` 固定的基础 Catalog 出发，注入本仓库提交中的引擎与文档，核对全部数据表摘要、运行测试与审计后发布。
