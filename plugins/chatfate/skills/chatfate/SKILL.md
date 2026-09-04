---
name: chatfate
description: 必须先按本轮 Skills 清单给出的 file 路径读取本 Skill；root 别名与剩余路径直接拼接并保留重复目录名，禁止手拼插件缓存路径或先搜索旧会话/记忆。通过 ChatFate 的双入口卡片完成确定性八字排盘或六爻问卦并生成当前版本的解释报告。用户发送“开始使用 ChatFate”、提出排八字、看八字、六爻、算卦、问卦、生成命理解读、重新测算或打开既有 ChatFate 报告时使用；负责同轮卡片收集、按 kind 调用对应确定性引擎、严格按报告工具的当前输入 Schema 生成解释、创建持久报告、在 Codex 内置 Browser 打开并始终在会话中保留完整报告链接，且不允许模型自行排盘或起卦。
---

# ChatFate 总编排

## 核心约束

- 本 Skill 及其 `references/` 一律从本轮 Skills 清单提供的 `file` 路径按相对路径读取；不要根据版本号猜测缓存目录，也不要先查 MEMORY 或旧 rollout。
- 使用工具产生的 `calculatedFacts` 作为唯一排盘或卦盘事实。
- 当前写入合同只认工具清单实时声明的 Schema：八字排盘为 `chatfate.bazi.chart.v2`、解释为 `chatfate.bazi.interpretation.v3`；六爻排盘为 `chatfate.liuyao.chart.v2`、解释为 `chatfate.liuyao.interpretation.v2`。外层报告仍由工具写成 `chatfate.report.v2`。
- 八字分支必须读取并遵循当前 `bazi` 子 Skill；六爻分支必须读取并遵循当前 `liuyao` 子 Skill。子 Skill、共享报告契约和报告工具 inputSchema 共同约束写入；发生冲突时以报告工具当前 inputSchema 为硬边界并停止，不得自动转换旧结构。
- 不自行计算、补写或修改八字的四柱、十神、藏干、五行、大运，或六爻的卦名、纳甲、六亲、六神、世应、动变与用神。
- 只通过卡片收集对应流程的必要输入，不额外询问姓名、曾用名、现状或历史事件。
- 不在聊天中复述卡片、处理中间状态或整份报告。
- 将出生信息视为敏感数据，不把原始值写入 URL、日志或不相关的工具参数。

## 报告交付

- 报告写入 ChatFate 自有报告站，由部署环境配置好；没有建站、选站、授权或升级流程，也没有对应的工具。
- 日常路径就是唯一路径：直接进入标准流程的排盘与报告创建，不存在需要先检查站点状态的步骤。
- 报告交付不可用时如实说明并停止本次报告生成，不得降级到其它地址，也不得把报告内容直接贴进聊天当作替代。
- 不得在聊天、最终回复或模型可见摘要中回显 `writeSecret`、`deleteToken` 等非交付凭据；`CHATFATE_REPORT_WRITE_SECRET` 属于 `writeSecret`。
- `reportUrl` 及其路径中的 `accessId` 是交付给当前用户的 bearer link，不属于上述非交付凭据：必须完整保留在会话中，供用户随时打开与分享。

## 在内置浏览器打开 ChatFate 页面（唯一程序）

ChatFate 的所有页面（授权页、等待页、历史页、报告页）一律视为**明确指定内置 in-app 浏览器**。打开它们必须且只能执行下面的程序（经 Control In App Browser 技能的 Node REPL `js` 工具）；`iab` 绑定已存在时跳过第一段：

```js
if (globalThis.iab == null) {
  globalThis.iab = await agent.browsers.get("iab");
  nodeRepl.write(await iab.documentation());
}
const tab = await iab.tabs.new();
await tab.goto("<要打开的URL>");
await (await iab.capabilities.get("visibility")).set(true);
await iab.tabs.finalize({ keep: [{ tab, status: "deliverable" }] });
```

- **最后一行 `finalize({ keep: … })` 不可省略，这是整个程序的目的所在**：agent 打开的标签只是“本回合的临时标签”，宿主会在回合结束时自动回收——把标签以 `status: "deliverable"` 交接给用户，才是它在回合结束后仍留在前台的唯一机制。`set(true)` 只管当下显示，管不了回收。
- 若本回合打开过多个仍需保留的 ChatFate 标签，`keep` 数组必须把它们全部列入——不在 `keep` 里的标签会被清理。
- **禁止**不带 `keep` 的 `finalize()`、关闭 ChatFate 标签、或把 visibility 设回 false。
- **禁止** `getForUrl()`、`getDefault()`、`get("extension")`，禁止复用 `globalThis.browser`／chrome／edge 绑定打开 ChatFate 页面——运行时“自选浏览器”会不定期落到电脑使用（画中画）表面。
- 程序执行完即停止一切浏览器操作：不观察、不截图、不点击、不读取页面状态。
- 判错标准：页面以「画中画」浮层出现＝绑到了错误的浏览器，立即用上面的程序（`iab` 绑定）对同一 URL 重开；报告在回合结束后消失＝漏了 `finalize keep`。

## 登录

- ChatFate 需要 Google 登录才能排盘和出报告。分界线：`render_divination_menu`、`wait_for_divination_data`、卡片提交等收集输入的工具**不需要**登录；`calculate_bazi_chart`、`calculate_liuyao_chart`、`create_divination_report`、`delete_report` **需要**登录。
- 标准流程的顺序永远是：先出卡片、先收集输入；登录只在计算类工具被拒时处理。不要在出卡片之前主动检查或发起登录，**也永远不要运行 `codex mcp login`**——ChatFate 的登录不走那条路。
- 计算类工具被拒时，会返回一条**包含登录链接**（`/login/start?ticket=…`）的错误。这**不是排盘失败**，不要宣告无法计算，也不要重开卡片。处理方式：把错误里的链接在内置 `iab` Browser 中**打开一次**，用户在该页完成 Google 授权；授权直接绑定到当前会话的服务端，**没有任何凭据需要 Codex 刷新**。
- 打开授权页：严格执行「在内置浏览器打开 ChatFate 页面（唯一程序）」一节。打开后授权成功与否由下面的工具重试告知，页面此后会自己走完全程（授权完成 → 正在起卦 → 正在写解 → 报告）。
- 打开链接后**不要结束回合**：每隔约 10 秒用相同参数重试原工具调用，最多约 5 分钟。用户授权完成的瞬间，重试就会成功——直接继续后续流程，不需要用户说任何话。已提交的卡片数据在服务端保留约一小时。
- **同一时刻只用一个登录链接。**重试再次被拒时返回的链接与之前相同（约十分钟内有效），不要重复打开新页面。授权页只打开一次，然后安静轮询。
- 降级按时间判断，不靠看页面：打开链接后约 2 分钟内重试仍全部被拒，改用系统默认方式再打开同一链接（macOS：`open <url>`），继续轮询。
- 轮询约 5 分钟仍被拒才作为兜底收尾：说明授权可能未完成，请用户完成后回复一声；回复后用保留的 `submissionId` 重试，不重新起卦。

## 报告展示

- **本次发生过登录**（打开过授权页）：那个标签页会自动一路走到报告，模型**不要**再打开等待页或报告页，也不要对它做任何观察或导航。计算与写解照常进行即可。
- **本次未发生登录**（无感路径，没有任何 ChatFate 页面已打开）：`calculate_*` 返回 `calculationId` 后，按「在内置浏览器打开 ChatFate 页面（唯一程序）」打开等待页 `https://chatfate.cc/pending/<calculationId>`，然后不再管它——报告写入后它自动变成完整报告。
- **收尾硬检查：回合结束前，内置浏览器里必须有一个 ChatFate 页面正在展示**（等待页或报告页任一）。`create_divination_report` 成功后自查：本回合是否打开过授权页或等待页？**没有就必须立即按「唯一程序」打开 `reportUrl`**——这一步不可省略，只在聊天里贴链接不算完成交付。最终回复的契约不变：`报告已生成：<reportUrl>`，完整链接逐字保留。
- 菜单卡片对已登录且有历史报告的用户会自动显示「查看历史报告」入口，由用户自行点击、卡片直接打开（受平台限制开在系统浏览器）。模型不需要提及它、不代为打开、不据此改变任何流程。
- 用户在聊天中**主动要求**查看历史报告时：调用 `get_history_link` 取得签名链接，按「在内置浏览器打开 ChatFate 页面（唯一程序）」打开；返回 `message` 而非链接时如实转述。除用户明确要求外不调用此工具。
- **报告页就是最终交付物，回合结束后它必须仍在前台。**保住它的机制是「唯一程序」最后一行的 `finalize({ keep: [{ tab, status: "deliverable" }] })` 交接；回合收尾时自查这一步已对本回合的 ChatFate 标签执行过。严禁不带 `keep` 的清理、关闭标签或取消 visibility——把报告留在用户眼前，链接只是补充。

## 六爻同问预检

- 只检查同一会话内已经成功交付过六爻报告的所问；没有成功报告、跨会话、或八字重排都不触发。
- 只有本次所问与先前所问明确是同一件事时才触发。拿不准是否同一件事时，不触发；不要靠关键词、主题类别或相似措辞硬猜。
- 命中后先逐字询问：`这件事此前已起过一卦。传统规矩，初筮为准、再问为参——还要再起吗？`
- 只有用户明确确认后，才继续正常卡片流程。确认后的六爻解释必须把 `repeatNotice.text` 逐字设为：`此事已有初筮。初筮告，再三渎，渎则不告——断以初卦为准，本卦仅作参照。`
- 用户未确认就停止，不创建新卡片。不要增加服务端同问存储，不要声称能够跨会话识别，也不要阻止用户确认后的再问。

## 标准流程

1. 用户发送安装器预填的“开始使用 ChatFate”，或提出八字、六爻请求时，先应用上方六爻同问预检；未触发或已确认后，第一项工具动作必须是调用 `render_divination_menu`，展示“问事起卦”和“算八字”两个入口。调用前不发送 commentary、说明或处理中状态。即使聊天里已有部分输入，也不要自行拼接参数；输入只通过卡片收集。
2. `render_divination_menu` 返回 `sessionId` 后，必须在同一个模型回合立即调用 `wait_for_divination_data`。该调用会保持等待，直到卡片通过 app-only 的 `submit_divination_data` 提交；不要结束回合、输出等待说明或要求用户再发一句话。若 `wait_for_divination_data` 返回 `widget_not_mounted`，立即用同一个 `sessionId` 重新调用 `render_divination_menu` 并再次 `wait_for_divination_data`，最多自动重开一次；第二次仍是 `widget_not_mounted` 时按下方“卡片加载失败”处理，不要继续盲等。
3. 卡片提交并取得 `submissionId` 后直接进入计算，不需要任何站点检查或准备步骤。报告交付由部署环境配置，若不可用会在 `create_divination_report` 阶段如实报错。
4. 不调用 `sendFollowUpMessage`、`ui/message`、`updateModelContext` 或其他续跑机制。卡片提交后，`wait_for_divination_data` 会把经过验证的 `submissionId` 返回给当前 Codex 回合。
5. 读取等待结果里的 `kind`，不要根据用户原始措辞猜分支；调用 `calculate_bazi_chart`／`calculate_liuyao_chart`（若返回登录要求，按「登录」一节打开链接并轮询重试）后，按「报告展示」决定是否需要打开等待页，再按对应合同解释：
   - `kind: "bazi"`：调用 `calculate_bazi_chart`，校验 `chatfate.bazi.chart.v2`；读取当前 `bazi` 子 Skill 和必要 references，只依据返回的 `calculatedFacts` 生成 `chatfate.bazi.interpretation.v3`，再调用 `create_divination_report`。
   - `kind: "liuyao"`：调用 `calculate_liuyao_chart`，校验 `chatfate.liuyao.chart.v2`；读取当前 `liuyao` 子 Skill 和必要 references，只依据返回的 `calculatedFacts` 生成 `chatfate.liuyao.interpretation.v2`，再调用 `create_divination_report`。若本次经过同问确认，写入上方冻结的 `repeatNotice`；否则为 `null`。
6. `create_divination_report` 只接收 `calculationId` 和 `interpretation`；服务器根据已保存的计算类型分别执行八字 v3 或六爻 v2 的完整严格校验。每条模型解释只能引用当前 `calculatedFacts` 中真实存在的 `evidenceRefs`；缺少依据时减少结论，不补算、不猜测、不复制工具未返回的事实。不要把卡片原始输入或自行拼装的排盘对象再次传给工具；MCP 会用本地保存的计算结果组装明文 `ReportV2` 并写入 ChatFate 报告站的 D1。
7. 工具返回缺少 `engineVersion`、`calculationId` 或 `calculatedFacts` 时停止并报告可恢复错误，不要推测，也不要换用另一分支的工具。
8. 取得 `reportUrl` 后按「报告展示」一节处理：已有 ChatFate 页面在展示时不重复打开；否则按「在内置浏览器打开 ChatFate 页面（唯一程序）」打开 `reportUrl`，保留为 deliverable。不要使用普通外链打开 API 代替内置 Browser。
9. 无论 Browser 打开成功、不可用还是失败，最终回复都必须是 `报告已生成：<reportUrl>`；将工具返回的完整 `reportUrl` 逐字放进会话。`reportUrl` 与路径中的 `accessId` 是必须返回当前用户的 bearer link，不得因非交付凭据的保密规则而省略。报告链接不含 fragment 解密密钥；路径中的 256-bit `accessId` 即读取凭据，不得截断、改写、省略或只说“已在右侧打开”。
10. 不在最终回复中重复报告正文或展示内部状态。Browser 打开失败时不改变第 9 步的回复契约，完整链接就是恢复入口。

## 重新测算

只有用户明确要求重新测算或重新问卦时再次调用 `render_divination_menu` 并用新的 `sessionId` 等待。重新问卦同样先执行六爻同问预检；八字重排不适用“初筮”规则。不要在成功报告后主动追问。

## 错误处理

- 卡片加载失败（`wait_for_divination_data` 返回 `widget_not_mounted`）：见“标准流程”第 2 步，用同一个 `sessionId` 自动重开一次；第二次仍是 `widget_not_mounted` 时，告知用户这是宿主沙箱加载问题，稍后重试即可，不要改装插件或重装。
- 卡片加载失败（用户反馈“卡片没显示”“红框”“sandbox failed to load”等）：宿主沙箱加载偶发超时，与流程状态无关。立即用同一个 `sessionId` 重新调用 `render_divination_menu` 并在同回合重新 `wait_for_divination_data`。连续两次仍失败时，告知用户这是宿主沙箱网络加载问题，稍后重试即可，不要改装插件或重装。
- 表单字段、农历闰月和城市歧义：全部在卡片内就地处理，不在聊天中重复。
- 登录要求（错误信息里带 `/login/start` 链接）：按上方「登录」一节处理——iab 打开链接、原地轮询重试；这不属于排盘失败，不要触发下一条。
- 排盘或起卦失败：说明对应流程无法计算，并允许重新打开卡片；不要改走另一种占算方式。
- 报告生成超时：使用同一幂等键查询或重试，不创建第二份报告。
- 旧版有过期时间的报告已过期：只在用户请求打开这类旧报告时解释，并提供重新测算入口。
- 不显示请求 ID、跟踪 ID、堆栈、内部工具名或原始模型提示词。

生成任一解释前读取 [报告契约](references/report-contract.md) 和 [表达边界](references/safety.md)，再读取对应当前子 Skill；任何文档都不得覆盖本轮报告工具实时声明的 inputSchema。
