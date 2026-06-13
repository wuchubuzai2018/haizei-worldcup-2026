# 2026 美加墨世界杯数据技能

> 百度体育世界杯数据抓取工具集 / Baidu Sports World Cup data scraper

**2026 年美加墨国际足联世界杯** 全量数据，包括 48 支球队、赛程、比赛详情、阵容、统计、排名等。零依赖、纯 Node.js CLI 工具，可作为 OpenCode/Claude Code 等 AI 助手的 Skill 使用，也可独立运行。

## ✨ 特性

- 🏆 **覆盖 7 大类数据**：球队配置、赛程、比赛详情、球队详情、球员详情、排名、淘汰赛
- 🇺🇳 **48 支球队全配置**：12 个小组 × 4 队，含分档/东道主/晋级状态
- 📅 **多维筛选**：按日期/小组/球队/赛事/数据维度查询
- 📊 **结构化 JSON 输出**：每个脚本输出标准化 JSON，便于二次处理
- 🤖 **AI 友好**：内置 12 个典型用户问题工作流（`references/workflows.md`）
- 🛡️ **反爬措施**：30 个真实 UA 池 + 随机选取 + 完整请求头模拟
- 📦 **零依赖**：仅使用 Node.js 内置模块（https/fs/path），无需 npm install

## 🚀 快速开始

```bash
# 1. 克隆仓库
git clone https://github.com/yourname/worldcup2026.git
cd worldcup2026

# 2. 列出 48 支球队
node skills/haizei-worldcup-2026-skill/scripts/worldcup-teams.js list

# 3. 查看今日赛程
node skills/haizei-worldcup-2026-skill/scripts/worldcup-schedule.js today

# 4. 查看 FIFA 排名前 10
node skills/haizei-worldcup-2026-skill/scripts/worldcup-rankings.js fifa 10

# 5. 查德国队阵容
node skills/haizei-worldcup-2026-skill/scripts/worldcup-team.js lookup 德国
```

## 📁 项目结构

```
worldcup2026/
├── README.md
├── LICENSE
├── .gitignore
└── skills/
    └── haizei-worldcup-2026-skill/      # 本项目唯一的 Skill
        ├── SKILL.md                      # Skill 入口（AI 路由索引）
        ├── data/
        │   └── teams.json                # 48 队分组配置（静态）
        ├── scripts/                      # 6 个抓取脚本 + UA 池
        │   ├── lib/
        │   │   └── user-agents.js        # 31 个真实浏览器 UA
        │   ├── worldcup-teams.js         # 球队配置（静态）
        │   ├── worldcup-schedule.js      # 赛程
        │   ├── worldcup-match.js         # 比赛详情（6 个 tab）
        │   ├── worldcup-team.js          # 球队详情（6 个 tab）
        │   ├── worldcup-player.js        # 球员详情（5 个 tab）
        │   └── worldcup-rankings.js      # 排名（积分榜/球员榜/FIFA）
        └── references/                   # 9 份渐进式加载文档
            ├── overview.md               # 赛事基本信息
            ├── teams.md                  # 球队配置
            ├── schedule.md               # 赛程
            ├── match.md                  # 比赛详情
            ├── team.md                   # 球队详情
            ├── player.md                 # 球员详情
            ├── rankings.md               # 排名
            ├── usage.md                  # 跨脚本组合用法
            └── workflows.md              # ⭐ 12 个用户工作流（AI 必读）
```

## 📖 文档导航

| 文档 | 用途 |
|------|------|
| [SKILL.md](skills/haizei-worldcup-2026-skill/SKILL.md) | Skill 入口、路由表、快速命令 |
| [references/workflows.md](skills/haizei-worldcup-2026-skill/references/workflows.md) | ⭐ **12 个用户场景 → 脚本调用链** |
| [references/usage.md](skills/haizei-worldcup-2026-skill/references/usage.md) | 跨脚本组合、数据流图 |
| [references/overview.md](skills/haizei-worldcup-2026-skill/references/overview.md) | 48 队分组一览、4 档分档、主办城市 |
| 各 `references/<topic>.md` | 每个脚本的字段说明 |

## 🎯 6 大脚本一览

| 脚本 | 命令示例 | 输出 |
|------|---------|------|
| `worldcup-teams.js` | `list` `group A` `find 巴西` | 48 队配置（静态）|
| `worldcup-schedule.js` | `today` `date 2026-06-14` `group B` `team 巴西` | 赛程（多维筛选）|
| `worldcup-match.js` | `info/analysis/lineup/live/stats/odds <matchId>` | 单场比赛 6 tab 数据 |
| `worldcup-team.js` | `lookup/info/schedule/lineup/history/stats <teamId>` | 单球队 6 tab 数据 |
| `worldcup-player.js` | `info/news/stats/schedule <playerId>` | 单球员 5 tab 数据 |
| `worldcup-rankings.js` | `standings` `fifa [N]` `players <分类>` | 排名数据 |

## 🤖 作为 AI Skill 使用

本项目按 OpenCode/Claude Code Skill 规范组织：

1. **入口**：[`SKILL.md`](skills/haizei-worldcup-2026-skill/SKILL.md) 包含 frontmatter + 路由表
2. **渐进式加载**：`references/*.md` 按需读取，避免上下文爆炸
3. **跨脚本工作流**：[`workflows.md`](skills/haizei-worldcup-2026-skill/references/workflows.md) 指导 AI 回答用户问题
4. **JSON 输出**：所有脚本返回结构化数据，AI 可直接解析

将 `skills/haizei-worldcup-2026-skill/` 复制到 AI 的 skills 目录即可使用。

> ⚠️ 本项目仅供学习研究，请勿高频抓取或商业用途。

## 🔧 开发与扩展

### 添加新抓取功能

1. 在 `scripts/` 添加新 JS 文件，复用 `lib/user-agents.js`
2. 在 `SKILL.md` 路由表添加一行
3. 在 `references/` 添加对应字段文档
4. 在 `workflows.md` 补充典型场景

### 数据流

```
teams.json (静态) ── teamId ──→ 球队详情
                          │
                          ├─ teamName ──→ 赛程 ── matchId ──→ 比赛详情
                          │
                          └─ playerId ──→ 球员详情
```

详见 [`references/usage.md`](skills/haizei-worldcup-2026-skill/references/usage.md)

## 📊 数据来源与限制

- **数据源**：百度体育 (https://tiyu.baidu.com)
- **更新频率**：实时（进行中比赛比分延迟 1-2 分钟）
- **赛程窗口**：约 4 天滚动，更早赛程需按日期循环查询
- **完整 104 场比赛**：通过按日期循环 `date <YYYY-MM-DD>` 命令获取
- **球员榜**：当前仅射手榜有数据，其它分类随赛事进行会丰富

## ⚠️ 免责声明

本项目仅供学习研究使用，数据版权归百度体育所有。请勿用于商业用途，遵守相关法律法规。

## 📄 License

[MIT](LICENSE)

## 🙏 致谢

- 数据来源：[百度体育](https://tiyu.baidu.com)
- Skill 设计参考：[skill-creator](https://github.com/anthropics/skills) 规范
- 灵感来源：2026 美加墨世界杯 🎉

## 👤 作者

- **爱海贼的无处不在**
- 微信公众号：**无处不在的技术**
