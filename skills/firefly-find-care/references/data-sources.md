# Data sources for firefly-find-care

罕见病专属。一手 > 二手；机构官方 > 第三方汇编；国际 + 中国双轨。

## 国际诊断网络（diagnostic_network 类查询主源）

| 源 | URL | 适合查 | 抓取注意 |
|---|---|---|---|
| **ORPHANET** | orpha.net | ORPHA codes、专家中心数据库（Reference / Expert / Specialised Service 三级标签）、流行病学 | 多语言版本，中文版滞后；用英文版 |
| **GARD (NIH Genetic and Rare Diseases)** | rarediseases.info.nih.gov | 病种概览、患者组织索引、临床试验链接 | 静态页，curl 即可 |
| **OMIM** | omim.org | 基因-疾病关联、表型描述 | 部分需注册；用于交叉验证 |
| **GeneReviews (NCBI)** | ncbi.nlm.nih.gov/books | 病种深度临床综述 | 公开，curl 可读 |
| **UDN (Undiagnosed Diseases Network US)** | undiagnosed.hms.harvard.edu | 未诊断病例的 US 申请通道、accepted patient stories | 申请门槛高（acceptance rate ~30%）；需 self-referral form |
| **UDP-EU (European Undiagnosed Diseases Program)** | udnetwork.eu | 欧盟未诊断网络 | 各成员国申请路径不同 |
| **RDCRN (Rare Diseases Clinical Research Network)** | rarediseasesnetwork.org | 20+ 个病种 consortium，含数据库 + 试验 + 自然史研究 | 按 consortium 浏览 |
| **CTSA Rare Disease Programs** | ncats.nih.gov | NIH 资助的临床转化网络 | 二级索引 |

## 中国 — 罕见病诊疗体系（最重要的本土源）

| 源 | URL | 适合查 | 抓取注意 |
|---|---|---|---|
| **中国罕见病诊疗协作网** | rarediseases.cn | 324 家网络医院（国家级 / 省级 / 市级三档）+ 哪些病种在哪些院 listed | 部分页面需登录；公众号"罕见病诊疗协作网"有更新 |
| **中国第一批罕见病目录（121 种）/ 第二批（86 种）** | 国家卫健委官网 + 文件下载 | 政策性病种界定 | 静态 PDF |
| **国家罕见病注册系统 (NRDRS)** | nrdrs.org.cn | 中国注册罕见病登记 | 部分需机构权限 |
| **北京协和医院罕见病多学科会诊** | pumch.cn | 全国疑难罕见病主流终点 | 院官网新闻栏目 |
| **上海儿童医学中心 / 复旦儿童医院 罕见病平台** | scmc.com.cn / shchildren.com.cn | 儿科罕见病主力 | 院官网 |
| **华西医院罕见病中心** | wchscu.cn | 西南罕见病主力 | 院官网 |
| **中山大学附属第一医院 / 浙大儿童医院 / 广州市妇女儿童医疗中心** | 各院域名 | 区域专科中心 | 院官网 |
| **遗传咨询师 / 临床遗传医师**：中华医学会医学遗传学分会 | 中华医学会官网 | 委员名单反查 | 静态页 |

## 临床试验（trial 类查询主源）

| 源 | URL | 适合查 | 抓取注意 |
|---|---|---|---|
| **ClinicalTrials.gov** | clinicaltrials.gov | 全球试验全集，按 condition + status filter | 公开 API，优先用 |
| **ChiCTR** | chictr.org.cn | 中国境内临床试验 | 反爬中等；mcp__chictr 可用时优先 |
| **ICTRP (WHO 国际平台)** | trialsearch.who.int | 跨国汇聚 | 元数据较粗 |
| **EU Clinical Trials Register** | clinicaltrialsregister.eu | 欧盟试验 | 静态页 |
| **病种制药公司 portal**（如 Sarepta DMD Hub、BioMarin Patient Resources） | 各公司页 | 直接看 sponsor 在招试验 | 各家 UI 差别大 |
| **国家药监局药物临床试验登记** | chinadrugtrials.org.cn | 含尚未在 ChiCTR 注册的国内药物试验 | 可能需登录态 |

## 病种基金会 / Foundation（doctor & expert_centre 反查 + 慈善赠药）

| 国际 | 适合 |
|---|---|
| NORD (rarediseases.org) | 美国患者组织 + financial assistance 项目 |
| EURORDIS (eurordis.org) | 欧洲 1000+ 患者组织 |
| Global Genes (globalgenes.org) | 国际 RARE 倡导 |
| Foundation for Rare Diseases / Disease-specific foundations（FOP Foundation、CFC International、SMA Europe、Cure SMA、PPMD 等） | 单病种深度资源 + Find a Doctor 页 |

| 中国 |  |
|---|---|
| 中国罕见病联盟 | 国家级伞形组织 |
| 北京瓷娃娃罕见病关爱中心 | OI / 罕见病 hub |
| 病痛挑战基金会 | 罕见病慈善 + 医疗援助 |
| 各单病基金会（蝴蝶宝贝关爱中心 EB、SMA 之家、罕见病发展中心 CORD 等） | 病种专属 |

## 已知抓取陷阱

- **ORPHANET expert centres**：列表分页，CDP 翻页比 API 稳；筛选器用 ORPHA + Country
- **中国罕见病诊疗协作网**：医院列表是 SPA，curl 拿不全，**用 CDP**
- **好大夫在线**：医生主页 URL 含会话参数，CDP 内点击进比手动构造 URL 可靠
- **微信公众号**（很多病种基金会主阵地）：搜索引擎索引滞后，CDP 直登可能更新更快
- **UDN 申请页**：表单分多步，走 GUI 而非程序化
- **ClinicalTrials.gov**：罕见病用 condition 字段精确匹配 ORPHA 病名 + 同义词，否则漏招
- **ChiCTR**：搜索结果列表的链接可能丢参数；用详情页的 trial registration number 当主键

## Subagent 派发模板（按 query_type 挑相关 1–3 行）

派发 subagent 时**不要**把整个 data-sources.md 塞 prompt。挑出本次任务相关的 1–3 行（"查 X 在 Y 源 + Z 源"），让 subagent 自己加载 web-access 处理具体抓取。

示例：

```
任务：在 ORPHANET expert centres 数据库查 ORPHA:365 (Pompe disease) 在中国大陆的认证中心；同时查中国罕见病诊疗协作网（rarediseases.cn）哪些医院 listed Pompe。每条返回中心名 / 城市 / ORPHA 标签等级 / 联系方式。

约束：必须加载 web-access skill 并遵循指引；只用 ORPHA 标识不传患者信息；超时 5 分钟。
```
