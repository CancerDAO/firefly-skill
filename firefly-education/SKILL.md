---
name: firefly-education
description: "生成罕见病患者教育手册。读 profile.json 选章节 → Markdown 手册（快速卡片/用药要点/生活指南/随访日历/费用导航/FAQ）→ Mermaid 疾病机制图 + 治疗决策树. Use when: 已确诊患者想要一份个性化患教材料；患儿家长需要长期照护参考手册；二线医生需要快速理解某个罕见病就诊要点. Triggers on: 患教手册, 罕见病科普, 病情解释, 用药指南, 个性化手册, education handbook, patient handbook."
---

# firefly-education — 患者教育手册

把临床信息转化为患者/家属能读懂的 Markdown 手册。不是百度百科，是 N=1 定制。

## 何时调用

- 已确诊（profile.json `confirmed_diagnosis` 存在）
- 用户说"帮我写个科普"、"给我出份患教材料"、"我要一份长期参考"
- firefly-organize / firefly-genetic-counseling 完成后，主动提示"要不要生成一份患教手册？"

## 协议

1. **读 profile.json**。提取疾病名、当前治疗、合并症、关注点。
2. **选章节**。从标准模板（`references/education-template.md`）按病种 / 阶段 / 患者角色（患者/家属）筛选。
3. **生成章节**（按优先级）：
   - **快速参考卡**（封面，打印携带）：急诊信息、主治医生、过敏、当前用药、紧急联系人
   - **My Health Summary**：1 段话解释"我得的是什么病"（基于 profile.json）
   - **疾病机制图**（Mermaid flowchart）：基因 → 蛋白 → 细胞 → 器官 → 症状
   - **当前用药**：每种药的作用、剂量、最佳服用时间、副作用管理、错过怎么办
   - **日常生活指南**：饮食（调 firefly-diet 的输出）、运动、睡眠、旅行准备
   - **随访日历**：未来 12 个月的检查时间轴 + 每项检查意义
   - **费用与援助**：医保报销路径、PAP、慈善援助（查 `firefly/references/resources-cn.md`）
   - **治疗决策树**（Mermaid）：如果 X 检查阳性 → Y 方案；如果 X 阴性 → Z 方案
   - **FAQ**：基于常见问题 + 本患者的具体疑问
   - **紧急情况**：什么算急症 / 去哪家医院 / 带什么资料
4. **合并文件**。单一 Markdown 输出到 `~/patient-education/{patient_id}_{YYYY-MM-DD}_罕见病患教手册.md`。
5. **可打印性检查**：去除 emoji 以外的花哨格式，确保 A4 打印可读。

## profile.json 交互

- **Reads**: 全部只读
- **Writes**: —

## 输出格式

单文件 Markdown，结构：

```markdown
# <患者名> 的罕见病手册
**更新：<日期>** | **版本：<编号>**

---

## 🚨 快速参考卡
[单页，打印携带]

## 📖 我得的是什么病
[1 段解释]

## 🧬 疾病机制
[Mermaid flowchart]

## 💊 我的用药
[表格]

## 🍽️ 饮食与生活
[指南]

## 📅 随访日历
[时间轴]

## 💰 费用与援助
[列表]

## 🌳 治疗决策树
[Mermaid]

## ❓ FAQ
[问答]

## 🆘 紧急情况
[应急卡]
```

## Safety Guardrails

- 不编造病种信息——每个医学主张都标注来源（Orphanet / OMIM / 指南 / 论文）
- 药物剂量**不**给出具体数字，只说"按医生处方服用"（剂量错了会出事）
- 治疗决策树只是"一般路径"，明确写"具体决策与主治医生商议"
- 避免给出预后预测（"还能活多久"这类问题一律转回医生）
- 多语言版本暂不做（YAGNI）

## References

- [references/education-template.md](references/education-template.md) — 手册完整章节模板库
