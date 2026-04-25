---
name: firefly-organize
description: "OCR 病历 → HPO 标准化 → DeepRare 多源诊断导航。完整 pipeline: 文件/对话输入 → 文本提取（PDF/图片）→ LLM 结构化 → HPO 术语映射 → 多源并行检索（PubCaseFinder/Phenobrain/OMIM/Orphanet/PubMed）→ LLM 共识排序 → 反思验证 → Top 5 诊断方向 + 检查建议 + 就诊摘要. Use when: 患者有病历/基因报告要梳理；确诊不了想做 AI 辅助诊断导航；要为 profile.json 初始化诊断档案. Triggers on: 病历整理, OCR, HPO, 诊断导航, DeepRare, 确诊不了, 看了很多医生, 症状排查, 罕见病诊断, 未确诊, 诊断方向."
---

# firefly-organize — 病历整理 + DeepRare 诊断导航

把患者的散落病历/描述/报告，转成结构化 profile.json + Top 5 诊断方向。这是 firefly 家族的入口级 companion。

## 何时调用

- 患者发来病历文件夹（PDF / 图片 / DOCX）
- 患者口述症状但手头没报告
- 已做 WES 但没人给出诊断方向
- profile.json 尚未创建
- 用户说"我看了很多医生都确诊不了"

## 协议（DeepRare 完整 pipeline）

### Step 1: 输入收集

**对话模式**：引导式收集
- 主诉 + 伴随症状（开放式问，不一次问太多）
- 发病时间线（起病年龄、进展速度、加重/缓解因素）
- 已就诊经历（科室、医院、结论）
- 已做检查（基因、血液、影像、活检——结果如何）
- 家族史（近亲类似症状、近亲婚配、民族）
- 生长发育史（儿童患者，里程碑）

**OCR 模式**：
1. 接受文件夹或单文件（PDF / JPG / PNG / DOCX）
2. 文本提取：`pdftotext`、`pytesseract`、或 LLM 视觉直接读
3. 重点识别：基因报告变异位点、IHC 结果、影像描述、病理描述
4. **dispatch subagent** 做重 LLM 结构化（长文本解析、多文档合并），主线程只编排

### Step 2: 患者档案初始化

生成 profile.json（schema 见 `../firefly/references/templates.md`）：
- demographics
- clinical（含 chief_complaint、onset_age、progression）
- family_history
- diagnostics_done
- genetic_findings（如有）

### Step 3: HPO 表型标准化

1. LLM 从临床文本提取表型描述（"走路不稳"、"抽搐"、"智力障碍"）
2. 映射到 HPO 标准术语（`HP:XXXXXXX`）。协议详见 `references/hpo-mapping.md`
3. 阈值：语义相似度 ≥0.8，去重，保留起病时间
4. 输出：标准化 HPO ID 清单，回写 `profile.json.clinical.hpo_terms`

### Step 4: 多源知识检索（并行）

详见 `references/multi-source-search.md` 和 `../firefly/references/deeprare-engine.md`。

Dispatch 多个 subagent 并行查询：
- PubCaseFinder（HPO-based case retrieval）
- Phenobrain（AI 表型-疾病预测）
- OMIM / Orphanet / GARD
- PubMed（最新文献）
- 相似病例库

### Step 5: LLM 共识排序

综合多源结果，生成 Top 5 诊断方向。每个方向附：
- 疾病名（中英 + ORPHA + OMIM）
- 匹配理由（哪些 HPO 支持）
- 不吻合点（哪些 HPO 不符）
- 发病率
- 信息来源引用 [1][2][3]

**永远用"建议排查 X 方向"，永远不说"你可能得了 X 病"。**

### Step 6: 反思验证

对每个候选做反向验证：
- 该疾病的典型表现是否与患者一致？
- 是否遗漏不典型亚型？
- 是否有致死性疾病需要紧急排除？
- 如初轮验证不通过，扩大检索重新分析

### Step 7: 基因组增强（如有 VCF）

- dispatch subagent 做 Exomiser 风格变异优先级（基因组 + 表型联合）
- ACMG 变异分类解读（交给 firefly-genetic-counseling 深度处理，organize 只做初筛）
- 表型-基因组多模态诊断

### Step 8: 输出

四份文件，写入 `patients/<patient_id>/`：

1. `profile.json`（结构化，符合 schema）
2. `timeline.md`（治疗时间线，Markdown 时间轴）
3. `diagnosis-directions.md`（诊断方向报告）
4. `就诊摘要.md`（带给下一位医生的 1-2 页总结）

模板见 `../firefly/references/templates.md`。

### Step 9: 检查建议

对每个 Top 5 方向，列出：
- 🔴 优先做（最能确认/排除的检查）
- 🟡 建议做
- 🟢 可选做

检查类型：基因（panel vs WES vs WGS）、代谢、影像、活检等。标注国内可做性和费用范围。

### Step 10: 专科医院推荐

基于 Top 5 方向，查 `../firefly/references/resources-cn.md` 推荐中国罕见病诊疗协作网 324 家医院中最合适的 3-5 家。

## profile.json 交互

- **Reads**: —（创建）
- **Writes**: 全部核心字段——demographics、clinical、diagnostics_done、diagnosis_directions

## Safety Guardrails

- **不诊断**。Top 5 是方向不是结论。
- HPO 映射如果不确定 → 标注"待医生确认"不强行匹配
- 不在 profile.json 里写任何未被证据支持的主张——每条都可追溯
- 基因数据隐私——OCR 过程中不把变异位点明文传给外部 API，尽可能本地处理
- 如 Top 5 包含致死性急症方向（肾衰、心衰、代谢危象） → 第一时间提示"建议立即就医而非等待诊断"
- 盲点自检：生存者偏差？种族代表性（gnomAD 偏向欧洲裔）？诊断后支持是否被忽略？
- 任何环节遇到致死性信号 → 调 firefly-mind check 心理状态 + 提示就医

## References

- [references/hpo-mapping.md](references/hpo-mapping.md) — HPO 标准化映射协议
- [references/multi-source-search.md](references/multi-source-search.md) — 并行多源检索协议
- `../firefly/references/deeprare-engine.md` — DeepRare 完整引擎协议（共享）
- `../firefly/references/templates.md` — profile.json schema + 输出模板（共享）
- `../firefly/references/evidence-grading.md` — 证据分级（共享）
- `../firefly/references/resources-cn.md` — 中国诊疗协作网医院（共享）
