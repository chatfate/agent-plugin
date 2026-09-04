# 报告契约

## 事实与解释分离

- `submissionId`：卡片提交后产生的短期不透明标识；卡片原始输入不进入模型上下文。
- `calculationId`：确定性排盘结果的短期不透明标识，创建报告时使用。
- `calculatedFacts`：排盘引擎输出的不可改写事实。
- `interpretation`：模型基于事实生成的结构化解释。
- `metadata`：引擎、排盘口径和 Schema 版本；排盘口径由报告凭据区固定展示。

解释不得重新声明与 `calculatedFacts` 不一致的八字或六爻事实。需要在正文提及这些数据时，逐字使用事实字段。

## 外层报告与写入版本

- 外层 `schemaVersion` 固定为 `chatfate.report.v2`。
- 八字当前写入只接受 `chatfate.bazi.chart.v2` 与 `chatfate.bazi.interpretation.v3`。
- 六爻当前写入只接受 `chatfate.liuyao.chart.v2` 与 `chatfate.liuyao.interpretation.v2`。
- 历史八字 interpretation 只用于旧加密报告读取，不迁移、不重写，也不得进入新报告写入。
- 新报告中的排盘事实与 interpretation Schema 合同保持不变；本次只改变存储与访问合同。
- `reportId`、`documentSerial`、明文 report envelope、receipt、幂等、删除和链接合同由 MCP 与 Schema 管理；Skill 不自行拼装或改写。

## 存储与访问

- 持有完整链接即拥有该报告的读取权限；链接应像凭据一样只交给预期读者。
- 明文 `ReportV2` 保存在 ChatFate 报告站的 D1，与当前用户关联；用户可随时自行删除自己的报告。
- 新报告的 `accessId` 是 256-bit 随机值，以 43 字符 base64url 放在 `/report/<accessId>` 路径中；新报告链接不含 fragment，也没有客户端解密密钥。
- `CHATFATE_REPORT_WRITE_SECRET` 保护 `POST` 写入；`deleteToken` 保护 `DELETE`，且只删除对应报告。读取不携带这两种写入或删除凭据。
- 新报告没有默认过期时间，默认保留至用户删除。

## 三层语域

每个 `modules`／`domains` 项均使用同一骨架：

- `duanyu`：先给方向判断；用判断句，不把置信度降级成含糊语气。
- `shiyi`：解释断语由哪些事实关系支持，并呈现正向与反向条件。
- `cankao`：给白名单内、可执行且有先后顺序的参考；凡保留的模块或域不得为空。

三层正文不得复制、拆句或按数组索引猜层。每个 Item 都保留非空 `evidenceRefs` 和 `confidence`；具体路径必须能从当前 `calculatedFacts` 解析。

## 八字 interpretation v3

- `schemaVersion`：`chatfate.bazi.interpretation.v3`。
- `synopsis`：3–5 条命局速写；每条至少引用两个独立事实组。
- `modules`：按 `dayMasterAndSeason → elementDynamics → tenGodPatterns → ganZhiRelations（可选）→ luckCycles` 的规范顺序输出。每个模块有 `duanyu`、`shiyi`、`cankao`。
- `domains`：固定且按 `career → wealth → love → health` 输出；每域有 `duanyu`、至少两条 `shiyi` 和非空 `cankao`。
- `overallGuidance.duanyu`：恰好 1 条跨域总断。
- `overallGuidance.shiyi`：恰好 1 条，解释总断依据和四域先后关系。
- `overallGuidance.cankao`：2–4 条，按优先级给判断与行动方向，不重新展开四域全文。
- `boundaries`：3–5 条，只写真实口径、缺失事实和适用边界，不收回正文已下的判断。

`overallGuidance` 三层正文规范化后不得重复；不得复制同一文本、拆句填层或把旧扁平数组自动映射成三层。

## 六爻 interpretation v2

- `schemaVersion`：`chatfate.liuyao.interpretation.v2`。
- `verdict.judgment`：非空总判断，至少引用两个独立事实组。
- `verdict.timing`：可为 `null`；非空时只引用 `calculatedFacts.timingTriggers` 的真实触发条件，不制造日期与结果保证。
- `verdict.action`：非空白名单动作，可复用 judgment 的事实路径。
- `repeatNotice`：只有总编排确认同一会话、同一件事的再次起卦后写入冻结原句；否则为 `null`。
- `safetyMode`：敏感问题为 `true`；此时 timing 为 `null`，不输出 timing 模块，并提供现实专业帮助。
- `modules`：按 `questionFocus → trend → movingLines → timing（可选）` 的规范顺序输出。每个保留模块都有非空 `duanyu`、`shiyi`、`cankao`。
- `boundaries`：3–5 条，集中承载候选未决、证据冲突、口径与传统方法限制。

## 报告交付

- 报告工具成功后，先尽力在 Codex 右侧内置 Browser 打开 `reportUrl`。
- 无论 Browser 是否成功，最终回复都固定为 `报告已生成：<reportUrl>`，并逐字保留完整 URL。
- 完整 URL 的路径中包含不可猜测的 `accessId`，报告链接不含 fragment 解密密钥。不能截断、改写或省略；用户关闭 Browser 后依靠会话中的同一完整链接重新打开。
- 除传给内置 Browser 打开外，该链接不写入日志、无关工具参数或公开位置，也不向非预期读者转发。
