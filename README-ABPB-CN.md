# ABPB 4.3.3（AiBo 分支）Windows x86 便携版

ABPB 是基于 **AiBo v4.3.3**（YaPB 分支，上游提交 `760180e`）重新命名的独立机器人插件。
它保留 AiBo 的 Human Copycat、Human Aim、本地学习、导航与对话功能，但使用完全独立的
`ABPB` 命名空间，可与 YaPB、SYPB 等其他机器人系统同时安装。

- 插件名：`ABPB`
- DLL：`addons/ABPB/bin/ABPB.dll`
- 配置：`addons/ABPB/conf/ABPB.cfg`
- 主命令前缀：`abpb`，短别名：`abp`
- 全部 cvar：`abpb_*`
- AMXX 模块库名：`abpb`

## 一、压缩包内容

```text
addons/
  ABPB/
    bin/ABPB.dll                 主插件（32 位 Metamod 插件）
    conf/ABPB.cfg                主配置（181 个 abpb_* cvar）
    conf/lang/*.cfg              界面语言（含简体中文 chs、繁体中文 cht）
    data/graph/de_dust2.graph    Dust2 自带导航图
    data/copycat/                Human Copycat 学习数据目录
    licenses/                    导航图授权与来源
  amxmodx/
    modules/abpb_amxx.dll        AMXX 模块（可选）
    scripting/include/abpb.inc   AMXX 原生函数声明
    scripting/abpb_sample.sma    AMXX 示例插件
README-CN.md                    本说明
INSTALL-CN.txt                  快速安装步骤
NOTICE-ABPB.md                  来源与授权说明
LICENSE                         GPL-2.0
```

## 二、安装

1. **关闭 CS 1.6 客户端 / HLDS 服务端。**
2. 把压缩包里的 `addons` 文件夹合并复制到游戏的 `cstrike` 目录。
   最终应存在：
   ```text
   cstrike/addons/ABPB/bin/ABPB.dll
   cstrike/addons/ABPB/conf/ABPB.cfg
   ```
3. 确认服务器已经安装并启用了 **Metamod**（`liblist.gam` 指向 `metamod.dll`）。
4. 编辑 `cstrike/addons/metamod/plugins.ini`，加入一行：
   ```text
   win32 addons/ABPB/bin/ABPB.dll
   ```
5. 启动服务器，控制台执行：
   ```text
   meta list
   ```
   应能看到 `ABPB`。再执行：
   ```text
   abpb version
   abpb add
   ```
   即可添加一个机器人。

> 本插件是 **Metamod 插件**，不要写进 `liblist.gam` 的 `gamedll`，也不要改名为
> `mp.dll`。它只支持 32 位（x86）GoldSrc。

## 三、与 YaPB / SYPB 等 BOT 共存

ABPB 的目录、DLL 名、Metamod 插件名、命令和 cvar 都与其他 BOT 分离，因此可以**安装**在同一台服务器上。但同一时间只应让**一个** BOT 系统管理机器人：

- 两个 BOT 管理器会 hook 同一批 GoldSrc 引擎函数、各自创建 fake client，并争抢同一批玩家槽位/edict。
- 如果两个系统都自动补位，32 槽服务器很容易被挤爆，常见结果是 `no free edicts` 或服务端直接退出（闪退）。
- ABPB 默认带有保护：换图时会检测 `yb_*`、`sypb_*`、`fapb_*`、`ebot_*`、`podbot_*`、`pb_*`、`zbot_*`、`zb_*`、`realbot_*` 等 cvar；发现其他 BOT 时自动把 `abpb_quota` 和 `abpb_autovacate` 设为 0，并在日志/控制台提示。
- 只有在你明确要自己协调多个配额时，才设置 `abpb_compat_auto 0`。

| 项目 | ABPB | YaPB | SYPB |
| --- | --- | --- | --- |
| 目录 | `addons/ABPB` | `addons/yapb` | `addons/sypb` |
| 主命令 | `abpb` / `abp` | `yb` / `yapb` | `sypb` |
| cvar 前缀 | `abpb_` | `yb_` | `sypb_` |
| 机器人配额 | `abpb_quota` | `yb_quota` | `sypb_quota` |

混用测试时：只保留一个 BOT 管理器；或把其他系统的 quota 设为 0，只在一个系统里手动加机器人。`abpb removebots` 只移除 ABPB 自己的机器人。

## 四、常用命令

所有命令都在服务器控制台执行，主前缀为 `abpb`，也可以使用短别名 `abp`。

### 机器人管理

| 命令 | 作用 |
| --- | --- |
| `abpb version` | 显示 ABPB 版本 |
| `abpb add [难度] [性格] [队伍] [模型] [名字]` | 添加一个机器人 |
| `abpb fill [队伍] [数量] [难度] [性格]` | 自动补足机器人 |
| `abpb removebots [instant] [队伍]` | 移除 ABPB 机器人 |
| `abpb list` | 列出当前 ABPB 机器人 |
| `abpb menu` | 打开机器人主菜单 |
| `abpb weapons [模式]` | 设置武器模式 |
| `abpb cvars` | 查看全部 cvar |
| `abpb graphmenu` | 打开导航图编辑器 |

### Human Copycat / 学习状态

| 命令 | 作用 |
| --- | --- |
| `abpb_human_status` | Human Copycat 运行状态 |
| `abpb_human_aim_status` | 瞄准教练状态 |
| `abpb_learning_status` | 本地学习状态 |
| `abpb_explore_status` | 地图探索状态 |
| `abpb_chat_status` | 程序化对话状态 |
| `abpb_final_status` | 4.3.3 综合状态 |
| `abpb_human_save` | 立即保存 Human Copycat 数据 |
| `abpb_human_reload` | 重新载入 Human Copycat 数据 |

## 五、常用 cvar

配置文件：`cstrike/addons/ABPB/conf/ABPB.cfg`
修改后可在控制台执行 `exec addons/ABPB/conf/ABPB.cfg`，或重启服务器。

| cvar | 默认 | 说明 |
| --- | ---: | --- |
| `abpb_quota` | `31` | 机器人补位数量（32 槽服务器留 1 个真人） |
| `abpb_compat_auto` | `1` | 检测其他 BOT，自动把 ABPB 配额/自动让位设为 0 |
| `abpb_quota_mode` | `normal` | `normal` / `fill` / `match` |
| `abpb_language` | `en` | 界面语言；简体中文改为 `chs`，繁体为 `cht` |
| `abpb_human_copycat` | `1` | 启用 Human Copycat |
| `abpb_human_aim_level` | `100` | 全局 Pro 瞄准上限；`-1` 为自适应 |
| `abpb_human_aim_skill` | `100` | 基础瞄准技能 |
| `abpb_human_reaction_skill` | `100` | 基础反应技能 |
| `abpb_human_pro_profiles` | `1` | 启用 42 个选手风格档案 |
| `abpb_human_hltv_skill` | `1` | 按 HLTV 排名权重调整能力 |
| `abpb_human_ct_patrol` | `1` | Dust2 CT 区域巡逻 |
| `abpb_learning` | `1` | 本地学习总开关 |
| `abpb_chat` | `1` | 机器人聊天 |
| `abpb_human_generative_chat` | `1` | 本地程序化生成对话 |
| `abpb_human_chat_language` | `profile` | `profile` 或强制 `en`/`zh`/`ru` 等 |
| `abpb_explore` | `1` | 几何探索 |
| `abpb_graph_analyze_auto_start` | `1` | 缺少导航图时自动分析 |

完整说明见 `ABPB.cfg` 中每个 cvar 上方的注释。

### 中文界面与中文聊天

```text
abpb_language "chs"              // 菜单/提示改为简体中文
abpb_human_chat_language "zh"    // 机器人聊天强制中文；profile 则按选手档案
```

## 六、AMXX 模块（可选）

只有需要在自己的 AMXX 插件里调用 ABPB 原生函数时才需要安装。

1. 复制 `addons/amxmodx/modules/abpb_amxx.dll`。
2. 在 `addons/amxmodx/configs/modules.ini` 增加：
   ```text
   abpb
   ```
   或在插件里使用 `#pragma loadlib abpb` 自动加载。
3. 把 `addons/amxmodx/scripting/include/abpb.inc` 放到
   `addons/amxmodx/scripting/include/`。
4. 参考 `abpb_sample.sma`，重新编译你的插件。

原生函数全部以 `abpb_` 开头，例如：

```pawn
native abpb_get_api_version();
native bool:abpb_game_has_bots();
native abpb_get_bot_count();
native bool:abpb_add_bot(Difficulty:difficulty = Difficulty:Normal,
                         Personality:personality = Personality:Normal,
                         CsTeams:team = CS_TEAM_UNASSIGNED);
native bool:abpb_is_user_bot(index);
```

模块库名为 `abpb`，与 YaPB 的 `yapb`、SYPB 的 `sypb` 不冲突。

## 七、故障排查

| 现象 | 处理 |
| --- | --- |
| `meta list` 没有 ABPB | 检查 `plugins.ini` 是否为 `win32 addons/ABPB/bin/ABPB.dll`；确认 Metamod 已启用、DLL 是 32 位 |
| `unknown command: abpb` | 插件未加载，先看 `meta list` 和服务器启动日志 |
| DLL 加载失败，提示缺少 `api-ms-win-crt-*` | 安装 Windows 更新/UCRT（Win7 需 KB2999226）或 VC++ 2015-2022 x86 运行库 |
| 没有导航图 | 把 `.graph` 放到 `addons/ABPB/data/graph/`，或让 `abpb_graph_analyze_auto_start 1` 自动分析 |
| 机器人不移动/卡住 | 检查地图是否有对应 `.graph`；删除旧图后重新分析 |
| 与 YaPB/SYPB 抢名额 | 降低各自的 quota，或关闭其中一个系统的自动补位 |
| 中文显示为英文 | 设置 `abpb_language "chs"` 并重载配置 |
| 想恢复默认 | 用压缩包内的 `conf` 覆盖，或在控制台执行 `abpb cvars defaults` |

日志与学习数据默认位于 `addons/ABPB/data/`（`logs/`、`copycat/`、`graph/` 等）。

## 八、构建与验证信息

- 上游：`keinzwei/AiBo`，tag `v4.3.3`，提交 `760180e`
- 本包：ABPB 4.3.3，Windows x86（i386）
- 编译器：Zig 0.17.1（Clang 22.1.8，`x86-windows-gnu`）
- 构建系统：CMake 4.4.4 + Ninja 1.13.2
- 链接：libc++ / libc++abi 静态链接，DLL 无未解析 C++ 符号
- 主机测试：292/293 通过；唯一未通过项为 `aim_baseline_compatibility`
  （Windows 数学库/浮点基线与 Linux 录制值存在微小差异，不影响功能）

## 九、授权与来源

ABPB 是 AiBo/YaPB 的修改分支，遵循 **GPL-2.0-or-later**。原项目、YaPB、PODBot、
RealBot/RBCC 及导航图数据各自保留其授权与署名。详见 `NOTICE-ABPB.md`、
`LICENSE` 和 `addons/ABPB/licenses/`。

Counter-Strike 与 Half-Life 归其各自权利人所有；本包不包含游戏本体。
