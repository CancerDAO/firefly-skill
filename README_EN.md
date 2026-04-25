<div align="center">

# firefly.skill

> *"Every rare condition deserves to be taken seriously."*

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-Skill-blueviolet)](https://claude.ai/code)
[![DeepRare](https://img.shields.io/badge/DeepRare-Nature%202026-brightgreen)](https://www.nature.com/articles/s41586-025-10097-9)

<br>

Been to 5 hospitals and still no diagnosis?<br>
Got a WES report saying "variant of uncertain significance" and spent the night googling?<br>
Diagnosed, but there's no drug in your country and no one else like you?<br>
You are not the first family to walk this path.<br>

**You need a bit of firefly. A light that keeps you company through the long night.**

<br>

Hand over your symptoms, records, and gene reports to Firefly.<br>
It walks with you to **shorten the diagnostic odyssey, one step at a time**.<br>
Top 5 diagnostic directions + suggested tests. ACMG/VUS interpretation.<br>
A one-page consultation summary to hand to the next doctor. Age-stratified disclosure scripts for children.<br>
Caregiver burden screening. Matching to patient organizations. English case packets for UDN.<br>
All in one Skill.

[Features](#features) · [Technology](#technology) · [Install](#install) · [Usage](#usage) · [**中文**](README.md)

</div>

---

## What Firefly Does

The pain of rare disease is not just rarity itself — it's the whole path: 5-7 years to diagnosis on average, then possibly no approved drug in your country, no peer patients, no genetic counselors. Most questions have no standard answer, but they can be organized, routed, and connected.

**Firefly consolidates these into an actionable navigation system:**

| Your situation | What Firefly does |
|---|---|
| Multiple hospitals, still undiagnosed | OCR records + HPO standardization + DeepRare multi-source search → Top 5 directions + priority tests |
| Genetic report is incomprehensible | Evidence-code-by-evidence-code ACMG interpretation + penetrance + pedigree + family risk |
| Planning PGT-M | Chinese PGT-M institutions, workflow, cost, success rates, alternatives |
| Want international second opinion | UDN / UDP-EU / Centogene comparison + English case packet generation |
| Odyssey-induced distress | PHQ-9 / GAD-7 / IES-R triple screening + direct crisis hotlines |
| Caregiver burnout | ZBI-22 + respite resource map + self-protection checklist |
| When to tell the child | SHARE-model age-stratified disclosure scripts (0-6 / 7-11 / 12-17 / adult) |
| PKU / Pompe diet | Disease-specific diet protocols + food exchange + metabolic crisis emergency card |
| Systematic medical records | N=1 data vault + 4-level sharing control (🔒 private → 🌐 anonymized public) |
| Finding peer patients | China Alliance for Rare Diseases + single-disease communities + NORD / EURORDIS |

---

## Features

### 1 entry point + 10 companion modules

```
firefly                     Family entry point, routes by situation

firefly-organize            OCR + HPO + DeepRare diagnostic navigation
firefly-genetic-counseling  ACMG variant interpretation + family risk + PGT-M
firefly-education           Personalized patient education handbook (Mermaid diagrams)
firefly-caregiver           Zarit burden screening + respite resources
firefly-mind                Mental health screening (PHQ-9 / GAD-7 / IES-R)
firefly-diet                Metabolic rare disease diets (PKU / Pompe / UCD / MSUD / GSD)
firefly-second-opinion      UDN / UDP-EU / China network case packets
firefly-vault               N=1 data vault + re-identification risk assessment
firefly-disclosure          Genetic risk disclosure (children / blood relatives / spouse)
firefly-patient-org         Patient organization + genetic counselor directory
```

Each module can be **triggered independently**. The system understands your context first, then guides next steps.

### Design Philosophy

Firefly is a **navigator and structurer**, not a diagnostic replacement.

Core decisions — diagnosis confirmation, treatment selection, reproductive planning — must happen between you, your attending physicians, and genetic counselors.

Firefly's role:
- **Shorten the diagnostic path** — DeepRare methodology (Nature 2026, 2,919 diseases, 64.4% recall@1) gives directions, not conclusions
- **Translate medical information** — ACMG / HPO / ORPHA terminology into usable language
- **Connect resources** — rare-in-1/2000 still means tens of thousands of people in China's 1.4B population; the gap is matching
- **See the emotion** — diagnostic odyssey is chronic trauma; screening and support cannot be absent

Firefly **will not**:
- Diagnose — only "directions worth investigating"
- Recommend treatments — only "options worth discussing with your doctor"
- Decide reproductive choices — presents all options (including not having children, adoption, donor gametes)
- Discourage you from following medical advice
- Say "nothing can be done" — there's always supportive care or research participation

---

## Technology

### DeepRare Diagnostic Methodology

Open-source reimplementation of the AI rare disease diagnostic pipeline (Nature 2026, [doi:10.1038/s41586-025-10097-9](https://www.nature.com/articles/s41586-025-10097-9)), covering 2,919 rare diseases, Recall@1 64.4% (exceeding human experts' 54.6%).

Core pipeline: HPO phenotype extraction → multi-source knowledge retrieval (PubCaseFinder + Phenobrain + OMIM + Orphanet + PubMed) → LLM consensus ranking → traceable reasoning chain + self-reflection validation.

### Rare Disease Cognitive Framework

Synthesized from 50 top global researchers across 6 schools (Schiff, Lupski, Valle, Rehm, Firth, etc.): 7 mental models + 10 decision heuristics + 6 internal tensions + 8 blind-spot self-checks.

Sample applications:
- **Genome-First** (mental model 1) — ask "what does the genome say" before stepwise symptom-based workup
- **Backup gene activation** (mental model 3) — SMN2 for SMA, HbF for sickle cell
- **Speed as medicine** (mental model 6) — the moral urgency of rare disease
- **Ancestry representation blind spot** — gnomAD biased European; East Asian allele frequencies may mislead

---

## Install

One line:

```bash
npx skills add CancerDAO/firefly-skill -g -a claude-code -y
```

Uses the [vercel-labs/skills](https://github.com/vercel-labs/skills) CLI to install all 11 skills to `~/.claude/skills/` at once. **Restart Claude Code** afterward.

More options (project-scoped install, multi-agent, single-companion install) — see [INSTALL.md](INSTALL.md).

---

## Usage

Enter any trigger scenario in Claude Code:

- "My kid's WES report has a c.xxxx variant classified as VUS" → auto-invoke `firefly-organize` + `firefly-genetic-counseling`
- "I've been caring for my SMA child for 8 years, I'm breaking down" → `firefly-caregiver` + `firefly-mind`
- "I want to submit a consultation to an overseas center, how do I prepare?" → `firefly-second-opinion`
- "What can a child with PKU eat?" → `firefly-diet`

Or invoke companions directly: `/firefly-mind` for screening / `/firefly-patient-org` to find peers.

---

## Data Flow

All modules share one `profile.json` (schema: [firefly/references/templates.md](firefly/references/templates.md)) — a patient's structured record, read/written by 10 companions per permission.

```
 OCR / dialogue
   ↓
 firefly-organize  →  Create profile.json + Top 5 directions
   ↓
 firefly-genetic-counseling  →  Fill genetic_findings + pedigree
   ↓
 firefly-education  →  Read profile → Generate handbook
 firefly-vault      →  Read profile → Build local vault
 firefly-mind       →  Write mental_health screening
 ...
```

---

## Contributing

Issues / PRs welcome. Rare-disease-specific:
- Patient organizations / communities / counselor additions (`firefly-patient-org/references/`)
- Diet protocol revisions (`firefly-diet/references/`, requires authoritative citations)
- Translation and terminology corrections
- Bug / behavior fixes

---

## License

MIT License — see [LICENSE](LICENSE)

---

## Acknowledgments

- DeepRare methodology: Nature 2026 paper authors
- HPO standardization: Köhler et al., Human Phenotype Ontology Consortium
- ACMG variant classification: Richards et al. 2015 + ClinGen
- China Rare Disease Diagnosis and Treatment Collaborative Network: 324 member hospitals + China Alliance for Rare Diseases
- 50 rare disease researchers whose collective cognition informed the framework (see `firefly/references/cognitive-framework.md`)

<br>

<div align="center">

**萤火 · Firefly** — *CancerDAO rare disease sibling project, isomorphic to [cancer-buddy-skill](https://github.com/CancerDAO/cancer-buddy-skill)*

</div>
