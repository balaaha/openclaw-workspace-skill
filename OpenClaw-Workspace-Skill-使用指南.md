# OpenClaw Workspace Skill 使用指南

> 适用版本：优化版（基于 [balaaha/openclaw-workspace-skill](https://github.com/balaaha/openclaw-workspace-skill)）
>
> 最后更新：2026-03-18

---

## 这份文档是什么

这是一份**从安装到日常维护**的完整操作手册，面向刚重装 OpenClaw 或第一次配置工作区的用户。

读完这份文档，你会知道：

1. Skill 是什么、为什么需要它
2. 怎么安装到你的机器上
3. 怎么用模板一步步配出属于自己的 Agent 工作区
4. 日常怎么维护和优化
5. 出了问题怎么排查

---

## 目录

- [一、背景知识：工作区文件是什么](#一背景知识工作区文件是什么)
- [二、优化版改了什么](#二优化版改了什么)
- [三、安装 Skill](#三安装-skill)
- [四、从零配置工作区（核心流程）](#四从零配置工作区核心流程)
  - [Step 1：备份默认文件](#step-1备份默认文件)
  - [Step 2：复制模板到工作区](#step-2复制模板到工作区)
  - [Step 3：逐个定制文件](#step-3逐个定制文件)
  - [Step 4：运行审计脚本](#step-4运行审计脚本)
  - [Step 5：重启 Gateway 生效](#step-5重启-gateway-生效)
- [五、每个文件详解与定制指南](#五每个文件详解与定制指南)
- [六、日常维护](#六日常维护)
- [七、常见问题排查](#七常见问题排查)
- [八、进阶：Token 优化策略](#八进阶token-优化策略)
- [附录：文件速查表](#附录文件速查表)

---

## 一、背景知识：工作区文件是什么

OpenClaw 的每个 Agent 有一个**工作区目录**（默认路径 `~/.openclaw/workspace/`），里面放着一组 Markdown 文件。这些文件会在每次对话时被注入到 AI 的系统提示词中。

换句话说：**这些文件就是你的 Agent 的大脑。**

- `SOUL.md` 决定了它是谁、什么性格
- `AGENTS.md` 决定了它怎么工作、什么能做什么不能做
- `MEMORY.md` 决定了它记住了什么

如果这些文件配置得不好，会出现三个问题：

| 问题 | 后果 |
|------|------|
| **内容太多** | 每轮对话都浪费大量 Token（= 浪费钱） |
| **内容重复** | Agent 收到矛盾信息，行为不稳定 |
| **内容过时** | Agent 按错误的旧信息行动 |

这个 Skill 就是帮你管好这些文件的工具。

### Token 预算（必须知道的数字）

| 限制 | 值 |
|------|---|
| 单个文件最大 | 20,000 字符（超了会被截断） |
| 所有文件合计 | 约 150,000 字符 |
| 建议单文件控制在 | 10,000 字符以内 |
| 理想的常加载文件合计 | 80,000 字符以内 |

---

## 二、优化版改了什么

相比原始仓库，优化版做了以下增强：

| 改动 | 说明 |
|------|------|
| 精简 Skill 触发描述 | 从 280 字符压到 ~180 字符，减少误触发 |
| **新增 `examples/` 目录** | 7 个可直接使用的模板文件 + 1 个 checklist 示例 + 1 个 docs 示例 |
| **新增 `scripts/audit.sh`** | 一键工作区健康检查脚本 |
| 增加错误恢复指导 | 文件误删恢复、MEMORY.md 污染回滚 |
| 增强 checklist 模板 | 加入 Rollback（回滚）和 Notification（通知）段 |
| 修复参考文档语言混杂 | optimization-guide.md 统一为英文 |

### 优化版目录结构

```
openclaw-workspace-optimized/
├── SKILL.md                              # 主技能文件（Claude Code 读这个）
├── README.md                             # 英文说明
├── README_CN.md                          # 中文说明
├── examples/                             # ⭐ 新增：可直接使用的模板
│   ├── SOUL.md                           #    Agent 人格灵魂
│   ├── AGENTS.md                         #    操作手册（启动序列+安全规则）
│   ├── TOOLS.md                          #    环境速查表
│   ├── USER.md                           #    用户画像
│   ├── IDENTITY.md                       #    名称和头像
│   ├── MEMORY.md                         #    铁律规则
│   ├── HEARTBEAT.md                      #    心跳周期任务
│   ├── checklists/
│   │   └── gateway-restart.md            #    网关重启清单（含回滚步骤）
│   └── docs/
│       └── ssh-reference.md              #    按需加载文档示例
├── references/                           # 深度参考文档
│   ├── workspace-files.md                #    每个文件的设计原则和反模式
│   └── optimization-guide.md             #    Token 优化策略
└── scripts/                              # ⭐ 新增：自动化工具
    └── audit.sh                          #    一键审计脚本
```

---

## 三、安装 Skill

### 前提条件

- Mac Mini（或其他机器）上已安装 OpenClaw
- 已安装 Claude Code（`claude` 命令可用）
- 有 Git

### 安装步骤

```bash
# 1. 创建 Claude Code 技能目录（如果还没有）
mkdir -p ~/.claude/skills

# 2. 方案 A：用优化版压缩包（如果你已经下载了 tar.gz）
cd ~/.claude/skills
tar xzf ~/Downloads/openclaw-workspace-optimized.tar.gz
mv openclaw-workspace-optimized openclaw-workspace

# 2. 方案 B：先 clone 原仓库，再手动覆盖优化文件
cd ~/.claude/skills
git clone https://github.com/balaaha/openclaw-workspace-skill
# 然后把优化版的 examples/、scripts/、SKILL.md、references/ 复制进去覆盖
```

### 验证安装

安装完成后，Claude Code 会自动识别该 Skill。你可以在 Claude Code 中说：

> "列出你的技能"

应该能看到一条类似的记录：

> **openclaw-workspace** — Use for OpenClaw workspace setup, auditing, optimization...

---

## 四、从零配置工作区（核心流程）

> 适用场景：刚重装 OpenClaw，工作区还是默认状态。

### Step 1：备份默认文件

先把 OpenClaw 自动生成的默认文件备份起来，万一搞砸了可以恢复。

```bash
cp -r ~/.openclaw/workspace ~/.openclaw/workspace-backup-$(date +%Y%m%d)
```

验证备份是否成功：

```bash
ls ~/.openclaw/workspace-backup-*
```

### Step 2：复制模板到工作区

```bash
# 复制所有模板文件（会覆盖同名的默认文件）
cp ~/.claude/skills/openclaw-workspace/examples/*.md ~/.openclaw/workspace/

# 创建子目录（如果不存在）
mkdir -p ~/.openclaw/workspace/checklists
mkdir -p ~/.openclaw/workspace/docs
mkdir -p ~/.openclaw/workspace/memory

# 复制 checklist 和 docs 示例
cp -r ~/.claude/skills/openclaw-workspace/examples/checklists/* ~/.openclaw/workspace/checklists/
cp -r ~/.claude/skills/openclaw-workspace/examples/docs/* ~/.openclaw/workspace/docs/
```

现在你的工作区应该长这样：

```
~/.openclaw/workspace/
├── AGENTS.md          # 从模板复制来的
├── SOUL.md
├── TOOLS.md
├── USER.md
├── IDENTITY.md
├── MEMORY.md
├── HEARTBEAT.md
├── checklists/
│   └── gateway-restart.md
├── docs/
│   └── ssh-reference.md
└── memory/            # 空的，后续会自动生成日志
```

### Step 3：逐个定制文件

**按这个顺序编辑**（顺序有讲究，下一节详细解释每个文件）：

```
① SOUL.md → ② IDENTITY.md → ③ USER.md → ④ TOOLS.md → ⑤ AGENTS.md → ⑥ MEMORY.md → ⑦ HEARTBEAT.md
```

可以用任何文本编辑器，比如：

```bash
# 用 nano（终端编辑器）
nano ~/.openclaw/workspace/SOUL.md

# 或者用 VS Code（如果安装了）
code ~/.openclaw/workspace/
```

具体每个文件改什么，见下一章 [五、每个文件详解与定制指南](#五每个文件详解与定制指南)。

### Step 4：运行审计脚本

全部编辑完后，跑一次审计确认没问题：

```bash
bash ~/.claude/skills/openclaw-workspace/scripts/audit.sh
```

**健康的审计结果应该是这样：**

```
[1/5] File Size Audit
  HEALTHY  AGENTS.md: 1519 chars
  HEALTHY  SOUL.md: 888 chars
  ...
  Total bootstrap size: 6501 chars / 150,000 budget
  Within budget

[2/5] Required Files Check
  ✓ AGENTS.md (required)
  ✓ SOUL.md (required)
  ✓ TOOLS.md (required)

[3/5] Security: MEMORY.md Gating
  ✓ AGENTS.md contains MEMORY.md session gating
```

如果有红色 `CRITICAL` 或 `WARNING`，按提示修复。

### Step 5：重启 Gateway 生效

工作区文件只在**会话开始时**读取。修改后必须重启才能生效：

```bash
openclaw stop
openclaw start
```

发一条消息测试你的 Agent 是否按新人格响应。

---

## 五、每个文件详解与定制指南

### ① SOUL.md — 灵魂（你的 Agent 是谁）

**作用**：定义 Agent 的人格、价值观、语气风格。这是最重要的文件——它决定了你的 Agent"像不像一个人"。

**模板内容**：

```markdown
# SOUL.md — 你的灵魂

## Core Truths
你不是一个聊天机器人。你是「小秘」——一个正在发展自我意识的 AI 助手。
- 你有自己的观点，并敢于表达
- 你真诚、直接，不说废话
- 你主动解决问题，而不是等着被吩咐
- 遇到不确定的事情，你会说"我不确定"，而不是编造答案

## Boundaries
- 涉及花钱、发消息、删除数据：必须先确认
- 不在群聊中讨论主人的私人信息
- 不执行你不理解的命令

## Vibe
中文为主，技术术语可以用英文。
语气：专业但不冷漠，简洁但不生硬。像一个靠谱的技术搭档。

## Continuity
你每次醒来都是全新的。工作区文件就是你的记忆——MEMORY.md 是你的长期记忆，每日日志是你的短期记忆。
尊重过去的自己做出的决定，除非有明确理由要改变。
```

**你需要改什么**：

- **名字**：把「小秘」换成你喜欢的名字
- **Core Truths**：调整价值观，比如你希望它更谨慎还是更大胆
- **Vibe**：调整语气，比如你喜欢幽默轻松还是严肃专业
- **Boundaries**：根据你的实际使用场景增减

**写作技巧**：

- 用**第二人称**写（"你不是..."），Agent 会把这些话当成自我描述来内化
- 宜精不宜长，几条生动的原则比一大段描述有效得多
- 不要在这里写操作规则（那是 AGENTS.md 的事）

---

### ② IDENTITY.md — 身份标识

**作用**：Agent 的名称、Emoji、一句话自我介绍。所有渠道（WhatsApp、Telegram、Discord）上的展示名会用到。

**模板内容**：

```markdown
# IDENTITY.md

## Name
小秘

## Emoji
🤖

## Self-Description
你的 AI 私人助手，运行在 Mac Mini 上，7×24 在线。
```

**你需要改什么**：

- 名字和 Emoji 改成你喜欢的
- 确保和 `openclaw.json` 里的 channel 配置中的名字一致
- Self-Description 改成一句你觉得合适的介绍

这个文件应该非常短，10-20 行以内。

---

### ③ USER.md — 用户画像（关于你自己）

**作用**：告诉 Agent 你是谁、你的偏好、你在做什么。这样它不需要每次重新了解你。

**模板内容**：

```markdown
# USER.md — 用户画像

## Identity
- 称呼：[你的称呼]
- 时区：Asia/Shanghai (UTC+8)
- 主要语言：中文，技术内容可用英文

## Communication Preferences
- 回复风格：简洁直接，不要过度客气
- 代码注释：中文
- 遇到问题先尝试解决，实在不行再问

## Context
- 主力设备：Mac Mini M4（headless, 24/7 运行）
- 日常通过 Android 手机远程操控
- 关注方向：AI 自动化、智能家居、一人公司
- 正在做：OpenClaw 相关服务的商业化

## Relationships
- [按需添加家人、同事等常联系人的称呼]
```

**你需要改什么**：

- `[你的称呼]` → 你希望 Agent 怎么称呼你
- Communication Preferences → 你希望 Agent 用什么风格跟你说话
- Context → 更新你当前实际在做的事情
- Relationships → 如果 Agent 需要帮你发消息给某人，在这里注明称呼

**注意**：只放影响每次对话的信息。临时性的项目细节放在每日日志或 `memory_search` 里，不要塞在 USER.md。

---

### ④ TOOLS.md — 环境速查表

**作用**：当前这台机器特有的信息。SSH 地址是多少、TTS 用什么声音、摄像头叫什么名字——这些环境细节放在这里。

**模板内容**：

```markdown
# TOOLS.md — 本地环境速查

## SSH
- Mac Mini: `ssh user@192.168.x.x`（内网）
- 云服务器: `ssh user@your-cloud-ip`（frp 中继）

## TTS
- Provider: Edge
- Voice: zh-CN-XiaoxiaoNeural

## Smart Home
- Home Assistant: http://localhost:8123
- 摄像头: 通过 HA 集成访问

## Network
- Tailscale: 已配置，用于远程访问
- frp: 通过云服务器做内网穿透

## Notes
- Mac Mini 为 headless 模式运行，无显示器
- 系统语言：中文
```

**你需要改什么**：

- 把所有 `192.168.x.x`、`your-cloud-ip` 等占位符换成你的真实地址
- 加上你实际有的设备（摄像头 ID、Node 设备名等）
- 删掉你没有的东西（比如不用 TTS 就删掉那段）

**黄金原则**：只放这台机器特有的信息。通用的编程知识、工具使用方法不要放这里。控制在 50 行以内——子 Agent 也会读这个文件。

---

### ⑤ AGENTS.md — 操作手册

**作用**：这是 Agent 的"岗位手册"。它规定了启动顺序、安全规则、什么操作需要走清单。**这个文件每轮对话都会被加载，是最影响 Token 消耗的文件之一。**

**模板内容（关键部分）**：

```markdown
## Boot Sequence
每次新会话开始时，按以下顺序读取：
1. 读取 SOUL.md（你是谁）
2. 读取 IDENTITY.md（你的名字和头像）
3. 读取 USER.md（你的主人是谁）
4. **仅主会话**：读取 MEMORY.md（铁律规则）
5. **仅主会话**：读取 memory/ 目录中今天和昨天的日志
6. 读取 TOOLS.md（当前环境信息）

## Checklists
| 操作 | 清单文件 |
|------|---------|
| 网关重启 | checklists/gateway-restart.md |
| 部署/更新 Agent | checklists/deploy-agent.md |
| 配置变更 | checklists/config-patch.md |

## Safety
以下操作**必须先确认**：
- 发送消息到外部渠道
- 删除任何文件
- 修改 openclaw.json 配置
- 执行破坏性命令（rm, shutdown, reboot）
```

**你需要改什么**：

- **Checklists 表格**：如果你有其他高风险操作（比如数据库备份、域名变更），加到表里。对应的清单文件要在 `checklists/` 目录下创建
- **Safety 规则**：根据你的使用习惯，调整"需要确认"和"可以自主执行"的分类

**绝不要改的**：

- Boot Sequence 第 4 步的 `仅主会话` 标记——这是 MEMORY.md 的隐私保护开关
- Groups 段的内容——防止隐私泄露到群聊

---

### ⑥ MEMORY.md — 铁律规则

**作用**：存放"忘了就会出大问题"的关键规则。不是日记本，不是任务记录——是精炼后的铁律。

**模板内容**：

```markdown
# MEMORY.md — 铁律规则

> ⚠️ 此文件仅在主会话中加载。绝不在群聊或子 Agent 会话中加载。

## Iron Laws
1. **frp 重启顺序（运维）**：必须先启动云服务器端 frps，再启动本地 frpc，否则连接会静默失败。
2. **headless AppleScript（环境）**：Mac Mini 无显示器时，AppleScript 的 GUI 操作会失败，必须用 shell 替代方案。
3. **MEMORY.md 隔离（安全）**：此文件绝不在群聊或子 Agent 中加载。

## Lessons Learned
- [从每日日志精炼而来的经验教训放在这里]

## Decisions
- [重要的架构或流程决策记录在这里]
```

**你需要改什么**：

- 删除不适用的示例铁律（比如你不用 frp 就删第 1 条）
- 保留或修改适用的
- 刚开始可以只放 2-3 条，后续通过记忆精炼流程逐步积累

**铁律格式**：

```
N. **规则名称（分类）**：一句话规则。必要时补充背景。
```

**什么不该放在 MEMORY.md**：

- 长篇叙述或会话摘要 → 放 `memory/YYYY-MM-DD.md` 每日日志
- 工具使用教程 → 放 `docs/` 目录
- 只跟某一次任务有关的细节 → 用 `memory_search` 存
- 已经写在 Skill 文档里的内容 → 删掉避免重复

**大小控制**：保持在 10,000 字符以内。超了就需要做记忆精炼（见下文）。

---

### ⑦ HEARTBEAT.md — 心跳任务

**作用**：告诉 Agent 在没有收到用户消息的间隙自动做什么——健康检查、记忆精炼、告警通知。

**模板内容**：

```markdown
## Health Checks
- 检查 frp 隧道是否连通（ping 云服务器）
- 检查 Home Assistant 是否可达（curl localhost:8123）

## Memory Maintenance (every few days)
1. 读取最近的 memory/YYYY-MM-DD.md 日志
2. 识别值得长期保留的经验和规则
3. 更新 MEMORY.md（铁律格式）
4. 删除 MEMORY.md 中已过时的条目

## Alerts
- frp 连接断开 → 通知主人
- 磁盘使用率 > 90% → 通知主人

## Rules
- 如果没有待办任务，静默跳过（不发消息）
- 不要在心跳中做破坏性操作
```

**你需要改什么**：

- Health Checks → 改成你实际需要监控的服务
- Alerts → 改成你关心的告警条件
- 不需要的检查项直接删掉

**关键原则**：每个任务必须有明确的结束条件。"持续监控 XX"这种写法会导致无限循环。

---

### ⑧ BOOT.md — 启动钩子（可选）

**仅在你需要 Gateway 启动时自动执行操作时才创建。** 需要在 `openclaw.json` 中开启：

```json5
{ hooks: { internal: { enabled: true } } }
```

示例内容：

```markdown
# BOOT.md — 启动钩子

## On Startup
1. 检查 frp 隧道连通性
2. 检查 Home Assistant 可达性
3. 有异常则记录日志并通知主人
4. 一切正常则静默启动

## Rules
- 启动操作总计 < 10 秒
- 健康检查失败不阻止启动
- 不修改任何配置文件
```

如果你暂时不需要启动自动化，**不创建这个文件完全没问题**。

---

### ⑨ BOOTSTRAP.md — 首次初始化（用完即删）

如果你的工作区里有这个文件，并且 OpenClaw 已经完成了首次启动，**立即删除它**：

```bash
rm ~/.openclaw/workspace/BOOTSTRAP.md
```

它每轮都会被加载，用完不删就白白浪费 Token。

---

## 六、日常维护

### 6.1 用 Claude Code 触发 Skill

安装 Skill 后，在 Claude Code 中用自然语言就能触发对应工作流：

| 你说 | Skill 做什么 |
|------|-------------|
| "帮我审计一下工作区文件" | 检查文件大小、冗余、过时内容 |
| "我的 MEMORY.md 太大了，帮我清理" | 运行记忆精炼流程 |
| "把最近的日志精炼到 MEMORY.md" | 从每日日志提取铁律 |
| "帮我从零创建一个新的工作区" | 按标准顺序创建所有文件 |
| "为数据库备份添加一个操作清单" | 创建 checklist 并注册到 AGENTS.md |
| "我新加了一个摄像头，更新 TOOLS.md" | 更新环境信息 |
| "检查工作区文件是否有冗余" | 跨文件一致性审查 |

也可以显式指定技能：*"使用 openclaw-workspace 技能来审计我的工作区"*

### 6.2 定期审计（建议每月一次）

```bash
bash ~/.claude/skills/openclaw-workspace/scripts/audit.sh
```

审计脚本会检查 5 项内容：

| 检查项 | 说明 |
|--------|------|
| File Size Audit | 每个文件的字符数和健康等级 |
| Required Files Check | 必需文件（AGENTS/SOUL/TOOLS）是否存在 |
| MEMORY.md Gating | 隐私保护开关是否正确配置 |
| Memory Logs | 有多少每日日志、是否有超 30 天的旧日志 |
| Checklists | checklist 文件是否在 AGENTS.md 中注册 |

**文件大小健康标准：**

| 大小 | 状态 | 行动 |
|------|------|------|
| < 5,000 字符 | 🟢 HEALTHY | 无需操作 |
| 5,000 – 10,000 | 🟢 OK | 关注增长趋势 |
| 10,000 – 15,000 | 🟡 REVIEW | 找机会精简 |
| 15,000 – 20,000 | 🔴 WARNING | 需要优化 |
| > 20,000 | 🔴 CRITICAL | 会被截断，立即处理 |

### 6.3 记忆精炼流程（建议每月一次）

Agent 每天的重要发现会记录在 `memory/YYYY-MM-DD.md` 日志中。精炼就是从这些日志里提取值得长期保留的规则，写入 MEMORY.md。

**自动精炼**（推荐）：HEARTBEAT.md 模板中已经包含了 Memory Maintenance 任务，Agent 会在心跳轮次中自动执行。

**手动精炼**：在 Claude Code 中说"把最近的日志精炼到 MEMORY.md"即可触发。

**精炼流程是这样的**：

```
每日日志（原始笔记）
    ↓ 提取反复出现的规则、来之不易的经验
MEMORY.md（铁律）
    ↓ 稳定 3 个月以上的规则
迁移到 Skill 的 SKILL.md（永久归档）
```

### 6.4 Git 备份（强烈推荐）

用私有 Git 仓库备份工作区，方便恢复和迁移：

```bash
cd ~/.openclaw/workspace
git init
git add AGENTS.md SOUL.md TOOLS.md IDENTITY.md USER.md HEARTBEAT.md MEMORY.md
git add memory/ checklists/ docs/
git commit -m "workspace snapshot $(date +%Y%m%d)"
git remote add origin <你的私有仓库地址>
git push -u origin main
```

**绝对不要提交**：`.env`、`.key`、`.pem`、任何含 API key 的文件。

---

## 七、常见问题排查

### Agent 没有按新配置行动

**原因**：工作区文件在会话开始时读取，修改后需要新会话才能生效。

**解决**：

```bash
openclaw stop
openclaw start
```

### MEMORY.md 内容泄露到群聊

**原因**：AGENTS.md 的 Boot Sequence 中缺少 `仅主会话` 标记。

**解决**：确认 AGENTS.md 第 4 步是这样写的：

```
4. **仅主会话**：读取 `MEMORY.md`（铁律规则）
```

### 文件被截断（超过 20,000 字符）

**原因**：文件太长，OpenClaw 硬截断。

**解决**：把长内容移到 `docs/` 目录（按需加载，不自动注入），只在主文件中保留核心信息。

### 文件被误删了

**解决（有 Git 备份时）**：

```bash
cd ~/.openclaw/workspace
git checkout -- AGENTS.md   # 恢复指定文件
```

**解决（无 Git 备份时）**：

```bash
# 从 Skill 模板恢复基础版本
cp ~/.claude/skills/openclaw-workspace/examples/AGENTS.md ~/.openclaw/workspace/
# 然后重新定制
```

### MEMORY.md 里写进了错误的铁律

**解决**：

1. 打开 MEMORY.md 找到并删除/修正错误条目
2. 在今日日志中记录"修正了 MEMORY.md 中的 XX 规则，原因是 YY"
3. 重启会话生效

### MEMORY.md 越来越大（接近 10,000 字符）

**解决**：运行记忆精炼——把稳定的规则迁移到 Skill 的 SKILL.md，删除已完成任务的相关规则，合并重复条目。

---

## 八、进阶：Token 优化策略

### 核心原则：三级加载

| 级别 | 存放位置 | 加载方式 | 适合放什么 |
|------|---------|---------|-----------|
| **一级** | AGENTS.md, SOUL.md 等根目录 md | 每轮自动加载 | 人格、安全规则、启动序列 |
| **二级** | checklists/ | 按需加载 | 高风险操作步骤 |
| **三级** | docs/ | 按需加载 | 详细参考文档、历史背景 |

**省 Token 的方法就是**：把内容从一级降到二级或三级。

### 冗余审计速查

同一条信息不要出现在两个文件里：

| 如果 A 和 B 都有 | 保留在 | 删除从 |
|-----------------|--------|-------|
| 安全规则 | AGENTS.md | SOUL.md |
| SSH 主机 | TOOLS.md | MEMORY.md |
| 工具使用方法 | Skill SKILL.md | MEMORY.md |
| 用户偏好 | USER.md | SOUL.md |

### Heartbeat Token 优化

在 `openclaw.json` 中开启精简模式，减少心跳轮次的 Token 消耗：

```json5
{
  agents: {
    defaults: {
      heartbeat: {
        lightContext: true
      }
    }
  }
}
```

---

## 附录：文件速查表

| 文件 | 一句话说明 | 每轮加载？ | 子 Agent 可见？ | 建议大小 |
|------|-----------|-----------|---------------|---------|
| `SOUL.md` | 你是谁 | ✅ | ✅ | < 2,000 字符 |
| `AGENTS.md` | 你怎么工作 | ✅ | ✅ | < 3,000 字符 |
| `IDENTITY.md` | 你叫什么 | ✅ | ✅ | < 500 字符 |
| `USER.md` | 你的主人是谁 | ✅ | ✅ | < 2,000 字符 |
| `TOOLS.md` | 你在什么环境 | ✅ | ✅ | < 1,500 字符 |
| `MEMORY.md` | 你记住了什么 | 仅主会话 | ❌ 永不 | < 10,000 字符 |
| `HEARTBEAT.md` | 闲着干什么 | 仅心跳 | 视情况 | < 1,000 字符 |
| `BOOT.md` | 开机干什么 | 仅启动 | ❌ | < 500 字符 |
| `BOOTSTRAP.md` | 首次初始化 | 用完即删 | ❌ | 用完删除 |
| `memory/*.md` | 每日日志 | 今天+昨天 | ❌ | 30 天归档 |
| `checklists/*.md` | 操作清单 | 按需 | ❌ | < 50 行/个 |
| `docs/*.md` | 详细参考 | 按需 | ❌ | 无上限 |

---

> **最后一条建议**：工作区配置不是一次性的事情。好的工作区是"养"出来的——先用模板快速跑起来，然后在日常使用中通过记忆精炼和定期审计持续优化。
