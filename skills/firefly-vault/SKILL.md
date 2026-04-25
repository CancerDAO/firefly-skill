---
name: firefly-vault
description: "帮患者建 N=1 个人数据仓。初始化目录结构 → 4 级分享控制（🔒私有/🔑授权/📊匿名/🌐公开）→ 访问日志 → 基因数据重识别风险评估。罕见病患者本身就是珍稀数据源，N=1 数据长期价值极高. Use when: 患者想系统化整理/备份自己的医疗数据；要和研究机构/医生共享数据前需要脱敏；要向其他患者公开匿名数据贡献科研. Triggers on: 建数据仓, 数据管理, 个人医疗数据, data vault, N=1, 数据分享, 匿名化, 基因数据隐私, 重识别."
---

# firefly-vault — N=1 数据仓

帮患者在本地建立结构化的医疗数据仓——所有检查报告、基因数据、治疗记录、随访数据一处管理，支持精细分享控制。

## 何时调用

- 患者想"系统整理我的医疗数据"
- 要和研究机构/医生/新平台分享数据前
- 基因检测后对隐私担忧
- 患者想把匿名数据贡献给 CancerDAO / 罕见病数据库

## 协议

1. **初始化目录**。在患者指定位置（默认 `~/patient-vault-<id>/`）创建：

   ```
   patient-vault/
   ├── profile.json              # 从 firefly-organize 迁入
   ├── timeline.md               # 治疗时间线
   ├── diagnostics/              # 所有检查报告
   │   ├── imaging/              # CT/MRI/超声
   │   ├── labs/                 # 血检尿检
   │   ├── pathology/            # 病理
   │   └── genetics/             # WES/WGS/panel 报告 + 原始 VCF
   ├── treatments/               # 每次治疗记录 + 应答评估
   ├── monitoring/               # 纵向监测数据
   ├── notes/                    # 医生访谈记录、Q&A
   ├── sharing-settings.json     # 分享级别配置
   └── access-log.ndjson         # 每次访问日志
   ```

2. **导入现有数据**。如果患者已有 profile.json（firefly-organize 产出），直接迁入。如果有散落的 PDF/图片，问用户想不想调 firefly-organize 做 OCR 结构化。

3. **4 级分享控制**（sharing-settings.json）：
   - 🔒 **private**：仅本人可读
   - 🔑 **authorized**：授权给特定人（医生、家属），邮箱/ID 白名单
   - 📊 **anonymized-for-AI**：脱敏后供 AI 训练/医学研究用（去姓名、出生日期精度到年、地域精度到省）
   - 🌐 **public**：完全匿名公开贡献（只保留临床特征、基因变异、治疗应答）

4. **基因数据重识别风险**（详见 `references/rare-disease-reidentification.md`）：
   - 提醒：罕见病 + 基因变异组合极易被重识别（即使去名字）
   - 罕见病目录 121 种常见病在百万级数据集里平均只有几十人
   - 共享基因原始数据（VCF/FASTQ）前强烈建议做 k-匿名检查
   - 家系数据尤其敏感——共享一人的数据等于暴露整个家族的遗传信息

5. **访问日志**（access-log.ndjson）：每次读/写/共享操作记一行 JSON（时间戳、操作、请求方、授权状态）。

## profile.json 交互

- **Reads**: 全部字段（用于初始化 vault + 分享预览）
- **Writes**: `sharing_settings`, `access_log`

## 输出格式

创建 vault 后输出摘要：

```markdown
# 数据仓已创建 · <patient_id>

## 位置
~/patient-vault-<id>/

## 已入库
- profile.json
- X 个检查报告
- Y 个治疗记录

## 当前分享级别
🔒 private（默认）

## 重识别风险评估
- 病种：<疾病名> (ORPHA:<id>)
- 国内估计患者数：X 人
- 风险级别：🔴高 / 🟡中 / 🟢低
- 建议：...

## 下一步
1. [ ] 如需授权医生访问 → 调整 sharing-settings.json `authorized` 级别 + 添加医生 ID
2. [ ] 如愿贡献科研 → 先 📊 anonymized，不要 🌐 public
3. [ ] 每月导出一次增量备份
```

## Safety Guardrails

- **永远不默认 🌐 public**——初始化一律 🔒 private
- 提醒：一旦 public 不可撤回
- 基因原始数据（VCF/FASTQ）单独一个 warning gate，切换到 authorized+ 级别前二次确认
- 未成年人数据：监护人操作，但提醒"孩子成年后有权重新决定分享级别"
- 不在本 companion 内直接"发送/上传"数据到外部——只管本地结构和设置，传输由用户手动执行

## References

- [references/vault-schema.md](references/vault-schema.md) — 目录结构、文件命名规则、sharing-settings.json 格式
- [references/rare-disease-reidentification.md](references/rare-disease-reidentification.md) — 重识别风险评估方法、k-匿名检查、家系数据特殊性
