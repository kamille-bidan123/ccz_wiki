# Smlmap.e5 与 Gate.e5 格式分析

## 结论

这两个文件都是 `Ls12` 多分块容器，但分块内部布局不同：

- `Smlmap.e5`：58 张战场缩略图，与 `Hm00.e5`～`Hm57.e5`、`Hexzmap.e5`、`Spalet.e5` 同编号。
- `Gate.e5`：186 个动态战场图块，组成 93 对状态图，供 EEX `0x58`“战斗障碍设定”替换战场中心点周围的 3×3 格。

社区资料把 `Smlmap.e5` 称为“序号与 HM 对应的 S 小地图”，把
`Gate.e5` 称为“地图小块，一般用于实现 S 地图的城门开关变化”：

- https://xycq.org.cn/forum/blog.php?tid=204321

本项目对文件内容、58 个战场尺寸、全部 EEX 脚本和 `Ekd5.exe` 的读取代码进行了交叉验证。

## Smlmap.e5

### 分块数量和尺寸

`Smlmap.e5` 有 58 个分块。设同编号 `Hexzmap.e5` 给出的战场尺寸为：

```text
格子列数 = columns
格子行数 = rows
```

则：

```text
缩略图宽度 = columns * 6
缩略图高度 = rows * 6
分块字节数 = columns * rows * 36
```

58 个分块全部严格符合这一关系。因此每个 48×48 像素的 Hm 战场格，
在 Smlmap 中对应 6×6 像素，线性比例为 1/8。

### 像素布局

分块是整张图片的逐行布局：

```text
offset = y * (columns * 6) + x
color_index = block[offset]
```

不能把它解释为一串独立的 6×6 小格。后者会生成明显的横向条纹和碎片。

每张图使用同编号 `Spalet.e5` 调色板。调色板通道顺序沿用项目已经验证的：

```text
文件三元组 -> 显示 RGB = (byte[1], byte[2], byte[0])
```

社区教程也说明 S 地图使用 246 色、不同地图具有各自的 `Spalet`：

- https://xycq.org.cn/forum/blog.php?tid=204321

原版部分 Smlmap 像素会使用 246～255 号索引；这些是调色板末尾的保留/
系统颜色，解析器不能因它们超过常用 246 色范围就判定图片损坏。

### 与 Hm 的关系

Smlmap 是 Hm 的视觉缩略图，但并非把 `Hm??.e5` 每隔 8 像素机械抽样就能
逐像素复原。对比结果表明其内容经过独立缩放、量化或导入处理。因此：

- 尺寸必须以 `Hexzmap.e5` 为准；
- 不能把 Hm 固定采样结果当作 Smlmap 的无损重建；
- 编辑 Hm 时若要兼容原引擎，应同步生成或导入 Smlmap。

## Gate.e5

### 分块结构

`Gate.e5` 有 186 个分块，每块解压后固定为 20,736 字节：

```text
20,736 = 3 * 3 * 48 * 48
```

每块包含九个 48×48 的战场格，先按格子逐行排列，再在每个格内按像素逐行
排列：

```text
for cell_y in 0..2:
    for cell_x in 0..2:
        for pixel_y in 0..47:
            for pixel_x in 0..47:
                read one palette index
```

渲染结果为 144×144 像素。Gate 没有独立调色板，必须使用当前战场编号的
`Spalet.e5`。

把每块解释成“9 帧 48×48 动画”会得到互不连贯的碎片；按 3×3 战场格
拼接后，城墙、城门、船、河岸和火场边缘均连续。

### 0x58 指令与 Gate 状态对

EEX `0x58` 的真实参数为：

```text
barrier_id
is_display
tile_type
x
y
focus_camera
skip_presentation
```

`Ekd5.exe` 中的分块换算为：

```text
is_display = false: Gate 分块 = 2 * barrier_id - 8
is_display = true : Gate 分块 = 2 * barrier_id - 7
```

因此：

| 障碍编号 | false 状态 | true 状态 |
| ---: | ---: | ---: |
| 4 | 0 | 1 |
| 5 | 2 | 3 |
| ... | ... | ... |
| 96 | 184 | 185 |

这正好覆盖 186 个分块。障碍编号 0～3 由引擎特殊处理，不读取 Gate。

### 覆盖位置

可执行文件中的实现以指令 `(x, y)` 转换后的战场格为中心，用两层
`-1..1` 循环复制九个 0x900 字节的格子：

```text
(x-1,y-1) (x,y-1) (x+1,y-1)
(x-1,y  ) (x,y  ) (x+1,y  )
(x-1,y+1) (x,y+1) (x+1,y+1)
```

靠近地图边界时会逐格检查范围，超出部分不写入。此前直接拿 Gate 与 Hm
寻找完全相同的裁剪块会出现一格左右的候选偏差，是因为 Gate 保存的是
另一状态的替换图，而非保证与静态 Hm 完全相同的副本；引擎循环已经明确
给出了中心点语义。

`tile_type` 会同时修改中心格的地形编号。原版 698 条 `0x58` 中常见值为：

| 地形编号 | 名称 | 次数 |
| ---: | --- | ---: |
| 27 | 船 | 426 |
| 17 | 城门 | 130 |
| 16 | 城内 | 86 |
| 13 | 大河 | 16 |
| 26 | 火 | 15 |
| 15 | 城墙 | 12 |
| 0 | 平原 | 12 |
| 12 | 小河 | 1 |

所以 Gate 的准确用途是“动态战场 3×3 替换块”，城门只是最典型的用途。
社区关卡教程也把“战场物体添加、战场障碍设定”用于放火与城门，并特别
提醒它会改变地形：

- https://www.xycq.org.cn/forum/viewthread.php?authoruid=337168&tid=201552

两个尾部布尔字段也已由本地旧引擎调用链定名：

- `focus_camera`：添加和移除障碍时都有效；为真时调用
  `0x0045503F`，把战场镜头定位到指令的 `(x,y)`。
- `skip_presentation`：只在添加障碍的 `0x00456140` 路径读取；为真时
  跳过镜头定位、地形对应音效和等待，但仍立即更新地形编号与 3×3
  Gate 图块。移除障碍的 `0x004567C1` 路径不接收该参数。

全部原版脚本的分布与这一结论一致：`skip_presentation=true` 的 337 条
全部是 `is_display=false` 的批量静默布置；移除路径只出现
`focus_camera` 的真假差异。`S_28` 的八条 `FFFF` 布尔原始值会被旧引擎
归一化为假，解析器仍保留其 `*_raw` 字节以便严格回写。

## 已实现代码

- `battle_aux_resources.py`
  - 验证 Smlmap 尺寸；
  - 渲染整图逐行布局的 Smlmap；
  - 验证并渲染 3×3 Gate 图块；
  - 将 `barrier_id + is_display` 换算为 Gate 分块编号。
- `tools/export_smlmap_gate.py`
  - 批量校验；
  - 导出单张或全部 Smlmap；
  - 按 Gate 分块编号导出；
  - 按 EEX 障碍编号和状态导出。

示例：

```bash
./venv/bin/python tools/export_smlmap_gate.py --analyze
./venv/bin/python tools/export_smlmap_gate.py --small-map 8
./venv/bin/python tools/export_smlmap_gate.py \
  --barrier 7 --displayed --palette-map 8
```

另外，`eex_schema.py` 中旧的 0x58“区域坐标”定义已修正为与二进制解析器和
写回器一致的七参数结构。
