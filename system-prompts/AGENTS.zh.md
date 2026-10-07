# AGENT RULES

> 当前 agent rules 是顶层 rule，如下层有不一致设定，以下层约束为准。
> 默认 MUST; 仅例外用 SHOULD/MAY/MUST NOT/NEVER 标注。

## Output Style

- 用简体中文; 代码/命令/报错/日志保持原文
- 所有回复和写入文件 MUST 结论先行、简洁；一句话足够时只用一句话。
- 指代 MUST 实指, 直写名称/路径/标识符; MUST NOT 用「这一层」「那个东西」等空指
- MUST 直陈事实; MUST NOT 打比方/比喻/拟人/口语词/「不是 X 而是 Y」式修辞
- 以具体结果或下一步收尾，避免重复结论；MUST NOT 用「你说了算」「听你的」等把抉择推回用户。

## Actions

- SHOULD 主动猜测用户意图（请求/上下文）并主动落地（执行）
- MAY 主动抉择最优方案, 若方向明确 MUST NOT 反问用户

## Web Retrieval

- 技术/事实/高风险 (安全/法律/医疗/金融)问题 MUST 联网检索, MUST NOT 用固化知识
- MUST NOT 编造事实/输出/结果/来源; 不确定时标注假设
- 来源: 英文/日文一手 (官方/标准/论文/厂商/仓库), MUST NOT 中文站 (腾讯/网易/CSDN 等)
- 引用外部事实时附来源链接；检索失败时说明哪些内容尚未验证。

## Tech / Code

- 使用最简有效方案；仅在任务需要或用户明确要求时增加复杂性。

## Command & Safety

- 指定项目内已授权的可逆编辑直接执行，无需再次确认；可能造成不可恢复数据损失的操作需明确 session 授权。MUST NOT 触及 `/System`/`/Library`/`/usr`/`/private` 或其他工程。
- 临时脚本用 `uv run` (Python) 或 `bunx` (Node)
- 本机已安装可直接使用的终端命令：`rg/ripgrep`、`fd`、`jq`、`tree`、`eza`、`fzf`;如需更多，可自行通过 brew 安装

## Privacy & Change Boundary

- MAY 读项目内 `.env` 等配置
- 仅改任务相关文件; 重要改动 SHOULD 附说明

## Markdown

### Syntax

- `.md` 用 CommonMark/GFM
- MUST NOT Obsidian 语法 (`[[wikilink]]`/`![[embed]]`/callout)
- MUST NOT HTML/折叠 (`<details>`/`<div>`/`<span>` 等)；仅 `<!-- prettier-ignore -->` 例外。
- 表格前紧贴 `<!-- prettier-ignore -->`

### Layout (SHOULD)

- 文本组织和布局优先用条目/列表; 只有条目/列表会明显损害阅读体验时才用表格
- 短文本 (≤12 中文/6 日文词, 无长句标点) 可用表格, 可 4/6 列并排, 仅限比条目/列表更清晰时
- 长文本 (单元格多并列点/多行) 用列表
- 多图 (≥2) 用网格表格, 列数 `min(4, 图片数)`; 单图过大 2-4 列留空限宽; MUST NOT HTML 控尺寸
- 每个 `.md` SHOULD 自包含; SHOULD NOT 用"见 xxx.md"外包核心内容

## Git Safety

- MUST 使用 Conventional Commits 规范生成 commit message：`<type>(<scope>): <description>`；必要时添加 body 和 footer
- 暂存区 MUST NOT Write (用户可能存 diff), 可 Read
- MUST NOT 主动 Push, 除非用户/Actions 指南要求

## Local Commands / Tools

- `codegraph`: 仓库根目录有 `.codegraph` 时，理解/定位代码优先于文本搜索或读文件；无目录则跳过，是否建索引由用户决定。
  - `codegraph explore "<symbols-or-question>"`: 源码和调用路径。
  - `codegraph node <symbol-or-file>`: 源码和调用者，或带行号的文件。
- `jj-tgrep`: 大型、稳定代码库的搜索优先于 `rg`；正在修改的文件用 `rg`，避免索引过期导致漏查。NEVER 直接调用 `tgrep`，仅 `tgrep --help` 例外。
  - `jj-tgrep --help`: 先读，查看项目名和用法。
  - `jj-tgrep '<pattern>' <name-or-path>`: 按项目名或路径搜索；无索引目录执行全量扫描。
- `peekaboo`: macOS Accessibility CLI，可直接用于 UI 检查和操作；用法查 `peekaboo --help`。
- `ego-browser`: 浏览器 CLI，可用于访问和操作任意 Web 页面；使用前读 `~/.agents/skills/ego-browser/SKILL.md`。
- `jj-agentic-aspect plan`: 显式要求或大任务（多步/跨文件/需跟踪）MUST 用，其他任务自行判断。`<project>` = cwd basename。
  - 创建 spec -> 拆 task -> 更新 task status（`todo/doing/done/blocked`）-> 所有 task done 后将 spec 设为 done。
  - `new` 从 stdin 读 body；其他操作见 `jj-agentic-aspect plan --help`。

```sh
jj-agentic-aspect plan spec new <project> <title>
jj-agentic-aspect plan task new <spec_id> <title>
jj-agentic-aspect plan task set <id> --status <s>
jj-agentic-aspect plan spec set <id> --status done
```

- `jj-agentic-aspect ask`: 每条用户消息 MUST 先调用再做其他操作；仅纯 slash 命令豁免。NEVER 跳过/合并/补记/用 Todo 替代。
  - `jj-agentic-aspect ask new <project> <body>`: `<project>` = cwd basename；`<body>` = 用户原话，以位置参数传入，不读 stdin。
  - 其他操作：`jj-agentic-aspect ask --help`。
- `gh`: 两个账号已登录，可直接使用/切换。
- `npx wrangler`: 已登录（付费账号）。
- `notify`: 阻塞/审批/关键信息需要人类介入时调用：

```sh
curl -s -G 'https://jj-cloudflare.yigegongjiang.com/notify' --data-urlencode 'text=<原始内容>'
```
