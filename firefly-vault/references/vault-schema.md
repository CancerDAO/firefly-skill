# Patient Vault Schema Reference

Machine-parseable schema for the N=1 patient data vault built by `firefly-vault`. All paths are relative to the vault root.

## 1. Directory structure

```
patient-vault/
├── profile.json                         # firefly-organize 产出的结构化档案
├── timeline.md                          # 人类可读治疗时间线
├── diagnostics/
│   ├── imaging/                         # YYYY-MM_modality_anatomy.dicom|png|pdf
│   ├── labs/                            # YYYY-MM_labtype.pdf
│   ├── pathology/                       # YYYY-MM_organ_biopsy.pdf
│   └── genetics/
│       ├── reports/                     # WES/panel 正式报告 PDF
│       └── raw/                         # VCF/FASTQ (separately warning-gated)
├── treatments/
│   └── YYYY-MM-DD_regimen-name/         # 一次治疗一个目录
│       ├── protocol.md                  # 方案、剂量、周期
│       ├── response.md                  # 影像学/标志物响应
│       └── side-effects.md              # CTCAE 分级不良反应
├── monitoring/
│   ├── YYYY-MM_panel.csv                # 定期复查面板快照
│   └── trend_<metric>.json              # 单指标长程趋势 (CEA, CA-125, …)
├── notes/
│   └── YYYY-MM-DD_visit.md              # 门诊/咨询记录
├── sharing-settings.json                # 4 级共享策略（本文件 §3）
└── access-log.ndjson                    # 审计日志（本文件 §4）
```

Per-directory intent:

| Path | Role | Write policy |
|---|---|---|
| `profile.json` | 规范化患者档案（唯一真源） | `firefly-organize` 独占写 |
| `timeline.md` | 从 profile.json 渲染的叙事视图 | 重新生成，不手改 |
| `diagnostics/` | 原始诊断证据（不可变） | append-only，拒绝原地改写 |
| `diagnostics/genetics/raw/` | 基因原始序列数据 | 额外 warning gate，默认只读挂载 |
| `treatments/` | 每次治疗自成子目录 | append per-event，允许补记但不删 |
| `monitoring/` | 时序监测数据（结构化） | 追加行 / 追加版本 JSON |
| `notes/` | 自由文本门诊笔记 | 日期前缀一次一文件 |
| `sharing-settings.json` | 4 级共享策略 | 改动需二次确认 |
| `access-log.ndjson` | 访问审计（append-only） | never truncate, never rewrite |

## 2. File naming conventions

| 规则 | 形式 | 示例 |
|---|---|---|
| 日期前缀 | `YYYY-MM-DD_` 或 `YYYY-MM_` | `2025-08-15_visit.md` |
| snake_case | 全小写 + 下划线 | `chest_ct_contrast` |
| 版本后缀 | `_v2`, `_v3` 递增（旧版保留） | `wes_report_v2.pdf` |
| 保密标记 | 文件名含 `⚠️` 前缀代表 warning-gated | `⚠️proband_trio.vcf` |
| 校验和后缀 | 可选 `.sha256` 并列文件 | `wes_report.pdf.sha256` |
| ASCII-safe | 禁用空格 / 中文 / 冒号 (跨平台兼容) | 中文信息放文件内部 |

Grammar (EBNF-lite):

```
filename   := date_prefix "_" body ("_v" digit+)? "." ext
date_prefix := year "-" month ("-" day)?
body       := [a-z0-9_]+
```

## 3. `sharing-settings.json` schema

```json
{
  "default_level": "private",
  "overrides": {
    "diagnostics/genetics/raw/**": {
      "level": "private",
      "warn_on_change": true,
      "require_second_confirm": true
    },
    "diagnostics/imaging/**": { "level": "private" }
  },
  "authorized_parties": [
    {
      "id": "dr_wang_pumch",
      "name_hash": "sha256:...",
      "role": "attending_physician",
      "allowed_paths": ["profile.json", "diagnostics/**", "treatments/**"],
      "expires_at": "2027-04-24",
      "contact": "encrypted:..."
    }
  ],
  "anonymization_ruleset": {
    "strip_fields": ["name", "exact_birth_date", "address", "id_number", "phone"],
    "generalize": {
      "birth_date": "year_only",
      "location": "province"
    },
    "k_anonymity_check": {
      "enabled": true,
      "min_k": 5,
      "quasi_identifiers": ["age", "sex", "ethnicity", "diagnosis"]
    }
  }
}
```

Level enum: `"private" | "authorized" | "anonymized_for_ai" | "public"`. Path matching uses glob semantics (`**` = recursive). Longest-prefix override wins; if multiple overrides match, the most restrictive `level` is applied.

## 4. `access-log.ndjson` format

One JSON object per line (newline-delimited JSON). Append-only; rotation by month to `access-log-YYYY-MM.ndjson.gz` but the live file is always `access-log.ndjson`.

```json
{"ts": "2026-04-24T10:30:00Z", "actor": "self", "action": "read", "path": "profile.json", "granted": true}
{"ts": "2026-04-24T11:00:00Z", "actor": "dr_wang_pumch", "action": "read", "path": "diagnostics/genetics/reports/wes_2025-08.pdf", "granted": true, "auth_method": "token"}
{"ts": "2026-04-24T11:05:00Z", "actor": "unknown", "action": "read", "path": "diagnostics/genetics/raw/proband.vcf", "granted": false, "reason": "warning_gate_triggered"}
```

Required fields: `ts` (RFC3339 UTC), `actor`, `action` ∈ {`read`, `write`, `export`, `share`, `revoke`}, `path`, `granted` (bool). Optional: `auth_method`, `reason`, `bytes`, `client_ip_hash`.

## 5. Import conventions

**From `firefly-organize` → `profile.json`**
- Hand-off is a file-level replacement of `profile.json`, but mtime of the source file is preserved via `cp --preserve=timestamps` (macOS: `cp -p`).
- Every replacement bumps `profile.json` key `version` (semver minor) and writes a `profile.json.bak-YYYY-MM-DD` snapshot alongside. Snapshots older than 12 months are compressed.
- Diff summary of changed top-level keys is appended to `timeline.md` under a `## Vault sync YYYY-MM-DD` heading.

**Ingesting diagnostic PDFs/images**
- Incoming file is moved (not copied) into the matching `diagnostics/<subdir>/` with name normalized to `YYYY-MM_<type>_<slug>.ext`.
- A SHA-256 is computed and appended to `diagnostics/<subdir>/CHECKSUMS.txt` (`<sha256>  <filename>` format, `sha256sum -c` compatible).
- If the incoming filename had Chinese/space characters, original name is preserved inside the vault's `CHECKSUMS.txt` as a trailing `# original: <name>` comment.
- Duplicate SHA-256 triggers dedup: the file is dropped and the existing path is logged to `access-log.ndjson` with `action: "dedup"`.

## 6. Backup / export

Monthly incremental archive:

```
backup_2026-04_<sha256[:12]>.tar.gz
```

- Contents: diff of `profile.json` vs previous month (unified diff) + any new files under `diagnostics/`, `treatments/`, `monitoring/`, `notes/` since last backup.
- Excludes: `diagnostics/genetics/raw/` (always excluded by default — exported separately with an explicit `--include-raw` flag and a warning gate).
- Naming: `<sha256[:12]>` is the short hash of the archive's own manifest (not the tarball itself) so users can verify integrity before decompression.
- Index file `backups/INDEX.ndjson` tracks `{archive, ts, bytes, manifest_sha256, covers_from, covers_to}` per archive.
- Full (non-incremental) export uses `full_YYYY-MM-DD_<sha>.tar.gz` and rebases the incremental chain.

Restore order: most recent `full_*` tarball, then all `backup_*` incrementals in chronological order.
