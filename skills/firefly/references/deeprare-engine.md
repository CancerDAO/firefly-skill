# DeepRare 诊断引擎 — 开源复现协议

> 基于 Nature 2026 论文复现（doi:10.1038/s41586-025-10097-9）
> Recall@1: 57.18%（HPO-only, 2919 种疾病）| 70.6%（HPO + 基因组多模态）
> 超越人类专家：64.4% vs 54.6%（10+ 年经验的罕见病专科医生）

## 架构概览

DeepRare 是一个三层 Agent 架构，萤火直接在 Claude 内复现：

```
┌─────────────────────────────────────┐
│         Central LLM Host            │
│  (Agent 编排 · 决策推理 · 记忆库)     │
└─────────────┬───────────────────────┘
              │
┌─────────────┴───────────────────────┐
│         Agent Servers               │
│  表型提取器 · 疾病标准化器 · 病例检索  │
│  表型分析器 · 知识检索器 · 基因型分析  │
└─────────────┬───────────────────────┘
              │
┌─────────────┴───────────────────────┐
│       External Data Sources         │
│  PubCaseFinder · Phenobrain · OMIM  │
│  PubMed · Web · 病例库 · 基因资源     │
└─────────────────────────────────────┘
```

## Stage 1: 表型提取与 HPO 标准化

### 1.1 从临床文本提取表型

从患者描述/病历中提取所有临床表型特征。

**提取 Prompt（复现 DeepRare hpo_extractor）：**
```
You are a medical expert specializing in rare diseases. 
Extract ALL clinical phenotype features from the following patient description.

For each phenotype:
1. Extract the exact clinical feature mentioned
2. Map it to the closest HPO (Human Phenotype Ontology) term
3. Provide the HPO ID (HP:XXXXXXX)
4. Rate confidence: HIGH (exact match) / MEDIUM (close match) / LOW (approximate)

Also extract:
- Age of onset
- Sex
- Progression pattern (progressive / stable / episodic)
- Family history indicators

Output as structured JSON:
{
  "phenotypes": [
    {"description": "...", "hpo_term": "...", "hpo_id": "HP:...", "confidence": "HIGH/MEDIUM/LOW"}
  ],
  "onset_age": "...",
  "sex": "...",
  "progression": "...",
  "family_history": "..."
}
```

### 1.2 HPO 术语验证与去重

- 对每个提取的 HPO 术语，验证 HPO ID 是否存在
- 去除重复的和层级冗余的术语（如果同时有 "Seizures" HP:0001250 和 "Generalized tonic-clonic seizures" HP:0002069，保留更具体的）
- 过滤低置信度匹配（confidence = LOW 的需要追问患者确认）

### 1.3 阴性表型收集

**关键创新**：DeepRare 同时收集患者明确没有的症状（阴性表型），用于鉴别诊断。
- 追问患者：确认哪些常见伴随症状不存在
- 阴性表型同样标注 HPO 术语，标记为 EXCLUDED

## Stage 2: 多源知识检索（并行执行）

这是 DeepRare 的核心——不依赖单一知识源，而是并行查询多个独立来源，然后共识排序。

### 2.1 PubCaseFinder 检索

PubCaseFinder（DBCLS 维护，300,000+ 病例报告）是罕见病表型匹配的核心工具。

**首选方式 — WebSearch 间接查询：**
```
查询策略：
1. 用 WebSearch 搜索：site:pubcasefinder.dbcls.jp + HPO 术语组合
2. 或用 WebSearch 搜索：HPO ID 列表 + "rare disease" + "differential diagnosis"
3. 从搜索结果中提取候选疾病及其 OMIM/ORPHA 编号
```

**备选方式 — API 直接查询（可能不稳定）：**
```
端点：https://pubcasefinder.dbcls.jp/api/get_ranked_list
参数：target=disease&hpo_id=HP:xxx;HP:yyy&lang=en&limit=20
注意：API 可能返回 502，此时回退到 WebSearch 方式
```

**备选方式 — 网页版手动查询：**
如以上均失败，提示用户直接访问 pubcasefinder.dbcls.jp 输入 HPO 术语查询。

### 2.2 Phenobrain 预测

Phenobrain 是基于深度学习的表型-疾病预测服务。

**注意：Phenobrain.org 服务可能不稳定或已迁移。**

**首选替代方案 — Monarch Initiative：**
```
查询策略：
1. 用 WebSearch 搜索：site:monarchinitiative.org + HPO ID
2. 或搜索：HPO 术语 + "associated diseases" + OMIM/Orphanet
3. Monarch 整合了 HPO-疾病关联数据，功能等效于 Phenobrain
```

**备选 — NCBI MedGen：**
```
查询：https://www.ncbi.nlm.nih.gov/medgen/?term=HPO_ID
从 MedGen 提取关联的疾病条目和 OMIM 交叉引用
```

**如 Phenobrain 可用：**
```
端点：https://phenobrain.org/api/（需确认最新地址）
方法：POST HPO 术语集合，获取 AI 预测的疾病排名
```

### 2.3 OMIM / Orphanet 知识库检索

```
对患者的核心症状组合，使用 WebSearch 并行查询：

1. OMIM 查询：
   WebSearch: "OMIM" + 核心 HPO 术语（英文）+ "rare disease"
   WebFetch: https://omim.org/entry/[编号] 获取详细条目
   
2. Orphanet 查询：
   WebSearch: "Orphanet" + 疾病关键词 + HPO 术语
   WebFetch: https://www.orpha.net/en/disease/detail/[ORPHA编号]
   
3. GARD 查询：
   WebSearch: site:rarediseases.info.nih.gov + 疾病关键词
   
4. NORD 查询（补充）：
   WebSearch: site:rarediseases.org + 症状关键词
   
5. 记录每个来源的候选疾病列表及 OMIM/ORPHA 编号
```

### 2.4 文献检索

```
1. PubMed 检索：用 HPO 术语组合 + "rare disease" 检索最新文献
   - 关注 Case Report 和 Clinical Study
   - 提取文献中报告的疾病诊断
2. Web 检索：用症状组合进行通用搜索
   - 关注权威医学网站的结果
   - 提取提到的候选疾病
```

### 2.5 相似病例匹配

```
基于患者的表型嵌入向量，在已确诊病例库中检索最相似的 K 个病例：
1. 将患者的 HPO 术语集合编码为语义嵌入向量
2. 与历史病例库中的嵌入向量计算余弦相似度
3. 返回 Top-5 最相似的已确诊病例及其最终诊断
4. 这些病例作为诊断推理的参考锚点
```

### 2.6 基因组分析（可选，如有 VCF 数据）

```
如患者提供了基因检测数据（VCF 文件或变异列表）：
1. 提取致病/可能致病变异（ACMG P/LP）
2. 提取 VUS 变异中与候选疾病基因相关的
3. 将基因证据与表型证据整合
4. 基因组增强后 Recall@1 从 57.18% 提升至 70.6%
```

## Stage 3: 共识排序与推理链生成

### 3.1 多源共识

```
汇总所有来源的候选疾病列表：
- PubCaseFinder Top-20
- Phenobrain Top-20
- OMIM/Orphanet 检索结果
- 文献检索提到的疾病
- 相似病例的诊断
- （可选）基因组分析候选

共识排序逻辑：
1. 统计每个候选疾病被多少个独立来源提及
2. 被多个来源一致认为高可能性的 → 排名靠前
3. 仅被单一来源提及的 → 排名靠后但不排除
4. 综合考虑每个来源的匹配得分
```

### 3.2 Top-5 诊断方向生成

**推理 Prompt（复现 DeepRare 核心推理）：**
```
Based on the following multi-source evidence for this patient:

Patient phenotypes (HPO): [列表]
Excluded phenotypes: [列表]
Age of onset: [年龄]
Family history: [信息]

PubCaseFinder results: [Top-20]
Phenobrain results: [Top-20]
OMIM/Orphanet results: [结果]
Literature findings: [结果]
Similar cases: [Top-5 相似病例]
Genetic findings: [如有]

Enumerate the top 5 most likely rare disease diagnoses, ordered from most to least likely.

For each diagnosis:
1. Disease name (Chinese + English) and ORPHA/OMIM ID
2. Detailed reasoning chain explaining WHY this diagnosis fits
3. Which phenotypes match (✅) and which don't match (❌)
4. Epidemiological data (prevalence, typical onset age)
5. Key confirmatory tests
6. Reference sources with citations [1][2][3]

Tag rare diseases with ## markup for emphasis.
Provide fewer diagnoses if you are not highly confident beyond the top few.
```

### 3.3 可追溯推理链（Traceable Reasoning）

DeepRare 的关键创新是每个诊断建议都有可追溯的推理链和可验证的参考文献。

**推理链结构（论文 Fig.4a 复现）：**
```
## [疾病名称]
### 诊断推理：
"This diagnosis is strongly supported by the patient's phenotype,
including [具体匹配的症状]. These are hallmark features of [疾病名].
[疾病] is a disorder caused by [致病机制].
The patient's [其他表型] are concordant with [疾病的表现]."

### 参考来源：
(1) Source: [文献/数据库名称]
    Content: [具体引用内容]
    URL: [可验证链接]
(2) Source: OMIM
    Content: OMIM entry #[编号] describes [相关信息]
    URL: https://omim.org/entry/[编号]
(3) Source: Similar case
    Content: Similar Case [N] describes a patient with [疾病],
    presenting with [相似症状], closely matching the patient's phenotype.
```

**参考文献准确性要求**（论文验证：平均准确率 0.954）：
- 每个引用的 URL 必须是真实存在的
- 引用的内容必须与实际来源一致
- 不编造不存在的文献或链接
- 如果无法找到可验证的参考文献，标注"基于领域知识推断，无直接文献引用"

## Stage 4: 自反思验证循环（Self-Reflection）

DeepRare 使用自反思循环来验证诊断结果的可靠性：

```
验证检查清单：
1. 每个 Top-5 诊断是否有至少 2 个独立来源支持？
2. 推理链是否存在逻辑矛盾？
3. 是否遗漏了患者的重要阴性表型？（阴性表型可能排除某些候选）
4. 参考文献是否真实可查？
5. 是否存在"表型模拟诊断"风险？（正确的临床类别但错误的具体疾病）

如果验证不通过：
- 扩大检索深度（更多来源、更宽的关键词）
- 重新评估被排除的候选
- 调整推理权重（避免过度强调非特异性症状）
```

## 已知失败模式（论文 200 例分析）

萤火在推理时需主动规避这些已知错误模式：

| 失败模式 | 占比 | 描述 | 规避策略 |
|---------|------|------|---------|
| 推理权重错误 | 41.0% | 过度强调非特异性症状，忽视更具鉴别力的特征 | 优先考虑特异性高的症状，降低非特异性症状权重 |
| 表型模拟诊断 | 38.5% | 识别了正确的临床类别但诊断了错误的具体疾病 | 在同一类别内做细致鉴别，关注区分性特征 |
| 病因关联诊断 | 15.5% | 诊断了与真正疾病相关但不同的疾病 | 检查候选疾病之间的病因学关系 |
| 推理事实错误 | 2.5% | 引用的医学事实不正确 | 对关键事实进行来源验证 |
| 证据链接错误 | 2.5% | 引用的参考文献不存在或不相关 | 验证每个 URL 和引用内容 |

## 多模态整合（HPO + 基因组）

当同时有表型和基因组数据时，采用论文的多模态整合策略：

```
整合流程：
1. 分别运行 HPO-only 诊断 和 基因组变异分析
2. 交叉验证：
   - HPO 候选中是否有疾病的致病基因出现在患者的变异列表中？
   - 基因组候选中的疾病表型是否与患者匹配？
3. 双重支持的候选疾病 → 高置信度，排名提升
4. 仅单一模态支持的 → 中等置信度
5. 两种模态矛盾的 → 需要人工审查

论文数据：
- Xinhua Hospital: HPO-only 39.9% → HPO+Gene 69.1% (Recall@1)
- Hunan Hospital: HPO-only 33.3% → HPO+Gene 63.6% (Recall@1)
```

## 14 个身体系统的专科表现

DeepRare 在论文中验证了跨 14 个身体系统的诊断能力（Fig.3a）：

| 系统 | 样本量 | 备注 |
|------|--------|------|
| 脑和神经 | 1,871 | 最大样本，DeepRare 表现最稳定 |
| 免疫系统 | 1,515 | |
| 血液/心脏/循环 | 1,066 | |
| 骨骼/关节/肌肉 | 1,025 | |
| 内分泌系统 | 922 | |
| 消化系统 | 729 | |
| 皮肤/毛发/指甲 | 688 | |
| 眼和视觉 | 676 | |
| 肾脏/泌尿 | 310 | |
| 肺和呼吸 | 222 | |
| 耳鼻喉 | 169 | |
| 女性生殖 | 66 | 样本较少 |
| 口腔/牙齿 | 61 | 样本较少 |
| 男性生殖 | 41 | 样本最少 |

## 执行清单

萤火执行 DeepRare 协议时的检查清单：

- [ ] HPO 术语提取完成，至少 3 个核心表型
- [ ] 阴性表型已收集
- [ ] PubCaseFinder 已查询（或等效替代）
- [ ] Phenobrain 已查询（或等效替代）
- [ ] OMIM/Orphanet 已查询
- [ ] 文献检索已完成
- [ ] 多源共识排序已完成
- [ ] Top-5 诊断方向已生成
- [ ] 每个方向有推理链和参考文献
- [ ] 自反思验证已通过
- [ ] 已检查已知失败模式
- [ ] 基因组数据已整合（如有）
