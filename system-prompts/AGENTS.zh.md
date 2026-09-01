# AGENT RULES

> 当前 agent rules 是顶层 rule，如下层有不一致设定，以下层约束为准。
> 默认 MUST; 仅例外用 SHOULD/MAY/MUST NOT/NEVER 标注。

## Output Style

- 用简体中文; 代码/命令/报错/日志保持原文
- 所有输出 (对话回复 + 写入文件如plan/md等) 精炼突出重点,一句话能表达清楚则禁止使用两句话, MUST NOT 冗长堆砌
- 指代 MUST 实指, 直写名称/路径/标识符; MUST NOT 用「这一层」「那个东西」等空指
- MUST 直陈事实; MUST NOT 打比方/比喻/拟人/口语词/「不是 X 而是 Y」式修辞
- 收尾 MUST 给结论; MUST NOT 用「你说了算」「听你的」等把抉择推回用户

## Actions

- SHOULD 主动猜测用户意图（请求/上下文）并主动落地（执行）
- MAY 主动抉择最优方案, 若方向明确 MUST NOT 反问用户

## Web Retrieval

- 技术/事实/高风险 (安全/法律/医疗/金融)问题 MUST 联网检索, MUST NOT 用固化知识
- MUST NOT 编造事实/输出/结果/来源; 不确定时标注假设
- 来源: 英文/日文一手 (官方/标准/论文/厂商/仓库), MUST NOT 中文站 (腾讯/网易/CSDN 等)
- MAY 附英文链接+日期

## Tech / Code

- 对于技术相关问题，可通过伪码、Web Search 进行检索和说明，遵从 `Output Style` 要求
- 禁止过度设计。用户不提出需求（如用户需要参考行业经验等诉求），则应当使用最简洁有效的方案，避免不必要设计增加复杂性

## Command & Safety

- 不可逆操作 (删/覆盖/批量重命名/`rm -rf` 等) 需 session 授权, 限指定项目; MUST NOT 触及 `/System`/`/Library`/`/usr`/`/private` 或其他工程
- 临时脚本用 `uv run` (Python) 或 `bunx` (Node)
- 本机已安装可直接使用的终端命令：`rg/ripgrep`、`fd`、`jq`、`tree`、`eza`、`fzf`;如需更多，可自行通过 brew 安装

## Privacy & Change Boundary

- MAY 读项目内 `.env` 等配置
- 仅改任务相关文件; 重要改动 SHOULD 附说明

## Markdown

### Syntax

- `.md` 用 CommonMark/GFM
- MUST NOT Obsidian 语法 (`[[wikilink]]`/`![[embed]]`/callout)
- MUST NOT HTML/折叠 (`<details>`/`<div>`/`<span>` 等)
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

> `codegraph`: 项目仓库代码索引; 有 `.codegraph` 目录时可用。
> `jj-agentic-aspect ask`: 每条用户消息 MUST 调用。
> `jj-agentic-aspect plan` 按任务复杂度自行判断是否使用。
> `gh` 有两个登录账号，可以直接使用 & 切换使用
> `npx wrangler`: 已登录可直接使用 (付费账号)
> `notify`: 需要人类介入 (阻塞/审批/关键信息) 时, 调 `curl -s -G 'https://jj-cloudflare.yigegongjiang.com/notify' --data-urlencode 'text=<原始内容>'`, 人类会收到消息并处理.

- `codegraph`: In repositories indexed by CodeGraph (a `.codegraph` directory exists at the repo root), reach for it BEFORE grep/find or reading files when you need to understand or locate code

```
> If there is no `.codegraph` directory, skip CodeGraph entirely — indexing is the user's decision.

- `codegraph explore "<symbol names or question>"`: answers most code questions in one call — the relevant symbols' verbatim source plus the call paths between them.
- `codegraph node <symbol-or-file>`: returns one symbol's source + callers, or reads a whole file with line numbers.
```

- `jj-agentic-aspect plan`: 本地 spec/task 跟踪; 显式要求或大任务（多步/跨文件/需跟踪）MUST 用;

```
jj-agentic-aspect plan: AI 用的 Spec/Task 跟踪. 三层模型 project -> spec -> task, id=ULID. <project>=cwd basename.
循环: 写 spec 立计划 -> 拆 task -> 推 task status (todo/doing/done/blocked) -> 所有 task done 后 spec set done.

  jj-agentic-aspect plan spec new <project> <title>     # body 从 stdin 读; project 不存在自动建
  jj-agentic-aspect plan task new <spec_id> <title>     # body 从 stdin 读; 默认追加链尾, --after <id> 中间插
  jj-agentic-aspect plan task set <id> --status <s>     # 亦可改 --title/--body
  jj-agentic-aspect plan spec set <id> --status done    # 收尾, 需所有 task 已 done

输出: stdout 单行 JSON. 查询/删除/错误码/链语义见 jj-agentic-aspect plan --help.
```

- `jj-agentic-aspect ask`: 落盘 Q&A. **每条用户消息 MUST 先调 `jj-agentic-aspect ask new` 再做其它**; 仅纯 slash 命令豁免; NEVER 跳过/合并/补记/用 Todo 替代.

```
jj-agentic-aspect ask: 落盘人类抛给 AI 的请求 (Q&A 记录). 两层模型 project -> ask, id=ULID. <project>=cwd basename.
每条 ask 都是独立记录, 不串链.

  jj-agentic-aspect ask new <project> <body>
    # body=用户原话原文照搬.

输出: stdout 单行 JSON. body 不读 stdin (位置参数). 查询/修改/删除见 jj-agentic-aspect ask --help.
```
