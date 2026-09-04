# 六爻事实与证据指南

## 事实路径

常用顶层路径：

- `calculatedFacts.baseHexagram`
- `calculatedFacts.changedHexagram`
- `calculatedFacts.ganZhiTime`
- `calculatedFacts.voidBranches`
- `calculatedFacts.lines[index]`
- `calculatedFacts.movingLinePositions`
- `calculatedFacts.shiLinePosition`
- `calculatedFacts.yingLinePosition`
- `calculatedFacts.hiddenSpirits`
- `calculatedFacts.useGod`
- `calculatedFacts.spiritSystems`
- `calculatedFacts.timingTriggers`
- `calculatedFacts.globalSpiritStars`
- `calculatedFacts.hexagramRelations`
- `calculatedFacts.derivedHexagrams`
- `calculatedFacts.guaShen`

具体值优先引用精确叶子路径。可选字段为空或不存在时，不得生成指向它的引用。

## 六爻路径

`calculatedFacts.lines[index]` 使用数组顺序保存六爻，但正文中的爻位必须以该对象的 `position` 字段为准，不要把数组下标直接写成“第几爻”。

可用子字段包括：

- `lineType`、`isChanging`、`movementState`、`movementLabel`
- `sixRelative`、`sixSpirit`
- `naJia`、`element`
- `isShi`、`isYing`
- `voidState`
- `strength.state|isStrong|specialStatus|evidence`
- `monthDayInfluence.monthAction|dayAction|evidence`
- `changedLine`
- `hiddenSpirit`
- `shenSha`
- `longLife`

字段是否可用以实际对象为准，不按列表补齐。

## 用神证据

按照以下层级引用：

1. `calculatedFacts.useGod.targetRelatives`
2. `calculatedFacts.useGod.groups[index].selectionStatus`
3. `calculatedFacts.useGod.groups[index].selected`
4. 对应候选的 `calculatedFacts.lines[index]`

若 `selectionStatus` 为 `ambiguous` 或 `missing`，只能按引擎给出的选择说明和候选分支进行解释；即使对象含有 `selected`，也不要把它写成无争议的唯一用神。

## 最低证据要求

- 卦名复述：一个卦名路径。
- 用神分析：用神组路径加对应爻路径。
- 旺衰分析：对应爻的 `strength` 加 `monthDayInfluence`。
- 动变分析：动爻位置加对应 `changedLine`；涉及变卦时再引用 `changedHexagram`。
- 世应比较：世爻与应爻各至少一个路径。
- 应期线索：对应 `timingTriggers[index]`，必要时加目标爻路径。
- 综合判断：至少两个来自不同事实组的路径。

## verdict 与三层责任

- `verdict.judgment`：至少两个不同事实组，例如用神组＋动爻，或世应＋月日作用。
- `verdict.timing`：每条必须直接引用 `calculatedFacts.timingTriggers[index]`；没有真实 trigger 时为 `null`。
- `verdict.action`：可复用 judgment 的事实路径，不得用行动文本冒充新事实。
- `duanyu`、`shiyi`、`cankao` 的层级由所在数组决定，不增加新字段；三层仍分别满足自身事实密度。
- `questionFocus` 主责所问与引擎用神映射；`trend` 主责卦势与经文释义；`movingLines` 主责动变；`timing` 主责真实触发条件。

## 静态经文豁免

卦辞原文由渲染器从版本化静态经文表读取，不是模型正文，因此不走 `evidenceRefs`。豁免只覆盖静态表逐字输出的经文与出处；模型生成的释经、盘面判断和落到所问的结论仍须引用 `calculatedFacts`。

## 不允许的证据

- `normalizedInput.*`、`warnings.*` 或 `interpretation.*` 不能放入 `evidenceRefs`。
- 不存在的候选、越界数组索引或空可选字段不能作为证据。
- 古籍名称、传统口诀、用户反馈和模型常识不是当前卦盘的事实证据。

## 输出前检查

- 每条路径都能从 `calculatedFacts` 根对象逐段解析。
- 文本中的每个卦名、爻位、六亲、六神、地支和状态都可在引用中找到。
- 没有把一个候选用神写成唯一用神。
- `high` 条目没有传统推断或吉凶判断。
- `verdict.judgment` 的路径来自至少两个事实组，非空 timing 直接落到 `timingTriggers`。
- 经文只来自版本化静态经文表；模型没有凭记忆补写引号。
- 每条文本只承载一个主要结论。
