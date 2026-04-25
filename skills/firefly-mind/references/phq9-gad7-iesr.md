# PHQ-9 / GAD-7 / IES-R 完整量表、评分与解释

> 本文件是 firefly-mind companion 的量表工作手册。供 LLM 逐题施测、计分、分级、生成解释语。

---

## 一、使用说明

- **筛查工具，非诊断工具**。得分只能提示"某一严重度区间的症状群存在的可能性"，不能替代精神科医生或临床心理科医生的面诊。
- **建议由临床医生确诊**。任何中重度结果都应转介至精神科 / 临床心理科。
- **施测频率建议**：
  - 🟢 正常/轻度：每 2 周自查一次
  - 🟡 中度：每周 1 次，追踪 4 周趋势
  - 🔴 中重-重度：按临床医生节奏，通常每 1-2 周
- **施测语境**：先共情再发问卷；不要"冷启动"直接列题；告知患者"每题想过去两周"；允许跳题但记录。
- **罕见病特有提醒**：如果患者经历长期未确诊（3+ 年）、多次误诊、被医生质疑"装病或心理作祟"，除 PHQ-9/GAD-7 外应同时施测 IES-R。
- **校订版本参考**：PHQ-9 中文版（邓云龙、戴晓阳等，2004/2014 修订）；GAD-7 中文版（何筱衍等，2010）；IES-R 中文版（吴坎坎、张雨青等，2011）。如遇到与本文档措辞略有不同的官方版本，以最新官方版为准。

---

## 二、PHQ-9（患者健康问卷-9，Patient Health Questionnaire-9）

**指导语**：在过去的两周里，你有多少时间被以下这些问题困扰？

**评分**（每题 0-3）：
- 0 = 完全不会（Not at all）
- 1 = 好几天（Several days）
- 2 = 一半以上时间（More than half the days）
- 3 = 几乎每天（Nearly every day）

### 完整 9 题

| # | 中文 | 英文 |
|---|------|------|
| 1 | 做事时提不起劲或没有兴趣 | Little interest or pleasure in doing things |
| 2 | 感到心情低落、沮丧或绝望 | Feeling down, depressed, or hopeless |
| 3 | 入睡困难、睡不安稳或睡眠过多 | Trouble falling or staying asleep, or sleeping too much |
| 4 | 感觉疲倦或没有活力 | Feeling tired or having little energy |
| 5 | 食欲不振或吃太多 | Poor appetite or overeating |
| 6 | 觉得自己很糟糕，或觉得自己很失败，或让自己、家人失望 | Feeling bad about yourself — or that you are a failure or have let yourself or your family down |
| 7 | 对事物专注有困难，例如阅读报纸或看电视时 | Trouble concentrating on things, such as reading the newspaper or watching television |
| 8 | 动作或说话速度缓慢到别人已经察觉？或者正好相反——变得比平常更烦躁或坐立不安 | Moving or speaking so slowly that other people could have noticed. Or the opposite — being so fidgety or restless that you have been moving around a lot more than usual |
| 9 | 有不如死掉或用某种方式伤害自己的念头 | Thoughts that you would be better off dead, or of hurting yourself in some way |

### 评分总表

| 总分 | 严重度 | 建议 |
|------|--------|------|
| 0-4 | 无抑郁 | 自助资源 + 每 2 周自查 |
| 5-9 | 轻度抑郁 | 自助资源 + 观察 + 每 2 周自查；若持续 ≥2 个月建议咨询 |
| 10-14 | 中度抑郁 | 心理咨询 + 互助群 + 每周自查；考虑药物评估 |
| 15-19 | 中重度抑郁 | 精神科 / 临床心理科就医 + 药物评估 |
| 20-27 | 重度抑郁 | 立即就医，精神科，考虑住院评估 |

### 第 9 题（自杀/自伤意念）单独条款 — 🚨 硬性触发

**只要第 9 题得分 ≥ 1（"好几天"或以上），无论总分高低，立即执行下列三步，不能跳过、不能延后、不能以"总分不高"为理由豁免**：

1. **立刻提供危机热线号码**：
   - 北京心理危机研究与干预中心：010-82951332（24 小时）
   - 全国心理援助热线：400-161-9995（按 1 成人 / 2 青少年）
2. **明确建议告知至少一位家人或亲近朋友**。用原话："现在这件事你一个人扛着太重了，能不能告诉一位你信任的家人或朋友？哪怕只是说'我最近状态不太好'也行。"
3. **强烈建议 24-72 小时内前往精神科或急诊心理科就医**。若得分为 3（几乎每天），建议当天就诊或拨打 120。

**同时记录**：在 `profile.json` 的 `mental_health.alerts` 中追加 `{ "type": "suicidal_ideation", "phq9_q9_score": X, "timestamp": <ISO>, "handled": true }`。

### 解释语模板

- **0-4**："你的得分是 X 分，提示过去两周没有明显的抑郁症状。这是一个好的起点。"
- **5-9**："你的得分是 X 分，提示轻度抑郁症状。大多数人在重大压力下都会经历这个区间。我们可以先看看自助资源，两周后再测一次。"
- **10-14**："你的得分是 X 分，提示中度抑郁症状。建议你考虑开始心理咨询，同时加入患者互助群不要一个人扛。"
- **15-19**："你的得分是 X 分，提示中重度抑郁症状。这个水平强烈建议你预约精神科或临床心理科门诊，必要时药物会很有帮助——药物不是软弱，是工具。"
- **20-27**："你的得分是 X 分，提示重度抑郁症状。请尽快就医，今天或明天。这不是夸张——现在你需要专业人士在身边。"

---

## 三、GAD-7（广泛性焦虑量表-7，Generalized Anxiety Disorder-7）

**指导语**：在过去的两周里，你有多少时间被以下这些问题困扰？

**评分**（每题 0-3）：0 = 完全不会 / 1 = 好几天 / 2 = 一半以上时间 / 3 = 几乎每天

### 完整 7 题

| # | 中文 | 英文 |
|---|------|------|
| 1 | 感到紧张、焦虑或急躁 | Feeling nervous, anxious, or on edge |
| 2 | 无法停止或控制担忧 | Not being able to stop or control worrying |
| 3 | 过度担心各种各样的事情 | Worrying too much about different things |
| 4 | 很难放松下来 | Trouble relaxing |
| 5 | 心神不宁，坐立难安 | Being so restless that it's hard to sit still |
| 6 | 变得容易烦恼或急躁 | Becoming easily annoyed or irritable |
| 7 | 感到害怕，好像有可怕的事情要发生 | Feeling afraid as if something awful might happen |

### 评分总表

| 总分 | 严重度 | 建议 |
|------|--------|------|
| 0-4 | 无焦虑 | 自助 + 定期自查 |
| 5-9 | 轻度焦虑 | 自助资源（呼吸/正念/运动）+ 观察 |
| 10-14 | 中度焦虑 | 心理咨询；考虑就医评估 |
| 15-21 | 重度焦虑 | 精神科 / 临床心理科就医，药物评估 |

### 解释语模板

- **0-4**："你的得分是 X 分，焦虑水平在正常范围内。"
- **5-9**："你的得分是 X 分，提示轻度焦虑。建议试试呼吸练习、规律运动、限制咖啡因和夜间刷屏。"
- **10-14**："你的得分是 X 分，提示中度焦虑。建议开始心理咨询；如果焦虑影响睡眠或日常生活，可以看精神科评估是否需要短期药物。"
- **15-21**："你的得分是 X 分，提示重度焦虑。建议尽快就医——重度焦虑通常对治疗反应很好，不要硬扛。"

---

## 四、IES-R（事件影响量表修订版，Impact of Event Scale-Revised）

**指导语**（罕见病奥德赛特化）：
> 以下的题目，请你围绕**最困扰你的那段求医经历**来回答——可以是被误诊的那段时间、反复求医却找不到答案的那段时间、被医生质疑"装病"的那段时间，或者你觉得最难熬的任何一段诊断过程。请评估在过去 7 天里，这些感受对你造成了多大程度的困扰。

**评分**（每题 0-4）：
- 0 = 完全没有（Not at all）
- 1 = 很少（A little bit）
- 2 = 有时（Moderately）
- 3 = 经常（Quite a bit）
- 4 = 总是（Extremely）

### 完整 22 题（按子维度分组）

#### 侵入维度（Intrusion，8 题）— 题目 1, 2, 3, 6, 9, 14, 16, 20

| # | 中文 | 英文 |
|---|------|------|
| 1 | 任何关于这件事的提醒都会让我重新体验当时的感受 | Any reminder brought back feelings about it |
| 2 | 我睡眠时会被打断 | I had trouble staying asleep |
| 3 | 别的事情都会让我不停想到它 | Other things kept making me think about it |
| 6 | 我不想去想它，但它还是会闯进我脑海 | I thought about it when I didn't mean to |
| 9 | 事情的画面会突然出现在我脑海里 | Pictures about it popped into my mind |
| 14 | 我发现自己表现或感觉得好像又回到了那时候 | I found myself acting or feeling like I was back at that time |
| 16 | 我有强烈的感受涌上来 | I had waves of strong feelings about it |
| 20 | 我梦到它 | I had dreams about it |

#### 回避维度（Avoidance，8 题）— 题目 5, 7, 8, 11, 12, 13, 17, 22

| # | 中文 | 英文 |
|---|------|------|
| 5 | 我避免让自己心烦意乱，只要一想到它或被它提醒 | I avoided letting myself get upset when I thought about it or was reminded of it |
| 7 | 我觉得事情好像没发生过，或者它不真实 | I felt as if it hadn't happened or wasn't real |
| 8 | 我远离任何会让我想到它的事物 | I stayed away from reminders about it |
| 11 | 我尝试不去想它 | I tried not to think about it |
| 12 | 我意识到我对它还有很多情绪没处理，但我选择不去碰 | I was aware that I still had a lot of feelings about it, but I didn't deal with them |
| 13 | 我对它的感受有些麻木 | My feelings about it were kind of numb |
| 17 | 我想把它从我记忆中抹掉 | I tried to remove it from my memory |
| 22 | 我尽量不谈论它 | I tried not to talk about it |

#### 过度警觉维度（Hyperarousal，6 题）— 题目 4, 10, 15, 18, 19, 21

| # | 中文 | 英文 |
|---|------|------|
| 4 | 我感到急躁和生气 | I felt irritable and angry |
| 10 | 我感到紧张不安，容易受惊 | I was jumpy and easily startled |
| 15 | 我发现难以入睡 | I had trouble falling asleep |
| 18 | 我难以集中注意力 | I had trouble concentrating |
| 19 | 关于它的提醒会让我身体有反应，比如出汗、呼吸困难、恶心或心跳加速 | Reminders of it caused me to have physical reactions, such as sweating, trouble breathing, nausea, or a pounding heart |
| 21 | 我感到警惕、戒备 | I felt watchful and on-guard |

### 评分方法

- 总分 = 22 题 0-4 分之和（范围 0-88）
- 三子维度分：各维度题目得分之和
  - 侵入（8 题，0-32）
  - 回避（8 题，0-32）
  - 过度警觉（6 题，0-24）

### 评分总表（按总分）

| 总分 | 严重度 | 解读 |
|------|--------|------|
| 0-23 | 正常 | 创伤应激症状在正常波动范围 |
| 24-32 | 轻度 | 有临床关注价值，建议自助+观察 |
| 33-36 | 中度 | 可能存在 PTSD 相关问题，建议专业评估 |
| 37+ | 重度 | 高度提示 PTSD 或其他创伤相关障碍，建议精神科/临床心理科就医 |

### 罕见病奥德赛的特化说明

- 传统上 IES-R 用于急性事件（事故、灾难、暴力）后的 PTSD 评估。本 companion 将其**借用于"长期诊断奥德赛创伤"**——即反复误诊、漫长等待、系统性不被相信所累积的慢性应激。
- 施测时务必明确指向"最困扰的那段求医经历"，而不是"你的整个疾病史"——避免过于笼统导致评分失真。
- 如果患者同时经历过急性医疗创伤事件（如 ICU、急救、临终目击等），可以分两次施测，分别锚定不同事件。
- 中度/重度结果应与精神科医生沟通是否满足 DSM-5 / ICD-11 PTSD 或适应障碍诊断标准。

### 解释语模板

- **0-23**："你的得分是 X 分，提示与这段求医经历相关的创伤应激反应在正常范围内。"
- **24-32**："你的得分是 X 分，提示轻度创伤应激反应。那段经历在你身上确实留下了痕迹，这是身体在保护你，不是软弱。"
- **33-36**："你的得分是 X 分，提示中度创伤应激反应。建议做一次专业的心理评估——尤其当你发现这些反应还在影响你现在的求医行为（比如不敢再看新医生）。"
- **37+**："你的得分是 X 分，提示重度创伤应激反应。你经历的那段比你意识到的更重。请看精神科或临床心理科，有针对性的治疗（如 EMDR、CPT、暴露疗法）对创伤后应激非常有效。"

---

## 五、跨量表整合决策树

按**最高严重度决定整体响应级别**（木桶原理——任一量表报警即触发高级别响应）。

```
定级逻辑（伪代码）：
level = max(phq9_level, gad7_level, iesr_level)
if phq9_q9_score >= 1:
    level = RED  # 硬性置顶
```

| 组合场景 | 综合级别 | 响应动作 |
|---------|---------|---------|
| 三个量表均为正常/轻度 | 🟢 | 自助资源 + 每 2 周自查 |
| 任一量表为中度，无自杀意念 | 🟡 | 心理咨询 + 互助群 + 每周自查 |
| 任一量表为中重度/重度 | 🔴 | 精神科/临床心理科就医 + 热线 + 告知家人 |
| PHQ-9 第 9 题 ≥1（无论总分） | 🔴 硬触发 | 立刻热线 + 24-72h 就医 + 告知家人，不可豁免 |
| PHQ-9 + GAD-7 均中度以上 | 🔴 | 合并抑郁焦虑，精神科评估药物 |
| IES-R 重度 + PHQ-9/GAD-7 轻度 | 🔴（针对创伤） | 转介创伤专长的临床心理师（EMDR/CPT） |
| 仅 IES-R 中度，其他正常 | 🟡 | 创伤聚焦的咨询 + 继续随访 |
| 照护者本人任一量表中度以上 | 🟡/🔴 | 照护者是 hidden patient，按本人分数独立建议 + 推荐照护者支持小组 |

---

## 六、引用

- Kroenke, K., Spitzer, R. L., & Williams, J. B. (2001). The PHQ-9: validity of a brief depression severity measure. *Journal of General Internal Medicine*, 16(9), 606-613.
- Spitzer, R. L., Kroenke, K., Williams, J. B., & Löwe, B. (2006). A brief measure for assessing generalized anxiety disorder: the GAD-7. *Archives of Internal Medicine*, 166(10), 1092-1097.
- Weiss, D. S., & Marmar, C. R. (1997). The Impact of Event Scale-Revised. In J. P. Wilson & T. M. Keane (Eds.), *Assessing psychological trauma and PTSD* (pp. 399-411). Guilford Press.
- 邓云龙, 戴晓阳 等. (2004/2014). PHQ-9 中文版的信效度研究. *中国临床心理学杂志*.
- 何筱衍, 李春波 等. (2010). 广泛性焦虑量表（GAD-7）在综合性医院的应用研究. *上海精神医学*.
- 吴坎坎, 张雨青 等. (2011). 事件影响量表修订版（IES-R）中文版在汶川地震灾区的信效度检验. *中国心理卫生杂志*.
