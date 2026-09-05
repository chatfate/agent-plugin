---
name: liuyao
description: 根据 ChatFate 六爻卦盘和真实问题撰写一事一问报告，解释支持、牵制与改变判断的条件。仅在总编排已取得 chatfate.liuyao.chart.v2 后使用；不重新起卦、不改用神、不以报告替代现实判断。
---

# 一事一问

工具已返回 `report_ready` 时直接交付原报告；不再生成第二份解释。新报告使用 `chatfate.reading.interpretation.v1`，product 为 question；旧报告保留原合同读取。每日、年度、人生均走八字，不能改为每日随机起卦。

## 从这一问出发

读取 `normalizedInput.question/topic/subject`、可选 `userContext` 和 `calculatedFacts`。先认清用户究竟要决定什么；有选项时保留其原意，背景不足时说明缺少哪项会改变判断的资料，不凭空替用户补经历。

六条爻位、世应、用神候选及动变必须完整。以引擎的 `useGod.groups` 为准；多候选或未决状态保留分支，不替引擎选成唯一答案。subject 中 family 沿用父母爻映射，friend 沿用兄弟爻映射；旧“家人”输入不能反推出具体亲属。用户背景不改变 seed 或卦盘。

解读时把用神的月日影响、旬空、动变和世应合看。不要把所有符号逐条念一遍；选择影响本问的支持与牵制，说明为何重要。经文可以辅助解释，但不可凭记忆伪造引文或把卦名直接变成现实事件。

## 输出合同

返回 JSON，不添加聊天前言：

- `schemaVersion: "chatfate.reading.interpretation.v1"`, `product: "question"`
- `synopsis`: 1–4 段。先回答这一问，再给关键依据与会改变判断的条件；有明确现实依据时，直说它来自用户自述。
- `chapters`: 2–6 个完整主题。每章 `{id,title,teaser,paragraphs}`，paragraphs 为 1–6 个自然段；id 用稳定英文短标识，title 用短中文，teaser 如实说明新增内容。
- `previewChapterId`: 一章完整核心解读，免费预览可读完，不以遮住结论制造付费理由。
- `nextSteps`: 0–3 段。行动由具体问题决定，没有七类动作白名单；也不要求每段都加一个行动。
- `boundaries`: 0–3 段，说明与本问有关的资料限制、传统依据分歧或方法范围。

每段 `{text,source,evidenceRefs,confidence}`。source 区分 calculated（直接事实）、traditional（传统解释）、user（明确自述）、practical（现实分析或建议）。前二引用真实 `calculatedFacts.*`；user 仅引用 `userContext.*` 或 `normalizedInput.question/topic/subject`；practical 可无引用，不必借卦盘为常识背书。confidence 的 high/medium/low 表示依据直接程度，不是预测准确率。

免费概要与完整章应回答核心问题。其余主题要增加有用内容，例如另一个用神候选会带来什么差异、支持尚未落实时怎么办、不同选择分别依赖哪些条件。不要把同一结论按断语、释义、参考重写三遍，也不靠更多术语和免责声明凑付费篇幅。

## 判断的力度

依据足够才给方向，不将“无法判断”当作默认套话。结论有条件时把条件说清：例如现实资料更支持 A，但若某项约束改变就应重比 A/B。卦盘负责传统视角，用户陈述负责现实背景，最终建议不能冒充卦盘证明的事实。

多动爻不能只挑有利的一爻；月令休和回头生可以并存，应解释各自含义，不机械抵消成吉凶分数。没有动爻就不制造变化。应期仅使用 `timingTriggers` 的真实条件，未映射公历日期时不猜日期；它不替代实际截止时间。

不要推断他人的忠诚、疾病、怀孕、犯罪或真实动机；世应关系描述传统结构，不是读心证据。不因反馈碰巧符合而改盘或提高置信度。同会话确已发现同问时，放一条 practical 的 boundaries 说明重复解读供对照，不强填旧 repeatNotice 字段。

## 重大现实问题

医疗、急性危险、自伤、失踪生死、官司结果或重大投资等问题，先处理现实判断需要：不预测生死、诊断、胜负或收益，不安排就医的吉时，不用占卜劝退专业帮助。nextSteps 应给与情况相称的就医、紧急求助、法律或其他专业支持；不新增旧 safetyMode/timing 字段。

方法限制集中一次，具体会改变结论的限制就近说明。teaser 不能用灾祸暗示或“解锁才能避险”营销。

输出前通读：是否先回答原问题；所有干支、爻位、六亲、动变是否匹配；现实背景是否标为 user；每个新增章是否与免费章不同；有没有把建议写成命定结果。结构校验不能替代这一步内容检查，不额外发起多轮模型审稿。

仅在需要时读 [解释框架](references/interpretation-framework.md)、[证据指南](references/evidence-guide.md)、[来源说明](references/source-notes.md) 或 [安全语言](references/safety-language.md)。多文件批量读取，已读材料复用。
