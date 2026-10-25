# 武器配置

每把武器一个文件，文件名 = 道具 id（`pistol.json` 对应道具表里的 `pistol`）。
这里是武器**战斗数值的唯一来源**：道具表 `items.json` 只管名称、描述、格子大小、掉率、售价，
武器条目里就算还写着这些字段，游戏也会忽略、以这里为准。

## 改完怎么生效

游戏不会直接读这个目录。改完要同步进两个工程，再**两边一起提交**：

```bash
infinity-rooms-client/tools/sync_resources.sh
infinity-rooms-server/tools/sync_resources.sh
```

脚本把 `weapons/*.json` 复制到各自的 `shared/weapons/`。两端数据必须一模一样，
不一样时客户端连服务端会在握手阶段被拒（提示道具数据不一致）。

## 字段

| 字段 | 含义 | 单位 / 取值 |
|---|---|---|
| `id` | 道具 id，必须和文件名一致 | |
| `weapon_type` | `melee` 近战挥砍 / `ranged` 远程射击 | |
| `damage` | 每一击（每发子弹）的伤害 | 点 |
| `attack_speed` | 每秒攻击次数，冷却 = 1 / 它 | 次/秒 |
| `attack_range` | 近战：挥砍够得着多远；远程：子弹能飞多远 | 像素 |
| `swing_arc` | 近战扇形的张角（整个扇面，不是半角），0 = 一条直线 | 度，0~360 |
| `max_targets` | 近战一下最多同时打中几只 | 只 |
| `pierce_walls` | 近战能不能隔墙打到 | true / false |
| `hit_delay` | 近战从出手到结算伤害的延迟（对上动画的打击帧） | 秒 |
| `bullet_speed` | 远程：子弹飞行速度 | 像素/秒 |
| `magazine_size` | 远程：弹匣容量 | 发 |
| `reload_seconds` | 远程：换弹时间 | 秒 |
| `crit_rate` | 暴击率（**暂未生效**，代码里还没读） | 0~1 |
| `crit_damage` | 暴击倍率（**暂未生效**） | 倍 |
| `knockback` | 击退力度（**暂未生效**） | |

远程的三项不填时用默认值：子弹 450 像素/秒、弹匣 12 发、换弹 1.2 秒。
近战武器不用写这三项。

参考尺度：屏幕宽 640 像素，徒手射程 24、扇形 60°，人物走路 80、奔跑 140 像素/秒。
