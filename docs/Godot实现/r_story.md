# 通用 R 剧情播放器

所有编号共用 `scenes/story/r_story.tscn`、`r_story_screen.gd` 和 `r_story_vm.gd`。
编号范围 0..58，按编号读取 `data/events/r_%02d.json`，不再在播放器里写死 R00。
主菜单默认仍从 R00 开始；标题场景根节点 Inspector 的 `New Game Script Id` 可改为其他编号。
单独打开 `r_story.tscn`，设置根节点 `Script Id` 后按 F6，也可预览其他剧情。
`r_prologue.tscn` 保留为兼容入口，使用同一脚本、默认编号 0。

## 运行时调试入口

运行后点击右上角“剧情调试 · F8”，或按 F8，选择 R00–R58 并点击“播放所选剧情”。主菜单和剧情播放中均可使用。打开选择器时暂停当前剧情，取消或 Esc 恢复；播放会终止旧剧情，从所选编号开头以默认状态重新运行。选择器自动定位当前剧情。

## 代码调用

```gdscript
var player = preload("res://scenes/story/r_story.tscn").instantiate()
player.autostart = false
add_child(player)
await player.play_script(1) # R_01，颍川之战后剧情
```

带历史状态运行：

```gdscript
await player.play_script(36, {
    "local_variables": {835: true},
    "global_variables": {2: 90}
})
var state = player.final_state
# 当前播放结束后可在同一实例继续加载另一个编号：
await player.play_script(37, state)
```

也可以在加入场景树前设置 `script_id` 和 `initial_state`，使用默认 autostart。
运行期间再次调用会被拒绝，避免两份剧情并发操作同一舞台。重看按钮保留当前编号和初始状态。
历史变量决定战果、人物生死和路线；单独传编号使用空历史状态，不自动伪造此前的战果。
JSON 存档的字符串数字键会归一化成整数。初始化命令只清理播放器的临时标志区 0..255，保留 300 段选择、700 段战果、800 段人物标志；该临时区边界是播放器约定，尚非旧引擎精确逆向结论。

## 移动和交互

- 同一事件列表连续的 `people_move` 合并为一批，先启动所有人物，再等待全部到位。
- 注释不截断移动批次；sleep、对白、转向等其他命令以及条件块边界会截断。
- 同一人物在批次内出现多次时，其路点串行执行；其他人物同时行走。
- 移动中朝向按实际位移自动计算，每个路点转弯时更新；到达后应用剧本 dire，65535 保留最后的移动朝向。北/南及许子将头像沿用已修正映射。
- 自由交谈场景等待玩家点击人物或底部人物按钮；“出阵”触发 battle_test。交谈结束返回等待。

## 支持范围与交接

解释器处理全部原版 R 文档实际出现的条件及事件命令：分支、else、变量、人物、室内/室外/中国地图、地图头像与叙事、音视频、章节和场所文字、队伍装备数据以及出阵交接。未知命令仍报告带偏移的错误。

`story_ended(result)` / `final_state` / SceneTree 的 `ccz_story_state` 返回编号、变量、队伍、装备、物品、形象变化和后续交接；R00 另外保留 `ccz_prologue_state` 兼容旧调用。
`end_scene` 返回 battle 交接；`script_goto` 返回原始 `script_index`；`end_set` 记录结局编号。不会把战后 R01 自动当成 R00 战斗接着播放。

这里是通用剧情播放器，尚未实现战斗引擎、战役跳转路由、存档和装备管理界面。队伍、出阵位图、获得物品等命令保留结构化状态供这些系统接入。
表现层仍有近似：地图文字排版、特殊头像状态位（大于7的原始值保留，暂用基础头像）、淡入淡出和音视频时序并非旧引擎逐帧复刻；现有唯一视频指令 video_id=1 映射 open.ogv，尚待原版运行对照。地图头像移动暂按单条命令等待，不属于人物移动批次。

## 验证

```sh
Godot --headless --path <project> --script res://scripts/verify_r_story.gd
Godot --headless --path <project> --script res://scripts/verify_r_prologue.gd
Godot --headless --path <project> --script res://scripts/verify_title.gd
```

通用测试对全部 59 个 R 文档执行两组历史/选择状态，共 118 次，使用真实表现层并跳过动画等待；不代表穷尽全部历史变量组合。另验 R01 人物交谈回到待命、真实多人并行 Tween、同人路点串行、等待屏障、嵌套 else、JSON 状态恢复。R00 独立原文基准继续比对 89/88 条命令、27/28 页对白和分支结果。

## 武将动作取帧修正

`r_person.gd` 已按原版 Ekd5.exe 的 `0x0048B920` 动作表取帧：动作0静止取帧0，行走按 `[0,1,0,2]` 循环；动作1..17分别取物理帧3..19，65535保留当前动作。原始透明槽照常保留。行走帧间隔仍为0.15秒，此节奏尚非旧引擎精确计时结论。

`verify_r_actions.gd` 对照独立从 EXE 提取的 `tests/r_action_expectations.json`，检查18种动作与四方向、行走序列、保留动作和移动结束。

中国地图 `Mmap[114]` 使用线性640×400像素，已修正此前错误的8×8重排导出；其余背景的图块布局不变。

## 音乐编号修正

按本地原版 Ekd5.exe `0x0041585E` 的转换，EEX音乐编号0..19加2后对应MP3音轨02..21；255停止，0播放第02轨。同曲仍在播放时保留进度，换曲或停止后重播从头开始。超范围编号报告失败并保留当前音乐。音乐专项验证为 `scripts/verify_r_music.gd`，基准来自独立EXE/播放清单审计。
