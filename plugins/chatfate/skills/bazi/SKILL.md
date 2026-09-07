---
name: bazi
description: 根据 ChatFate 八字排盘与用户自述，撰写今日指引、年度主题或人生命盘的自然中文报告。仅在总编排已取得八字计算结果后使用；不手工排盘，不替用户编造经历。
---

# 八字解读

如果工具已返回 `report_ready`，直接由总编排交付原报告，不重新生成解释。daily/annual新报告输出 `chatfate.reading.interpretation.v1`；旧版报告继续由原合同读取，不要求迁移。

## life：完整方法与单次原文

reading.product 为 life 时，使用本次 `calculate_bazi_chart({submissionId, lifeAuthoringVersion:"life.v1"})` 返回的完整 `authoring.instructions`，其 `authoring.version` 为 `life.v1`。方法、表达和字段说明均已随事实取得，不读取本地方法文件或额外参考。缺少方法或版本不匹配时保留提交，说明插件与 MCP 版本尚未配套，不自行补方法或重开卡片。

唯一事实输入是本次 `chatfate.bazi.life-calculation.v1.minimalFacts`，userContext 单独作为用户自述。依照服务端提供的完整方法，由当前模型亲自一次完成原文，直接提交 `{schemaVersion:"chatfate.bazi.life.interpretation.v1", product:"life", text:完整原文}`，不增加内容检查回合或第二作者。

life 到此分支结束，不读取下文 daily/annual 规则、通用合同或编辑参考。古籍旁证由显示侧独立附注，Writer 不核验出处或填写引用字段。

下文只用于 daily/annual 的 `chatfate.bazi.chart.v2`；旧life报告仍按原版本读取。

## 先看所问与现有依据

读取本次 `calculatedFacts`、`reading` 和可选 `userContext`。八字工具会隐藏出生输入，`normalizedInput` 缺席不影响解读。不要为补充故事而追问私密经历，也不把卡片外的猜测写成用户自述。

- daily 必须有日期匹配的 `dailyTransit`。把当日日干支及 `relations` 与本命合看，月柱只作较长背景；不把一篇长期性格分析换上今天的日期。
- annual 必须有对应目标年的 `liunian`。若有 `annualTransit.months`，可比较具体节气月段的月柱及本命关系；它们从立春延续至次年立春，不等同公历月。只有旧事实时就写全年主题，不自行生成月柱。
- `userContext` 是用户陈述，不是排盘结果。没有背景时可围绕命局提出具体反思角度，但不能编造职业、婚恋状态或正在发生的事。有背景时，先回答用户关心的问题，避免每一章都落到工作安排上。

## 输出合同

返回 JSON 对象，不另写聊天前言：

- `schemaVersion: "chatfate.reading.interpretation.v1"`
- `product`: 本次的 `daily` 或 `annual`
- `synopsis`: 1–4 段，先给主要判断与最关键的理由、牵制或条件。
- `chapters`: 2–6 个按题目需要选择的完整主题；每章 `{id,title,teaser,paragraphs}`。id 为稳定英文短标识；title 是短中文标题；teaser 如实概括这一章新增的问题；paragraphs 为 1–6 个自然段。
- `previewChapterId`: 选一章完整、直接回答核心问题的内容供免费预览。
- `nextSteps`: 0–3 段，按实际情境给下一步，不按动作白名单机械选词。
- `boundaries`: 0–3 段，只写相关的方法限制、资料缺口或分歧；方法说明集中一次。

每段 Passage 是 `{text,source,evidenceRefs,confidence}`：

- `source: "calculated"`：复述计算资料，引用真实 `calculatedFacts.*`。
- `source: "traditional"`：说明传统取象及其推导，引用所用计算资料；一般为 medium 或 low。
- `source: "user"`：复述明确的 `userContext.*`。不可把自己的推论归给用户。
- `source: "practical"`：现实分析、建议或方法限制；可引用背景与相关事实，也可以 `evidenceRefs: []`，不用硬挂排盘路径包装常识。

`confidence` 为 high/medium/low，描述依据直接程度，不是事件概率。混合段落应拆开，或按其主要结论标 practical；不能把传统解释标成计算事实。

## 怎样写出新增价值

概要让人马上读懂这一问。免费完整章把最关键的判断讲透，包含真实理由和必要限制。其他章节分别回答新的问题：相反依据怎样权衡、另一领域如何适用、何种条件改变判断、某个时间段与另一个有何不同。不得把概要改写成长文或把同一句话拆成三个层次。

标题不必固定，篇幅由信息决定。今日通常两章就够；年度需要更充分的依据时再展开。teaser 不制造悬念，不以凶煞、隐患或付费揭晓引诱解锁；付费章不能只有免责声明、术语表或通用的“确认—安排—复盘”。

依据足够才断；有支持也有牵制时说明二者分别影响什么。行动来自用户的问题和现实条件，例如学习方法、创作取舍、生活照料、沟通或选项比较，不把所有问题改造成项目管理。没有行动必要时，允许只把问题解释清楚。

## 保持可核验

- 事实只读，引用路径必须存在；具体干支、十神、关系和时间要逐项对应。需要查路径时读 [证据指南](references/evidence-guide.md)。
- 五行数量不是旺衰得分；不输出尚未经规则计算的喜用神、格局或合化结论。可以充分解释已有的时令、显隐、十神和关系，不能用省略传统依据来冒充稳妥。
- 冲、合、刑、害先指传统关系，不直接证明事故、婚变、他人动机或现实成败。神煞只作补充，不能单星定论。
- 时间口径以实际 profile 为准：T1的校正计算时钟用于四柱及起运；旧profile仅日时口径按旧说明。流年按立春民用日期切年，dailyTransit取当天正午实际时刻，两者在交节日可不同；annualTransit月份保留真实交节时刻。大运表只列年份时，不声称知道精确交运月日。
- 不依赖用户事后反馈改盘或宣称预测验证，不提供健康诊断、死亡预言、投资收益或开运消费保证。涉及真实重大问题时先依据现实情况和对应专业帮助。

写完自行通读一次：是否回答了本问；用户自述是否被标清；免费章是否完整；每个新增章是否带来不同信息；付费前后的结论是否一致。结构检查通过不等于文字有价值，不额外调用模型逐章审稿。

只有方法难以取舍时读 [解释框架](references/interpretation-framework.md)；涉及经典出处读 [来源说明](references/source-notes.md)，涉及敏感表述读 [安全语言](references/safety-language.md)。需要多份时批量读取，已读内容直接复用。
