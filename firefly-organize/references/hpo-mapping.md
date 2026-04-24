# HPO Mapping Protocol (LLM-Executed)

## 1. 目的

把患者自由文本描述的表型（如"走路不稳"、"抽搐"、"智力发育迟缓"）映射到 **Human Phenotype Ontology (HPO)** 标准术语（`HP:XXXXXXX`），以便：

- 多源检索（PubCaseFinder / Orphanet / OMIM）可直接喂入
- 跨库可比、跨语言可比
- 诊断级数据结构（diagnostic grade），支持 DeepRare 多源融合

**标准来源**：human-phenotype-ontology.org；Köhler S. et al. (2021) *The Human Phenotype Ontology in 2021*, Nucleic Acids Res.

---

## 2. 映射协议（LLM 执行）

### Step A: 提取表型候选

- 从患者描述 / 病历 / OCR sidecar 中识别临床相关表型词
- **区分 症状 vs 体征**：主诉（subjective）vs 查体/影像/化验所见（objective）
- **区分 当前 vs 既往**：需标记 onset 时间窗
- 非表型信息（家族史、用药史、社会史）另存，不进入 HPO 列表

### Step B: 候选 HPO 术语查询

- 对每个候选词调用 HPO 官方词表：
  - 语义检索（LLM 向量相似度）或
  - HPO JSON API (`https://hpo.jax.org/api/hpo/search`)
- 返回 top-K (K=5) candidate HPO IDs + labels + definitions

### Step C: 语义相似度阈值判断

- 阈值：余弦相似度 **≥ 0.8**（建议嵌入模型：BioLORD 或同等生物医学模型）
- LLM 二次判定语义一致性，不仅靠字符串匹配
- 若无任何候选 ≥0.8 → 标注 `待确认`（`confidence: low`），**不强行匹配**

### Step D: 去重与层级整合

- HPO 是 DAG，父子术语可能同时出现
- 保留最具体（叶子）术语；父术语仅在**唯一附加信息**时保留
- 示例：`HP:0001250 (Seizure)` 是 `HP:0002197 (Generalized-onset seizure)` 的祖先；若病人明确为 generalized-onset，仅保留子术语

### Step E: 元数据附加

对每个保留的 HPO term 附加：

| 字段 | 值域 | 对齐 ontology |
|---|---|---|
| onset | congenital / infantile / childhood / juvenile / adult | HPO Onset (HP:0003674) |
| severity | mild / moderate / severe / profound | HPO Severity (HP:0012824) |
| laterality | left / right / bilateral / unilateral | HPO Laterality (HP:0012831) |
| frequency | very frequent / frequent / occasional / very rare | HPO Frequency (HP:0040279) |

---

## 3. 常见表型映射示例表（中英对照 + HP ID）

### 神经系统

| 中文 | English | HPO ID |
|---|---|---|
| 癫痫发作 | Seizure | HP:0001250 |
| 全面性起病癫痫 | Generalized-onset seizure | HP:0002197 |
| 局灶性起病癫痫 | Focal-onset seizure | HP:0007359 |
| 肌张力低下 | Hypotonia | HP:0001252 |
| 肌张力增高 | Hypertonia | HP:0001276 |
| 共济失调 | Ataxia | HP:0001251 |
| 肌阵挛 | Myoclonus | HP:0001336 |
| 智力障碍 | Intellectual disability | HP:0001249 |
| 全面性发育迟缓 | Global developmental delay | HP:0001263 |
| 发育倒退 | Developmental regression | HP:0002376 |
| 言语发育迟缓 | Delayed speech and language development | HP:0000750 |
| 运动发育迟缓 | Motor delay | HP:0001270 |
| 小头畸形 | Microcephaly | HP:0000252 |
| 大头畸形 | Macrocephaly | HP:0000256 |

### 骨骼与结缔组织

| 中文 | English | HPO ID |
|---|---|---|
| 身材矮小 | Short stature | HP:0004322 |
| 骨折易发 | Recurrent fractures | HP:0002757 |
| 关节过度活动 | Joint hypermobility | HP:0001382 |
| 脊柱侧弯 | Scoliosis | HP:0002650 |
| 皮肤过度延展 | Hyperextensible skin | HP:0000974 |

### 皮肤

| 中文 | English | HPO ID |
|---|---|---|
| 咖啡牛奶斑 | Café-au-lait spot | HP:0000957 |
| 毛细血管扩张 | Telangiectasia | HP:0001009 |
| 皮肤皱纹增多 | Premature skin wrinkling | HP:0007392 |

### 眼科

| 中文 | English | HPO ID |
|---|---|---|
| 先天性白内障 | Congenital cataract | HP:0000519 |
| 视神经萎缩 | Optic atrophy | HP:0000648 |
| 视网膜色素变性 | Retinitis pigmentosa | HP:0000510 |
| 晶状体脱位 | Ectopia lentis | HP:0001083 |

### 听力

| 中文 | English | HPO ID |
|---|---|---|
| 感音神经性听力损失 | Sensorineural hearing impairment | HP:0000407 |
| 传导性听力损失 | Conductive hearing impairment | HP:0000405 |

### 代谢

| 中文 | English | HPO ID |
|---|---|---|
| 高氨血症 | Hyperammonemia | HP:0001987 |
| 低血糖发作 | Hypoglycemia | HP:0001943 |
| 代谢性酸中毒 | Metabolic acidosis | HP:0001942 |
| 乳酸性酸中毒 | Lactic acidosis | HP:0003128 |
| 酮症 | Ketosis | HP:0002919 |

### 肝

| 中文 | English | HPO ID |
|---|---|---|
| 肝肿大 | Hepatomegaly | HP:0002240 |
| 肝酶升高 | Elevated hepatic transaminase | HP:0002910 |
| 脾肿大 | Splenomegaly | HP:0001744 |

### 免疫

| 中文 | English | HPO ID |
|---|---|---|
| 反复感染 | Recurrent infections | HP:0002719 |
| 反复细菌感染 | Recurrent bacterial infections | HP:0002718 |
| 中性粒细胞减少 | Neutropenia | HP:0001875 |
| 淋巴细胞减少 | Lymphopenia | HP:0001888 |

### 心脏

| 中文 | English | HPO ID |
|---|---|---|
| 肥厚型心肌病 | Hypertrophic cardiomyopathy | HP:0001639 |
| 扩张型心肌病 | Dilated cardiomyopathy | HP:0001644 |
| 室间隔缺损 | Ventricular septal defect | HP:0001629 |
| 房间隔缺损 | Atrial septal defect | HP:0001631 |
| 主动脉瓣狭窄 | Aortic valve stenosis | HP:0001650 |

### 面容

| 中文 | English | HPO ID |
|---|---|---|
| 特殊面容 | Abnormal facial shape | HP:0001999 |
| 眼距增宽 | Hypertelorism | HP:0000316 |
| 低耳位 | Low-set ears | HP:0000369 |
| 厚唇 | Thick lower lip vermilion | HP:0000179 |
| 小颌 | Micrognathia | HP:0000347 |

---

## 4. 模糊 / 混淆术语处理

| 原文 | 处理 |
|---|---|
| "智力低下" | 旧术语；用 `HP:0001249 Intellectual disability`，按严重度分 mild (HP:0001256) / moderate (HP:0002342) / severe (HP:0010864) / profound (HP:0002187) |
| "发育迟缓" | 区分 global (`HP:0001263`) vs specific domain（speech/motor/cognitive） |
| "抽搐" | 区分 癫痫发作 seizure (`HP:0001250`) vs 抽动 tic (`HP:0100033`) — 问持续时间、是否伴意识改变 |
| "肌无力" vs "肌张力低" | muscle weakness (`HP:0001324`) vs hypotonia (`HP:0001252`) — 前者主动力量差，后者被动张力低 |
| "发育倒退" | `HP:0002376` Developmental regression（既往获得技能丢失），与静止性发育迟缓 (`HP:0001263`) 区分 |
| "抽筋" | 区分 cramp (`HP:0003394`) vs spasm vs seizure — 问部位、意识、持续时间 |

---

## 5. 质量检查清单（pipeline 末）

- [ ] 所有 HPO ID 格式正确（`HP:` + 7 位数字）
- [ ] 没有已废除术语（查 HPO `obsolete` list）
- [ ] 没有冗余（父子术语去冗余，保叶子）
- [ ] 每个术语有 `onset` 标注（`unknown` 可接受）
- [ ] 总数合理：罕见病诊断通常 **5–30 个 HPO**
  - 过少（<5）：信息不足，检索噪音大
  - 过多（>30）：可能混入非特异性/误导性术语

---

## 6. 输出 JSON 格式（对齐 profile.json schema）

```json
[
  {
    "hpo_id": "HP:0001250",
    "label_en": "Seizure",
    "label_zh": "癫痫发作",
    "onset": "infancy",
    "severity": "moderate",
    "frequency": "frequent",
    "laterality": null,
    "source_text": "从 3 个月起反复抽搐，每日 1–2 次",
    "confidence": "high"
  },
  {
    "hpo_id": "HP:0001263",
    "label_en": "Global developmental delay",
    "label_zh": "全面性发育迟缓",
    "onset": "infancy",
    "severity": "severe",
    "frequency": null,
    "laterality": null,
    "source_text": "1 岁不能独坐，不会叫爸妈",
    "confidence": "high"
  }
]
```

---

## 7. 引用

- Köhler S. et al. (2021) The Human Phenotype Ontology in 2021. *Nucleic Acids Res* 49:D1207–D1217.
- Arnaud, R. et al. (2022) BioLORD: learning ontological representations from definitions for biomedical concepts.
- DeepRare pipeline (Nature 2026) — HPO mapping as Step 1 of multi-source rare disease diagnostic reasoning.
- HPO official search API: `https://hpo.jax.org/api/hpo/search?q=<term>`
