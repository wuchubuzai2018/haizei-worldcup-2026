---
name: haizei-worldcup-2026-skill
description: 获取2026年国际足联世界杯（美加墨世界杯）数据技能。从百度体育抓取48支参赛球队的分组配置、赛程、比赛详情（分析/阵容/赛况/统计/指数）、球队详情（资料/阵容/历史成绩/数据）、球员详情（资料/动态/数据/赛程）、排名数据（积分榜/球员榜/FIFA排名）等。当用户需要查询2026世界杯球队、赛程、比赛数据、阵容、历史成绩、排名时使用此技能。A skill for retrieving 2026 FIFA World Cup data. Scrapes from Baidu Sports.
---

# 2026年美加墨世界杯数据获取

从百度体育抓取 2026 美加墨世界杯数据。包含 48 支球队的分组配置、赛程、比赛详情、球队详情、球员详情、排名等。

## 路由表

| 用户需求 | 脚本 | 参考文档 |
|---------|------|---------|
| 赛事基本信息、48队分组 | `scripts/worldcup-teams.js info` | [overview.md](references/overview.md) |
| 球队配置（静态）| `scripts/worldcup-teams.js` | [teams.md](references/teams.md) |
| 全部赛程 / 今日赛程 / 按组按队查询 | `scripts/worldcup-schedule.js` | [schedule.md](references/schedule.md) |
| 比赛情报 / 阵容 / 赛况 / 统计 / 指数 | `scripts/worldcup-match.js` | [match.md](references/match.md) |
| 球队资料 / 阵容 / 历史成绩 / 数据 | `scripts/worldcup-team.js` | [team.md](references/team.md) |
| 球员资料 / 动态 / 数据 / 赛程 | `scripts/worldcup-player.js` | [player.md](references/player.md) |
| 排名 / 球队榜 / 球员榜 / FIFA排名 | `scripts/worldcup-rankings.js` | [rankings.md](references/rankings.md) |
| **典型用户问题处理** | — | [**workflows.md**](references/workflows.md) ⭐ |
| 跨脚本组合用法 | — | [usage.md](references/usage.md) |

> ⭐ **AI 必读**：[workflows.md](references/workflows.md) 包含 12 个真实工作流（用户问 X 时该怎么做）。

## 快速命令

### 球队（静态配置）
```bash
node scripts/worldcup-teams.js list              # 全部48队
node scripts/worldcup-teams.js group A           # A组
node scripts/worldcup-teams.js find 加拿大        # 按名查 teamId
```

### 赛程
```bash
node scripts/worldcup-schedule.js today          # 今日赛程
node scripts/worldcup-schedule.js date 2026-06-14
node scripts/worldcup-schedule.js group B        # B组赛程
node scripts/worldcup-schedule.js team 巴西      # 巴西的赛程
```

### 比赛详情（matchId 来自赛程）
```bash
node scripts/worldcup-match.js info <matchId>     # 基本信息
node scripts/worldcup-match.js analysis <matchId> # 情报/历史/预测
node scripts/worldcup-match.js lineup <matchId>   # 首发/替补
node scripts/worldcup-match.js live <matchId>     # 事件流/场地
node scripts/worldcup-match.js stats <matchId>    # 技术统计
node scripts/worldcup-match.js odds <matchId>     # 博彩赔率
```

### 球队详情（teamId 来自 teams.json）
```bash
node scripts/worldcup-team.js lookup 加拿大
node scripts/worldcup-team.js info <teamId>       # 资料+荣誉
node scripts/worldcup-team.js schedule <teamId>   # 该队赛程
node scripts/worldcup-team.js lineup <teamId>     # 教练+球员
node scripts/worldcup-team.js history <teamId>    # 历届世界杯
node scripts/worldcup-team.js stats <teamId> 2026 世界杯
```

### 球员详情（playerId 来自球队阵容）
```bash
node scripts/worldcup-player.js info <playerId>          # 资料+能力雷达+荣誉+转会
node scripts/worldcup-player.js news <playerId>          # 球员新闻
node scripts/worldcup-player.js stats <playerId> 2026 世界杯
node scripts/worldcup-player.js schedule <playerId>      # 出场记录
```

### 排名数据
```bash
node scripts/worldcup-rankings.js standings              # 12组积分榜
node scripts/worldcup-rankings.js fifa 20               # FIFA 排名前20
node scripts/worldcup-rankings.js players 进球          # 射手榜
node scripts/worldcup-rankings.js players 助攻 10       # 助攻榜前10
```

## 加载策略

1. **先看 [workflows.md](references/workflows.md)**：了解用户问题 → 脚本调用的对应关系
2. **简单查询**：直接根据上表运行命令
3. **字段说明**：参考对应的 `references/<topic>.md`
4. **数据流/组合**：参考 [usage.md](references/usage.md)

## 注意事项

- 数据从百度体育实时抓取，进行中比赛比分延迟约 1-2 分钟
- 赛程页默认展示约 4 天窗口，更早赛程需按日期循环查询
- 球队配置（teams.json）为静态，仅 2026 年赛事
- 球队/球员名仅支持中文（如"巴西"），不支持英文
- 比赛时间均为北京时间（UTC+8）

## 作者

- 爱海贼的无处不在
- 微信公众号：无处不在的技术
