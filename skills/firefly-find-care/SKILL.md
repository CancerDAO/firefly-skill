---
name: firefly-find-care
description: "活体查找罕见病的专家中心、专科医生、临床试验、诊断网络入口（UDN/UDP-EU/中国罕见病诊疗协作网）。**只做资源发现，不做诊断或治疗判断**。典型问题：杭州周边谁能看 Pompe 病？我家孩子 SMA，国内外有什么基因治疗试验在招？这个 ORPHA 号有没有 ORPHANET 认证的 expert centre？AADC 缺乏症的国内随访体系在哪？输入：profile.json（确诊/疑似病种 + ORPHA + HPO 表型 + 家庭所在地 + 能否跨城/跨境）+ 一个具体诉求。输出：排序后的资源短名单（含联系/转诊路径、地址、所需材料、活动状态）。和 firefly-patient-org 互补——patient-org 查已 curated 的病友组织/遗传咨询师库，find-care 走多 subagent 联网做活体发现。Triggers on: 找专家, 找罕见病专科医生, 哪家医院能看, 找诊断中心, 罕见病诊疗协作网, ORPHANET, UDN, RDCRN, 找临床试验, 罕见病试验, 哪里有基因治疗, 找专科中心, 罕见病随访."
---

# firefly-find-care — 资源发现（活体）

帮罕见病家庭找资源，不替你做医疗判断。

罕见病比常见病更难找资源——单病专家中心常常全国只有 2-3 家，国际诊断网络（UDN/UDP-EU）申请门槛高，临床试验招募分散在 ORPHANET / ClinicalTrials.gov / ChiCTR 不同源，而且病友组织生命周期短（很多老群已停止活跃）。这个 skill 把它做成一次可执行的并行调研：理解你的具体诉求 → 分派多个子 agent 去权威源 → 汇总成短名单。

## When to use

触发场景（用户说类似这些话）：
- "孩子 6 岁确诊 Pompe 病（庞贝氏症），上海能看的医生是谁？酶替代治疗去哪做？"
- "我们这个 AADC 缺乏症 ORPHA:35687，国内有没有专家中心？孩子需要长期随访"
- "SMA 1 型，听说国外有基因治疗试验，怎么找？"
- "DMD 杜兴肌营养不良，想申请加入 RDCRN 数据库怎么操作？"
- "确诊不了，5 家医院都没头绪，UDN / UDP-EU 申请有没有同病种的成功路径？"

## What this skill is NOT

**不是诊断、不是基因咨询、不是治疗判断。**

- 不解读 ACMG 变异分类 → `firefly-genetic-counseling`
- 不评估家系遗传风险 → `firefly-genetic-counseling`
- 不做基于 HPO 的鉴别诊断 → `firefly-organize`（DeepRare 流程）
- 不准备跨境会诊材料 → `firefly-second-opinion`
- 不查已 curated 的病友组织/遗传咨询师库 → `firefly-patient-org`

我们做的就是：**告诉你哪里有人能做这件事 + 怎么联系**。

## Preflight

### Profile completeness

读 `patients/<patient_code>/profile.json`，至少需要：

| 字段 | 必需？ | 用途 |
|---|---|---|
| `confirmed_diagnosis` 或 `suspected_diagnoses[]` | 必需 | 决定查哪些专科 / ORPHA expert centres |
| `orpha_id` | 强烈推荐 | ORPHANET / RDCRN / UDN 检索主键 |
| `hpo_terms[]` | 推荐 | 未确诊场景下用于扩散搜索 |
| `geo.current_city` + `geo.willing_radius` | 必需 | 否则全国/全球范围太散 |
| `economic.budget_band` | 推荐 | 影响公立 / 自费 / 海外比例 |
| `patient_age` + `caregiver_role` | 推荐 | 罕见病多为儿童，成人专科 vs 儿科分流 |

**字段不全时**：不要让 subagent 瞎跑。先回过头问 1–3 个关键字段，再调研。

### Subagent 能力前置

`firefly-find-care` 依赖 host agent 能并行派发子 agent + 加载 `web-access` skill。当前可靠运行环境：
- ✅ Claude Code（用 Agent 工具派 subagent，subagent 加载 vendored `../web-access`）
- ⚠️ Codex / OpenCode / Cursor：subagent 派发能力不一致，可降级为主 agent 串行联网

降级模式下，主 agent 自己用 WebSearch/WebFetch 走一遍同样的数据源清单，速度慢但流程一致。

## Core workflow

### Step 1 — Clarify the ask

把用户那句话翻译成可调研的查询，写到 `patients/<patient_code>/reports/find-care/<query-slug>/QUERY.md`：

```yaml
query_type: expert_centre | doctor | trial | diagnostic_network | combined
clinical_target:
  disease_name: 庞贝氏症 / Pompe disease
  orpha_id: ORPHA:365
  hpo_anchors: [HP:0003560, HP:0001263]
  service_needed: 酶替代治疗（Myozyme）+ 长期随访
geo:
  primary_city: 上海
  acceptable_cities: [北京, 杭州, 南京]
  cross_border: 视必要性而定
constraints:
  budget: 国内医保为主 / 慈善赠药可考虑 / 不考虑自费海外
  patient_age_band: 6yo 儿科
  timing: 1 个月内能挂到号
patient_profile_ref: patients/PT-XXXX/profile.json
```

不确定的字段当面问用户，**别让 subagent 替你猜**。

### Step 2 — Plan parallel investigations

按 `query_type` 决定派几个 subagent。每条调研路线一个，互不依赖。

**典型分派：**

| query_type | Subagent A | Subagent B | Subagent C | Subagent D |
|---|---|---|---|---|
| expert_centre | ORPHANET expert centres database：按 ORPHA + 国家筛 | 中国罕见病诊疗协作网（324 家网络医院） + 病种是否在该院 listed | RDCRN（US Rare Diseases Clinical Research Network）：该病种是否有 consortium | （可选）该病种基金会的"Find a Doctor"页 |
| doctor | 中华医学会医学遗传学分会 + 中华医学会儿科学分会代谢学组 / 神经学组（视病种）专家名单 | 好大夫在线该病种 top 医生 + 出诊医院（CDP，因强反爬） | 院官网医生介绍页（亚专业 + 出诊安排） | Pubmed：该医生近 5 年发表是否对口此病种 |
| trial | ClinicalTrials.gov：disease 字段精准查 ORPHA 病名 + status=Recruiting | ChiCTR：中国境内罕见病试验 | RDCRN consortium 自有试验登记页 | 大型基金会 / 制药公司的病种 portal（如 Sarepta DMD portal） |
| diagnostic_network | UDN（US，未诊断疾病网络）申请通道 + 国际患者 acceptance 路径 | UDP-EU（欧盟版） | 中国国家罕见病注册系统 / 北京协和疑难病中心 / 上海儿童医学中心罕见病诊治平台 | （可选）民间 DeepRare-style 服务（含潜在质量风险） |
| combined | 上述按需混合，控制在 4–6 个 subagent | | | |

### Step 3 — Dispatch (并行)

每个 subagent 用 Agent 工具启动（CC）或主 agent 串行（其他 host），prompt 模板：

```
任务：<具体目标，例如：在 ORPHANET 的 expert centres 数据库查 ORPHA:365 (Pompe disease) 在中国大陆的认证中心列表，每个含中心名 / 城市 / 联系方式 / ORPHA 标签级别（Reference Centre / Expert Centre / Specialised Service）>

约束：
- **必须加载 web-access skill 并遵循指引**（vendored at ../web-access）
- 数据源限定一手（ORPHANET / RDCRN / UDN / 院官网 / 国家级注册系统），不接受社交媒体软文作为唯一证据
- 罕见病数据极敏感：不要把患者真实病历内容暴露给查询参数；只用 ORPHA / disease name / HPO 这些去识别化标识
- 单 subagent 最多 5 分钟，超时返回"未完成 + 已采集到的部分"

输出格式（JSON 写到 patients/<patient_code>/reports/find-care/<query-slug>/raw/<subagent-name>.json）：
{
  "source_type": "orphanet|rdcrn|udn|hospital_page|trial_registry|society_directory|foundation",
  "source_url": "...",
  "fetched_at": "ISO8601",
  "items": [
    {
      "name": "...", "url": "...",
      "evidence_quote": "原文片段",
      "fields": {
        "orpha_label": "Reference Centre / Expert Centre / null",
        "city": "...", "specialty": "...",
        "team_lead": "...",
        "intake_path": "...",
        "language": "...",
        "last_updated": "..."
      }
    }
  ],
  "notes": "执行中遇到的问题、未覆盖的子目标"
}
```

**并行而非串行**：CC 一次性派 4 个 subagent，prompt 只说要什么，**不指定 web-access 内部步骤**——子 agent 自己判断该用 WebSearch 还是 CDP。其他 host 降级为主 agent 串行。

### Step 4 — Merge & score

收到所有 subagent 的 JSON：

1. **De-duplicate** — 同一中心 / 医生在多源出现合并，证据列表叠加
2. **Score by fit** — 按 [references/scoring-rubric.md](references/scoring-rubric.md) 维度打分
3. **Filter** — 删除明显不匹配的（地理范围外 / 不接此病种 / 试验已停招 / 中心不接收外院患者）
4. **Rank** — 按总分排序，输出 top 5–8

打分维度（详细权重见 rubric）：
- ORPHA / RDCRN 认证等级（Reference > Expert > Specialised > 未认证）
- 病种匹配度（明确做该病 vs 一般罕见病科室）
- 地理可及性（罕见病专家中心稀缺，权重低于癌症 find-care）
- 入组 / 接诊可行性（外院患者是否接受、转诊门槛）
- 证据强度（一手 > 二手；多源互证 > 单源）
- （仅医生类）出诊频率 + 该病种发表
- （仅试验类）招募状态 + 距离 + 入排标准初筛

### Step 5 — Write shortlist

写到 `patients/<patient_code>/reports/find-care/<query-slug>/SHORTLIST.md`，按 [references/output-template.md](references/output-template.md)。

每条目必含：
- 名称（中心 / 医生 / 试验编号）
- ORPHA / RDCRN 认证等级（如有）
- 匹配度（高/中/低 + 一句话理由，**不是 "我推荐"**）
- 关键事实（位置 / 团队 / 频率 / 试验状态 / 接诊门槛）
- **联系 / 转诊路径**（具体到平台 + 操作步骤 + 所需材料）
- 证据来源 URL 列表
- 潜在风险 / 限制（费用 / 等候期 / 异地医保 / 试验排除标准 / 需先做什么检测）

末尾**必须**有：

```
> 这是资源发现的结果，不是医学推荐。是否真的合适，需要带这份清单和你的主诊医生 / 遗传咨询师讨论。挂号或入组前由对方医生评估。
> 罕见病数据极易被重识别——和上述资源沟通时，注意只发送必要的 de-identified 信息（ORPHA / HPO / 关键检测结果），不要把完整 profile.json 直接外发。
```

## Role behavior

罕见病家庭里 **caregiver 是最常见的主用户**（很多患者是儿童 / 认知受影响），所以 role 行为和 cancer-buddy 略不同：

- **role=patient**：成人罕见病患者本人，第二人称"你"，挂号/入组路径以患者本人操作为准
- **role=caregiver**：父母 / 配偶 / 成年子女，第二人称"你"但任务理解为帮家人办，路径包括儿童身份证明 / 遗传咨询陪同等
- **role=family**（兄弟姐妹 / 远亲）：仅生成"如何支持主照护者找资源"的简版，不直接派 subagent 大调研

## Disclosure 行为

如果 `profile.disclosure_state.suppressed=true`（罕见病高发情况：未告诉孩子诊断 / 未告诉有遗传风险的兄弟姐妹）：
- SHORTLIST 里**避免**用强标签性表述（"成人致死性疾病"、"严重残障"等），用临床中性语
- 联系路径里**避免**指向公开患者社群 URL（暴露病种），改给"通过医生转介"类路径
- 详见 `../firefly-disclosure/` 的 disclosure 等级规则

## Output

```
patients/<patient_code>/reports/find-care/<query-slug>/
  ├── QUERY.md              # Step 1 的查询定义
  ├── SHORTLIST.md          # 最终给用户的短名单
  └── raw/
      ├── subagent-A.json
      ├── subagent-B.json
      └── ...
```

`<query-slug>` 形如 `pompe-orpha365-shanghai-2026-04`。

## Safety

- **绝不**承诺"这家中心能治好你 / 能给你确诊"
- **绝不**写 ACMG 变异分类、用药决策、剂量建议——遇到这类需求路由回 `firefly-genetic-counseling` 或主诊医生
- **绝不**鼓励通过非正规渠道（黄牛、付费内推、未注册的"诊断中介"）就诊
- **罕见病数据隐私是硬规则**：
  - subagent 调研时只用 ORPHA / disease name / HPO 这些去识别化标识，绝不传 profile.json 的患者姓名 / 出生年月 / 详细检测结果
  - SHORTLIST 输出文件本身存本地，不主动上传
  - 提示用户和外部资源沟通时也走 de-identified
- 临床试验匹配 **必须**带"匹配 ≠ 符合入组，具体以研究中心预筛为准"
- UDN / UDP-EU 等国际诊断网络的 acceptance rate 较低（UDN ~30%），不能让用户产生"申请 = 一定能确诊"的预期
- 跨境推荐时同步提示费用量级、签证、医疗记录跨境运输（可路由到 firefly-second-opinion）

## References

- [data-sources.md](references/data-sources.md) — 罕见病专属数据源清单 + 已知抓取陷阱
- [scoring-rubric.md](references/scoring-rubric.md) — 排序打分维度和权重
- [output-template.md](references/output-template.md) — SHORTLIST.md 模板
- [rd-expert-centres-cn-seed.md](references/rd-expert-centres-cn-seed.md) — 中国大陆罕见病专家中心种子库（按病种 / 协作网医院）
- 共用：`../../references/` 的 `cognitive-framework.md` / `evidence-grading.md` / `templates.md`
- 联网底层依赖：`../web-access/SKILL.md`（subagent 必须加载）
- 互补 skill：`../firefly-patient-org/SKILL.md`（已 curated 病友组织/遗传咨询师库）
