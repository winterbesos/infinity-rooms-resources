# 僵尸配置

僵尸是一族敌人：**一个 YAML 文件就是一种僵尸**，文件名 = 僵尸 id（小写字母开头，字母数字下划线）。
加一种僵尸不用改代码，加个文件、同步、升协议版本就行。推荐用内容工作台（`tools/workbench`）的「僵尸」页改：
判定框和动画都有预览。

```
zombies/
├── walker.yaml        普通僵尸的数值、判定框、动画参数
└── walker/            它的图集（一张精灵表 png）
    └── walk.png
```

改完同步进两个工程（yaml 进两端 `shared/zombies/`，图只进客户端 `assets/zombies/<id>/`），再一起提交：

```bash
infinity-rooms-client/tools/sync_resources.sh
infinity-rooms-server/tools/sync_resources.sh
```

数值和判定框算进内容哈希，两端不一致握手就拒；改了要升 `NetProtocol.VERSION`。
动画（`anim_*` 和 png）是纯表现，不进哈希，服务端也不读。

写法和 `weapons/` 一样，只认一层 `字段: 值`。没写的字段用默认值（`shared/enemy_types.gd` 的 `ZOMBIE_DEFAULTS`）。

## 怎么打

- **子弹**按头 / 身体 / 腿三块判定框分别判：子弹线**最先碰到**哪块就打哪块，伤害乘上那块的倍率（`head_mult` 等）。
  判定框就是画面：没出美术时游戏里直接用这三块框画占位小人；出了美术要照图量框。
- **近战**一律算身体（近战在地面上判定，没有高度）。
- **打腿**：腿上累计挨到 `leg_threshold` 点伤害（算完倍率）就**瘸腿**，瘸了不会好：
  移速换成 `crippled_speed`，判定框换成 `crippled_*` 那一套，动画换成 `anim_crippled_*`。
  一下打死的不算瘸（直接死了）。

## 字段

| 字段 | 含义 |
|---|---|
| `id` / `name` | id（和文件名一致）/ 名称 |
| `hp` `speed` `radius` `damage` `touch_cd` | 血量、移速（像素/秒）、地面碰撞半径、接触伤害、接触间隔（秒） |
| `spawn_rooms` | 出现在哪些房型，逗号分隔：`normal,elite,boss,final_boss`（安全房不刷怪） |
| `spawn_min_day` | 从第几天开始出现 |
| `spawn_weight` | 刷怪权重。每间房先买 BOSS 和重胎，剩下的预算里碎屑（权重 10）和各种僵尸按权重抢；0 = 不刷 |
| `spawn_cost` | 占多少预算（碎屑 1、重胎 2.5、污核 6） |
| `head_mult` `body_mult` `legs_mult` | 打中各部位的伤害倍率 |
| `head_x/_y/_w/_h` … `legs_*` | 站着时三块判定框：x = 框中心往前偏多少，y = 框底边离地多高，w / h = 宽高（像素，按朝右写，朝左自动镜像） |
| `leg_threshold` | 腿上累计多少伤害就瘸，0 = 打不瘸 |
| `crippled_speed` | 瘸了以后的移速 |
| `crippled_head_*` … `crippled_legs_*` | 瘸了以后的三块判定框 |
| `anim_faces_left` | 素材里的僵尸是朝左画的就填 true |
| `anim_file` `anim_frame_w` `anim_frame_h` `anim_cols` `anim_rows` | 图集（一张精灵表，放在 `zombies/<id>/` 下）和切格方式。帧号从 0 起按行从左往右数 |
| `anim_foot_x` `anim_foot_y` | 帧里脚底那个像素（对准游戏里的脚底） |
| `anim_idle_*` `anim_walk_*` `anim_crippled_idle_*` `anim_crippled_*` | 四个动作：站着、走、瘸了趴着、瘸了爬。`_frames` 帧序列（`0-3`、`4,5,6,5`、`11-8` 倒放），`_fps` 每秒帧数。待机空着就停在行走的第一帧 |

⚠️ 子弹在离地 35 像素（`BULLET_HEIGHT`）的高度飞，矮的目标要往镜头方向站一点才打得到 ——
爬着的僵尸只有几像素高，基本要斜着往下打。调框的时候用游戏里的 `/hitbox on` 看子弹线落在哪块上。
