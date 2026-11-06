# 敌人配置

每种敌人一个 YAML 文件，文件名 = 种类（`grunt` 碎屑、`brute` 重胎、`boss` 污核、`dummy` 训练场木桩）。
这里是敌人数值的**唯一来源**，写法和 `weapons/` 一样（只认一层 `字段: 值`，见那边的 README）。

改完同步进两个工程，再两边一起提交：

```bash
infinity-rooms-client/tools/sync_resources.sh
infinity-rooms-server/tools/sync_resources.sh
```

敌人数值算进道具内容哈希：两端不一致时客户端握手直接被拒。改了要升 `NetProtocol.VERSION`。
也可以用内容工作台（`tools/workbench`）的「敌人」页改，它会一起做同步。

## 字段

| 字段 | 含义 | 单位 |
|---|---|---|
| `kind` | 种类，必须和文件名一致 | |
| `name` | 名称（控制台、调试信息里显示） | |
| `hp` | 血量 | 点 |
| `speed` | 移动速度。玩家走路 80、奔跑 140 | 像素/秒 |
| `radius` | 碰撞半径，也是身体半宽。近战的受创圆半径 = 它 + 子弹半径 | 像素 |
| `height` | 身体画出来多高（从脚底往上）。子弹的受创框 = 宽 2 × radius、高 height | 像素 |
| `damage` | 接触伤害 | 点 |
| `touch_cd` | 同一只两次接触伤害的间隔，不然贴着就是每帧掉血 | 秒 |

没写的字段用代码里的默认值（`shared/enemy_types.gd` 的 `DEFAULTS`）。

⚠️ `radius` / `height` 也是**画面**：客户端按它们画怪的方块和木桩，子弹判定也用同一个框。
出了正式素材要照图重量。

⚠️ 只能改已有种类的数值。新增一种敌人要写代码（AI、刷怪表、画法），不是加个文件就行 ——
多出来的文件游戏会忽略并警告。
