# R 剧本音乐编号审计

结论：EEX 的音乐编号是逻辑编号，不能直接当成 CD 音轨号或 MP3 文件名前缀。普通编号进入 CD 播放器前加2；255是停止。当前 Godot 遗漏加2，且错误把0作为停止。此前只核对 mp3list 的 CD 音轨映射，漏掉了它上游的 EEX 转换。

## 本地原版证据

Ekd5.exe SHA-256：`199b1b67cf966ead46c49b430b9aab84bdc7d4f0e8dfa8e0ebeec5da280b01e0`。

- `0x0041581C`：音乐指令处理函数；以参数类型标签9读取原始音乐编号。
- `0x0041584D`：与255比较，等于255则进入停止分支。
- `0x0041585E`：机器码 `83 C2 02`，即 `add edx,2`。
- `0x00415867`：将加2后的编号传给 `0x004745FE`。
- `0x004745FE → 0x004745B1 → 0x00474458`：编号继续传递；`0x004744AF` 调用 IAT 地址 `0x00486400` 对应的 `CDAudioPlayTrack`。
- `0x00415882`：255分支调用停止方法 `0x00474619`，最终调用 `CDAudioStop`。
- `0x00474486..0x00474496`：如果同一轨仍在播放，则直接成功返回。当前 Godot 每条 play_track 都先 stop，另有重复编号使音乐从头开始的问题。

`Koeicda.dll` 将 CD 音轨号转发为消息0x1979；`mp3serv.dat` 用 `track−1` 索引 `mp3list.txt`。后两步已在 [程序与配置说明](../../文件与格式/程序与配置文件说明.md) 中记录。完整编号链为：

```text
EEX track_id=n → CD音轨=n+2 → mp3list零起算索引=n+1 → 对应MP3
EEX track_id=255 → 停止
```

不要删除 mp3list 的第一条空行；它对应CD数据轨1。但仅“跳过空行”并不能替代上游加2的转换。

## 原版 R 实际使用编号

统计全部59个已导出R文档，音乐指令452条，其中停止215条。

| 剧本音乐编号 | 应播放的文件／操作 | 指令出现次数 |
| --- | --- | --- |
| 0 | 02-AudioTrack 02.mp3 | 18 |
| 11 | 13-AudioTrack 13.mp3 | 22 |
| 12 | 14-AudioTrack 14.mp3 | 5 |
| 13 | 15-AudioTrack 15.mp3 | 16 |
| 15 | 17-AudioTrack 17.mp3 | 50 |
| 16 | 18-AudioTrack 18.mp3 | 91 |
| 17 | 19-AudioTrack 19.mp3 | 4 |
| 18 | 20-AudioTrack 20.mp3 | 20 |
| 19 | 21-AudioTrack 21.mp3 | 11 |
| 255 | 停止 | 215 |

例如 R00 在偏移1401处使用编号11，应播放第13轨；R01偏移90使用编号16，应播放第18轨。R09等剧本使用编号0，应播放第2轨，不能静音。

## 音频资源与当前代码

21个导入MP3与 `ccz/SoundTrk` 原文件逐一比对 SHA-256，全部一致。因此这里是运行时映射问题，未发现音乐文件在导入时被改写或交换。

当前 `tools/godot_template/scripts/story/r_story_screen.gd` 及实际项目同名脚本在 play_track 分支：

```gdscript
if id not in [0, 255]:
    var path := "res://assets/audio/music/%02d-audiotrack_%02d.mp3" % [id, id]
```

应仅把255作为停止，其余合法编号先换算CD音轨，再按清单解析资源。正常重复播放同一轨时应保留进度。对超范围编号需单独验证；原版底层 `0x004745B1` 还限制传入CD音轨 `<22`，不可仅凭目录里存在第22轨就推断所有编号都有效。上表实际R编号均在该限制内。

后续修复已完成：Godot播放代码已加入编号加2、0正常播放、255停止、同曲保留进度及范围检查，并通过音乐专项和R剧情回归测试。上文代码描述为修复前审计记录。无需重新解包音频。

复核工具：`./venv/bin/python tools/audit_r_music.py`。完整文件哈希、各编号的剧本示例及统计见 [audit.json](audit.json)，核心原版代码见 [engine_evidence.txt](engine_evidence.txt)。
