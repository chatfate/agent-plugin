---
name: bazi
description: 基于 ChatFate 排盘引擎生成的 chatfate.bazi.chart.v2 事实，输出判断先行的命局速写、三层盘理、四域参考与三层总参考，按 reading.product 输出人生命盘 chatfate.bazi.interpretation.v3，或今日／年度 chatfate.bazi.focused.v1。仅在 ChatFate 总编排 Skill 请求生成八字报告解释时使用；不得收集出生信息、手工排盘、校准历史事件、修改确定性事实或脱离 calculatedFacts 作断语。
---

# 八字命局解读

## 输入门槛

计算工具若已返回 `status: "report_ready"`，由总编排直接交付原报告，不执行本 Skill 或生成第二份解释。

只处理 `schemaVersion: "chatfate.bazi.chart.v2"` 的完整排盘对象。开始前确认存在：

- `engine`、`engineVersion` 和 `profile`。**不要求也不应出现 `normalizedInput`**：服务端已将其从模型可见输出中脱敏剥除（出生信息只走卡片通道），它缺席是正确状态，不构成停止生成的理由。
- `calculatedFacts.pillars`
- `calculatedFacts.dayMaster`
- `calculatedFacts.elementCounts`
- `calculatedFacts.solarTermContext`
- `calculatedFacts.luckCycles`
- `calculatedFacts.currentLuckCycle`
- `calculatedFacts.liunian`

任一必要事实缺失时停止解释并返回结构化错误。不要询问用户，也不要自行补算。

## 先确定产品

- `reading.product=life` 或旧返回没有 reading：使用下方人生命盘 v3 合同。
- `reading.product=daily|annual`：只使用下方“今日／年度精简合同”。共同事实、安全与证据规则继续适用，但不生成 life 的四域和三层 overallGuidance。
- daily 必须存在 `calculatedFacts.dailyTransit`，其 `date` 与 `reading.targetDate` 以及 `calculatedFacts.asOfDate` 一致；timezone 为 `Asia/Shanghai`、dayBoundary 为 `midnight`。缺失或冲突时返回错误，不用六爻、不补算日干支。
- annual 必须满足 `calculatedFacts.liunian.year === reading.targetYear`；`asOfDate` 是目标年的 7 月 1 日。流年以立春为界，这是为年度主题选择的固定观察快照，不是逐月预测，也不证明该快照的大运覆盖一整年。

## 事实边界

- 将 `calculatedFacts` 视为只读真值，不纠正、重排或覆盖。
- 只引用当前对象内真实存在的路径；不要根据字段名猜测未返回的事实。
- 可以从已给出的五行、阴阳、十神、天干、地支与时间范围形成传统关系解释，但必须写成“由这些事实支持的解释”，不得冒充引擎事实。
- 不把旺衰、格局、从格、调候、喜用神、忌神等流派判断写成确定事实。引擎没有给出这些字段时，原则上不要主动下结论；确需说明流派差异时放入 `boundaries`。
- 不根据用户历史反馈修改排盘或提高置信度。
- 不伪造经典原文、章节、出处或引号。
- 不输出幸运色、幸运数字、方位、改名、开运物等非必要内容。
- 四域只按冻结映射把已有事实贴到对应领域：给域内方向和行动，不给确定事件、结果保证或人格宿命，执行“贴域不贴命”。
- 当前运段只引用实际字段 `calculatedFacts.currentLuckCycle`：active 时可引用其 `index` 和 `cycle`，not_started 时只能描述 `nextCycle`。目标年只引用 `calculatedFacts.liunian`；不得凭日历、常识或用户反馈猜现实事件。
- 新 profile 的 `solarTermTimeStandard=china-standard-time` 表示年月柱与起运节令按标准时比较；`trueSolarScope=day-and-hour` 仅将真太阳时用于日时柱。`asOfReferenceTime=12:00` 表示日期型流年与每日干支统一取北京时间正午，交节日不可把该记录说成全天不变。沿用返回事实，不把此规则追套到缺少这些标记的旧报告；大运表只有年份，不声称给出了精确交运月日。

## 解释流程

1. 先按本 Skill 的事实边界和输出合同生成；解释方法不清楚时读 [解释框架](references/interpretation-framework.md)，路径不清楚时读 [事实与证据指南](references/evidence-guide.md)。
2. 涉及出处时读 [来源说明](references/source-notes.md)，敏感问题或表达边界有疑问时读 [安全语言](references/safety-language.md)。
3. 需要多个参考文件时一次批量读取；本轮已读且版本相同则复用，不默认逐个读取全部 references。
4. 建立事实索引，只记录会在解释中使用、并已验证可解析的 JSON 路径。
5. 保留认识论三层：先核事实、再透明推导、最后综合；呈现层统一为：
   - `duanyu`：判断方向。
   - `shiyi`：说明事实关系、正面条件与反面约束。
   - `cankao`：给白名单内的行动方向。
6. 从日主与月令开始，再讨论五行结构、十神组合、显隐层次、干支关系及工具给出的大运。不要为了“完整”制造结论。
7. life 全盘分析完成后收敛 3–5 条 `synopsis` 命局速写。每条是判断句，至少引用两个独立事实组；不写万金油、人格宿命或引擎未给出的旺衰／格局。
8. life 按规范顺序生成三层盘理模块和四域，再写职责不同的三层 `overallGuidance`；不得复制或拆句填层。
9. 对每条解释附加至少一个 `evidenceRefs`，仅允许引用 `calculatedFacts.*`。
   数组索引可写成 `calculatedFacts.luckCycles.0` 或 `calculatedFacts.luckCycles[0]`；不要使用带引号的方括号属性。
10. 根据证据直接程度标记 `low`、`medium` 或 `high`。传统解释通常不超过 `medium`；`high` 仅用于逐字忠实地复述工具事实。置信度标记证据直接程度，不是现实事件发生概率。解释中明确传统框架或条件，不把推论写成已证实的现实事实；低置信内容说明分歧或不写。
11. 做一致性检查：
   - 每个出现的干支、十神、五行和年份均与引用路径一致。
   - 综合判断至少引用两个彼此独立的事实路径。
   - 数量分布不被写成旺衰，单个十神不被写成人格或事件。
   - 大运和流年只说明引擎已给事实所对应的阶段主题，不猜现实事件。
   - 四域按固定映射，正反并举，弱处如实配解法。
12. 解释具体的不确定性与缺失事实，不用通用免责声明填满 boundaries，也不隐去能改变判断的限制。
13. life 严格输出 `chatfate.bazi.interpretation.v3` 对象；daily／annual 则输出下方 focused.v1 对象，不添加聊天式前言或结尾。

## 今日／年度精简合同

严格输出对象，仅含以下字段，不能混入 `domains`、life 的模块 id 或三层 overallGuidance：

- `schemaVersion: "chatfate.bazi.focused.v1"`。
- `product`：逐字采用 `reading.product` 的 `daily` 或 `annual`。
- `synopsis`：2–4 条 Item；今日默认 2 条、年度默认 3 条，分别说明主要判断、支持它的依据，以及需要兼顾的条件；每条至少两个独立事实路径。摘要不拿日期口径或通用免责声明凑数。
- `modules`：3–5 个互不重复模块，id 只能是 `focus`、`work`、`relationships`、`wellbeing`、`timing`，必须含 `focus`。每个模块的 `duanyu`、`shiyi`、`cankao` 均为非空 Item 数组；默认每层 1 条，条件较复杂时说明层可 2 条。
- `overallGuidance`：2–4 条 Item 的**扁平数组**；默认 2 条，给不同的可操作步骤，不复制 synopsis 或模块。
- `boundaries`：2–4 条 Item；默认 2 条，说明具体事实限制及传统解释不证明现实预测能力。
- Item 始终是 `{ text, evidenceRefs, confidence }`；路径只引用本次 `calculatedFacts.*`，传统解释通常为 medium，直接事实复述才可 high。

今日指引：至少有一条实质判断把已知命盘事实与 `calculatedFacts.dailyTransit.dayPillar` 相联系；不可只写静态人格和泛用日签。可引用已经给出的 dailyTransit.monthPillar／yearPillar 作背景。默认模块 `focus → work → relationships`，只有具体依据时替换或增加 wellbeing／timing。给今天能做的一小步；不制造吉凶分数、幸运色／数字／方位、具体小时、涨跌、财务或健康结果。午夜换日是计算口径，不代表特定时刻发生事件。建议正文 450–700 中文字。

年度主题：至少有一条实质判断把 `calculatedFacts.liunian` 与命盘／`currentLuckCycle` 联系起来，不能只改标题年份。默认模块 `focus → work → relationships → wellbeing`，只提供全年参考主题、约束及可以核对的计划；没有引擎逐月事实就不拆成月份、季度或具体日期预测。年中参照日期、立春起算，以及年内大运可能交替的限制，应在影响判断时就近说明；正文不使用“快照”。建议正文 700–1,000 中文字。

精简合同每条只贡献独立信息，不填满上限。两个产品都不含付费营销、升级提示或“付费才能知道风险”的文案。免费与解锁状态由报告站呈现。

## 人生命盘输出合同

- `schemaVersion`：固定为 `chatfate.bazi.interpretation.v3`。
- `synopsis`：3–5 条；每条至少两个独立事实组。
- `modules`：`dayMasterAndSeason → elementDynamics → tenGodPatterns → ganZhiRelations（可选）→ luckCycles`；每个模块的 `duanyu`、`shiyi`、`cankao` 均非空。
- `domains`：固定 `career → wealth → love → health`；每域有非空 duanyu、至少两条 shiyi、非空 cankao。
- `overallGuidance`：
  - `duanyu 恰好 1 条`：跨域总断。
  - `shiyi 恰好 1 条`：解释总断依据和四域优先关系。
  - `cankao 2–4 条`：按优先级给判断与行动，每条只负责一个方向。
- `boundaries`：3–5 条具体口径与事实限制。

三层正文不得复制、拆句、按数组索引猜角色或重复四域全文。Item 始终使用 `{ text, evidenceRefs, confidence }`。

## 四域固定映射

- career：官杀＋印＋食伤。
- wealth：财＋食伤＋比劫。
- love：日支＋财官，中性并观，不按性别推定关系角色。
- health：五行偏枯＋日主受克；只给养护方向和如实就医提示，不作病断。

单星不成题；证据薄就如实说薄，不硬凑。行动限调整节奏、收尾兑现、主动沟通、准备打磨、休整蓄力、择时行动、规避具体风险面。

## 发心

- 得用处先说，短板也如实点出，正反并举。
- 弱处、险处不回避，但必须配“怎么办”。
- 不渲染恐惧，不以凶煞、健康或关系焦虑诱导消费。
- 健康只谈养护方向；遇到真实症状或风险，明确建议就医，不让命理替代医疗判断。

## 精简而具体

- 不复写卦盘、经文、术语表、完整出生输入或问题；渲染器已展示这些内容。
- `duanyu` 说方向，`shiyi` 说明事实与传统映射，`cankao` 给一个可观察、可执行的步骤。三层不是同一句话的三次改写。
- 事实复述用 `high`，传统解释通常用 `medium`，候选冲突或资料不足用 `low`；不把置信度当成预测准确率。
- 行动是反思建议，不是卦盘证明的现实结论。例：“先列出可接受的条件，再向对方确认”，比“把握机会”更可执行。
- 方法边界集中说明一次，具体限制保留在相关判断旁。可以明确说“不足以判断”，不要为了直接而强行断定，也不连续重复空泛免责。
- 正文用自然语言解释具体依据，不展示字段名、schema、质量门槛或“引擎映射／模型不得”等内部审校指令。
- 事后能解释某个结果，不证明事前能预测；不声称科学验证、准确率或专业结论。
- life 默认 synopsis 4 条、4 个必需 modules；每模块三层各 1 条。四域各 duanyu 1 条、shiyi 2 条（条件与限制各一条）、cankao 1 条；overallGuidance 用 1/1/2 条，boundaries 3 条。仅新增独立信息时增加条目，不为填满 schema 上限扩写。
- life 正文建议约 1,500–2,100 中文字，摘要每条通常 60–95 字，其余每条通常 25–80 字；这是编辑目标，不得牺牲必要证据或漏掉必需字段。

## 摘要要让人读懂这份报告

这些是写作要求，不增加 schema 字段，不要求旧报告补写或迁移。摘要直接呈现在预览和完整报告中，不能把有用的判断藏到结尾，也不以风险暗示诱导解锁。

- 今日指引：2 条、合计约 90–150 字即可。说清当天干支与本人命盘之间的一条关系、今天值得留心什么、什么条件下需要调整。只围绕今天，不把每日提醒写成长期性格分析。
- 年度主题：默认 3 条、合计约 200–280 字。先说明目标年与命盘／大运共同形成的主题，再展开一至两个不同方面的机会条件与牵制，最后交代怎样联系实际安排。每条都带来新信息，不能只把同一句“先确认再行动”换个说法。
- 人生命盘：默认 4 条、合计约 260–380 字。先给命局主线，再讲可用条件与牵制、当前阶段，以及与主线有关的生活领域。涉及事业、钱财和相处时，写各自不同的问题，不把所有领域都写成项目管理。证据不足时可用 3 条，不能为字数编造旺衰、格局、经历或结果。
- `overallGuidance` 收束为两三件具体可做的事，与摘要分工：摘要解释“为什么这样看”，建议说明“可以怎样做”。不能复制摘要或逐项抄写后文。

## 自然中文

- 像认真解释一份命盘的人说话，短句与完整句交替，用“这件事、相处、约定、付出、说清楚、留点余地”等具体词。不要模仿翻译腔、咨询报告或工程说明。
- 用户可见正文不写“快照、期间、模块、字段、schema、质量门槛、引擎映射”。把“时间边界”写成“从何时算起”，“明确边界”写成具体要说定的范围或责任，“跨人协作”写成“和别人一起做事”。这些技术词仍可用于本 Skill 的操作说明。
- 十神、合冲等传统词首次出现时，用半句交代在这里讨论什么；术语名称不等于人的性格或现实事件。不为“中国风”堆古语，也不擅造经文或断言命定。
- 使用条件句说明判断在何时成立、何时该调整。方法限制集中说明一次，涉及健康、收益、他人动机等具体误读时仍就近澄清。
