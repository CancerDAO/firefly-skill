# Multi-Source Diagnostic Search Protocol

## 1. 目标

从标准化 HPO 术语组合出发，**并行查询多个权威源**，得到候选疾病排名。对齐 DeepRare (Nature 2026) 多源融合 pipeline，服务罕见病诊断导航。

---

## 2. 数据源清单

| 源 | 类型 | 覆盖 | 访问 | 优先级 |
|---|---|---|---|---|
| **PubCaseFinder** | HPO-based case retrieval | 2.8M+ cases | HTTP API + web | 🔴 must-have |
| **Phenobrain** | AI 表型-疾病预测 | ~8000+ 疾病 | web API / LLM 调用 | 🔴 |
| **OMIM** | 权威孟德尔遗传数据库 | ~16000 phenotypes | HTTP API (需 API key) / web | 🔴 |
| **Orphanet** | 欧洲罕见病官方数据库 | ~6500 rare diseases | HTTP API / web | 🔴 |
| **GARD (NIH)** | 美国罕见病数据库 | ~7000 rare diseases | web + JSON | 🟡 |
| **PubMed** | 文献检索 | 35M+ 论文 | E-utilities API | 🟡 |
| **ClinVar** | 变异致病性 | 2M+ variants | HTTP API | 🟡（含基因数据时）|
| **DECIPHER** | 基因 + 表型 | 罕见 CNV/SNV | 需注册 | 🟢 |
| **HPO annotation** | HPO-disease links | OMIM/Orphanet 注释 | JSON 下载 | 🟡（fallback）|

---

## 3. 并行调度策略

**Wave 1（必跑，最互补）**：
- PubCaseFinder + Orphanet + Phenobrain 三路并行

**Wave 2（补充）**：
- OMIM + PubMed 并行

**Wave 3（仅当有基因数据）**：
- ClinVar + gnomAD + DECIPHER 并行

各 wave 之间可视作独立的并行子任务（按 host agent 的并行机制执行），结果汇总后进入融合层。失败源不阻塞后续。

---

## 4. 查询模板

### PubCaseFinder
- **Input**: HPO ID list
- **Endpoint**: `https://pubcasefinder.dbcls.jp/api/...`
- **Output**: ranked disease list + similarity score
- **注意**: 去除过度常见 HPO（如单独 `HP:0001263` global developmental delay）以减少噪音；建议 ≥5 个 HPO 才调用

### Orphanet
- **Input**: HPO IDs
- **Endpoint**: `https://api.orphadata.com/` (rare disease / phenotype association)
- **Filter**: by prevalence（`<1/1000000` / `1-9/1000000` / `1-9/100000` 等）, inheritance (AD/AR/XL/MT)

### OMIM
- **Input**: HPO IDs 或 clinical key terms
- **Endpoint**: `https://api.omim.org/api/` (需 API key，`OMIM_API_KEY` 环境变量)
- **Output**: MIM number + phenotype title；映射 `OMIM:XXXXXX` → Orphanet ID（通过 mim2gene / HPO annotation 交叉表）

### PubMed
- **Input**: HPO labels + key clinical terms（英文）
- **E-utilities**: `esearch.fcgi` → `efetch.fcgi`
- **筛选**: `case reports[pt] OR review[pt]`，近 10 年优先

### Phenobrain
- **Input**: HPO IDs
- **机制**: LLM-based scoring，返回 disease candidates
- **定位**: 独立第二意见（不与 PubCaseFinder 耦合）

### ClinVar（仅有变异数据时）
- **Input**: gene symbol 或 `chr:pos:ref:alt`
- **Endpoint**: `https://eutils.ncbi.nlm.nih.gov/entrez/eutils/esearch.fcgi?db=clinvar`
- **Output**: pathogenicity classification (Pathogenic / Likely pathogenic / VUS / Likely benign / Benign) + 关联疾病

---

## 5. 结果融合策略

### 5.1 归一化
- 每源返回 Top 20 疾病 + 原始分数
- 分数归一化：min-max 或 z-score 到 `[0, 1]`

### 5.2 融合方法
- **主方案**: Weighted Borda count（跨源 rank 加权累加）
- **次方案**: 加权平均归一化分数

### 5.3 权重建议（可调）

| 源 | 权重 |
|---|---|
| PubCaseFinder | 0.35 |
| Orphanet | 0.25 |
| Phenobrain | 0.20 |
| OMIM | 0.15 |
| PubMed | 0.05 |

### 5.4 去重
- OMIM / ORPHA / GARD 同一疾病通过交叉表（mondo, HPO annotation）合并
- 疾病 canonical ID 优先 MONDO，其次 ORPHA，其次 OMIM

---

## 6. LLM 共识排序（DeepRare Step 5）

- **输入**: 融合后 Top 20
- **提示**: 逐疾病对比患者 HPO，列举：
  - `support`: 患者表现与该病典型表现一致的 HPO
  - `against`: 与该病不一致或矛盾的 HPO
  - `missing`: 该病典型但患者未报告的 HPO（可能是未检查）
- **输出**: Top 5 疾病 + 证据引用（`[1][2][3]` 对应文献或数据源）
- **安全约束**: 永远表述为"**建议排查 X 方向**"，不说"**诊断为 X**"

---

## 7. 反思验证（DeepRare Step 6）

对 Top 5 中每个候选疾病：

1. **反向匹配**：列该病典型 HPO 集合，与患者 HPO 对照
   - 匹配率 `<50%` → 降权或剔除
2. **致死性急症筛查**：对每个候选标注是否涉及
   - 肾衰 / 心衰 / 代谢危象（高氨血症、乳酸酸中毒、低血糖）/ 严重感染风险 / 恶性肿瘤易感
   - 若涉及 → **单独预警模块**，优先于诊断排序展示

---

## 8. 失败回退

| 场景 | 处理 |
|---|---|
| 某源无响应 | 跳过，记录到 report footer (`sources[].status = "failed"`) |
| PubCaseFinder down | 提升 Orphanet + OMIM 权重至 0.35 + 0.30 |
| API rate limit | 减小批量、分批查；retry 最多 2 次，指数退避 |
| API key 缺失 | 跳过该源，warn 用户（不 fatal） |
| 所有必跑源都失败 | Fatal — 不输出伪造结果，返回 error report |

---

## 9. 缓存与隐私

- **可缓存**: HPO 组合 → 疾病 list 映射（24h TTL）
- **不可缓存**: 含患者标识（姓名、ID、出生日期）的查询
- **外部传输**: 只传 HPO IDs + 脱敏表型描述；基因变异以 `gene + variant notation` 匿名形式传，不带患者 ID
- 本地缓存路径建议 `~/.cache/firefly/query-cache/`，按 `sha256(sorted_hpo_ids)` 命名

---

## 10. 输出格式（给下游共识排序用）

```json
{
  "patient_id": "firefly-0001",
  "query_hpo": ["HP:0001250", "HP:0001263", "HP:0001252"],
  "queried_at": "2026-04-24T10:30:00Z",
  "sources": [
    {
      "name": "PubCaseFinder",
      "queried_at": "2026-04-24T10:30:05Z",
      "status": "success",
      "results": [
        {
          "orpha_id": "ORPHA:98896",
          "omim_id": "310200",
          "mondo_id": "MONDO:0010679",
          "disease_en": "Duchenne muscular dystrophy",
          "disease_zh": "杜氏肌营养不良",
          "score": 0.92,
          "rank": 1,
          "supporting_hpo": ["HP:0001252", "HP:0001324"]
        }
      ]
    },
    {
      "name": "Orphanet",
      "status": "success",
      "results": [ /* ... */ ]
    },
    {
      "name": "Phenobrain",
      "status": "failed",
      "error": "API timeout after 30s"
    }
  ],
  "fusion": {
    "method": "weighted_borda",
    "weights": {
      "PubCaseFinder": 0.35,
      "Orphanet": 0.25,
      "Phenobrain": 0.20,
      "OMIM": 0.15,
      "PubMed": 0.05
    },
    "top_candidates": [
      {
        "rank": 1,
        "mondo_id": "MONDO:0010679",
        "disease_en": "Duchenne muscular dystrophy",
        "fused_score": 0.89,
        "source_ranks": {"PubCaseFinder": 1, "Orphanet": 2, "OMIM": 1}
      }
    ]
  },
  "warnings": [
    "Phenobrain unavailable; weights redistributed to PubCaseFinder (+0.10) and Orphanet (+0.10)"
  ]
}
```

---

## 11. 引用

- DeepRare (Nature 2026) — multi-source rare disease diagnostic pipeline.
- Mitsuhashi, N. et al. (2022) PubCaseFinder: a case-report-based, phenotype-driven differential-diagnosis system for rare diseases. *JMIR Med Inform*.
- Orphanet API docs: `https://api.orphadata.com/`
- OMIM API terms: `https://www.omim.org/help/api`
- NCBI E-utilities: `https://www.ncbi.nlm.nih.gov/books/NBK25501/`
- MONDO Disease Ontology: `https://mondo.monarchinitiative.org/`
