# 中国大陆罕见病专家中心种子库

人工维护的种子库。**用途：subagent 调研时的起点 + 验证锚**，不替代实时调研。条目随时间会过时，每次使用前必须用一手源（院官网 / 协作网官网 / 公众号）核实当前状态。

> 数据时效：本文档最后人工核对 2026-04。每条带 `last_verified` 字段；若该字段距今 > 6 个月，必须重新核实。

## 国家级综合罕见病中心

### 北京协和医院 罕见病多学科会诊中心

- 城市：北京
- 涵盖病种：广谱（疑难罕见病、未确诊病例为主）
- 平台：罕见病多学科会诊门诊（疑难罕见病专科门诊）
- 准入：本院疑难病转介；外院通过转诊申请
- 协作网定位：国家罕见病诊疗协作网 **国家级牵头单位**
- last_verified: 2026-04
- 一手源：pumch.cn

### 上海交通大学医学院附属上海儿童医学中心

- 城市：上海
- 涵盖：儿科罕见病、罕见遗传代谢病、罕见血液病
- 协作网定位：协作网国家级
- last_verified: 2026-04

### 复旦大学附属儿科医院

- 城市：上海
- 涵盖：儿科罕见病全谱、新生儿筛查相关代谢病、神经罕见病
- 协作网定位：协作网国家级
- last_verified: 2026-04

### 广州市妇女儿童医疗中心

- 城市：广州
- 涵盖：儿科罕见病、产前诊断、新生儿筛查
- last_verified: 2026-04

### 浙江大学医学院附属儿童医院

- 城市：杭州
- 涵盖：儿科罕见病、遗传代谢病、新生儿筛查
- last_verified: 2026-04

### 四川大学华西医院 罕见病中心

- 城市：成都
- 涵盖：成人罕见病为主，部分儿科
- 协作网定位：协作网国家级（西南片区）
- last_verified: 2026-04

### 中国医学科学院北京协和医学院 神经科学罕见病平台

- 城市：北京
- 涵盖：神经罕见病（SMA / DMD / ALS / 罕见脑病）
- last_verified: 2026-04

### 中国人民解放军总医院（301）罕见病诊疗

- 城市：北京
- 涵盖：罕见病全谱、罕见免疫病
- last_verified: 2026-04

## 区域 / 病种专长中心（部分典型示例）

### 北京大学第一医院 神经内科罕见病

- 城市：北京
- 病种专长：罕见脑病、Wilson 病、罕见运动障碍

### 北京积水潭医院 骨科罕见病

- 城市：北京
- 病种专长：成骨不全症（OI 瓷娃娃）、罕见骨病

### 中山大学附属第一医院

- 城市：广州
- 病种专长：神经肌肉罕见病、SMA、DMD

### 西安交通大学第一附属医院

- 城市：西安
- 病种专长：西北罕见病牵头

### 哈尔滨医科大学附属第二医院

- 城市：哈尔滨
- 病种专长：东北罕见病、罕见血液病

### 山东大学齐鲁医院

- 城市：济南
- 病种专长：罕见血液病、罕见代谢病

### 重庆医科大学附属儿童医院

- 城市：重庆
- 病种专长：西南儿科罕见病

## 国际网络中国接入点

### UDN US — 中国患者申请路径

- self-referral form 接受国际患者
- 实际 acceptance 概率较低（~30% 整体，国际患者更低）
- 一旦 accepted，NIH 承担美国境内住院 + 检查费用，但跨境运输、签证、住宿用户自理
- 申请: undiagnosed.hms.harvard.edu/apply

### UDP-EU — 欧盟未诊断网络

- 不同成员国受理路径不同
- 同上：accepted 后基本免费

### RDCRN consortia — 病种特异

- 20+ consortia，按病种参与（FOP、CFC、SMA、ALD 等）
- 部分 consortium 接受国际患者纳入自然史研究（无费用）
- 申请: rarediseasesnetwork.org/cms/<consortium-name>

## 单病种主要 foundation（中国）

### 北京瓷娃娃罕见病关爱中心

- 病种 hub：成骨不全症 (OI)、并联多种罕见病关爱
- 联系：通过 CORD（罕见病发展中心）

### 蝴蝶宝贝关爱中心

- 病种：大疱性表皮松解症 (EB)

### SMA 之家

- 病种：脊髓性肌萎缩症

### 罕见病发展中心 (CORD)

- 角色：罕见病伞形 NGO，对接多 foundation

### 病痛挑战基金会

- 角色：罕见病慈善 + 医疗援助 + 政策倡导

## 每条目期望字段（重新核实时按此扩充）

```yaml
centre_name: ...
city: ...
type: 综合罕见病 / 单病种专长 / 国际网络入口
covered_diseases:
  - orpha_id: ORPHA:xxx
    disease_name: ...
    depth: 国家级 | 区域级 | 偶发接诊
team:
  multidisciplinary: true | false
  involved_specialties: [临床遗传, 神经, 代谢, 影像, 病理, 遗传咨询, 心理, 康复]
  team_lead: ...
intake:
  external_patient: 接受 / 仅院内 / 需转诊
  required_referral_letter: true | false
  required_documents: [基因报告, 影像, 监护人身份证明, 推荐信]
  appointment_lead_time_weeks: 2 / 4 / 8 / 12+
fees:
  consultation_fee: ...
  insurance_coverage: 院内医保 / 协作网内转诊报销 / 自费
  charity_aid_available: ...
international_credentials:
  orphanet_label: Reference / Expert / Specialised / null
  rdcrn_member: true | false
  udn_alternative: true | false
contact:
  appointment_phone: ...
  rd_clinic_email: ...
  wechat_account: ...
last_verified: YYYY-MM-DD
primary_source: <URL>
```

## 子 agent 用法

派发 subagent 调研某城某病种专家中心时：

1. 主 agent 从本表挑出该城 / 邻近城市的候选医院列表（≤5 家）
2. 给每个 subagent 分一家，prompt 类似：
   ```
   核实 [医院名] 当前是否为 [疾病名 / ORPHA:xxx] 的专家中心，按 rd-expert-centres-cn-seed.md 的字段格式输出。
   一手源：院官网、院微信公众号、近 6 个月内的院方报道、罕见病诊疗协作网官网。
   必须加载 web-access skill 并遵循指引。
   只用 ORPHA / 病名标识，不传患者信息。
   ```
3. 收回结果后**反向更新本文档**（如发现新信息），保持种子库新鲜度

## 不在表里的院

不代表不接此病种——只是种子库还没覆盖。subagent 也可以从中国罕见病诊疗协作网（324 家）出发反向发现。
