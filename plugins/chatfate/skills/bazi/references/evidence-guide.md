# 事实与证据指南

## 可直接引用的事实

- `calculatedFacts.pillars.year|month|day|hour`
  - `stem`、`branch`
  - `stemElement`、`stemYinYang`、`branchElement`
  - `tenGod`
  - `hiddenStems[index].stem|tenGod|weight`
- `calculatedFacts.dayMaster.stem|element|yinYang`
- `calculatedFacts.elementCounts`
- `calculatedFacts.solarTermContext.previousName|previousAt|nextName|nextAt`
- `calculatedFacts.luckCycles[index]`
  - `startAgeYears`、`startYear`、`endYear`
  - `pillar` 及其子字段
- `calculatedFacts.currentLuckCycle`
  - active：`status|index|cycle|precision|nearBoundary`
  - not_started：`status|nextCycle|precision|nearBoundary`
- `calculatedFacts.liunian.year|pillar|dayMasterTenGod`
- `calculatedFacts.shenSha[index]`
- `calculatedFacts.ganzhiRelations[index]`

引用对象路径适合概括该对象；声称具体值时优先引用精确叶子路径。数组下标必须真实存在。

## 透明推导

以下内容可以作为解释，但不是引擎事实：

- 依据两个 `*Element` 字段说明基础生克关系。
- 依据多个 `tenGod` 字段说明某类传统主题反复出现。
- 依据显性天干与藏干分别说明“表层/补充层”的观察角度。
- 依据运柱和日主字段说明某十年区间可能放大的传统主题。

透明推导必须：

1. 引用所有参与推导的事实路径。
2. 使用 `medium` 或 `low`。
3. 使用判断句；不确定性集中进入 boundaries，不用“传统框架下可能”“可作为线索”等模板稀释正文。

## 不可从当前事实推出

除非当前对象明确给出对应字段，否则不能推出：

- 日主旺衰及评分。
- 格局、从格、专旺或合化是否成立。
- 喜神、用神、忌神。
- 流月、具体应期和现实事件。
- 婚姻、职业、财富、健康或具体人生事件。

`shenSha` 与 `liunian` 已是合法事实，但只可按实际字段引用；神煞单星不定吉凶，流年事实不等于现实事件。四域解读是对已有事实的固定传统映射，不授权补算以上禁区。

## 证据密度

- 单纯复述一个事实：至少一个精确路径。
- 柱间关系：至少两个柱位路径。
- 十神组合：至少两个十神路径。
- `synopsis`：每条至少两个独立事实组，例如柱位＋五行分布、十神组合＋当前运段。
- 域断语：至少引用该域固定映射中的两个相关事实路径；证据不足就如实说薄。
- 当前段提示：active 只引用 `calculatedFacts.currentLuckCycle.index|cycle`，not_started 只引用 `nextCycle`，不得混用。
- 当前年提示：必须引用 `calculatedFacts.liunian`，不得自行计算当前年事实。
- 综合主题：至少两个来自不同事实组的路径。
- 不确定性：引用导致限制的现有上下文；不要引用不存在的字段来证明“缺失”。

## 输出前检查

- 路径能从 `calculatedFacts` 根对象逐段解析。
- 文本中的每个具体干支、十神、五行、年龄和年份都可在引用中找到。
- `high` 条目没有解释性形容词。
- synopsis、四域和 overallGuidance 的综合判断都有两个独立事实组。
- 当前大运没有把 `nextCycle` 写成当前运，当前年没有扩写为现实事件。
- 每条不超过一个主要结论，避免一个引用支撑多项无关断语。
