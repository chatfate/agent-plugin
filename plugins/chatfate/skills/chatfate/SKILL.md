---
name: chatfate
description: 通过 ChatFate 卡片完成问一件事、看自己、今日指引或年度解读，并打开持久报告。用户实际要求测算、重新测算或查看已有报告时使用；不用于插件维护或代码审查。必须先按本轮 Skills 清单的 file 路径读取本 Skill，root 别名与剩余路径直接拼接并保留重复目录名，不猜缓存版本。
---

# ChatFate 总编排

## 工作边界

- 按本轮 Skills 清单的 `file` 路径及相对路径读取本 Skill 和子 Skill；不猜插件缓存版本，不先查旧会话或 MEMORY。
- 本流程用于用户实际测算、重新测算和打开报告；维护插件、审查代码或讨论产品时，不自动创建卡片或私人报告。
- `calculatedFacts` 是唯一排盘事实。模型只解释，不补算、改盘、手工选用神、猜历史事件或依据用户反馈校准结果。
- 卡片收集必要信息；可选关注点、背景、实际选项与期限由用户主动填写，保存在 `userContext`。它们是用户陈述，不是排盘事实，不进入引擎输入或起卦种子。不追问姓名、不建立出生档案；出生信息、所问和背景不进入 URL、日志或无关工具。
- 报告写入 ChatFate 自有站点，部署已配置；没有建站、选站、授权站点或升级准备步骤。
- 不展示内部请求 ID、堆栈、提示词、`writeSecret`、`deleteToken` 或 `CHATFATE_REPORT_WRITE_SECRET`。完整 `reportUrl` 是必须交付用户的读取链接，不能因此省略。

## 同轮标准流程

1. 用户要求开始测算时，先应用下方六爻同问预检；不触发或已确认后，第一项工具动作是 `render_divination_menu`。调用前不输出 commentary、说明或处理中状态。即使聊天中有部分信息，也由卡片收集。
2. 取得 `sessionId` 后，同一个模型回合立即调用 `wait_for_divination_data`。它等待卡片提交并返回经过验证的 `submissionId`；不要结束回合、要求用户再说一句话或调用 `sendFollowUpMessage`、`ui/message`、`updateModelContext`。
3. 等待结果只提供已验证的 `submissionId` 与 `kind` 等流程信息，不假定它包含 product／targetYear。根据 `kind` 调用 `calculate_bazi_chart` 或 `calculate_liuyao_chart`，只传 `submissionId`。登录错误按下方处理，不重开卡片。若返回 `status: "report_ready"` 与 `reportUrl`，表示已经有可复用的报告：在本轮已有的 ChatFate 标签导航到该 reportUrl（没有则新开），按第 7 步交付完整原链接，跳过第 4–6 步；不读取子 Skill、不再生成 interpretation、不调用 create。其他成功结果必须有 `calculationId`、`engineVersion`、`calculatedFacts` 和对应 chart 版本；以计算返回的 `reading.product` 确定产品，`daily` 还须有匹配 `reading.targetDate` 的 `calculatedFacts.dailyTransit`。旧返回没有 reading 时，仅按八字 life／六爻 question 兼容。
4. 成功排盘后，本轮尚未打开登录页时，打开 `https://chatfate.cc/pending/<calculationId>`；它会自动转到报告。本轮已打开登录页时沿用该页，不重复开等待页。页面交付和第 5 步文件读取互不依赖，可同时调度；首次需要读取 Browser 技能时也并入同一次文件读取。不要等等待页加载完成、截图或查进度后才开始解读；浏览器不可用时直接继续报告生成。
5. 只读取实际分支的子 Skill：八字为 `../bazi/SKILL.md`，六爻为 `../liuyao/SKILL.md`。与 [报告契约](references/report-contract.md)、[表达边界](references/safety.md) 一次批量读取；当前上下文已读且版本未变时直接复用。子 Skill 的扩展 references 只在相应事实、规则或出处不清楚时读取，不默认加载另一分支或逐文件重复读取。
6. 根据计算结果、reading 和用户明确提供的 userContext，生成一次 `chatfate.reading.interpretation.v1`。围绕本问选择相关篇章，先给实质回答，再解释支持关系与改变判断的条件；不按固定领域凑篇幅，不要求每章附行动。检查来源、证据路径和具体事实后调用 `create_divination_report({ calculationId, interpretation })`。写作时直接自查：概要是否先回答本问，传统解释是否有可理解的推导，是否清楚区分用户陈述与现实建议；证据不足就收窄判断，不靠字数或术语撑深度，也不把“稳住节奏、结合实际、适时调整”当作整条结论。无需再调用一次模型审稿。不要把原始输入或整个 chart 重新传回。
7. 工具成功后，沿用本轮已交付的 ChatFate 标签；本轮从未打开过登录页或等待页时，立即打开工具返回的 `reportUrl`。最终回复固定为 `报告已生成：<reportUrl>`，完整链接逐字保留，不复述报告正文。

正常路径只有：卡片 → 同轮等待 → 确定性计算 → 解释 → 一次报告写入。不要先搜索外网、查安装状态、轮询站点、重复计算、重读已在上下文中的文档或为润色重复写入报告。校验失败只修正指出的字段，沿用同一 `calculationId` 重试；网络超时沿用相同 interpretation 重试。

## 当前写入合同

四种产品都写入 `chatfate.reading.interpretation.v1`；当前精确字段见[报告契约](references/report-contract.md)，需要机器可读 Schema 时读取 `chatfate://schemas/reading-interpretation-v1`，不重复加载。

- question 使用 `chatfate.liuyao.chart.v2`，先回答用户具体问题；未给比较选项或期限时，不替用户编造。
- life 使用 `chatfate.bazi.chart.v2`，从命局特点解释到用户关心的生活议题；没填关注点时，选最有依据的主题，不机械填满事业、财富、感情、健康。
- daily 使用八字与服务器北京时间当天的 `dailyTransit`；不改成随机六爻，不借出生干支假写当日内容。
- annual 使用八字、目标年 `liunian` 及引擎返回的 `annualTransit` 节气月份。7 月 1 日用于固定年度参照；月份起止只用引擎真实节气日期。没有月份事实时不补算，不将月份解释写成事件保证。

外层 `chatfate.report.v2`、reading 和 userContext 由服务器从保存的计算结果生成，模型只传 calculationId 与 interpretation。旧 interpretation 合同仅为已经开始的旧客户端流程保留兼容，新流程不选旧模板。

付费产品先展示概要和一个有实质依据的完整篇章，再由用户按需解锁；今日全部免费。预览必须能回答本问，不能把关键条件藏在付费后，也不将恐吓或预测保证作为解锁理由。价格和权益以报告页为准，正文不促销。

## 在内置浏览器打开页面

先读取本轮可用的 Control In App Browser 技能，并按该技能初始化浏览器运行时。ChatFate 的授权、等待、历史和报告页面都使用 `iab`。已有本轮 ChatFate 交付标签及其持久变量时，直接用它导航和保留，不再创建标签。仅在尚未建立这些绑定时执行以下初始化，后续复用变量，不能重复声明；不要使用 `globalThis`：

```js
const chatfateBrowser = await agent.browsers.get("iab");
nodeRepl.write(await chatfateBrowser.documentation());
```

读完返回的完整浏览器文档后，再创建本轮第一个交付标签；已有标签则跳过创建并复用：

```js
let chatfateTab = await chatfateBrowser.tabs.new();
await chatfateTab.goto("<URL>");
await (await chatfateBrowser.capabilities.get("visibility")).set(true);
await chatfateBrowser.tabs.finalize({ keep: [{ tab: chatfateTab, status: "deliverable" }] });
```

- 本回合有多个需要保留的 ChatFate 标签时，`keep` 列出全部。禁止不带 `keep` 的 `finalize()`、关闭交付标签或隐藏浏览器。
- 后续只调用 `chatfateTab.goto(...)`、显示浏览器和带 `keep` 的 `finalize(...)`；标签名称采用初始化时真实使用的变量名，不重复执行 `tabs.new()`。
- 不用 `getForUrl()`、`getDefault()`、extension、chrome 或其他已有 browser 绑定猜浏览器。出现画中画表示选错，使用 `iab` 重开同一 URL。
- 这是交付动作，不为报告生成做浏览器截图、点击、读页面状态或重复导航；登录结果由原工具重试确认。标签在回合结束后保留依赖 `finalize({ keep: … })`，仅 `set(true)` 不足。
- 内置 Browser 不可用或打开失败时，不阻塞已生成报告的交付：仍逐字返回完整 `reportUrl`，作为恢复入口。不要把浏览器失败说成报告失败。

## 登录与恢复

- `render_divination_menu`、`wait_for_divination_data` 和卡片提交无需登录；计算、报告创建和删除需要 Google 登录。先收集输入，仅在受保护工具返回登录要求时处理，永远不要运行 `codex mcp login`。
- 错误包含 `/login/start?ticket=…` 时，在 `iab` 打开该链接一次，并保留 deliverable。登录绑定当前服务端会话，无需刷新凭据。
- 不结束模型回合：约每 10 秒以原参数重试原工具，最多约 5 分钟。授权成功即继续，不能重复打开登录链接或让用户重填。约两分钟始终被拒时，可以系统默认方式打开同一链接一次作为兼容兜底，继续原工具重试。
- 约五分钟仍未成功，说明授权可能未完成；用户回复完成后沿用保留的 `submissionId` 或 `calculationId` 重试。已提交信息保留约一小时。
- 本轮打开过登录页时，它会沿计算和写解状态自动转到报告；不要另开等待页或报告页。

## 六爻同问预检

仅检查同一会话内已成功交付六爻报告的所问。没有成功报告、跨会话或八字重排都不触发；拿不准是否同一件事时，不触发，不根据主题关键词猜测。

明确是同一件事时，先逐字询问：`这件事此前已起过一卦。传统规矩，初筮为准、再问为参——还要再起吗？`

用户明确确认后继续卡片流程，可在 boundaries 用 practical 说明“重复解读只宜作为对照，不据此取代先前判断”。这只是使用建议，不证明首次或重复测算有预测能力。未确认不创建新卡片，不增加跨会话同问识别，不向新合同添加旧 repeatNotice 字段。

## 已有报告和异常

- 用户主动要求查看历史时调用 `get_history_link`，按上方程序打开返回链接；返回 `message` 时如实转述。普通菜单会自行显示历史入口，模型不主动调用。
- 用户明确重新测算时才新建 `sessionId`，六爻仍做同问预检。成功交付后不主动追问。
- `widget_not_mounted` 或用户报告卡片未显示：用同一个 `sessionId` 重开并在同轮再次等待，最多自动重开一次；仍失败时说明宿主卡片加载失败，可稍后重试，不改装插件。
- 字段、闰月和城市歧义留在卡片内处理。事实缺失或计算失败时如实说明可恢复错误，不猜测、不改走另一分支。
- daily 额度已使用且原报告删除时，说明今天已用额度并结束该次请求；不重排、换出生资料或转其它免费产品规避限制。daily 正在保存或检查暂时失败时，用原 submissionId 重试计算，不先生成解释。
- 报告上传繁忙或响应丢失：沿用同一 `calculationId` 和 interpretation 重试，不再起卦、不建第二份报告。质量或 schema 校验失败按返回字段修正，不重写无关段落。
- 报告站不可用时如实说明，不另建站点、不换地址、不把整份正文贴进聊天替代交付。
- 新报告的 256-bit `accessId` 用于不可猜测地定位报告，读取仍受生成账号和权益检查；早期未关联账号的报告可能保留持链接读取的兼容口径。无论哪种口径，都必须向用户交付完整 reportUrl，保留路径、不截断、不改写，不写日志或公开位置。
