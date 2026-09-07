# 报告契约

本文件供 daily/annual/question 使用；life 标准路径使用计算结果中 `authoring.version: life.v1` 对应的完整 `authoring.instructions`，所需方法与原文契约已随事实返回，不再额外读取本文件。

## 写入与恢复

- 模型只提交 `{ calculationId, interpretation }`。`submissionId`、`calculationId` 是短期不透明标识，外层报告固定为 `chatfate.report.v2`。
- daily/annual/question新解释用 `chatfate.reading.interpretation.v1`。`product` 为 daily／annual／question，必须等于计算工具返回的 reading.product；八字对应前两种，六爻对应 question；新life使用下方独立合同。
- chart、reading、userContext、报告编号、删除凭据与链接都由服务器从已保存的计算结果生成。不得重传原始输入、手拼 envelope、改写事实或给用户添加背景。
- 旧八字 v3、focused v1 和六爻 v2 解释只兼容已开始的旧客户端流程及历史读取。新流程不使用旧三层模板。
- 同一 calculationId 重试返回原报告；已经成功写入后，换一份解释不会覆盖它。网络失败保留原参数；life技术格式或保存失败保留原文并如实报告，不二次修稿；其他产品仅修改指出字段。

## life原文合同

life读取本次单份minimalFacts与完整 `authoring.instructions`，一次形成17 MODULE。新life interpretation仅有schemaVersion:"chatfate.bazi.life.interpretation.v1"、product:"life"、text三个字段；精确资源为chatfate://schemas/bazi-life-interpretation-v1。正文原文唯一权威保存，不另生成结构化副本；不加逐段source/evidenceRefs/confidence，不要求六句固定说明字段。格局/喜用是有条件的传统解释，不需要先由机器定论。

服务器仅按 Schema 验证字段与非空字符串，不扫描正文含义、不对自然语言判断设拒收门禁；模块与标签的解析诊断用于展示降级，保留原文。新事业、财富栏目按方法表达修订 r2，旧报告标签继续识别，不要求迁移。本人可读的免费范围为基础命盘资料与完整Hero命局图版，图版后进入解锁区；“命局与性情”及以后的分析正文不提前下发。旧reading-life仅保留原有synopsis概览与实际章节名称，不返回整章或个性化teaser；旧完整正文不迁移。该范围只控制读取投影，不改变Writer和完整保存合同。技术格式或保存失败如实报告；网络重试仅复用同一原文和calculationId，不调用第二模型重写。

## daily/annual/question解释形状

精确 JSON Schema 可由 `chatfate://schemas/reading-interpretation-v1` 读取；其life枚举仅保留旧客户端兼容，新life不选择此结构。其他产品当前写作结构：

```text
schemaVersion: "chatfate.reading.interpretation.v1"
product: "daily" | "annual" | "question"
synopsis: Passage[1..4]
chapters: { id, title, teaser, paragraphs: Passage[1..6] }[2..6]
previewChapterId: 已有章的 id
nextSteps: Passage[0..3]
boundaries: Passage[0..3]
```

章节 id 是不重复的小写字母、数字或连字符，开头须为字母，最多 48 字符。标题最多 48 字、teaser 最多 180 字；teaser 准确说明这一章解释什么，不作营销悬念。每个 Passage：

```text
{ text, source: "calculated" | "traditional" | "user" | "practical",
  evidenceRefs: string[], confidence: "high" | "medium" | "low" }
```

- calculated：引擎真实输出，引用 `calculatedFacts`，正文数值与事实逐字相符。
- traditional：传统体系内的解释，引用支持该判断的 `calculatedFacts` 具体关系，并说清推导与反向条件；不能把引用存在当作现实预测成立。置信度不高于 medium。
- user：明确来自用户的陈述，引用实际存在的 `userContext` 或六爻 `normalizedInput`；不得把背景反写成排盘推出的经历。
- practical：现实建议或常识，可不带证据引用；不要为了凑 evidenceRefs 把一条普通建议假装成命理结论。

每段最多 1600 字、16 个引用是容量上限，不是写作目标。拆分确有不同来源的事实与解释；不要拆成重复的短句填表。概要先直答本问，各章自然展开不同依据；行动只在有帮助时给，不设动作词白名单，不强制每章都有建议。用户正文不出现字段名、质量门槛、模型审校指令；方法说明集中一次，具体限制在相关判断旁说清。

## 产品依据与预览

- daily 必须把当天 dailyTransit 与本命事实合看；不编造幸运色、小时吉凶、价格涨跌等未计算内容。
- annual 结合目标年与本命。按月解释只用 annualTransit.months 的真实节气段、干支和关系，不将节气月当公历月，不保证事件会发生。
- question 围绕所问解释卦意、用神、动变及冲突。比较题只分析用户真实给出的选择；缺少条件时直说不能据此区分，不凭空指定 A/B 谁更好。敏感问题按表达边界收窄，不输出命理诊断或交易指令。
- life 使用上述独立原文合同，四域完整且独立，弱信号短写，遵循17模块职责。以下previewChapterId等字段只属于其他产品。

previewChapterId 指向一篇完整、可独立阅读的篇章；与概要一起交代当前判断、主要依据与重要限制。其余篇章提供新的分析，不复述预览，不把必要限制留作付费内容。nextSteps 和 boundaries 可以为空；正文已充分说清时，不另凑重复条目。

## 存储与访问

- 新报告链接用于定位；读取仍需生成报告的 Google 账号与对应权益。早期未关联账号的报告保留持链接读取的兼容访问。
- 报告保存在 ChatFate 自有站点的 D1，默认保留至用户删除。卡片暂存用于本次继续和恢复，不建立可复用出生档案。
- accessId 为 256-bit 随机值，放在 `/report/<accessId>` 路径。新链接没有 fragment 或客户端解密密钥。
- writeSecret 只保护写入，deleteToken 只删除对应报告。不要把这些凭据交给读者或写进正文。

## 交付

工具成功后沿用本轮 ChatFate 内置 Browser 标签打开返回的 reportUrl；不可用时仍交付完整链接。最终回复 `报告已生成：<reportUrl>`，逐字保留路径，不截断、不改写、不复述报告正文。链接不写日志、公开位置或无关工具，也不向非预期读者转发。
