# 典型工作流

按用户问题分类，给出具体的脚本调用链。AI 助手可参考这些模式回答用户。

## 速查表

| 用户问什么 | 用什么脚本 | 关键步骤 |
|-----------|-----------|---------|
| 今天/明天有什么比赛 | `worldcup-schedule.js today/tomorrow` | 单次调用 |
| X队下一场几点打？ | `worldcup-team.js schedule` + 筛 next | 查 teamId → 拉赛程 → 找最近 |
| 正在直播的比赛 | `worldcup-schedule.js today` + 状态过滤 | 筛 `statusId === '1'` |
| X队首发阵容 | `worldcup-match.js lineup` | 查该队今天比赛 → 拉阵容 |
| X队球员列表 | `worldcup-team.js lineup` | 单次调用 |
| X队历史战绩 | `worldcup-team.js history` | 单次调用 |
| X球员能力如何 | `worldcup-player.js info` | 雷达图数据 |
| X队 FIFA 排名 | `worldcup-rankings.js fifa` | 遍历查 teamName |
| 小组出线情况 | `worldcup-rankings.js standings` | 筛 isQualified |
| 射手榜 | `worldcup-rankings.js players 进球` | 单次调用 |
| 某场比赛分析 | `worldcup-match.js analysis` | 查 matchId → 拉分析 |
| 直播/比赛时间 | `worldcup-schedule.js date` | 查具体日期 |

---

## 工作流 1：今日观赛指南

**用户问**：「今天有什么世界杯比赛？几点开赛？」

**执行**：
```bash
node scripts/worldcup-schedule.js today
```

**输出处理**（伪代码）：
```js
const matches = JSON.parse(stdout);
for (const m of matches) {
  const status = m.statusId === '2' ? '已结束' : 
                 m.statusId === '1' ? '进行中' : '未开赛';
  console.log(`${m.time} ${m.stage} ${m.homeTeam} vs ${m.awayTeam} (${status})`);
}
// 已结束比赛补一句比分
```

**额外增强**：
- 用户问"哪场最值得看" → 排序 by `hot` 字段
- 用户问"有直播吗" → 过滤 `hasLive === true`

---

## 工作流 2：正在直播

**用户问**：「现在有什么比赛在直播？」

**执行**：
```bash
node scripts/worldcup-schedule.js today | jq '.[] | select(.statusId=="1")'
```

或者日期+状态组合：
```bash
node scripts/worldcup-schedule.js today
# Python 过滤
```

**进一步**：对每场正在直播的比赛，可拉取实时赛况：
```bash
node scripts/worldcup-match.js live <matchId>
```

---

## 工作流 3：球队下一场比赛

**用户问**：「巴西下一场几点打？对手是谁？」

**执行链**：
```bash
# 1. 查 teamId
node scripts/worldcup-team.js lookup 巴西
# 得到 teamId: b92535372efd0c6bc7df7b84b9c7b577

# 2. 查赛程
node scripts/worldcup-team.js schedule b92535372efd0c6bc7df7b84b9c7b577 世界杯
```

**Python 处理**（找未来最近一场）：
```python
import json, subprocess
from datetime import datetime
data = json.loads(check_output(['node', 'worldcup-team.js', 'schedule', teamId, '世界杯']))
matches = data['competitions']['世界杯']['matches']
future = [m for m in matches if m['statusId'] == '0']  # 未开赛
if future:
    next_match = future[0]
    print(f"下一场: {next_match['date']} {next_match['time']}")
    print(f"对手: {next_match['homeTeam']} vs {next_match['awayTeam']}")
```

---

## 工作流 4：球队核心球员

**用户问**：「德国队谁最贵？有哪些明星球员？」

**执行**：
```bash
node scripts/worldcup-team.js lineup d885ec68c7b46dbfd4a8f6e41d577ba0
```

**Python 处理**（按身价排序）：
```python
data = json.loads(check_output(['node', 'worldcup-team.js', 'lineup', teamId]))
all_players = [p for g in data['players'] for p in g['players']]
# 转换身价字符串为数字
def parse_value(v):
    if not v: return 0
    return float(v.replace('万', '').replace('亿', '00'))  # 简单处理
top10 = sorted(all_players, key=lambda p: parse_value(p.get('value', '0')), reverse=True)[:10]
for p in top10:
    print(f"{p['name']} ({p['number']}号) - {p.get('club','')} - {p.get('value','')}")
```

---

## 工作流 5：球员详情深挖

**用户问**：「姆巴佩什么水平？效力哪个队？世界杯数据如何？」

**执行链**：
```bash
# 1. 找到球员（在某球队的阵容里）
node scripts/worldcup-team.js lineup <teamId>
# 从 players[].players[] 找 playerId

# 2. 拉详细资料（含能力雷达）
node scripts/worldcup-player.js info <playerId>

# 3. 拉赛事数据
node scripts/worldcup-player.js stats <playerId> 2026 世界杯
```

---

## 工作流 6：比赛预测参考

**用户问**：「X vs Y 谁更可能赢？」

**执行**：
```bash
node scripts/worldcup-match.js analysis <matchId>
```

**重点提取**：
- `prediction.homeWinRate` / `prediction.awayWinRate`（AI 胜率）
- `prediction.similarOddsHistory`（相似赔率下历史胜负比例）
- `intelligence`（双方近期状态对比）
- `records`（H2H 历史战绩）

**回答模板**：
> 根据百度分析，X 胜率 35%，Y 胜率 50%。相似赔率历史中主胜占 30%。X 近期状态低迷（5场1胜），Y 状态出色（4胜2平）。

---

## 工作流 7：FIFA 排名查询

**用户问**：「中国/日本/阿根廷 FIFA 排名第几？」

**执行**：
```bash
node scripts/worldcup-rankings.js fifa 100
```

**Python 过滤**：
```python
data = json.loads(check_output(['node', 'worldcup-rankings.js', 'fifa', '100']))
team_data = next((r for r in data['rankings'] if team_name in r['teamName']), None)
if team_data:
    change = int(team_data['positionChanged'])
    arrow = '↑' if change > 0 else ('↓' if change < 0 else '—')
    print(f"{team_data['teamName']}: 第{team_data['rank']}名 {team_data['points']}分 {arrow}{abs(change)}")
```

---

## 工作流 8：小组出线形势

**用户问**：「A组现在什么情况？哪些队晋级了？」

**执行**：
```bash
node scripts/worldcup-rankings.js standings
```

**Python 处理**：
```python
data = json.loads(check_output(['node', 'worldcup-rankings.js', 'standings']))
group_a = data['groups'][0]  # A组
for team in group_a['list']:
    status = '✓已晋级' if team['isQualified'] else ('✗已淘汰' if team['isRelegated'] else '·待定')
    print(f"  {status} {team['teamName']} {team['played']}场 {team['winDrawLoss']} 积分{team['points']}")
```

---

## 工作流 9：完整比赛报告（已结束比赛）

**用户问**：「昨晚墨西哥那场比赛谁赢了？有什么精彩瞬间？」

**执行链**：
```bash
# 1. 找到比赛
node scripts/worldcup-schedule.js date 2026-06-12
# 得到 matchId

# 2. 比赛基本信息
node scripts/worldcup-match.js info <matchId>

# 3. 赛况（事件流+文字播报）
node scripts/worldcup-match.js live <matchId>

# 4. 技术统计
node scripts/worldcup-match.js stats <matchId>
```

**回答模板**：
> 墨西哥 2-0 战胜南非。第8分钟博瓦迪利亚打入一球（实际上是乌龙？），控球率 60%，射正 4-2。技术统计显示墨西哥在进攻端全面压制。

---

## 工作流 10：跨多场次的对比

**用户问**：「X组和Y组哪组竞争更激烈？」

**执行**：
```bash
node scripts/worldcup-rankings.js standings
```

**Python 处理**：
```python
data = json.loads(check_output(['node', 'worldcup-rankings.js', 'standings']))
for group_idx in [0, 1]:  # A组, B组
    g = data['groups'][group_idx]
    group_name = chr(ord('A') + group_idx)
    total_goals = sum(int(t['goals'].split('/')[0]) + int(t['goals'].split('/')[1]) 
                      for t in g['list'] if t['goals'])
    print(f"{group_name}组: {total_goals} 总进球")
```

---

## 工作流 11：球队历史 vs 当前

**用户问**：「巴西队世界杯历史成绩怎么样？今年能走多远？」

**执行**：
```bash
# 历史成绩
node scripts/worldcup-team.js history b92535372efd0c6bc7df7b84b9c7b577

# 当前赛事数据
node scripts/worldcup-team.js stats b92535372efd0c6bc7df7b84b9c7b577 2026 世界杯

# 阵容
node scripts/worldcup-team.js lineup b92535372efd0c6bc7df7b84b9c7b577
```

---

## 工作流 12：新闻/动态关注

**用户问**：「C罗最近有什么新闻？」

**执行链**：
```bash
# 1. 找 playerId
node scripts/worldcup-team.js lookup 葡萄牙
# 然后 lineup 找 C罗
node scripts/worldcup-team.js lineup <portugalId>
# 从 players[].players[] 找 playerId

# 2. 拉新闻
node scripts/worldcup-player.js news <playerId>
```

---

## 错误处理模式

### 未找到球队
```
Error: 未找到球队: X
```
→ 提示用户："没找到球队X，请检查拼写。可用 `node worldcup-teams.js list` 查看所有48支球队。"

### 未找到比赛
```
Error: 未找到赛程 JSON 数据标记
```
→ 提示用户："赛程数据获取失败，可能是网络问题，请稍后重试。"

### 比赛未开赛
```
status: 未开赛
matchStatusText: 未开赛
```
→ 拉不到 lineup（阵容未确认）→ 提示用户："该比赛阵容尚未公布，开赛前2小时左右会更新。"

### 比赛进行中查历史数据
- 状态: 进行中
- lineup 已确认但 stats 暂无
→ 提示用户："比赛进行中，技术统计和赛况约1-2分钟延迟。"

---

## 中文/英文球队名

支持的查询方式：
- 中文：「德国」、「巴西」、「韩国」
- 部分匹配：「阿根」→ 阿根廷
- 不支持英文：「Germany」需先查 teams.json

如果用户用英文问，可提示用中文。

---

## 注意事项

- 数据延迟：实时数据 1-2 分钟延迟
- 静态数据：teams.json 不会自动更新（如有球队变化需手动更新）
- matchId 编码：URL 中需做 URL-encoding，脚本已自动处理
- 球员/球队ID：跨脚本通用，可直接串联
- 比赛时间：均以北京时间（UTC+8）为准
