---
name: firefly-patient-org
description: "连接罕见病患者到病友组织、遗传咨询师、专科医生。按病种/地域/语言匹配中国罕见病联盟、病种单病社群、国际 NORD/EURORDIS、遗传咨询师名录。Use when: 患者或家属说'想找病友'、'想找同病人互助群'、'需要遗传咨询师'、'哪里能找到专科医生'、'有没有 XX 病的患者组织'. Triggers on: 患者组织, 病友群, 罕见病联盟, 遗传咨询师, NORD, EURORDIS, 单病社群, 互助群, 找病友."
---

# firefly-patient-org — 患者组织连接

帮患者和家属找到同路人、找到专业支持资源。罕见病的社群力量是治疗外的第二支柱。

## 何时调用

- 患者说"想找得同一种病的人聊聊"
- 家属说"有没有这个病的患者组织"
- 医生建议"你可以找个遗传咨询师"，但患者不知去哪找
- 患者刚确诊，需要联系到"第一个同伴"

## 协议

1. **识别病种**。从 `profile.json` 的 `confirmed_diagnosis` 或用户描述提取疾病名 + ORPHA ID。
2. **匹配组织**。按下列优先级查 `references/china-rd-alliance.md` + `references/international-orgs.md`：
   - 国内单病社群（病种专属）
   - 中国罕见病联盟合作组织
   - 国际单病组织（NORD / EURORDIS / 病种专属 foundation）
3. **遗传咨询师匹配**（如需要）。查 `references/genetic-counselor-directory.md`，按地域筛选。
4. **语言适配**。默认给中文资源；患者有跨境需求或病种国内无组织，才给英文资源。
5. **输出**。每个匹配项包含：组织名、联系方式、官网、活动频率、近期活动、加入门槛。

## profile.json 交互

- **Reads**: `confirmed_diagnosis`, `demographics.ethnicity`（用于国际组织语种推荐）
- **Writes**: —

## 输出格式

```markdown
# 患者组织匹配 · <疾病名> · <日期>

## 国内组织（推荐度 ⭐⭐⭐）
### 组织名
- 官方渠道：微信公众号 / 官网
- 联系方式：邮箱 / 入群方式
- 近期活动：YYYY-MM-DD 活动名
- 备注：活跃程度、主要服务对象

## 国际组织
（同上）

## 遗传咨询师
### 姓名 / 机构 / 城市
- 擅长领域
- 咨询方式（线上/线下）
- 联系渠道

## 下一步
1. [ ] 加入 XX 社群
2. [ ] 联系 YY 咨询师预约
```

## Safety Guardrails

- 不推荐来路不明的民间组织——只用 `references/` 里的经认证清单
- 提醒用户：分享病历到社群前务必匿名化
- 不推荐商业化咨询服务（除非是有资质的医疗机构）
- 若某病种国内无组织，坦诚说"目前国内这个病的组织还在筹建/暂无，可以先联系 YY 代为咨询"

## References

- [references/china-rd-alliance.md](references/china-rd-alliance.md) — 中国罕见病联盟 + 单病社群索引
- [references/international-orgs.md](references/international-orgs.md) — NORD / EURORDIS / 病种国际组织
- [references/genetic-counselor-directory.md](references/genetic-counselor-directory.md) — 遗传咨询师名录
