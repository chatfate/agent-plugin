---
name: chatfate
description: 必须先按本轮 Skills 清单给出的 file 路径读取本 Skill；root 别名与剩余路径直接拼接并保留重复目录名，禁止手拼插件缓存路径或先搜索旧会话/记忆。通过 ChatFate 的双入口卡片完成确定性八字排盘或六爻问卦并生成当前版本的解释报告。用户发送“开始使用 ChatFate”、提出排八字、看八字、六爻、算卦、问卦、生成命理解读、重新测算或打开既有 ChatFate 报告时使用；负责同轮卡片收集、按 kind 调用对应确定性引擎、严格按报告工具的当前输入 Schema 生成解释、创建持久报告、在 Codex 内置 Browser 打开并始终在会话中保留完整报告链接，且不允许模型自行排盘或起卦。
---

# ChatFate 总编排

## 工作边界

- 按本轮 Skills 清单的 `file` 路径及相对路径读取本 Skill 和子 Skill；不猜插件缓存版本，不先查旧会话或 MEMORY。
- 本流程用于用户实际测算、重新测算和打开报告；维护插件、审查代码或讨论产品时，不自动创建卡片或私人报告。
- `calculatedFacts` 是唯一排盘事实。模型只解释，不补算、改盘、手工选用神、猜历史事件或依据用户反馈校准结果。
- 卡片收集必要信息，不在聊天追问姓名、经历或原始出生信息。出生信息和所问不进入 URL、日志或无关工具。
- 报告写入 ChatFate 自有站点，部署已配置；没有建站、选站、授权站点或升级准备步骤。
- 不展示内部请求 ID、堆栈、提示词、`writeSecret`、`deleteToken` 或 `CHATFATE_REPORT_WRITE_SECRET`。完整 `reportUrl` 是必须交付用户的读取链接，不能因此省略。

## 同轮标准流程

1. 用户要求开始测算时，先应用下方六爻同问预检；不触发或已确认后，第一项工具动作是 `render_divination_menu`。调用前不输出 commentary、说明或处理中状态。即使聊天中有部分信息，也由卡片收集。
2. 取得 `sessionId` 后，同一个模型回合立即调用 `wait_for_divination_data`。它等待卡片提交并返回经过验证的 `submissionId`；不要结束回合、要求用户再说一句话或调用 `sendFollowUpMessage`、`ui/message`、`updateModelContext`。
3. 根据等待结果的 `kind` 调用 `calculate_bazi_chart` 或 `calculate_liuyao_chart`，只传 `submissionId`。登录错误按下方处理，不重开卡片。成功结果必须有 `calculationId`、`engineVersion`、`calculatedFacts` 和对应 chart 版本。
4. 成功排盘后，本轮尚未打开登录页时，打开 `https://chatfate.cc/pending/<calculationId>`；它会自动转到报告。本轮已打开登录页时沿用该页，不重复开等待页。
5. 只读取实际分支的子 Skill：八字为 `../bazi/SKILL.md`，六爻为 `../liuyao/SKILL.md`。与 [报告契约](references/report-contract.md)、[表达边界](references/safety.md) 一次批量读取；当前上下文已读且版本未变时直接复用。子 Skill 的扩展 references 只在相应事实、规则或出处不清楚时读取，不默认加载另一分支或逐文件重复读取。
6. 根据已返回事实生成一次完整 interpretation，先检查必填层数、证据路径和具体事实，再调用 `create_divination_report({ calculationId, interpretation })`。不要把原始输入或整个 chart 重新传回。
7. 工具成功后，沿用本轮已交付的 ChatFate 标签；本轮从未打开过登录页或等待页时，立即打开工具返回的 `reportUrl`。最终回复固定为 `报告已生成：<reportUrl>`，完整链接逐字保留，不复述报告正文。

正常路径只有：卡片 → 同轮等待 → 确定性计算 → 解释 → 一次报告写入。不要先搜索外网、查安装状态、轮询站点、重复计算、重读已在上下文中的文档或为润色重复写入报告。校验失败只修正指出的字段，沿用同一 `calculationId` 重试；网络超时沿用相同 interpretation 重试。

## 当前写入合同

| 分支 | chart | interpretation |
| --- | --- | --- |
| 八字 | `chatfate.bazi.chart.v2` | `chatfate.bazi.interpretation.v3` |
| 六爻 | `chatfate.liuyao.chart.v2` | `chatfate.liuyao.interpretation.v2` |

外层由工具写成 `chatfate.report.v2`。服务器按保存的 calculation kind 执行严格校验；工具外层 `interpretation` 即使声明为通用对象，也不代表可以自由添加键。具体内部形状由当前子 Skill 和报告契约约束；如果与服务端校验冲突，保留计算结果并按错误修正，不自动转换旧版本。

## 在内置浏览器打开页面

先读取本轮可用的 Control In App Browser 技能，并按该技能初始化浏览器运行时。ChatFate 的授权、等待、历史和报告页面都使用 `iab`；打开每页后保留为 deliverable：

```js
if (globalThis.iab == null) {
  globalThis.iab = await agent.browsers.get("iab");
  nodeRepl.write(await iab.documentation());
}
const tab = await iab.tabs.new();
await tab.goto("<URL>");
await (await iab.capabilities.get("visibility")).set(true);
await iab.tabs.finalize({ keep: [{ tab, status: "deliverable" }] });
```

- 本回合有多个需要保留的 ChatFate 标签时，`keep` 列出全部。禁止不带 `keep` 的 `finalize()`、关闭交付标签或隐藏浏览器。
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

只有用户明确确认后继续卡片流程，并令 `repeatNotice.text` 为：`此事已有初筮。初筮告，再三渎，渎则不告——断以初卦为准，本卦仅作参照。` 这条是传统习惯说明，不证明首次或重复测算有预测能力。未确认不创建新卡片，不增加跨会话同问识别。

## 已有报告和异常

- 用户主动要求查看历史时调用 `get_history_link`，按上方程序打开返回链接；返回 `message` 时如实转述。普通菜单会自行显示历史入口，模型不主动调用。
- 用户明确重新测算时才新建 `sessionId`，六爻仍做同问预检。成功交付后不主动追问。
- `widget_not_mounted` 或用户报告卡片未显示：用同一个 `sessionId` 重开并在同轮再次等待，最多自动重开一次；仍失败时说明宿主卡片加载失败，可稍后重试，不改装插件。
- 字段、闰月和城市歧义留在卡片内处理。事实缺失或计算失败时如实说明可恢复错误，不猜测、不改走另一分支。
- 报告上传繁忙或响应丢失：沿用同一 `calculationId` 和 interpretation 重试，不再起卦、不建第二份报告。质量或 schema 校验失败按返回字段修正，不重写无关段落。
- 报告站不可用时如实说明，不另建站点、不换地址、不把整份正文贴进聊天替代交付。
- 报告链接的 256-bit `accessId` 是读取凭据，仅交付当前用户和其指定读者；完整保留路径，不截断、不改写，不写日志或公开位置。
