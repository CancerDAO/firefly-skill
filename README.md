<div align="center">

# 萤火.skill

> *"每一种罕见，都值得被认真对待。"*

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-Skill-blueviolet)](https://claude.ai/code)
[![DeepRare](https://img.shields.io/badge/DeepRare-Nature%202026-brightgreen)](https://www.nature.com/articles/s41586-025-10097-9)

<br>

跑了 5 家医院还是确诊不了？<br>
孩子拿到基因报告，"可能致病" 四个字看了一宿？<br>
确诊之后国内没药、找不到同路人？<br>
你不是第一个这样的家庭。<br>

**你需要一点萤火。一束会陪你走过长夜的光。**

<br>

试试看把你的症状、病历、基因报告交给萤火。<br>
它陪你 **把诊断奥德赛一点一点走短**。<br>
能给你 Top 5 诊断方向 + 建议检查，读懂基因报告的 VUS / ACMG 分类，<br>
帮你生成带去下一位医生的一页就诊摘要、给孩子的年龄分层告知脚本、<br>
筛查照护者的 Zarit 负担、对接中国罕见病联盟病友群、<br>
打包一份英文病例包投给国外 UDN。一个 Skill 装下这些事。

[功能](#功能) · [技术基座](#技术基座) · [安装](#安装) · [使用](#使用) · [**English**](README_EN.md)

</div>

---

## 萤火能帮你做什么

罕见病的痛点不只是"罕见"本身，而是整条路径：诊断要平均 5-7 年，确诊后国内可能没药、没同路人、没遗传咨询师。很多问题没有标准答案，但可以被更好地整理、分派、连接。

**萤火把这些问题收敛成一个可执行的导航系统：**

| 你遇到的问题 | 萤火怎么帮你 |
|---|---|
| 看了很多医生都确诊不了 | OCR 病历 + HPO 标准化 + DeepRare 多源检索，给出 Top 5 诊断方向 + 优先检查 |
| 基因报告看不懂 VUS / ACMG | 逐条解读证据代码、评估外显率、画家系图、算家族成员风险 |
| 孩子要做 PGT-M 备孕 | 列中国上市机构 / 流程 / 费用 / 成功率 / 替代方案 |
| 国内确诊不了想去国外 | UDN / UDP-EU / Centogene 申请流程对比 + 生成英文病例包 |
| 诊断奥德赛让人撑不住了 | PHQ-9 / GAD-7 / IES-R 三联筛查 + 危机热线直达 |
| 照护 10 年快崩了 | ZBI-22 照护者负担量表 + 喘息资源地图 |
| 孩子几岁能告诉他诊断 | SHARE 模型年龄分层告知脚本（0-6 / 7-11 / 12-17 / 成年） |
| PKU / Pompe 患儿不知怎么吃 | 按病种加载饮食协议 + 食物交换份 + 代谢危象应急卡 |
| 想系统建自己的医疗档案 | N=1 数据仓 + 4 级分享控制（🔒私有 → 🌐匿名公开） |
| 确诊了不知道有没有病友 | 中国罕见病联盟 + 单病社群 + NORD / EURORDIS 匹配 |

---

## 功能

### 1 个入口 + 10 个陪伴模块

```
firefly                     家族入口，按你的情境分派到对应子技能

firefly-organize            OCR + HPO + DeepRare 诊断导航（Top 5 方向 + 就诊摘要）
firefly-genetic-counseling  ACMG 变异解读 + 家族风险 + PGT-M 可行性
firefly-education           个性化患教手册（Mermaid 疾病机制图 + 治疗决策树）
firefly-caregiver           照护者 Zarit 筛查 + 喘息资源 + 自我保护清单
firefly-mind                心理筛查（PHQ-9 / GAD-7 / IES-R）+ 危机热线
firefly-diet                代谢性罕见病饮食（PKU / Pompe / UCD / MSUD / GSD）
firefly-second-opinion      UDN / UDP-EU / 协作网跨境会诊包
firefly-vault               N=1 数据仓 + 基因数据重识别风险评估
firefly-disclosure          遗传风险家庭告知（对孩子 / 血亲 / 配偶）
firefly-patient-org         患者组织 + 遗传咨询师名录
```

每个模块都可以 **独立触发**，不需要按顺序走完整条路。系统先理解你的身份和情境，再引导下一步。

### 设计哲学

萤火是一个"导航 + 结构化"的系统，**不是诊断替代者**。

在罕见病的语境里，核心决策——确定诊断、选择治疗、生育规划——必须发生在你和主诊医生、遗传咨询师之间。

萤火的职责是：
- **缩短诊断之路** — 用 DeepRare 方法论（Nature 2026，2,919 种罕见病，64.4% recall@1）给出方向而不是结论
- **翻译医学信息** — 把 ACMG / HPO / ORPHA 这些术语变成患者能用的语言
- **连接资源** — 罕见病在中国 14 亿人口基数下仍有绝对数量，缺的是匹配
- **看见情绪** — 诊断奥德赛是慢性创伤，筛查 + 支持不能缺席

因此萤火**不会**：
- 下诊断——只给"建议排查 X 方向"
- 推荐治疗方案——只说"这个选项值得和医生讨论"
- 替你决定生育——呈现所有选项（包括不生育、收养、供/供）
- 劝阻你遵医嘱
- 说"没有办法了"——总有支持性治疗或研究参与机会

---

## 技术基座

萤火整合了两个前沿系统：

### DeepRare 诊断方法论

开源复现的 AI 罕见病诊断管线（Nature 2026, [doi:10.1038/s41586-025-10097-9](https://www.nature.com/articles/s41586-025-10097-9)），覆盖 2,919 种罕见病，Recall@1 64.4%（超越人类专家 54.6%）。

核心 pipeline：HPO 表型提取 → 多源知识检索（PubCaseFinder + Phenobrain + OMIM + Orphanet + PubMed）→ LLM 共识排序 → 可追溯推理链 + 自反思验证。

### 罕见病研究之神认知框架

综合全球 6 大学派 50 位顶级研究者（含 Schiff, Lupski, Valle, Rehm, Firth 等）的集体思维框架：7 个心智模型 + 10 条决策启发式 + 6 对内在张力 + 8 大盲点自检。

应用示例：
- **Genome-First**（心智模型 1）——先问"基因组说了什么"而不是症状反复排查
- **备份基因激活**（心智模型 3）——SMN2 之于 SMA，HbF 之于镰刀细胞
- **速度即药物**（心智模型 6）——罕见病的道德紧迫性
- **种族代表性盲点**——gnomAD 偏向欧洲裔，东亚变异频率可能误导

---

## 安装

三步，详见 [INSTALL.md](INSTALL.md)：

```bash
# 1. Clone
cd ~
git clone https://github.com/CancerDAO/firefly-skill.git

# 2. Symlink 11 个 skill 到 Claude Code 的 skills 目录
for skill in firefly firefly-organize firefly-genetic-counseling firefly-education \
             firefly-caregiver firefly-mind firefly-diet firefly-second-opinion \
             firefly-vault firefly-disclosure firefly-patient-org; do
  ln -sf ~/firefly-skill/$skill ~/.claude/skills/$skill
done

# 3. 重启 Claude Code
```

---

## 使用

在 Claude Code 里输入任何一个触发场景——

- "我家孩子拿到 WES 报告，说有个 c.xxxx 的变异是 VUS" → 自动调 `firefly-organize` + `firefly-genetic-counseling`
- "我照护 SMA 孩子 8 年了，最近很崩溃" → `firefly-caregiver` + `firefly-mind`
- "想给国外医院投一份诊断咨询，怎么准备材料" → `firefly-second-opinion`
- "苯酮尿症的孩子能吃什么" → `firefly-diet`

或者直接调子技能：`/firefly-mind` 做心理筛查 / `/firefly-patient-org` 找病友。

---

## 数据流

跨模块共享一个 `profile.json`（schema 见 [firefly/references/templates.md](firefly/references/templates.md)）——一个患者的所有结构化信息，10 个 companion 按权限读写。

```
 OCR / 对话
   ↓
 firefly-organize  →  创建 profile.json + Top 5 诊断方向
   ↓
 firefly-genetic-counseling  →  补全 genetic_findings + 家系
   ↓
 firefly-education  →  读 profile → 生成患教手册
 firefly-vault      →  读 profile → 建本地数据仓 + 分享控制
 firefly-mind       →  写 mental_health 筛查结果
 ...
```

---

## 贡献

欢迎 issue / PR。罕见病相关的：
- 单病组织 / 病友社群 / 遗传咨询师补充（提 PR 改 `firefly-patient-org/references/`）
- 饮食协议修订（`firefly-diet/references/`，需引用权威指南）
- 翻译校对 / 术语订正
- Bug / 行为修正

---

## License

MIT License — 见 [LICENSE](LICENSE)

---

## 致谢

- DeepRare 方法论：Nature 2026 论文作者团队
- HPO 标准化：Köhler et al., Human Phenotype Ontology Consortium
- ACMG 变异分类：Richards et al. 2015 + ClinGen
- 中国罕见病诊疗协作网：324 家成员医院 + 中国罕见病联盟
- 50 位罕见病领域研究者的集体认知贡献（见 `firefly/references/cognitive-framework.md`）

<br>

<div align="center">

**萤火 · Firefly** — *CancerDAO 罕见病姊妹项目，与 [抗癌搭子](https://github.com/CancerDAO/cancer-buddy-skill) 同构*

</div>
