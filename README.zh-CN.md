# RetroBoxDB NGPC

[English](README.md) | 中文

SNK NeoGeo Pocket Color的单文件 SQLite 保存库。公开的 Catalog 只含元数据（校验值、DAT 与来源记录、头部字段、打包配方和程序），不含 ROM 数据，不能独立恢复文件；完整库保留在本地。

| 项目 | 数值 |
| --- | --- |
| 原始大小 | 源 ZIP 183 个，93.0 MiB（No-Intro 130 个，RetroAchievements 集合 53 个）；解压后 ROM 183 个，261.8 MiB |
| 入库后大小 | 完整库 37.9 MiB；公开 Catalog 3.5 MiB（不含 ROM 数据） |
| 比例 | 完整库为原 ZIP 的 40.8%，为解压后 ROM 总量的 14.5% |
| 使用的技术 | 存储 v4：128 KiB 块按 SHA256 去重，按 No-Intro 游戏族顺序装入最大 256 MiB 的 LZMA2 实体组（字典 256 MiB）；逐块 SHA256、逐对象 CRC32／MD5／SHA1／SHA256 校验；源 ZIP 由 TorrentZip 配方逐字节重建 |
| 导出性能 | Intel(R) Core(TM) i7-8650U CPU @ 1.90GHz，空闲负载，Python 3.14.4，含全部校验。按最新 DAT 整套导出（`export_set.py`，128 个文件，逐个按 DAT 哈希校验）：52.5 MiB/s，平均 27 毫秒／个；单个文件冷缓存（每次清空缓存，需解压所在组的前段）：ROM 平均 0.943 秒，TorrentZip 平均 1.184 秒 |

## 下载与说明

| 文件／文档 | 内容 |
| --- | --- |
| [RetroBoxDB.NGPC.Catalog.sqlite](https://github.com/rshi0212/RetroBoxDB-NGPC/releases/latest/download/RetroBoxDB.NGPC.Catalog.sqlite) | 公开 Catalog（Release 附件，附 `SHA256SUMS`） |
| [存储 v4 说明](RetroBoxDB.Storage-v4.zh-CN.md)／[English](RetroBoxDB.Storage-v4.en.md) | 各平台的存储评估、内容、RA、中文名与维护 |
| [Technical design](RetroBoxDB.Storage-v4.Technical-Design.en.md) | 存储格式、平台适配、增量更新、校验 |
| [RA 清单](reports/ra-ngpc-games.csv)／[汇总](reports/ra-ngpc.json)、[构建报告](reports/ngpc-build-report.json)、[审计处理](reports/audit-resolution-20261004.md) | 逐项数据 |

## 本平台的存储选择与特殊情况

全部本地收藏实测 18 种块／组组合（`assessment/data/storage-experiment-ngpc.json`）：最小为 256 KiB / 256 MiB 32.94 MiB；按规则（最小值 0.5% 以内选块最小、再选组最小）采用 128 KiB / 256 MiB 33.02 MiB。ZIP 92.97 MiB，逐文件 LZMA 64.83 MiB。

- 头部：开头 64 字节（`COPYRIGHT BY SNK CORPORATION` 或 ` LICENSED BY SNK CORPORATION`、启动地址、软件 ID、子版本、彩色模式、标题）存入 `ngp_hardware`；BIOS 没有卡带头，记为 `unclassified`。
- RetroAchievements 把 NeoGeo Pocket 和 NeoGeo Pocket Color 放在同一个主机（14）和同一个目录下。两个库都导入该目录：本库收 `.ngc` 文件，`.ngp` 文件跳过，由 [RetroBoxDB-NGP](https://github.com/rshi0212/RetroBoxDB-NGP) 收录。RA 报告只统计与本库有关的游戏。

## 内容

| 项目 | 数值 |
| --- | --- |
| ROM 记录／游戏组／发行版本 | 160／76／128 |
| 各版 DAT 覆盖 | 20240506-123728：128/128；20260626-085623：128/128；20260919-122044：128/128 |
| 不在任何 DAT 的本地 ROM | 32 |
| RetroAchievements 集合中的 ROM 文件 | DAT 中有 22，仅 RA 收录 30，哈希不在最新 RA 快照 1（[清单](reports/ra-ngpc-collection-unknown.csv)）；仍缺本地 ROM 的 RA 游戏见 [缺口清单](reports/ra-ngpc-missing.csv) |
| No-Intro DB Export＋Dump Log 20260919-122044 | 128 个档案、215 个文件身份、215 条有文档的硬件声明；Dump Log Verified 81 |
| RetroAchievements（console 14） | 有成就的游戏 41 个：本地有 ROM 41（54 个 ROM），ROM 在兄弟库中 0，仅 DAT 有 0，仅 DB 文件 0，无 No-Intro 对应 0 |
| 中文名 | 132 条记录中 132 条有中文（83 个唯一名）；本地 ROM 128 个有中文名 |
| 完整库审计 | 165 个对象、2 个组、171 个 ZIP 配方，全部通过 |

源 ZIP 均可由 TorrentZip 配方逐字节重建（`v_file_checksums.exported_bytes_equal_source`）。

## 使用

```bash
# 用 Catalog 内嵌引擎做只读审计（stats、checksums FILE_ID、help 同理）
python3 -B -c 'import sqlite3,sys; c=sqlite3.connect(sys.argv[1]); s=c.execute("SELECT content FROM resources WHERE name=?",("engine.py",)).fetchone()[0]; c.close(); exec(compile(s,"RetroBoxDB:engine.py","exec"))' ./RetroBoxDB.NGPC.Catalog.sqlite audit
# 完整库：按 DAT 版本、1G1R、RA 成就、TorrentZip／裸 ROM 组合导出
python3 -B tools/export_set.py RetroBoxDB.NGPC.sqlite OUT --set 1g1r --ra achievements --container torrentzip --layout ra-category
# 增量加入新 DAT、DB Export／Dump Log、ROM 与 RA 快照
python3 -B tools/update_db.py RetroBoxDB.NGPC.sqlite --discover --ra --catalog RetroBoxDB.NGPC.Catalog.sqlite
```

只需 Python 3.10+ 标准库。`resources` 中的 `engine.py` 等是可执行代码，只应从自己构建或 SHA256 已核对的 Release 附件中执行。发布由 `.github/workflows/publish-catalog.yml` 完成：工作流从 `release/catalog-release.json` 固定的基础 Catalog 出发，注入本仓库提交中的引擎与文档，核对全部数据表摘要、运行测试与审计后发布。
