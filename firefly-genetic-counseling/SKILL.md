---
name: firefly-genetic-counseling
description: "遗传咨询：ACMG 变异分类解读 → 遗传模式判定（AD/AR/XL/线粒体/新生突变）→ 家族成员风险评估 → PGT-M 可行性 → 生育指导 → 建议检测亲属名单。罕见病绕不开的核心能力. Use when: 患者有基因检测报告不会读（VUS、可能致病等术语）；备孕/妊娠夫妻发现携带者状态；有家族史但不知如何评估亲属风险；要做胚胎植入前诊断 PGT-M. Triggers on: ACMG, VUS, 变异分类, 遗传模式, 常染色体显性, 隐性, X 连锁, PGT-M, 胚胎植入前诊断, 产前诊断, 携带者筛查, 家族史评估, 遗传咨询."
---

# firefly-genetic-counseling — 遗传咨询

把基因检测报告转化为家庭能用的信息——对我意味着什么？对家人意味着什么？对下一代意味着什么？

## 何时调用

- 患者有 WES/panel 报告但看不懂（"VUS 是什么？"）
- 医生说"这是隐性遗传病"，家属不懂含义
- 备孕夫妻发现一方或双方携带者
- 已确诊患者问"我弟弟要不要查？我女儿要不要查？"
- 要做 PGT-M / 产前诊断

## 协议

1. **读 profile.json** 的 `genetic_findings` + `family_history`。如果没有基因数据，先引导患者去调 firefly-organize 做 OCR 结构化。

2. **ACMG 变异分类解读**。对每个变异：
   - 说明 ACMG 类别（致病 P / 可能致病 LP / 临床意义不明 VUS / 可能良性 LB / 良性 B）
   - 列出支持证据代码（PVS1 / PM2 / PP3 等）的含义
   - gnomAD 人群频率解读
   - ClinVar 当前分类 + 是否存在冲突
   - **VUS 处理**：不因"不确定"就决定行动，但不忽略——建议 3-5 年后复查分类更新
   - 详见 `references/acmg-classification.md`

3. **遗传模式判定**：
   - 常染色体显性（AD）：子代 50% 风险，外显率需评估
   - 常染色体隐性（AR）：两个携带者配对 25% 患病、50% 携带者
   - X 连锁（XL）：男女差异大，母亲携带者风险计算
   - 线粒体（mt）：母系遗传，异质性
   - 新生突变（de novo）：再发风险极低（种系嵌合除外）
   - 详见 `references/inheritance-patterns.md`

4. **家族成员风险评估**：
   - 绘制家系图（Mermaid pedigree）
   - 标注已知患者、已知携带者、未检测亲属
   - 按遗传模式计算每个亲属的先验风险
   - 给出建议检测名单（按亲缘远近、性别、年龄分层）

5. **PGT-M 可行性**（如夫妻备孕/已孕）：
   - 确认变异是否可定位（需要 trio/informative 位点）
   - 评估中国上市的 PGT-M 机构（详见 `references/pgt-protocol.md`）
   - 费用范围 + 医保情况 + 时间周期
   - 成功率评估（受年龄、卵子数量影响）
   - 替代方案：产前诊断（绒毛膜 CVS 11-13w / 羊水 16-20w）

6. **生育指导**（按具体情境）：
   - 双方均非携带者：正常妊娠 + 常规产检
   - 一方携带者：对方筛查 + 正常妊娠
   - 双方携带者（AR）：PGT-M / 产前诊断 / 接受风险 / 不生育 / 供精供卵——平等呈现，不替代决定
   - 已患者生育：每胎按遗传模式评估

## profile.json 交互

- **Reads**: `clinical`, `family_history`, `genetic_findings`
- **Writes**: `genetic_findings`（补全 ACMG 细节）、`family_history`（补全 pedigree_notes、affected_relatives）

## 输出格式

```markdown
# 遗传咨询报告 · <患者名> · <日期>

## 变异解读
### 变异 1: <基因> <HGVS>
- ACMG 类别：<P/LP/VUS/LB/B>
- 证据代码：<PVS1, PM2, ...>
- gnomAD 频率：<AF>
- ClinVar 分类：<分类>（如有冲突标注）
- 通俗解释：<2-3 句>

## 遗传模式
<AD/AR/XL/mt/de novo>

## 家系图

\`\`\`mermaid
graph TD
...
\`\`\`

## 家族成员风险
| 亲属 | 关系 | 先验风险 | 建议检测 | 优先级 |
|---|---|---|---|---|
| 弟弟 | 同胞 | 25% 患病、50% 携带者 | 目标变异位点 | 🔴高 |

## 生育选项（如适用）
### 选项 A: 自然妊娠 + 产前诊断
- 风险：...
- 时间：...
- 费用：...

### 选项 B: PGT-M
...

（不做推荐，只呈现 pros/cons）

## 下一步
1. [ ] 建议 <亲属> 检测 <变异>
2. [ ] 联系 <医院> 遗传咨询门诊进一步评估
3. [ ] 如备孕：预约 PGT-M 咨询 / 产前诊断
```

## Safety Guardrails

- **不替患者决定生育**——呈现所有选项（包括不生育、收养、供精供卵）
- VUS 不当致病——明确"目前证据不足以判断"
- 家族成员检测涉及告知义务——与 firefly-disclosure 配合
- 中国 PGT-M 合法性有限制（需高风险指征 + 医学伦理委员会批准）——不误导患者可以无条件选择
- 跨国/跨代数据涉及 GINA（美国）/中国人类遗传资源管理等法规，跨境传输需合规

## References

- [references/acmg-classification.md](references/acmg-classification.md) — ACMG 2015 变异分类标准 + 证据代码详解
- [references/inheritance-patterns.md](references/inheritance-patterns.md) — 5 种遗传模式风险计算
- [references/pgt-protocol.md](references/pgt-protocol.md) — 中国 PGT-M 上市机构、流程、费用、成功率
- `../firefly/references/evidence-grading.md` — 证据分级（共享）
