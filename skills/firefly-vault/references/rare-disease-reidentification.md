# Rare Disease Re-identification Risk

Reference for `firefly-vault` sharing decisions. Rare disease patients face elevated re-identification risk even in de-identified datasets; this file summarizes the evidence and operational defenses.

## 1. Why rare disease re-identification is high-risk

- **Small-cohort arithmetic.** A disease with prevalence <1/2000 (the EU Orphanet threshold) gives on the order of ≤700k patients worldwide and often <1000 per province in China. Any quasi-identifier combination quickly collapses cohort size to 1.
- **Disease + geography + age ≈ fingerprint.** For ultra-rare diseases (<1/50,000) the triple {diagnosis, province, age±5y} is frequently unique even before genetics are added.
- **Pedigree multiplies exposure.** Sharing proband data reveals partial genotype and disease status of all first-degree relatives; trio/quartet data implicates everyone in the pedigree at once.
- **Literature as an identifier.** Published case reports ("a 34-year-old female, Zhejiang, compound heterozygous X + Y") already reduce the anonymity set. Re-sharing the same data amplifies, not averages, the risk.
- **Irreversibility.** Unlike a leaked password, a leaked genome cannot be rotated. Public disclosure is permanent.

## 2. Empirical evidence

| Paper | Finding |
|---|---|
| **Homer et al. 2008, *PLoS Genet*** — *Resolving individuals contributing trace amounts of DNA to highly complex mixtures using high-density SNP genotyping microarrays* | Aggregate allele frequencies from as few as 1000 SNPs are sufficient to determine whether a specific individual is present in a GWAS cohort. Triggered NIH's dbGaP lockdown of summary statistics. |
| **Gymrek et al. 2013, *Science*** — *Identifying personal genomes by surname inference* | Y-STR haplotypes + public genealogy databases recovered the surnames of ~50 anonymous male 1000 Genomes participants, leading to full re-identification. |
| **El Emam et al. 2011, *JAMIA*** — *A systematic review of re-identification attacks on health data* | Meta-analysis of 14 attacks: median re-identification rate 26% in "de-identified" datasets that retained quasi-identifiers. |
| **Shabani & Borry 2018, *Eur J Hum Genet*** — *Rules for processing genetic data for research purposes in view of the new EU General Data Protection Regulation* | Articulates that one relative's consent cannot cover the shared-DNA privacy interest of family members. |
| **Erlich, Shor, Pe'er & Carmi 2018, *Science*** — *Identity inference of genomic data using long-range familial searches* | 60% of US individuals of European descent identifiable via third-cousin matches in consumer genealogy DBs (~1.3M samples). Threshold keeps dropping. |
| **Bonomi, Huang & Ohno-Machado 2020, *Nat Rev Genet*** — *Privacy challenges and research opportunities for genomic data sharing* | Comprehensive review of attack surfaces (membership, attribute, completion) and defenses. |
| **Hagestedt et al. 2020** — *MBeacon: privacy-preserving beacons for DNA methylation data* | Demonstrates that even methylation beacons leak membership, extending Homer-style attacks to epigenomic data; applies directly to rare disease methylation panels. |

## 3. K-anonymity checking (operational)

**Definition.** A dataset is k-anonymous with respect to a set of quasi-identifiers (QIs) if every combination of QI values appears in at least k records.

**Rare-disease QIs typically include:**
- Age (binned ±5y)
- Sex
- Ethnicity
- Diagnosis (ICD-10 level + Orphanet code)
- Broad region (province, not city)
- Optional: family structure (trio / singleton), consanguinity flag

**Recommended min-k:**
- Research cohort (DUA-gated): k ≥ 5
- Anonymized-for-AI feed (semi-public embeddings / summaries): k ≥ 5 plus suppression of exact diagnosis code for ultra-rare
- Full public release: k ≥ 20, phenotype-only, no raw genomic data

**Tools.**
- [ARX](https://arx.deidentifier.org/) — GUI + library, supports k / l-diversity / t-closeness, generalization hierarchies
- [sdcMicro](https://cran.r-project.org/package=sdcMicro) — R package, batch pipelines
- Custom pandas:
  ```python
  import pandas as pd
  QIs = ["age_bin", "sex", "ethnicity", "diagnosis", "province"]
  min_k = df.groupby(QIs).size().min()
  assert min_k >= 5, f"k-anonymity violated: min group size = {min_k}"
  ```
- **Beyond k-anonymity.** k alone does not prevent attribute disclosure. For sensitive attributes (specific variant, HIV status), layer l-diversity (≥ l distinct sensitive values per QI group) or t-closeness (sensitive value distribution within each group within t of the global distribution).

## 4. Family data multiplicative risk

- One proband WES ≈ 25% leakage of each sibling and 50% leakage of each parent/child at every locus (basic Mendelian transmission math).
- Trio data (proband + both parents) uniquely pins the nuclear family structure; publishing trio VCF = de facto publishing the parents' genomes at read-through coverage.
- Pedigrees with multiple affected relatives compound identifiability: `P(identifiable) ≈ 1 - ∏(1 - p_i)` across relatives where `p_i` is per-person re-id probability.
- **Third-cousin radius.** Erlich et al. 2018 showed a consumer genealogy database covering ~2% of the target population enables finding a third-cousin match for ≥60% of that population. For Chinese cohorts the equivalent databases are growing (23魔方, WeGene); patient should assume this threshold will be crossed within ~5 years.
- **Defense.**
  - Never share trio raw data at Anonymized-for-AI level.
  - Require explicit per-relative consent (not just proband consent) for Authorized level when pedigree info is included.
  - Prefer phenotype abstractions over raw variants when discussing family history.
  - For rare-disease pedigrees in literature, redact family structure diagrams before sharing downstream — a pedigree with 2 affected siblings + consanguinity flag is often uniquely identifying within a province.

## 5. Operational sharing guidance by level

- **🔒 Private** — Local only. No network egress. `diagnostics/genetics/raw/**` defaults here and requires a second confirmation to change. No anonymization needed because no sharing occurs.
- **🔑 Authorized** — Named recipient, signed Data Use Agreement (DUA), path-minimal (only the subset relevant to the consultation), time-boxed (`expires_at`), and logged in `access-log.ndjson`. No blanket "all data" grants.
- **📊 Anonymized-for-AI** — QI generalization (year-only birth, province-only geography), k ≥ 5 check against the recipient's cohort if known, strip free-text fields that may contain incidental identifiers, redact literature quotes that could act as fingerprints.
- **🌐 Public** — k ≥ 20 over phenotype + treatment response only. No raw sequence. No imaging with embedded metadata. Explicit research-use declaration. One-way door: flag `irrevocable: true` in `sharing-settings.json` to force a final confirmation.

## 6. Mitigation techniques

- **Differential privacy** — Add calibrated noise to aggregate statistics (counts, allele frequencies). Budget ε tightly for rare-disease counts; naive ε=1 on a cohort of 3 gives near-zero privacy.
- **Federated learning** — Data never leaves the vault; only gradient/weight updates cross the boundary. Appropriate for multi-institution rare disease ML.
- **Trusted execution environments (TEE)** — Intel SGX / AMD SEV / AWS Nitro Enclaves run analysis code against sealed data; the code is attested and the data is never exposed in plaintext to the operator.
- **Selective disclosure via DID** — On-chain decentralized identifier with verifiable credentials and zero-knowledge proofs. Patient proves "I have diagnosis X" without revealing which patient they are. Integrates with CancerDAO BioLinkX contribution attestation.

## 7. Chinese legal context

- **《人类遗传资源管理条例》(2019)** — Any research collecting/using/storing Chinese human genetic resources requires 遗传办 (HGRAC) filing or approval. Cross-border transfer (including sending FASTQ/VCF to an overseas collaborator or cloud) requires separate approval and may be denied.
- **《个人信息保护法》(PIPL, 2021)** — Treats health and genetic data as 敏感个人信息 (sensitive personal information), requires explicit separate consent, and mandates a PIPIA (个人信息保护影响评估) before processing.
- **《数据安全法》(2021)** + **数据出境安全评估办法 (2022)** — Cross-border transfer of important data or large-volume personal information requires CAC Security Assessment (网信办安全评估). Genetic data almost always qualifies.
- **Research project filing** — 遗传办 requires registration of each research project using HGR, not a blanket lab license. Re-sharing data for a new project means a new filing.

## 8. Patient-facing key warnings (simple language)

- **即使去掉名字，你罕见的病 + 你住的省 + 你的年龄，在数据集里可能只有你一个人。** "Anonymous" is not anonymous for rare disease.
- **家系数据共享 = 暴露整个家族。** 你分享 trio 数据时，你父母的遗传信息也跟着出去了 — 他们没有按同意书。
- **一旦公开不可撤回。** 基因组不能改密码。发出去之前多停三分钟。
- **病例报告就是身份证。** 学术文献里一句"34 岁女性，浙江，XXX 基因复合杂合"，搜索引擎能找回你。
- **认可方（🔑）不等于放心方。** 医生也可能换邮箱、丢U盘、被钓鱼。最小必要路径 + 过期时间是标配，不是客气。

## 9. Pre-share checklist (companion self-check)

Before promoting any path from 🔒 → 🔑/📊/🌐, `firefly-vault` must walk this checklist and log the outcome in `access-log.ndjson`:

1. **Disease prevalence known?** If <1/10,000, force recipient confirmation and raise min_k by 5.
2. **Family data included?** If yes, are all implicated relatives consented? If not, block and reduce path scope.
3. **QI combination uniqueness.** Run k-anonymity check against a reference cohort if available (ClinVar submitters, registry peers); else use self-cohort of size 1 = k=1 → fail.
4. **Free-text scan.** `notes/` and `treatments/*/response.md` may contain incidental identifiers ("my daughter's elementary school near Huaxi hospital"). Run regex + LLM scan before export.
5. **Literature cross-check.** Has this case been published? If yes, the published description is already the identifier — sharing more here doesn't reduce risk, just accelerates attribution.
6. **Reversibility test.** Ask the user out loud: "If this became fully public tomorrow, could you live with it?" If no, do not promote to 🌐.
7. **Legal gate.** If data leaves mainland China, verify HGRAC filing + CAC Security Assessment status. Default to blocking if either is missing.

## 10. Further reading

- GA4GH Regulatory and Ethics Work Stream — *Framework for Responsible Sharing of Genomic and Health-Related Data* (2019 revision)
- NIH Genomic Data Sharing Policy (updated 2023)
- 《涉及人的生命科学和医学研究伦理审查办法》(2023, 国家卫健委)
- Global Alliance for Genomics and Health — *Data Use Ontology* (DUO) for encoding consent constraints as machine-readable tags
