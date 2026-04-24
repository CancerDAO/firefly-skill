---
name: firefly-second-opinion
description: "跨境/跨院会诊病例包生成。UDN（美国未诊断疾病网络）/ UDP-EU / 中国罕见病诊疗协作网申请材料、英文病例包（≤2 页 PDF-ready markdown）、审稿清单、基因数据脱敏说明. Use when: 患者跑遍国内专家确诊不了想申请 UDN；要请国外专家做 second opinion；要跨省申请协作网医院会诊. Triggers on: UDN, undiagnosed disease network, UDP-EU, 跨境会诊, 国外专家, second opinion, 海外会诊, 协作网会诊, 二次诊断, 病例包."
---

# firefly-second-opinion — 跨境/跨院会诊包

为"确诊不了但想更进一步"的患者生成专业会诊材料。

## 何时调用

- 国内多家三甲确诊不了，患者考虑国外
- 要申请 UDN / UDP-EU / Centogene 等国际罕见病会诊项目
- 要跨省请中国罕见病诊疗协作网专家会诊
- 已有主诊医生但要去另一家医院 second opinion

## 协议

1. **确认目标受众**。不同项目要求不同：
   - UDN（美国）：只接受美国公民/PR，申请材料严格（详见 `references/udn-application.md`）
   - UDP-EU：欧洲未诊断项目，跨国协作
   - Centogene / Blueprint Genetics：商业临床会诊
   - 中国罕见病诊疗协作网：国内跨院，免申请费
   - 单病国际 registry（如 IRDiRC）
2. **整理病例**（读 profile.json，全部只读）：
   - HPO 术语清单（标准化的）
   - 家族史（含家系图）
   - 已做检查时间线
   - 既往诊断假设与排除依据
   - 基因检测结果（变异、ACMG 分类）
   - 当前治疗与应答
3. **生成英文病例包**（≤2 页，PDF-ready markdown）。模板见 `references/case-packet-template-en.md`。结构：
   - Header：anonymized ID, age, sex, ethnicity, date
   - Chief complaint + key clinical findings (HPO terms)
   - Family history + pedigree
   - Diagnostic workup to date (chronological)
   - Genetic findings (variants with ACMG class, gnomAD AF)
   - Current management
   - Specific consultation question（关键——问得越具体越有用）
4. **脱敏检查**：
   - 姓名 → anonymized ID
   - 出生日期精度到年
   - 地址精度到省/国
   - 亲属姓名去除
   - 机构名保留（对专家评估诊断能力有用）
   - 照片/影像：只保留临床相关部位，去除标识性信息
5. **申请材料清单**（per 项目）：表格、授权书、费用说明、时间预期。
6. **审稿建议**：让主治医生审一遍再发，避免信息冲突。

## profile.json 交互

- **Reads**: 全部只读
- **Writes**: —

## 输出格式

```markdown
# Consultation Case Packet · <anonymized_id> · <YYYY-MM-DD>

## For: <target program/institution>

## Chief Complaint
[英文主诉]

## Key Clinical Findings
- HPO Terms: HP:XXXX (label), HP:XXXX (label), ...
- Physical: ...
- Developmental: ...

## Family History
[文字描述]

Pedigree:

\`\`\`mermaid
graph TD
...
\`\`\`

## Diagnostic Workup to Date
| Date | Test | Result | Institution |

## Genetic Findings
| Gene | Variant | Zygosity | ACMG | gnomAD AF |

## Current Management
[治疗现况]

## Specific Consultation Question
> [明确、可回答的 1-2 个问题]

## Data Privacy
All identifiers removed. Raw genomic data available upon signed data use agreement.
```

（单独生成应用材料清单和脱敏说明。）

## Safety Guardrails

- **绝不**在未脱敏前发送任何包含姓名/完整生日/地址的材料
- 跨境传输基因原始数据前必须有法律授权（中国《人类遗传资源管理条例》）
- UDN 只接受美国居民，国内患者申请会被拒——要先筛目标适配性
- 商业会诊（Centogene 等）要告知患者价格范围（数千美元）
- 会诊结果回传后要更新到 profile.json，但不要替代主治医生决策
- 英文材料建议请专业翻译或双语医生校对

## References

- [references/udn-application.md](references/udn-application.md) — UDN / UDP-EU / 协作网申请流程对比
- [references/case-packet-template-en.md](references/case-packet-template-en.md) — 英文病例包完整模板 + 写作规范
