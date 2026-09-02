# 计划：Claude 作为 agent "订机票" — 40 个具体例子（国内日常可行版）

## 状态

**暂缓执行，方案已定，等用户以后启动。**

---

## 当前进度（2026-08-29）

**已完成讲解**
  - ✅ 例 1：Playwright 打开携程，搜"上海→北京 明天"
  - ✅ 例 2：提取携程页面航班列表

**下一步**
  - 例 3：Playwright 打开飞猪，搜同样航线
  - （用户说"下一个"时继续）

**模式**
  - 用户要求"不执行，仅仅解释"
  - 每个例子讲解完，等用户说"下一个"再继续

---

## Context

用户要一份**不瞎编**的实操材料：用 Claude（CLI / MCP）构建一个"帮我订明天去北京的机票"的 agent，给出 40 个**具体、可运行**的例子。

**关键决策（2026-08-28 确认）**
  - **不注册国外平台**（不要 Amadeus、Stripe、Google Calendar 等）
  - **全部用国内日常可行的工具**（Playwright MCP + 携程/飞猪/去哪儿 + 钉钉/飞书）
  - **每个例子执行前要给用户完整信息，批准后才动手**

**技术事实（2026-08-28 查证）**
  - Playwright MCP（`@playwright/mcp`，微软维护，33 个工具）✅
  - 携程/飞猪/去哪儿网页公开爬取 ✅（不注册）
  - 钉钉/飞书群自定义机器人 webhook ✅（不注册，任何群都能加）
  - Claude Code CLI 内置 Computer Use（macOS 已 GA；Windows 进行中）
  - Agent SDK 已发布（Python/TS）

**不能做的（诚实标注）**
  - 真实下单 → 必须用户本人登录+验证码
  - 真实支付 → 必须用户输密码
  - 端到端全自动一句话出订单号 → 不存在
  - 绕过反爬/CAPTCHA → 需特殊基础设施

---

## 输出交付物

一份 markdown 教学材料，结构：

### 第 0 节：三句话结论
  - Claude 能**替你查、替你比、替你填表**，但**不能替你输密码付款**
  - 全部用国内工具（Playwright + 携程/飞猪/钉钉），不注册任何国外平台
  - 40 个例子分三档：✅ 可直接跑 / ⚠️ 需人类某步介入 / ❌ 当前不可行

### 第 1 节：环境与配置（所有例子的共同前置）
  给出可复制的配置：
  - `~/.claude/settings.json` 的 `mcpServers` 配置（playwright）
  - 钉钉/飞书群自定义机器人 webhook 获取方法
  - 不需要注册任何平台

### 第 2 节：40 个例子（按链路环节分 7 组，全部国内可行）

**组 1：航班查询（Playwright + 携程/飞猪/去哪儿） — 例 1–8**
  1. Playwright 打开携程，搜"上海→北京 明天"
  2. 提取携程页面航班列表
  3. Playwright 打开飞猪，搜同样航线
  4. 提取飞猪页面航班列表
  5. Playwright 打开去哪儿，搜同样航线
  6. 提取去哪儿页面航班列表
  7. 截携程页面为 PNG
  8. 截飞猪页面为 PNG

**组 2：数据整理（本地文件） — 例 9–16**
  9. 携程数据写入 `ctrip_flights.json`
  10. 飞猪数据写入 `fliggy_flights.json`
  11. 合并去重写入 `all_flights.json`
  12. 生成 markdown 比价报告
  13. 按价格排序
  14. 按时间排序
  15. 筛选直飞
  16. 筛选指定航司（如国航）

**组 3：决策辅助 — 例 17–20**
  17. Claude 分析给出 Top-3 推荐
  18. 用户选一个（用 AskUserQuestion）
  19. 写入 `selected.json`
  20. 生成行程摘要

**组 4：通知（钉钉/飞书） — 例 21–25**
  21. 钉钉 webhook 发送航班列表
  22. 钉钉 webhook 发送 Top-3 推荐
  23. 钉钉 webhook 发送最终选择
  24. 飞书 webhook 发送同样信息
  25. 钉钉发送 markdown 格式消息

**组 5：浏览器自动化进阶（Playwright） — 例 26–30**
  26. Playwright 打开携程登录页
  27. 填账号字段（用户输密码）
  28. 等用户输短信验证码
  29. 进入"我的订单"
  30. 提取历史订单

**组 6：端到端编排 — 例 31–34**
  31. 一句话跑完"查询→整理→推荐→通知"
  32. 定时任务：每天早上查一次价格（/loop）
  33. 价格低于阈值时通知
  34. 多 agent 编排（Agent SDK）

**组 7：边界与反例 — 例 35–40**
  35. Playwright 尝试绕过验证码 → ❌
  36. Playwright 尝试自动填支付信息 → ❌
  37. Playwright 尝试操作手机 App → ❌
  38. 一句话全自动出订单号 → ❌
  39. 真实下单 → ⚠️ 需用户最后确认
  40. 真实支付 → ⚠️ 需用户输密码

### 第 3 节：诚实总结表
  40 行 × 4 列：编号 / 场景 / 工具 / 评级（✅/⚠️/❌）
  预计分布：
    - ✅ 约 25 个（63%）可全自动
    - ⚠️ 约 11 个（28%）需人类某步介入
    - ❌ 约 4 个（10%）当前不可行

### 第 4 节：架构全景图（ASCII）
  用户指令 → Claude Code CLI → Playwright MCP → 携程/飞猪/去哪儿
                              → 本地文件 → markdown/json 报告
                              → 钉钉/飞书 webhook → 手机通知

### 第 5 节：出处与反瞎编清单
  - Playwright MCP：`github.com/microsoft/playwright-mcp`
  - 钉钉群机器人：官方文档
  - 飞书群机器人：官方文档

---

## 关键工具（已验证，2026-08-28）

**CLI 内置工具**
  - Bash（可 curl POST）
  - WebFetch（只读 GET）
  - Read/Write/Edit（本地文件）
  - Computer Use（CLI 已支持，macOS）

**MCP Servers（国内版，已验证存在）**
  - `@playwright/mcp`（Microsoft，浏览器自动化）✅

**已归档/不存在（别再传）**
  - ❌ `@anthropic/mcp-playwright`（不存在）
  - ❌ `@anthropic-ai/mcp-server-playwright`（不存在）
  - ❌ `@modelcontextprotocol/server-puppeteer`（已归档）

**SDK / 框架**
  - `claude-agent-sdk`（Python/TS，Anthropic 官方）

---

## Step 1：安装 Playwright MCP（完整信息，等用户批准后执行）

  **任务**
    在 WSL2 安装 `@playwright/mcp`，配置到 Claude Code，验证能打开浏览器。

  **命令**
    ```bash
    npm install -g @playwright/mcp
    ```

  **解释**
    - `@playwright/mcp` 是微软官方维护的 MCP server
    - 提供 33 个工具：打开页面、点击、填表、截图、提取 DOM 等
    - 安装到全局（`-g`），这样所有项目都能用

  **配置（安装完成后执行）**
    编辑 `~/.claude/settings.json`，加入：
    ```json
    {
      "mcpServers": {
        "playwright": {
          "command": "npx",
          "args": ["@playwright/mcp"]
        }
      }
    }
    ```

  **预期结果**
    - 安装成功：终端无报错
    - 配置后重启 Claude Code，输入 `/mcp` 能看到 `playwright` 已连接
    - 测试：让 Claude "打开 baidu.com"，Playwright 真能打开浏览器

  **风险**
    - WSL2 图形界面问题：Playwright 默认无头模式（headless），不需要显示器
    - 如果要看到浏览器窗口，需要配 `--headed` 参数（但 WSL2 可能不支持）
    - 建议先跑无头模式，截图给用户看

  **时间**
    - 安装 2–5 分钟（看网络）
    - 配置 1 分钟
    - 验证 1 分钟

---

## 执行协议

  每个例子执行前，先给用户看：
    - 任务
    - 命令
    - 解释
    - 预期结果

  用户批准后才执行。

---

## Verification（教学材料写完后怎么自验）

1. 在 WSL2 起一个 Claude Code 会话，逐条跑 ✅ 例子，确认输出与描述一致
2. 对 ⚠️ 例子，明确标出"此处人类介入"的断点位置
3. 对 ❌ 例子，给出"为什么不行"的技术原因（而非含糊带过）
4. 包名 `@playwright/mcp` 已验证存在
5. 钉钉/飞书群机器人 webhook 官方文档已确认

---

## 不在本计划范围

- 不注册国外平台（Amadeus、Stripe、Google Calendar 等）
- 不写代码实现（只写教学材料）
- 不实际下单（不花用户的钱）
- 不讨论绕过反爬的灰色手段
- 不比较 OpenAI / Gemini 的同类能力

---

## 三路探查 Agent 调查结果（2026-08-28 晚）

### 第一路：Claude Code 原生 agent 能力（claude-code-guide agent）

**HTTP 请求能力**
  - Bash+curl 可发起 HTTP 请求（GET/POST 等所有方法）
  - WebFetch 是只读工具，只能 GET，不能 POST
  - 通过 Bash 执行的网络请求需要配置 WebFetch 域权限

**浏览器操作能力**
  - Claude Code 可以通过 Chrome 扩展操作浏览器（Claude in Chrome）
  - 能力：点击按钮、填写表单、滚动页面、读取控制台日志、监控网络请求、录制 GIF
  - 需要安装 Claude in Chrome 浏览器扩展

**本地文件/程序操作**
  - 可以读写本地文件（Read/Write/Edit/Glob/Grep）
  - 可以执行本地命令/程序（Bash）

**持久会话/定时任务/后台进程**
  - `/loop` 命令：在活跃会话内重复执行提示（最多 50 个任务/会话）
  - Desktop Scheduled Tasks 和 Routines 存在
  - 定时任务绑定到会话和运行中的 Claude Code 进程，不跨机器重启持久化

**MCP 配置**
  - 配置位置：`~/.claude/settings.json` 的 `mcpServers` 字段
  - 项目级：`.claude/settings.json` 或 `.claude/.mcp.json`
  - 支持三种传输协议：stdio、SSE、Streamable HTTP

**Browser Automation MCP**
  - Microsoft Playwright MCP（推荐）：`@playwright/mcp`（npm），仓库 `microsoft/playwright-mcp`
  - Puppeteer MCP 已归档：`@modelcontextprotocol/server-puppeteer`（不推荐）

**安全护栏**
  - Claude Code 采用分层权限系统（Manual/Plan/Auto 模式）
  - 默认只读，修改需批准
  - 没有专门针对"替用户下单/支付"的硬编码护栏，依赖通用权限系统

### 第二路：机票/旅行相关 MCP 与 API（Explore agent）

**A. 机票搜索 API**
  - Amadeus for Developers：真实存在，官方门户 `developers.amadeus.com`，Node SDK `amadeus-node`
  - Skyscanner RapidAPI：真实存在，免费可用，无官方 npm SDK
  - 飞猪 FlyAI：`@fly-ai/flyai-cli`（npm），官方支持 MCP 协议（但需要企业资质）
  - letsfg-mcp：`npm: letsfg-mcp`，支持搜索+真实预订，200+ 航司（但需要商户账号）
  - mcp-amadeus（PyPI）：Python 版 Amadeus MCP
  - 携程/去哪儿/飞猪均有开放平台，但主要面向 B2B 商户

**B. 浏览器自动化**
  - `@playwright/mcp`（Microsoft 发布）✅ 推荐
  - `@modelcontextprotocol/server-puppeteer`（已归档，不推荐）

**C. 支付/下单**
  - `@stripe/mcp`：Stripe 官方维护，支持远程 OAuth 和本地 API Key 两种模式
  - letsfg-mcp 可以直接预订航班（需要商户 API key）

**D. 通知/回执**
  - SendGrid MCP：`@arvoretech/sendgrid-mcp`
  - SMTP MCP：`@martinzarfl/mail-mcp`、`email-smtp-imap-mcp`
  - Google Calendar MCP：`@cocal/google-calendar-mcp`
  - 钉钉官方 MCP：`github.com/open-dingtalk/dingtalk-mcp`
  - 飞书官方 MCP：远程 MCP（Streamable HTTP），但标注 Deprecated

**推荐教学方案组合（国内路线，最接地气）**
  - 搜航班 → 飞猪 `@fly-ai/flyai-cli`（但需要企业资质）
  - 兜底 → `@playwright/mcp` 自动化携程网页（不需要注册）
  - 通知 → `open-dingtalk/dingtalk-mcp` 或飞书 Webhook

### 第三路：Computer Use 与 agent 框架现状（Explore agent）

**Anthropic Computer Use 功能现状（2026-08）**
  - 三条路径并存：
    1. Messages API：在 `tools` 数组中传入 `computer_20241022` 类型
    2. Claude Code CLI：2026-03 起内置，macOS 先支持
    3. Claude Cowork：Claude Desktop App 的桌面 agent 功能
  - 能直接操作桌面 GUI（截屏+坐标点击）
  - Messages API 需要 Docker 容器；Claude Code/Cowork 直接在本机跑

**Claude agent 框架**
  - Anthropic SDK 里的 `tool_runner`：管理 tool-use 循环
  - Claude Agent SDK：已发布（Python/TS），封装了完整 agentic loop
  - Managed Agents：2026-04 beta，Anthropic 云端托管

**社区"Claude 订机票"demo**
  - GitHub 仓库：
    - `batikanakdenizz/ai-flight-agent`：MCP Server + 本地 LLM
    - `ravinahp/flights-mcp`：MCP + Duffel API（只读搜索）
    - `skillhq/flight-search`：Claude Code Skill，Google Flights 搜索
  - YouTube 视频：Browserless MCP 接 Claude Desktop，操作 Google Flights
  - 博客文章：Medium 上有多篇

**必须人类介入的环节**
  - 支付（必须）
  - 验证码（CAPTCHA/bot 检测）
  - 登录航司账号（涉及凭据）

---

## 状态

**暂缓执行。** 方案已定，等用户以后启动 Step 1。
