# 颍川之战前 R 剧情

主菜单“开始游戏”进入通用场景 `scenes/story/r_story.tscn`（编号0），读取 `data/events/r_00.json`。
这是从原版 `ccz/R_00.eex` 导出的 AST，不是重新编写的对白。R_00 共一场景、一事件段；R_01 已经是颍川之战后的内容，不应在此接着播放。

## 本次实现

- 原版 R 内场景 30 → `Mmap[70]`；通行数据保留 `Pmap[30]`。
- 曹操、许子将、士兵01、士兵02的出场、移动、转向、动作及对应头像。
- `&人物名` 分隔的原始对白；每页最多三行，文字逐字显示，点击或空格/回车显示全句，再次操作进入下一页。
- 原版两项选择“正是吾心所愿／似乎言之过早”；只执行选中的 case 和匹配的后续条件块。
- 局部变量300分别设为 true / false，全局变量类型2由50变为60 / 40；队伍和装备命令参数保留到剧情状态中。
- 脚本指定的环境音与音乐、开始绘制、淡出和人物清理。
- 结尾停在“前往颍川／战前剧情结束”过渡页，可重看或返回主菜单；返回菜单不重复播放开场视频。

## 文件与接入

| 文件 | 用途 |
| --- | --- |
| `scripts/story/r_story_vm.gd` | 条件块、变量、分支及命令执行；未知命令明确中断 |
| `scripts/story/r_story_screen.gd` | 背景、对白、选项、音频、输入与过渡 |
| `scripts/story/r_person.gd` | 原版 R 形象与坐标投影 |
| `tests/r_prologue_expectations.json` | 从原始 EEX 独立计算的两条分支对白和命令偏移基准 |
| `scripts/verify_r_prologue.gd` | 对照两条分支的自动化验证 |

人物绘制沿用解析器仓库 `r_scene_resources.py`：

```text
anchor_x = (a-b)*8+312
anchor_y = (a+b)*4-216
sprite_top_left = (anchor_x, anchor_y-41)
depth = anchor_y+23
part = pmapobj_id*2 + (direction in [0,3])
mirror_x = direction in [1,3]
frame = 0 if action_id == 0 else action_id + 2  # 有效动作1..17；行走按[0,1,0,2]
portrait_slot = 0 if logical_face_id == 0 else logical_face_id+7
```

方向0使用背面（北），方向2使用正面（南）；方向1/3分别镜像正面/背面。原先的0/2正背映射已按实际画面反馈纠正，并同步修正解析器仓库的预览函数。Data.e5 的 face_id 是逻辑编号，不能直接作为 PNG 槽位：许子将206→213，士兵01为173→180，士兵02为177→184；曹操默认仍为0。

播放结果可从 `story_ended` 信号、页面的 `final_state`，或 SceneTree 元数据 `ccz_prologue_state` 取得，包括局部/全局变量、队伍与装备原始参数、选项和 `next_battle_id=0`。暂未写入存档，也未实现颍川战斗，故“继续游戏”仍维持原状态。

## 验证与精度边界

两条分支分别执行89/88条命令（含选择），显示27/28页对白；自动测试逐项比对原始 EEX 的命令偏移、执行顺序、完整对白和变量结果。还验证了四名人物初始落点、原版投影与镜像、真实 Tween 移动终点和菜单入口。

播放器已扩展为按编号运行全部 R 文档的共用实现，详见 [通用 R 播放器](r_story.md)。连续人物移动现在并行启动并等待全部完成，同一人物的路点仍按顺序执行。步速默认112像素/秒，睡眠参数每单位0.01秒，可在 Inspector 调节；尚不宣称旧引擎逐帧复刻。
