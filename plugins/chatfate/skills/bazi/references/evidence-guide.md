# 分支说明

life按计算结果的authoring.instructions读取minimalFacts，正文不要求逐段引用/source/confidence；格局与喜用由完整方法解释，不能冒充引擎事实。下文路径指南只用于daily/annual与旧合同。

# 八字依据与来源

calculated/traditional 段只引用实际存在的 calculatedFacts 路径；user 段只引用 userContext；practical 段可引用相关背景或使用空数组。实际建议不能因挂了一个日柱引用就变成计算事实。

基础资料包括 pillars、dayMaster、elementCounts、solarTermContext、luckCycles、currentLuckCycle、liunian、ganzhiRelations、shenSha。藏干只有实际返回的 stem 与 tenGod，不假设存在 weight。时辰不详时 hour 不存在，不引用它。

currentLuckCycle 为 active 时使用 cycle；not_started 时使用 nextCycle。precision 是出生输入精度，大运年表不等于精确交运日期。

每日新增 dailyTransit 的 dayPillar/monthPillar/yearPillar 与可选 relations。每条关系写明 transit 是 day 还是 month、natalPillar 对应哪一柱、type、chars 等；不可把当天关系当作本命永久关系。

年度新增 annualTransit.months：12 个按真实交节时刻划定的区间，index 0 为立春起的寅月，index 11 延伸到下一年立春。startAt/endAt 是北京时间绝对时刻，pillar 的十神相对本命日主，relations 只描述该流月与本命的成对关系。没有列出的三柱成局、流月与大运关系或精确交运日期，不能自行声称已由工具计算。

柱位、十神、元素、时间和关系应逐项对应引用；引用数组可用点号下标或数字方括号。不要引用不存在的字段来证明“缺失”。一个 source=calculated 段不能夹带性格、吉凶和现实事件判断。

月份关系和出生信息虽然可计算，格局、旺衰、喜用神及现实结果并未由这些字段得到验证。引用存在只证明路径有效，不证明推论成立。
