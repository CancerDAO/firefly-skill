# Firefly Shared Templates

整个 firefly 家族使用的共享数据结构与模板。所有读写 `profile.json` 的 companion 必须遵循本 schema。

---

## profile.json — 患者档案总线

跨 companion 的数据总线。每个患者一个 `profile.json`，所有读写操作遵循下列 schema。

### 字段定义

```json
{
  "patient_id": "<anonymized_code>",
  "created_at": "YYYY-MM-DD",
  "updated_at": "YYYY-MM-DD",
  "demographics": {
    "age": null,
    "sex": null,
    "ethnicity": null,
    "proband_role": "self | parent | spouse | sibling | other"
  },
  "clinical": {
    "chief_complaint": "",
    "onset_age": null,
    "progression": "static | progressive | episodic | remitting-relapsing",
    "hpo_terms": [
      {"hpo_id": "HP:0001250", "label_zh": "癫痫", "label_en": "Seizure", "onset": "infancy"}
    ],
    "physical_findings": [],
    "developmental_milestones": {}
  },
  "family_history": {
    "consanguinity": false,
    "affected_relatives": [],
    "pedigree_notes": ""
  },
  "diagnostics_done": [
    {"test": "WES", "date": "YYYY-MM", "result_summary": "", "file": "diagnostics/wes_YYYY-MM.pdf"}
  ],
  "genetic_findings": [
    {
      "gene": "DMD",
      "variant": "c.xxx",
      "hgvs_p": "p.xxx",
      "zygosity": "hemizygous",
      "inheritance": "XL",
      "acmg_class": "pathogenic | likely_pathogenic | VUS | likely_benign | benign",
      "evidence_codes": ["PVS1", "PM2"]
    }
  ],
  "diagnosis_directions": [
    {
      "disease": "Duchenne muscular dystrophy",
      "orpha_id": "ORPHA:98896",
      "omim_id": "310200",
      "confidence": "high | medium | low",
      "supporting_hpo": [],
      "evidence_refs": []
    }
  ],
  "confirmed_diagnosis": null,
  "treatments_tried": [],
  "current_treatments": [],
  "caregivers": [],
  "mental_health": {
    "phq9": null,
    "gad7": null,
    "iesr": null,
    "last_screened": null
  },
  "sharing_settings": {"level": "private | authorized | anonymized | public"},
  "access_log": []
}
```

### 读写矩阵

| companion | reads | writes |
|---|---|---|
| firefly-organize | — (创建) | demographics / clinical / diagnostics_done / diagnosis_directions |
| firefly-genetic-counseling | clinical / family_history / genetic_findings | genetic_findings / family_history |
| firefly-education | 全部只读 | — |
| firefly-caregiver | demographics / clinical | caregivers |
| firefly-mind | demographics | mental_health |
| firefly-diet | confirmed_diagnosis / current_treatments | — |
| firefly-second-opinion | 全部只读 | — |
| firefly-vault | 全部 | sharing_settings / access_log |
| firefly-disclosure | family_history / genetic_findings | — |
| firefly-patient-org | confirmed_diagnosis / demographics | — |

### 一致性规则

1. schema 变更必须同步更新本文件 + 所有 companion 的 SKILL.md 读写声明
2. 任何 companion 写之前先检查 `updated_at`，避免并发覆盖
3. HPO 术语必须用 `HP:xxxxxxx` 标准 ID，不允许自由文本
4. 基因名用 HGNC 官方符号，变异用 HGVS 命名法
5. 日期统一 ISO 8601 (`YYYY-MM-DD`)

---

## 患者档案卡（Patient Profile Card）

人类可读摘要，由 `firefly-organize` 在完成 OCR/HPO 结构化后生成。

```markdown
# 患者档案卡 · <patient_id>

**更新时间**：YYYY-MM-DD

## 基本信息
- 年龄 / 性别 / 民族
- 先证者关系：<self/parent/spouse/sibling/other>

## 主诉与病程
- 主要症状：
- 起病年龄：
- 进展模式：<static/progressive/episodic/remitting-relapsing>

## HPO 表型清单
| HPO ID | 中文 | 英文 | 起病时间 |
|---|---|---|---|
| HP:XXXXXX | ... | ... | ... |

## 家族史
- 近亲婚配：是/否
- 受累亲属：

## 已做检查
| 日期 | 检查 | 结果摘要 | 机构 |

## 诊断方向（Top 5）
1. 疾病名（ORPHA:xxx / OMIM:xxx）| 置信度 | 支持 HPO

## 下一步建议
- 🔴 优先：
- 🟡 建议：
- 🟢 可选：
```

---

## 就诊摘要（带给下一位医生）

患者可打印带去就诊，帮助新医生快速了解既往诊疗路径。

```markdown
# 就诊摘要 · <patient_id> · 截至 YYYY-MM-DD

## 主要问题
[1-2 句话描述核心症状]

## 已排除方向
- 疾病 A（依据：XXX 检查阴性）
- 疾病 B（依据：XXX 不符）

## 当前诊断方向（Top 3）
1. 疾病 X（匹配度、核心理由）
2. 疾病 Y
3. 疾病 Z

## 已做检查清单
| 日期 | 机构 | 检查 | 结果 |

## 希望医生帮助确认
- [ ] 是否需要做 XXX 检查？
- [ ] XXX 方向是否还值得排查？
- [ ] 是否有未尝试的经验性治疗？
```

---

## 诊断方向报告模板

`firefly-organize` 在多源检索 + LLM 共识排序后输出。每个方向包含：

```markdown
## 方向 N: <疾病中文名> · <Disease English Name>

- **ORPHA / OMIM / GARD**：ORPHA:xxx / OMIM:xxx / GARD:xxx
- **发病率**：1/XXXXX
- **遗传模式**：AD / AR / XL / mt / sporadic
- **置信度**：🟢高 / 🟡中 / 🔴低

### 匹配理由
- 患者 HP:XXXXXX（中文）↔ 该病典型表现（引用 [1]）
- ...

### 不吻合点
- 患者缺少 HP:YYYYYY（该病常见于 XX% 病例）
- ...

### 建议确认检查
- 🔴 优先：
- 🟡 建议：
- 🟢 可选：

### 信息来源
[1] Orphanet rare disease entry ORPHA:xxx
[2] PubMed PMID:xxxxxx
[3] OMIM xxxxxx
```

**关键约束**：永远用"建议排查 X 方向"，永远不说"你可能得了 X 病"。
