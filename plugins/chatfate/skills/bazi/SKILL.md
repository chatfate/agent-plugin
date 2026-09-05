---
name: bazi
description: 基于 ChatFate 排盘引擎生成的 chatfate.bazi.chart.v2 事实，输出判断先行的命局速写、三层盘理、四域参考与三层总参考，形成 chatfate.bazi.interpretation.v3。仅在 ChatFate 总编排 Skill 请求生成八字报告解释时使用；不得收集出生信息、手工排盘、校准历史事件、修改确定性事实或脱离 calculatedFacts 作断语。
---

# 八字命局解读

## 输入门槛

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

## 事实边界

- 将 `calculatedFacts` 视为只读真值，不纠正、重排或覆盖。
- 只引用当前对象内真实存在的路径；不要根据字段名猜测未返回的事实。
- 可以从已给出的五行、阴阳、十神、天干、地支与时间范围形成传统关系解释，但必须写成“由这些事实支持的解释”，不得冒充引擎事实。
- 不把旺衰、格局、从格、调候、喜用神、忌神等流派判断写成确定事实。引擎没有给出这些字段时，原则上不要主动下结论；确需说明流派差异时放入 `boundaries`。
- 不根据用户历史反馈修改排盘或提高置信度。
- 不伪造经典原文、章节、出处或引号。
- 不输出幸运色、幸运数字、方位、改名、开运物等非必要内容。
- 四域只按冻结映射把已有事实贴到对应领域：给域内方向和行动，不给确定事件、结果保证或人格宿命，执行“贴域不贴命”。
- 当前运段只引用实际字段 `calculatedFacts.currentLuckCycle`：active 时可引用其 `index` 和 `cycle`，not_started 时只能描述 `nextCycle`。当前年只引用 `calculatedFacts.liunian`；不得凭日历、常识或用户反馈猜现实事件。

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
7. 全盘分析完成后收敛 3–5 条 `synopsis` 命局速写。每条是判断句，至少引用两个独立事实组；不写万金油、人格宿命或引擎未给出的旺衰／格局。
8. 按规范顺序生成三层盘理模块和四域，再写职责不同的三层 `overallGuidance`；不得复制或拆句填层。
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
13. 严格输出 `chatfate.bazi.interpretation.v3` 对象，不添加聊天式前言或结尾。

## 输出合同

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
- 事后能解释某个结果，不证明事前能预测；不声称科学验证、准确率或专业结论。
- 默认 synopsis 3 条、4 个必需 modules；每模块三层各 1 条。四域各 duanyu 1 条、shiyi 2 条（条件与限制各一条）、cankao 1 条；overallGuidance 用 1/1/2 条，boundaries 3 条。仅新增独立信息时增加条目，不为填满 schema 上限扩写。
- 正文建议约 1,200–1,800 中文字，每条通常 25–70 字；这是编辑目标，不得牺牲必要证据或漏掉必需字段。
