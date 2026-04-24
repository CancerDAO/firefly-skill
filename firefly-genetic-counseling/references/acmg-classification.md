# ACMG 变异致病性分类参考

## 1. 概述

ACMG (American College of Medical Genetics and Genomics) / AMP (Association for Molecular Pathology) 2015 年联合发布的《Standards and guidelines for the interpretation of sequence variants》(Richards et al., *Genet Med* 17(5):405-424) 是目前国际主流的胚系变异致病性分类标准，广泛应用于临床基因检测报告。

**五级分类系统**：
- **Pathogenic (P)** — 致病
- **Likely Pathogenic (LP)** — 可能致病（致病性 ≥90%）
- **Variant of Uncertain Significance (VUS)** — 意义未明
- **Likely Benign (LB)** — 可能良性
- **Benign (B)** — 良性

ACMG 2015 发布后，ClinGen (Clinical Genome Resource) 下设的 SVI (Sequence Variant Interpretation) 工作组及各基因特异的 VCEP (Variant Curation Expert Panel) 对原始标准做了多轮精细化修订。具体基因（BRCA1/2、TP53、CDH1、MYH7、PTEN、RASopathy 等）有专门的 VCEP specifications，应优先采用。

## 2. 证据代码完整清单

### 2.1 致病方向证据（Pathogenic，从强到弱）

#### Very Strong（非常强）
- **PVS1** — Null variant (无功能变异)：nonsense、frameshift、canonical ±1/2 splice site、initiation codon、single-exon 或 multi-exon deletion，出现在**已知 loss-of-function (LoF) 是致病机制**的基因中。
  - 关键前置条件：该基因的 LoF 是已建立的致病机制（如 BRCA1/2、NF1、DMD）。对于功能获得型致病机制（如 HRAS、FGFR3 achondroplasia），PVS1 不适用。

#### Strong（强）
- **PS1** — 与已知致病变异导致相同氨基酸改变（不同核苷酸改变但同一蛋白产物）
- **PS2** — 患者中确认的 de novo（父母亲缘关系经分子验证），家族史阴性
- **PS3** — 体外或体内功能研究充分支持有害效应（需 well-established 的实验体系）
- **PS4** — 在患病队列中的频率显著高于对照（通常要求 OR 明显 >5，95% CI 不跨 1）

#### Moderate（中等）
- **PM1** — 位于 mutational hot spot 或已建立的关键功能域（无良性变异的结构域）
- **PM2** — 在 gnomAD 等对照数据库中缺失或极低频（按 ClinGen SVI 2020 refinement 降级为 Supporting）
- **PM3** — 隐性遗传病中，与另一已知致病变异**在 trans 位置**（需家系验证或 phasing）
- **PM4** — 蛋白长度改变：non-repeat 区域的 in-frame indel 或 stop loss
- **PM5** — 同一密码子的新 missense 改变，该位点已知另一 missense 为致病
- **PM6** — 推测 de novo，但未经亲缘关系确认

#### Supporting（支持）
- **PP1** — 家系内多个受累成员共分离（cosegregation）
- **PP2** — Missense 变异位于 missense 是主要致病机制、且良性 missense 率低的基因
- **PP3** — 多种计算工具一致预测有害（按 ClinGen 建议使用 REVEL/CADD/SpliceAI 等具体阈值）
- **PP4** — 患者表型 / 家族史高度特异于单一遗传病因
- **PP5** — Reputable source 报告为致病 **（2018 年 ClinGen SVI 已废除，不再使用）**

### 2.2 良性方向证据（Benign）

#### Stand-alone（独立）
- **BA1** — 人群数据库等位基因频率 >5%（单独即可判定良性，但 ClinGen 针对某些高频致病变异如 HFE c.845G>A 设置例外）

#### Strong（强）
- **BS1** — 等位基因频率明显高于该疾病预期（按流行率公式计算阈值）
- **BS2** — 在早发全外显疾病预期的健康成年人中观察到（纯合/半合子形式）
- **BS3** — 良好的功能研究显示无有害效应
- **BS4** — 家系内受累成员中缺乏共分离（lack of segregation）

#### Supporting（支持）
- **BP1** — 在只有 truncating 变异致病的基因中出现的 missense 变异
- **BP2** — 完全外显显性遗传病中与致病变异 in trans，或与致病变异 in cis
- **BP3** — 重复区域内无已知功能的 in-frame indel
- **BP4** — 多种计算工具一致预测无有害效应
- **BP5** — 患者已有其他分子病因（alternate molecular basis）
- **BP6** — Reputable source 报告为良性 **（2018 年已废除）**
- **BP7** — 同义变异（synonymous），无剪接影响预测，且核苷酸非高度保守

## 3. 组合规则（Combination Rules）

### 3.1 Pathogenic 判定（满足任一组合）
- 1 PVS1 + ≥1 PS
- 1 PVS1 + ≥2 PM
- 1 PVS1 + 1 PM + 1 PP
- 1 PVS1 + ≥2 PP
- ≥2 PS
- 1 PS + ≥3 PM
- 1 PS + 2 PM + ≥2 PP
- 1 PS + 1 PM + ≥4 PP
- ≥3 PM
- 2 PM + ≥2 PP
- 1 PM + ≥4 PP

### 3.2 Likely Pathogenic 判定
- 1 PVS1 + 1 PM
- 1 PS + 1-2 PM
- 1 PS + ≥2 PP
- ≥2 PM
- 1 PM + ≥2 PP
- ≥4 PP

### 3.3 Benign 判定
- 1 BA1（独立）
- ≥2 BS

### 3.4 Likely Benign 判定
- 1 BS + 1 BP
- ≥2 BP

### 3.5 VUS 判定
- 不满足上述任何致病或良性组合
- 或致病与良性证据并存产生冲突（contradictory）

### 3.6 贝叶斯框架（Tavtigian et al. 2018）
ClinGen 将 ACMG 规则重构为贝叶斯点值系统（Bayesian framework），每级证据对应分数：
- Supporting = 1 分 / Moderate = 2 分 / Strong = 4 分 / Very Strong = 8 分
- Pathogenic ≥10 分；Likely Pathogenic 6-9 分；VUS 0-5 分；Likely Benign -1 至 -6 分；Benign ≤-7 分
- 该框架允许证据强度灵活升降（如 PM2 降为 Supporting、PP3 升为 Moderate）

## 4. ClinGen 重要修订

### 4.1 PVS1 决策树（Abou Tayoun et al. 2018, *Hum Mutat*）
针对不同 LoF 机制对 PVS1 进行细化：
- Nonsense/frameshift → 是否触发 NMD (nonsense-mediated decay)？
- 受影响外显子是否 biologically relevant？
- 删除是否保留 reading frame？
- Splice site ±1/2 → 是否产生 cryptic splice？
根据决策路径，PVS1 可降级为 Strong / Moderate / Supporting。

### 4.2 PP3/BP4 计算证据阈值（Pejaver et al. 2022, *AJHG*）
推荐使用 REVEL 阈值：
- REVEL ≥0.773 → PP3_Moderate
- REVEL ≥0.932 → PP3_Strong
- REVEL ≤0.016 → BP4_Strong
- 剪接：SpliceAI ≥0.2 → Supporting；≥0.5 → Moderate

### 4.3 PS1/PM5 codon-level 考量
同一密码子不同核苷酸变化若产生相同氨基酸 → PS1；产生不同但同样有害的氨基酸 → PM5。需考虑该密码子是否跨剪接位点（可能同时影响 splicing）。

### 4.4 PM2 降级
ClinGen SVI 2020 建议 PM2 在多数情况下降为 Supporting（PM2_Supporting），因为"罕见"本身证据较弱。

### 4.5 VCEP 基因特化
- **BRCA1/ENIGMA VCEP**：对 BRCA1/2 的剪接、missense、VUS 再分类有专门规则
- **TP53 VCEP**：PS3 功能证据阈值基于 Giacomelli et al. 2018 saturation mutagenesis
- **CDH1 VCEP**：PVS1 应用受限（hereditary diffuse gastric cancer）
- **MYH7 VCEP**：missense 证据权重高（HCM）
- **RASopathy VCEP**：PP2/BP1 的 gene list 明确

## 5. VUS 处理原则

- **不按致病处理**：不因 VUS 建议预防性手术、PGT-M、家系级联检测
- **不按良性处理**：保留监测通道
- **不作为临床决策单一依据**：需结合表型、家族史
- **定期重评**：建议每 2-3 年请检测实验室重新分类
- **ClinVar 监测**：关注 ClinVar 中该变异的分类动态与冲突（conflicting interpretations 在 ClinVar 约占 17%，需仔细判读）
- **功能研究投入**：对家族中反复出现的 VUS 可建议 RNA 研究、细胞功能实验、家系共分离分析以推动再分类
- **RNA 研究**：对疑似剪接影响的 VUS，血源 RNA + minigene assay 是推动再分类的主要手段
- **家系级联 VUS 检测的雷区**：VUS 在亲属中阳性不能作为"遗传学证据"，避免告诉亲属"你也有这个变异所以风险升高"
- **患者沟通**：明确使用"现有证据不足以判定"而非"可能有问题"，避免过度焦虑或虚假安心

## 6. gnomAD 频率解读

### 6.1 数据库版本
- **gnomAD v2.1**：exomes ~125,748；genomes ~15,708（GRCh37）
- **gnomAD v3.1**：genomes ~76,156（GRCh38）
- **gnomAD v4**：截至本文整合 exomes + genomes 超过 80 万样本（GRCh38）

### 6.2 频率阈值
- **PM2**：多数罕见孟德尔病，allele frequency < 1/10,000（ClinGen 建议 AF < 0.0001；对极罕见病可放宽到"absent"）
- **BA1**：AF >5%（>0.05）独立良性（特定致病变异例外，如 HFE、GJB2 c.109G>A 等 ClinGen 白名单）
- **BS1**：阈值为疾病特异。公式：BS1 阈值 = 疾病最大可信等位基因频率（由流行率、penetrance、遗传模式、等位基因异质性决定，参考 Whiffin et al. 2017）

### 6.3 亚群频率
必须查看 population-specific AF：
- 某变异在 gnomAD total 中罕见，但在特定亚群（East Asian、Ashkenazi Jewish 等）富集 → 谨慎应用 PM2
- 奠基者效应变异（founder mutation）在亚群中频率高但仍致病

## 7. 中国临床解读注意

- ACMG 是美国指南，中华医学会医学遗传学分会 (CSMG) 于 2017 年发布《遗传变异分类标准与指南》，基本与 ACMG 2015 一致，加入中国人群数据库参考建议
- 中国人群数据库：
  - **ChinaMAP**：中国代谢解析计划，~10,000 样本，WGS
  - **WBBC**：Westlake BioBank for Chinese，覆盖长江流域、珠江流域人群
  - **NyuWa**：中国科学院发布，~3,000 WGS
  - **PGG**：Population Genetics & Genomics database（中科院上海营养所）
  - **GenomeAsia 100K**：包含中国多族群
- gnomAD East Asian (EAS) 子集可部分代表汉族人群，但建议结合中国本土数据库尤其对中国特异罕见变异
- 实验室报告常见问题：
  - VUS 被过度声称为 LP（尤其是携带 PP3 单证据时）
  - LP 与 P 混淆报告，临床处置需按实际证据强度
  - 忽略 BA1/BS1 例外清单（如 HFE c.845G>A / p.Cys282Tyr、GJB2 c.109G>A 等）
  - 未标注使用的 ACMG 修订版本（2015 原版 vs ClinGen VCEP 特化）
  - PM2 未按 ClinGen 2020 降级为 Supporting
- **HBOC 场景**：BRCA1/2 变异分类应优先采用 ENIGMA VCEP 规则
- **儿童神经发育障碍场景**：全外显子/基因组（WES/WGS）结合 trio 测序显著增强 de novo 判定（PS2 应用）
- **肿瘤 panel 场景**：注意区分胚系 vs 体细胞，体细胞变异用 AMP/ASCO/CAP 2017 四级分类，不直接套用 ACMG

## 8. 常见误用避免

1. **不用 PP5 / BP6**：2018 年已废除，直接查原始证据而非引用他人报告
2. **PP3 单独不充分支持 P**：多工具一致仅为 supporting，不能作为主要证据
3. **PP1 家系共分离证据门槛**：通常要求 ≥3 受累成员 + 2 代以上才够 PP1_Strong
4. **de novo 证据要分级**：
   - PS2：亲缘关系经分子验证（SNP/STR 指纹）
   - PM6：未验证亲缘，仅依赖家族口述
5. **PS3/BS3 功能研究**：需评估实验体系是否 well-established，是否代表体内相关机制
6. **PM3 必须确认 in trans**：父母 phasing 或家系验证，不能仅凭"检测到两个变异"即判
7. **PVS1 基因特异性**：必须确认 LoF 为该基因的致病机制，不是所有基因都适用
8. **复合证据不双重计分**：同一数据不可同时用于多条证据（如同一计算工具不能同时支持 PP3 多次）
9. **剪接变异不默认 PVS1**：仅 canonical ±1/2 适用；深内含子或外显子内 synonymous 影响剪接的变异需走 PS3（RNA 实验）或 PP3（SpliceAI）

## 9. 引用

1. Richards S, Aziz N, Bale S, et al. (2015) Standards and guidelines for the interpretation of sequence variants: a joint consensus recommendation of the American College of Medical Genetics and Genomics and the Association for Molecular Pathology. *Genet Med* 17(5):405-424.
2. Biesecker LG, Harrison SM, ClinGen Sequence Variant Interpretation Working Group. (2018) The ACMG/AMP reputable source criteria for the interpretation of sequence variants. *Genet Med* 20(12):1687-1688.
3. Abou Tayoun AN, Pesaran T, DiStefano MT, et al. (2018) Recommendations for interpreting the loss of function PVS1 ACMG/AMP variant criterion. *Hum Mutat* 39(11):1517-1524.
4. Tavtigian SV, Greenblatt MS, Harrison SM, et al. (2018) Modeling the ACMG/AMP variant classification guidelines as a Bayesian classification framework. *Genet Med* 20(9):1054-1060.
5. Pejaver V, Byrne AB, Feng BJ, et al. (2022) Calibration of computational tools for missense variant pathogenicity classification and ClinGen recommendations for PP3/BP4 criteria. *Am J Hum Genet* 109(12):2163-2177.
6. Whiffin N, Minikel E, Walsh R, et al. (2017) Using high-resolution variant frequencies to empower clinical genome interpretation. *Genet Med* 19(10):1151-1158.
7. 中华医学会医学遗传学分会. (2017) 遗传变异分类标准与指南. *中华医学遗传学杂志* 34(5):593-601.
