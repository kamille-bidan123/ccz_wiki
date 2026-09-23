# `ccz` 目录文件用途与格式分析

## 1. 分析范围与结论级别

本文针对仓库中的 [`ccz`](https://github.com/lometsj/ccz_eex_parse/blob/master/ccz) 目录做分析，目标是回答两件事：

1. 这些文件分别是干什么用的。
2. 它们的“具体格式”能分析到什么程度。

本文的结论分为 3 级：

- `已验证`：可由当前仓库文件头、文件大小、现有解析代码直接验证。
- `高可信推断`：由 `cczmod` 社区资料与当前目录结构互相印证，但没有在本仓库里完整反解。
- `待进一步反编译/解包`：只能确认用途，暂时不能给出稳定的二进制字段定义。

### 1.1 引擎版本与证据优先级

仓库内 [`ccz/Ekd5.exe`](https://github.com/lometsj/ccz_eex_parse/blob/master/ccz/Ekd5.exe)
属于旧引擎，不是 Star 新引擎。本文按以下顺序采用证据：

1. 本地资源的字节分布、受控差分和可逆回写；
2. 本地 `Ekd5.exe`、DLL 的实际装载点、读写地址和调用链；
3. 与本地文件年代和结构吻合的早期轩辕春秋资料；
4. Star、带版本号的新引擎资料仅作候选名或研究线索。

除非本地旧引擎机器码或旧资源能再次验证，否则第 4 类资料不会直接写成
本项目的字段结论。尤其不能因为新引擎帖子给出了版本号，就假定同名字段、
记录长度、补丁地址或脚本机制在旧引擎中通用。

## 2. 目录总体观察

当前 `ccz` 目录不是“单个 `.ccz` 资源包”，而是一个已经展开后的游戏目录，包含：

- 主程序与动态库：`Ekd5.exe`、`*.dll`、`mp3serv.dat`
- 事件脚本：`R_*.eex`、`S_*.eex`
- 存档：`Sv??d.e5s`、`Sv??e.e5s`、`Sv??s.e5s`、`Svcmn.e5s`
- 图形资源：大量 `.e5`
- 音效与音乐：`.wav`、`SoundTrk/*.mp3`
- 视频：`.avi`
- 辅助文本/注册表配置：`README.txt`、`mp3list.txt`、`font.reg`、`font2.reg`、`AutoDLL.ini`

按扩展名统计：

| 扩展名 | 数量 | 说明 |
| --- | ---: | --- |
| `.e5` | 91 | 游戏专用资源文件，包含图片、地图、图标、数据表等 |
| `.e5s` | 31 | 存档文件 |
| `.eex` | 117 | 事件脚本 |
| `.wav` | 89 | 音效 |
| `.mp3` | 21 | 背景音乐 |
| `.avi` | 4 | 开场/过场/结局视频 |
| `.dll` | 6 | 图标/功能扩展/兼容库 |
| `.exe` | 1 | 主程序 |
| `.dat` | 1 | MP3 播放相关辅助程序 |

## 3. 文件族总表

### 3.1 脚本与存档

| 文件名/模式 | 用途 | 格式判断 | 结论级别 |
| --- | --- | --- | --- |
| `R_00.eex` ~ `R_58.eex` | R 剧本，通常对应剧情/内外场景段落 | 明确是 `EEX` 脚本格式 | `已验证` |
| `S_00.eex` ~ `S_57.eex` | S 剧本，通常对应战斗/关卡事件 | 明确是 `EEX` 脚本格式 | `已验证` |
| `Sv00d.e5s` ~ `Sv09d.e5s` | 每个存档槽的“人物/仓库/核心状态”数据 | 中文 21724 / 日文 21709 字节双布局 | `已验证` + `社区佐证` |
| `Sv00e.e5s` ~ `Sv09e.e5s` | 战场出场配置、胜败文本、事件变量 | 115×15 出场记录 + 4096×u16 变量 | `已验证` |
| `Sv00s.e5s` ~ `Sv09s.e5s` | 每个存档槽的战场单位运行状态与单位占位网格 | 头部 + 115×26 字节单位记录 + 40×40 占位表 | `已验证` + `旧引擎调用点` |
| `Svcmn.e5s` | 三条结局标志、104 项物品图鉴及 Logo 花色选择 | 固定 108 字节 | `已验证` + `旧引擎调用点` |

### 3.2 数据与文本

| 文件名 | 用途 | 格式判断 | 结论级别 |
| --- | --- | --- | --- |
| `Data.e5` | 人物、物品、商店、兵种、地形、策略等主数据表 | Ls12；10 个分块均可稳定拆包和字段级导出 | `已验证` |
| `Imsg.e5` | 说明、战役名、列传、台词和制作人员 | 921×200 字节 GBK/NUL 定长记录 | `已验证` |

### 3.3 地图、人物、特效、UI 图像

| 文件名/模式 | 用途 | 格式判断 | 结论级别 |
| --- | --- | --- | --- |
| `Face.e5` | 头像 | `.e5` 图像资源 | `已验证` + `社区佐证` |
| `Pmapobj.e5` | R 形象，即 R 剧本中人物立绘/行走形象 | Ls12；成对图块，可稳定浏览、定位和回写 | `已验证` |
| `Unit_atk.e5` | S 形象攻击动作 | Ls12 动作帧，可稳定浏览和回写 | `已验证` |
| `Unit_mov.e5` | S 形象移动动作 | Ls12 动作帧，可稳定浏览和回写 | `已验证` |
| `Unit_spc.e5` | S 形象特殊/待机/防御动作 | Ls12 动作帧，可稳定浏览和回写 | `已验证` |
| `Mmap.e5` | R 场景背景图；外场景、内场景、中国地图等 | Ls12；115 张 640×400 索引图 | `已验证` |
| `Pmap.e5` | 74 个 R 内场景的 100×100 二值通行/障碍网格 | Ls12；每块 16 字节头 + 10000 个 u16 | `已验证` + `社区佐证` |
| `Hm00.e5` ~ `Hm57.e5` | S 战场地图 | 专用地图资源；长宽优先取同编号 Hexzmap | `已验证` |
| `Hexzmap.e5` | S 地形长宽与逐格地形编号 | Ls12 容器；30种地形编号已确认 | `已验证` |
| `Smlmap.e5` | S 小地图；每格缩为 6×6 像素 | Ls12；58 张整图逐行索引图 | `已验证` |
| `Gate.e5` | 0x58 指令使用的动态战场 3×3 替换块 | Ls12；186 个 3×3×48×48 分块 | `已验证` |
| `Logo.e5` | 开场、战前标题、结局、Game Over、单挑背景等画面 | Ls12 索引图；13 块尺寸表已验证 | `已验证` |
| `Logo_p.e5` | `Logo.e5` 调色板 | Ls12；13 个 256 色调色板 | `已验证` |
| `Pmpalet.e5` | `Mmap.e5` 调色板 | Ls12；与 115 个 Mmap 槽对应 | `已验证` |
| `Spalet.e5` | S 地图调色板 | Ls12；与 58 个战场对应 | `已验证` |
| `Item.e5` | 物品 32×32 图片，与 Data icon 字段对应 | Ls12；104 个 32×32 索引图 | `已验证` |
| `Meff.e5` | 小型策略/普通特效 | 动画头、步骤表、像素组及尾控制动作均由旧引擎验证 | `已验证` |
| `Mcall00.e5` ~ `Mcall08.e5` | 大型策略动画 | 同 Meff 动画结构；支持双像素组及末尾内嵌调色板 | `已验证` |
| `Mark.e5` | 血条、火、船等小图标 | 67 个子图的偏移/宽高表已验证 | `已验证` |
| `Weather.e5` | 天气图 | Ls12；5 张 216×200 索引图 | `已验证` |
| `Hitarea.e5` | 攻击范围高亮 | Ls12；范围图尺寸已验证 | `已验证` |
| `Effarea.e5` | 穿透/作用范围高亮 | Ls12；范围图尺寸已验证 | `已验证` |
| `U_select.e5` | 出战选择、练武场、对话背景等界面资源 | Ls12；5 个 UI 分块尺寸已验证 | `已验证` |
| `Font.e5` | 旧目录遗留的 50 个 16×16 单色字形；本地旧引擎不装载 | 50×32 字节 + 2 字节重复尾行；文件内无字码表 | `已验证` |

### 3.4 程序、库与配置

| 文件名 | 用途 | 格式判断 | 结论级别 |
| --- | --- | --- | --- |
| `Ekd5.exe` | 游戏主程序/引擎 | 标准 PE32 EXE | `已验证` |
| `AutoDLL.dll` | 自动存档/快速点击等补丁功能 | 标准 PE32 DLL | `已验证` |
| `AutoDLL.ini` | `AutoDLL.dll` 配置 | INI 文本 | `已验证` |
| `Cmdicon.dll` | 攻击/策略/道具/待命菜单图标 | PE32；4 个 16×16 8-bit RT_BITMAP | `已验证` |
| `Itemicon.dll` | 物品 16×16/32×32 UI 图标 | PE32；210 个 8-bit RT_BITMAP | `已验证` |
| `Mgcicon.dll` | 31 组策略图标 | PE32；62 个 16×16/32×32 8-bit RT_BITMAP | `已验证` |
| `Mapatr.dll` | 地形 0～27 的情报图 | PE32；28 个 48×48 8-bit RT_BITMAP | `已验证` |
| `Koeicda.dll` | 把旧引擎 `CDAudio*` API 转接到 `mp3serv.dat` | PE32；20 个导出，4 个窗口消息转接实现 | `已验证` |
| `mp3serv.dat` | 接收 `Koeicda.dll` 消息、按 `mp3list.txt` 播放 MP3 | UPX 压缩 PE32；消息接收端已验证 | `已验证` |
| `mp3list.txt` | MP3 曲目列表 | 文本 | `已验证` |
| `font.reg` / `font2.reg` | 字体注册表配置 | Windows `.reg` | `已验证` |
| `README.txt` | 原始说明文档 | 文本，编码非 UTF-8 | `已验证` |

### 3.5 音视频

| 文件名/模式 | 用途 | 格式判断 | 结论级别 |
| --- | --- | --- | --- |
| `Se*.wav`、`Se_e_*.wav`、`Se_m_*.wav` | 游戏音效 | 标准 PCM WAV | `已验证` |
| `SoundTrk/*.mp3` | 背景音乐 | 标准 MP3 | `已验证` |
| `Logo.avi` / `Press.avi` / `open.avi` / `end.avi` | 开场/按键提示/结尾视频 | 标准 AVI | `已验证` |

## 4. 各类格式的具体分析

## 4.1 `.eex`：事件脚本文件

这是当前仓库里最明确的一类格式。

### 4.1.1 文件头

样本 [`ccz/R_00.eex`](https://github.com/lometsj/ccz_eex_parse/blob/master/ccz/R_00.eex) 的前 16 字节为：

```text
45 45 58 00 01 02 00 00 00 00 0E 00 00 00 01 00
```

可以稳定确认：

- `0x00-0x03`：魔数 `EEX\0`
- `0x04-0x05`：版本/格式号，当前样本为 `0x0201`
- `0x06-0x09`：保留字段
- `0x0A` 之后：场景偏移表

### 4.1.2 结构层级

根据 [`parse_eex.py`](https://github.com/lometsj/ccz_eex_parse/blob/master/parse_eex.py) 和已有文档：

```text
EEX
├─ Header
├─ Scene Offset Table
├─ Scene[]
│  ├─ section_count (u16)
│  ├─ Section[]
│  │  ├─ section_len (u16)
│  │  ├─ test commands...
│  │  ├─ 0x0000
│  │  ├─ event_len (u16)
│  │  ├─ event commands...
│  │  └─ 0x0000
```

### 4.1.3 命令编码

- 命令码为 `u16` 小端
- 数值参数多为 `u8/u16/u32`
- 字符串为 `GB18030`；常规文本兼容 GBK，但原版存在四字节字符
- 事件层面可区分“条件段”和“执行段”

当前仓库已整理出完整命令表，见：

- [`docs/EEX二进制格式说明.md`](../EEX脚本/EEX二进制格式说明.md)
- [`docs/曹操传文件结构说明.md`](曹操传文件结构说明.md)

### 4.1.4 命名规律

本目录中共有：

- `R_*.eex` 59 个
- `S_*.eex` 58 个

结合社区通用说法与当前项目命名，可以认为：

- `R` 更偏剧情/场景段落
- `S` 更偏战斗/关卡段落

## 4.2 `.e5s`：存档文件

### 4.2.1 文件组织

当前目录中有 31 个 `.e5s`：

- `Sv00d/e/s.e5s` 到 `Sv09d/e/s.e5s`：10 个存档槽，每槽 3 个文件
- `Svcmn.e5s`：公共存档

文件大小模式非常稳定：

| 文件 | 大小 |
| --- | ---: |
| 中文版 `Sv??d.e5s` | 21724 |
| 日文版 `Sv??d.e5s` | 21709 |
| `Sv??e.e5s` | 10330 |
| `Sv??s.e5s` | 5891 |
| `Svcmn.e5s` | 108 |

### 4.2.2 社区已知分工

`cczmod` 社区对三件套的描述较一致：

- `Sv??d.e5s`：人物信息、仓库、主要进度状态
- `Sv??s.e5s`：场景信息
- `Sv??e.e5s`：扩展状态，但公开资料没有完全统一的字段定义
- `Svcmn.e5s`：通关/宝物等公共信息

### 4.2.3 当前目录里的 `.e5s` 混有两种语言布局

继续比对后确认，这个目录下的 `d` 存档包含中、日文两种原版布局：

#### 中文版 45 字节头布局

- `Sv00d.e5s`
- `Sv01d.e5s`
- `Sv02d.e5s`
- `Sv07d.e5s`
- `Sv08d.e5s`
- `Sv09d.e5s`

共同特征：

- 文件大小 `21724`
- 开头常见 `05 03 03 00 00` 或近似模式
- 套用 `ccz_origin.xml` 中的保存偏移后，可稳定读出人物块

#### 日文版 30 字节头布局

- `Sv03d.e5s`
- `Sv04d.e5s`
- `Sv05d.e5s`
- `Sv06d.e5s`

共同特征：

- 文件大小 `21709`
- 开头为 `00 01 03 02 02 ...`
- `0x05` 可见 Shift-JIS `シナリオ名`
- 同槽 `Sv03s.e5s` 的战场文本也是 Shift-JIS 日文
- 人物表从 `0x14CD` 开始

它不是未知引擎或 MOD 格式。日文版文件头显示文字区是 30 字节，中文版
是 45 字节，因此金钱起的逻辑字段整体提前 `0x0F`：

| 字段 | 中文版 | 日文版 |
| --- | ---: | ---: |
| 金钱 | `0x32` | `0x23` |
| 忠奸值 | `0x56` | `0x47` |
| 装备/仓库起点 | `0x57` | `0x48` |
| 物品数量起点 | `0x2AF` | `0x2A0` |
| 人物表起点 | `0x14DC` | `0x14CD` |

两种人物表都是 `512 × 0x20 = 0x4000` 字节，并恰好延伸到文件末尾。

### 4.2.4 当前样本头部特征

[`ccz/Sv00d.e5s`](https://github.com/lometsj/ccz_eex_parse/blob/master/ccz/Sv00d.e5s) 开头可直接看到：

- 等级/标题类文本区域
- 大量固定长度字段
- 从社区拆解贴可知前 `0x00-0x55` 一带是读档界面显示信息

[`ccz/Sv00s.e5s`](https://github.com/lometsj/ccz_eex_parse/blob/master/ccz/Sv00s.e5s) 也能看到一段场景标题与关卡信息，符合“场景态”定位。

### 4.2.5 `Sv??d.e5s` 人物块结构

结合社区拆解、[`ccz_origin.xml`](https://github.com/lometsj/ccz_eex_parse/blob/master/ccz_origin.xml) 偏移和本地样本验证，两种语言布局的逻辑结构相同：

- 中文版人物区：`0x14DC..0x54DC`
- 日文版人物区：`0x14CD..0x54CD`

其中人物数据区长度恰好是：

```text
0x54DC - 0x14DC = 0x54CD - 0x14CD = 0x4000 = 16384 bytes
16384 / 0x20 = 512
```

也就是：

- 一共 `512` 个人物槽位
- 每个槽位 `0x20` 字节

当前基于本地样本可较高可信地得到以下字段表：

| 相对偏移 | 长度 | 含义 | 说明 |
| --- | ---: | --- | --- |
| `+0x00` | 2 | `face_id` | 头像编号 |
| `+0x02` | 1 | `pmapobj_id` | R 形象编号 |
| `+0x03` | 1 | `camp_or_side` | 阵营/敌我标志 |
| `+0x04` | 5 | 五维基础值 | `str/vit/int/avg/luk` 的半值存储 |
| `+0x09` | 5 | 战斗面板值 | 常与基础值相同，领导者/有装备者会不同 |
| `+0x0E` | 2 | `hp` | 当前/存档 HP |
| `+0x10` | 1 | `mp` | 当前/存档 MP |
| `+0x11` | 1 | `force_id` | 兵种编号 |
| `+0x12` | 1 | `level` | 等级 |
| `+0x13` | 1 | `exp` | 经验 |
| `+0x14` | 3 | 武器三元组 | `id / lv / exp` |
| `+0x17` | 3 | 防具三元组 | `id / lv / exp` |
| `+0x1A` | 3 | 辅助物三元组 | `id / lv / exp` |
| `+0x1D` | 1 | `sortie_count` | 出战次数 |
| `+0x1E` | 1 | `kill_count` | 击退/击杀次数 |
| `+0x1F` | 1 | `retreat_count` | 撤退次数 |

例如 [`ccz/Sv00d.e5s`](https://github.com/lometsj/ccz_eex_parse/blob/master/ccz/Sv00d.e5s) 在 `0x14DC` 的第 0 个人物块，可与 [`parse.json`](https://github.com/lometsj/ccz_eex_parse/blob/master/parse.json) 的曹操数据互相印证：

- `face_id = 0`
- `pmapobj_id = 0`
- `level = 3`
- `weapon_id = 23`
- `armor_id = 38`
- `hp = 117`
- `mp = 33`

而第 1、2 个人物块分别能对上夏侯惇、张辽的头像号、R 形象、兵种、装备和基础能力。

### 4.2.6 `Sv??e.e5s` 与 `Sv??s.e5s`

- `Sv??s.e5s` 中能直接看到胜利条件等场景文本，符合“场景态”定义
- `Sv??s.e5s[0x515]` 起不是此前按余数猜测的 176 条记录，而是由旧
  引擎保存循环确认的 115×26 字节运行时单位表；随后固定 `0x640`
  字节是 40×40 单位占位表，格值为单位槽号、`FF` 为空
- `Sv??e.e5s[0x0000..0x06BC]` 是 115×15 字节出场表：
  我军 15 槽、友军 20 槽、敌军 80 槽
- 友军、敌军槽分别逐项对应 S 剧本 `0x46`、`0x47` 指令，已验证人物
  编号、坐标、AI 类型、AI 附加参数联合体、兵力参数和等级
- `Sv??e.e5s[0x06BD..0x083D]` 是胜败条件文本，中文为 GBK、日文为
  Shift-JIS
- `Sv??e.e5s[0x083E]` 的首个 `u32` 是回合上限
- `Sv??e.e5s[0x085A..EOF]` 是 4096 个 `u16le` 事件变量槽
- `03~06` 槽位的 `s` 文件使用 Shift-JIS 文本；其物理分区和单位表
  偏移仍与中文版相同

### 4.2.7 结论

`.e5s` 不是通用容器，而是固定槽位的游戏运行状态快照。若后续需要继续拆：

1. 优先从 `Sv??d.e5s` 入手。
2. 先区分中文 45 字节头和日文 30 字节头，不要混用偏移表。
3. 以“人物块 + 仓库块 + 场景头部”方式分段。
4. 可对同一槽位做存档前后差分，快速锁定字段。

## 4.3 `.e5`：并不是单一格式，而是一整个专用资源族

这是 `ccz` 目录里最需要区分的一点。

`.e5` 只是“资源文件后缀”，并不意味着内部都是同一种二进制布局。  
本次进一步扫描后，可以确认当前目录里的 `.e5` 至少分成 6 个家族：

| 家族 | 数量 | 代表文件 | 说明 |
| --- | ---: | --- | --- |
| `Ls12` 容器 | 21 | `Data.e5` `Face.e5` `Mmap.e5` | 一批共用统一封装头的资源 |
| `Hm` 战场地图 | 58 | `Hm00.e5` ~ `Hm57.e5` | S 战场底图 |
| `Mcall` 策略动画 | 9 | `Mcall00.e5` ~ `Mcall08.e5` | 大型策略动画独立家族 |
| 文本资源 | 1 | `Imsg.e5` | 可直接看到 GBK 文本 |
| 图标/原始小资源 | 1 | `Mark.e5` | 无 `Ls12` 头 |
| 字体资源 | 1 | `Font.e5` | 无 `Ls12` 头 |

### 4.3.1 `Ls12` 头的资源容器

以下 21 个文件头部都以完全相同的 16 字节开始：

```text
4C 73 31 32 20 20 20 20 20 20 20 20 20 20 20 20
ASCII: "Ls12            "
```

即：

- [`ccz/Data.e5`](https://github.com/lometsj/ccz_eex_parse/blob/master/ccz/Data.e5)
- [`ccz/Effarea.e5`](https://github.com/lometsj/ccz_eex_parse/blob/master/ccz/Effarea.e5)
- [`ccz/Face.e5`](https://github.com/lometsj/ccz_eex_parse/blob/master/ccz/Face.e5)
- [`ccz/Gate.e5`](https://github.com/lometsj/ccz_eex_parse/blob/master/ccz/Gate.e5)
- [`ccz/Hexzmap.e5`](https://github.com/lometsj/ccz_eex_parse/blob/master/ccz/Hexzmap.e5)
- [`ccz/Hitarea.e5`](https://github.com/lometsj/ccz_eex_parse/blob/master/ccz/Hitarea.e5)
- [`ccz/Item.e5`](https://github.com/lometsj/ccz_eex_parse/blob/master/ccz/Item.e5)
- [`ccz/Logo.e5`](https://github.com/lometsj/ccz_eex_parse/blob/master/ccz/Logo.e5)
- [`ccz/Logo_p.e5`](https://github.com/lometsj/ccz_eex_parse/blob/master/ccz/Logo_p.e5)
- [`ccz/Meff.e5`](https://github.com/lometsj/ccz_eex_parse/blob/master/ccz/Meff.e5)
- [`ccz/Pmap.e5`](https://github.com/lometsj/ccz_eex_parse/blob/master/ccz/Pmap.e5)
- [`ccz/Mmap.e5`](https://github.com/lometsj/ccz_eex_parse/blob/master/ccz/Mmap.e5)
- [`ccz/Pmapobj.e5`](https://github.com/lometsj/ccz_eex_parse/blob/master/ccz/Pmapobj.e5)
- [`ccz/Pmpalet.e5`](https://github.com/lometsj/ccz_eex_parse/blob/master/ccz/Pmpalet.e5)
- [`ccz/Smlmap.e5`](https://github.com/lometsj/ccz_eex_parse/blob/master/ccz/Smlmap.e5)
- [`ccz/Spalet.e5`](https://github.com/lometsj/ccz_eex_parse/blob/master/ccz/Spalet.e5)
- [`ccz/U_select.e5`](https://github.com/lometsj/ccz_eex_parse/blob/master/ccz/U_select.e5)
- [`ccz/Unit_atk.e5`](https://github.com/lometsj/ccz_eex_parse/blob/master/ccz/Unit_atk.e5)
- [`ccz/Unit_mov.e5`](https://github.com/lometsj/ccz_eex_parse/blob/master/ccz/Unit_mov.e5)
- [`ccz/Unit_spc.e5`](https://github.com/lometsj/ccz_eex_parse/blob/master/ccz/Unit_spc.e5)
- [`ccz/Weather.e5`](https://github.com/lometsj/ccz_eex_parse/blob/master/ccz/Weather.e5)

这说明：

- `Ls12` 不是 `.e5` 的通用魔数，而是 `.e5` 某个子家族的统一封装头
- 头部固定 16 字节，真实内容从 `0x10` 开始
- 这批文件内部没有明显的裸 BMP/PNG/RIFF 等标准文件签名
- 尾部也没有容易一眼识别的索引表/结束标记
- 更像“统一封装层 + 类型相关负载”的资源格式，而不是单纯加了一个短文件头的裸资源

继续参考 `kaodata` 项目后，现在可以把这个结论再推进一步：

- `Ls12` 与 `LS11` 属于同一家压缩封装
- `0x10` 开始是 `256` 字节字典表
- `0x110` 开始是块索引区
- 每个块索引记录为 `12` 字节，按大端序存：
  - `compressed_size`
  - `uncompressed_size`
  - `data_offset`
- 直到遇到 `00 00 00 00` 结束索引
- 若 `compressed_size == uncompressed_size`，该块是未压缩直存
- 否则用 `kaodata` 中 `ls11.py` 的位流 + 回溯复制算法可解压

也就是说，`Ls12` 现在已经不只是“疑似容器”，而是可以稳定读取索引表并完成分块解压的资源封装。

### 4.3.2 `Ls12` 的本地验证结果

把 `kaodata` 的解压思路套到当前目录后，已经能稳定读出这些信息：

| 文件 | 分块数 | 解压总量 |
| --- | ---: | ---: |
| `Data.e5` | 10 | 42079 |
| `Face.e5` | 228 | 1167360 |
| `Pmap.e5` | 74 | 1481184 |
| `Mmap.e5` | 115 | 29440000 |
| `Logo.e5` | 13 | 1592320 |
| `Item.e5` | 104 | 106496 |
| `Unit_atk.e5` | 151 | 7421952 |
| `Weather.e5` | 5 | 216000 |

从这些数字还能看出一些很强的侧面证据：

- `Face.e5` 解压后是 `228 × 5120`，很像 228 张等规格头像资源
- `Item.e5` 解压后是 `104 × 1024`，与物品图标数非常接近
- `Weather.e5` 解压后是 `5 × 43200`，像 5 张等规格天气图
- `Unit_atk.e5` 解压后是 `151 × 49152`，明显是大量固定尺寸动作帧/动画块

目前还没最终确认“解压后像素块如何进一步映射到宽高/调色板”，但至少可以确认：

- `Ls12` 负责的是“分块压缩封装”
- 解压后的负载很多已经是规则化的固定大小资源块
- 真正的图像解释逻辑应当在 `Ls12` 解压之后继续做

因此，`Data.e5` 在逻辑上确实存放数据表；当前仓库里的物理文件是 `Ls12` 封装态，但这层封装已经有可靠解法。

### 4.3.3 `Hm??.e5` 战场地图格式（已解码）

`Hm00.e5` 到 `Hm57.e5` 已确认是无文件头、无压缩的 8 位索引图，
并不是另一种压缩容器。其结构为：

```text
HmXX.e5
└─ Cell[]（按地图格先横后纵）
   └─ 48 × 48 个 palette index（格内按扫描线顺序）
```

关键结论：

- 每个战场格固定为 `48 × 48 = 2304` 字节。
- 文件大小除以 `2304` 即地图格总数。
- 地图宽高均不超过 40 格，即像素边长不超过 1920。
- `HmXX.e5` 自身不保存显式宽高；权威长宽位于同目录 `Hexzmap.e5`
  的同编号分块。
- `Hexzmap` 分块前两字节分别是 `横向格数 × 3` 和 `纵向格数 × 3`，
  后续恰有 `横向格数 × 纵向格数` 个逐格地形编号。
- 当 `Hexzmap.e5` 缺失、无法解包或对应分块校验失败时，可以在 40×40
  范围内枚举因数，并比较相邻格边缘的像素连续性来恢复行列数。
- `Spalet.e5` 恰有 58 个调色板块，与 58 个 `Hm` 文件按编号一一对应。
- 调色板块为 738 或 768 字节，即 246 或 256 个三字节颜色。
- 调色板通道顺序沿用当前项目已验证的 `(1, 2, 0)` 重排。

例如：

| 地图 | 格数 | 像素尺寸 |
| --- | --- | --- |
| `Hm00.e5` | 20×20 | 960×960 |
| `Hm03.e5` | 28×20 | 1344×960 |
| `Hm13.e5` | 20×28 | 960×1344 |
| `Hm27.e5` | 32×40 | 1536×1920 |
| `Hm41.e5` | 40×28 | 1920×1344 |
| `Hm56.e5` / `Hm57.e5` | 40×40 | 1920×1920 |

导出工具：

```bash
./venv/bin/python tools/export_battle_map.py 0
./venv/bin/python tools/export_battle_map.py --all
```

导出器的尺寸来源优先级为：

1. 单张导出时显式传入的 `--columns`
2. 同目录 `Hexzmap.e5` 的权威长宽
3. 相邻 48×48 格块的边缘色差分析

如果把整个文件直接当作一张逐行位图，会出现横向条纹。原因是文件先保存
完整的 48×48 格块，再保存下一个格块，必须先进行格块重排。

### 4.3.4 文本、动画、字体等其他 `.e5` 家族

进一步确认了 3 类此前容易混在一起的 `.e5`：

- `Imsg.e5`：不是 `Ls12`，而是可直接看到 GBK 文本流的文本资源
- `Mcall00.e5` ~ `Mcall08.e5`：不是 `Ls12`；短头部依次记录步骤数、A/B 图组数、内嵌调色板数等字段，如 `19 19 19 01 64 00 00 00`
- `Font.e5` / `Mark.e5`：也都不属于 `Ls12` 家族

这说明后续若写解包器，至少要分 4 套处理逻辑：

1. `Ls12` 家族
2. `Hm??.e5` 地图家族
3. `Mcall??.e5` 动画家族
4. `Imsg.e5` / `Font.e5` / `Mark.e5` 这类特例家族

### 4.3.5 调色板/索引图配套资源

下列文件按社区资料均是调色板或索引图配套：

- `Logo_p.e5`
- `Pmpalet.e5`
- `Spalet.e5`
- `Mark.e5`
- `Weather.e5`
- `Hitarea.e5`
- `Effarea.e5`

这一类通常不是复杂数据表，而是供引擎直接取用的图像/调色板资源。

## 4.4 `Data.e5`：主数据表的“逻辑结构”与“物理封装”要分开看

### 4.4.1 逻辑结构

[`parse_data.py`](https://github.com/lometsj/ccz_eex_parse/blob/master/parse_data.py) 明确把 `Data.e5` 视为以下内容的来源：

- 人物数据
- 物品数据
- 商店数据
- 兵种数据
- 地形适应数据
- 策略数据

[`ccz_origin.json`](https://github.com/lometsj/ccz_eex_parse/blob/master/ccz_origin.json) 中也给出了相应偏移：

| 项目 | 偏移/数量 |
| --- | --- |
| 人物 | `0x18c` / `0x200` |
| 物品 | `0x418c` / `0x68` |
| 商店 | `0x4bb4` |
| 兵种 | `0x53dc` / `0x35` |
| 地形 | `0x5973` |
| 策略 | `0x5fc7` / `0x44` |

代码里还能读出每条记录的逻辑字段，例如：

- 人物：名字、头像编号、R 形象编号、阵营、五维、HP/MP、兵种、等级、装备
- 物品：名字、类型、特效、价格、图标、基础值、升级值
- 策略：名字、说明、类型、目标、范围、耗蓝、图标、各兵种习得等级

### 4.4.2 物理封装上的重要异常
当前目录内的 [`ccz/Data.e5`](https://github.com/lometsj/ccz_eex_parse/blob/master/ccz/Data.e5) 只有 `10302` 字节，但 `ccz_origin.json` 给出的偏移已经超过 `0x5fc7`。  
现在结合 `Ls12` 解压结果，可以把这个“异常”说得更准确：

- `Data.e5` 的物理文件确实是压缩封装态
- 解压后得到的主负载总长是 `42079`
- 这个主负载和 `ccz_origin.json` 的偏移体系非常接近，但不是从同一个起点开始

关键对应关系是：

```text
decoded_data_payload_offset = configured_offset - 0x18C
```

其中 `0x18C` 正好就是 `Game_Unit_Offset`。

本地验证结果如下：

| 配置项 | `ccz_origin.json` | 解压后有效偏移 |
| --- | --- | --- |
| 人物 | `0x18C` | `0x0000` |
| 物品 | `0x418C` | `0x4000` |
| 商店 | `0x4BB4` | `0x4A28` |
| 兵种 | `0x53DC` | `0x5250` |
| 地形 | `0x5973` | `0x57E7` |
| 策略 | `0x5FC7` | `0x5E3B` |

这意味着：

- 你之前写的偏移体系并没有错
- 只是它针对的是“带前导区的逻辑数据视图”
- 而 `Ls12` 解压后直接拿到的是“去掉前导 `0x18C` 后的主数据负载”

进一步验证也支持这一点：

- 解压数据 `0x0000` 就能找到 `曹操`
- `0x0020` 就能找到 `夏侯惇`
- `0x0040` 就能找到 `张辽`
- `0x4000` 开始进入物品主数据区

也就是说，对当前 `Data.e5` 最实用的理解是：

1. 先做 `Ls12` 解压
2. 再把现有偏移统一减去 `0x18C`
3. 之后你现有的记录解析逻辑就能继续沿用

这也是为什么从工程角度看，`Data.e5` 这一部分其实已经很接近“打通”了。

## 4.5 `Imsg.e5`：921 条定长文本记录

[`ccz/Imsg.e5`](https://github.com/lometsj/ccz_eex_parse/blob/master/ccz/Imsg.e5) 可直接看到大量 GBK 中文字节流，且 [`parse_data.py`](https://github.com/lometsj/ccz_eex_parse/blob/master/parse_data.py) 用它读取：

- 人物介绍
- 兵种说明
- 物品说明
- 策略说明

当前已进一步确认：

- 文件固定为 `184200 = 921 × 200` 字节。
- 每条是 GBK、NUL 结束、补零到 200 字节。
- 七条英文标记分隔物品、策略、战役、兵种、列传、撤退、致命一击和
  制作人员八区。
- 致命一击前 21 条由 EXE 的特殊人物表直接索引，后 30 条按 Data 的
  `critical_text_type` 每组三句随机选择。
- 原版支持单槽原位回写；440200 字节扩展版必须使用另一套偏移。

完整分区表和导出工具见
[`数据与文本文件说明.md`](数据与文本文件说明.md)。

## 4.6 图像资源类 `.e5`

结合社区资料，可以把当前目录中的主要图像文件理解为下表：

| 文件 | 作用 |
| --- | --- |
| `Logo.e5` | 开场、战前标题、结束、Game Over、单挑背景等 |
| `Mmap.e5` | R 场景背景，含外景、内景、中国地图 |
| `Pmap.e5` | R 地形 |
| `Pmapobj.e5` | R 人物形象 |
| `Face.e5` | 头像 |
| `Unit_atk/mov/spc.e5` | S 战场人物动作帧 |
| `Hm??.e5` | S 战场底图 |
| `Hexzmap.e5` | S 地形 |
| `Smlmap.e5` | 战场小地图；尺寸由同编号 Hexzmap 的格数乘 6 得出 |
| `Gate.e5` | 战斗障碍的 3×3 格双状态替换块；城门、船、火等均会使用 |
| `Meff.e5` | 小型策略特效 |
| `Mcall??.e5` | 大型策略特效 |
| `Item.e5` / `Itemicon.dll` | 物品图标 |
| `Mark.e5` | 血条、火焰、船等小图标 |
| `Mapatr.dll` | 地形图标 |
| `Mgcicon.dll` | 策略图标 |
| `Cmdicon.dll` | 菜单图标 |
| `Hitarea.e5` | 攻击范围覆盖图 |
| `Effarea.e5` | 穿透范围覆盖图 |
| `Weather.e5` | 天气图 |
| `U_select.e5` | 出战/练武场/对话背景等 UI 图 |

### 4.6.1 `Mmap.e5` 的当前工程结论

`Mmap.e5` 现在已经可以给出比“R 场景背景”更具体的工程结论：

- 共 `115` 个资源块
- 每块解压后固定为 `256000` 字节
- 这正好等于 `640 * 400`

但这里最关键的一点是：

**这 `256000` 字节不是线性 `640x400` 像素流。**

如果直接按“每行 640 像素、共 400 行”的方式解释，会得到明显的横向条纹图。  
本地重排实验已经确认，`Mmap.e5` 的正确像素布局应为：

- 图像逻辑尺寸：`640x400`
- 存储顺序：按 `8x8 tile` 排列
- 也就是 `80 x 50` 个 tile，每 tile `64` 字节

换句话说，工程上应该按：

1. 解压得到 `256000` 字节索引像素
2. 以 tile 为单位顺序读取
3. 再拼回 `640x400`

而不是把这 `256000` 字节直接当成线性 framebuffer。

另外，当前项目中 `background_set` 到 `Mmap.e5` 的槽位映射可按下列规则理解：

- `outside_map_id = n` -> `Mmap` 槽位 `n`
- `inside_map_id = n` -> `Mmap` 槽位 `40 + n`
- `chinese_map_id` -> `Mmap` 槽位 `114`

这与当前样本里实际出现的：

- 外场景 id
- 内场景 id
- 中国地图 id

范围能够对齐。

调色板方面，当前工程预览链使用：

- `Pmpalet.e5` 同槽位调色板块
- 当前实装的通道顺序为 `120`

### 4.6.2 R 内场景坐标的当前结论

曹操传 R 剧本里，人物在内场景中的出现/移动坐标目前可以明确为：

- 不是像素坐标
- 不是普通笛卡尔坐标
- 而是内场景逻辑网格坐标

现有教程给出的典型例子是：

- 最上边的人：`(32,32)`
- 右下方的人：`(36,32)`
- 左下方的人：`(32,36)`

这说明：

- `x`、`y` 两个轴分别沿两条斜向增长
- 其本质更接近等角/斜轴地图坐标

工程上因此可以得出两个直接结论：

1. `people_display` / `people_move` 在 R 内场景中的 `x,y` 不能直接按屏幕像素理解。
2. 若要把人物正确叠到 `Mmap.e5` 背景图上，必须结合 `Pmap.e5` 提供的网格/地形数据。

也就是说：

- `Mmap.e5`：负责背景图像
- `Pmap.e5`：负责 R 场景逻辑网格/坐标层

当前仅用 `Mmap` 还不足以精确做人物落点预览。

### 4.6.3 `Pmap.e5` 的当前工程结论

`Pmap.e5` 的物理结构和网格语义均已确认：

- 也是 `Ls12` 容器
- 共 `74` 个资源块
- 每块解压后固定为 `20016` 字节
- 每块为 `16` 字节头部 + `100×100×u16le` 网格
- 所有块头的中间六项都是 `100,100,1,200,100,1`，与宽、高、
  平面数和每行 200 字节相符
- 头部首项在文件中只出现 `23/54/56/61`，但旧引擎载入后会将它覆盖
  为实际 Mmap 场景号；末项恒为 `0x3A41`，旧引擎没有读取调用，应按
  旧格式尾字原样保留
- 穷举原版 `740000` 个网格单元，恰好只有 `0xFFFF`（可通行）和
  `0xF0FF`（障碍）两种

社区文档所说的 `FF` 无障碍、`F0` 障碍是观察单元高字节的简写；
按文件的小端 `u16` 读取就是上述两个完整数值。

`Pmap` 只覆盖 R 内场景。第 `i` 块对应 `Mmap` 第 `i+40` 块（0
起算），即社区资料中的“第一个 Pmap 对应第 41 个 Mmap”。因此 74
块正好覆盖 `Mmap` 槽位 40～113；前 40 个外场景和最后一个中国地图
不带 Pmap。

### 4.6.3.1 `Pmap -> Mmap` 投影的当前修正

结合轩辕春秋的实测公式、反汇编代码和多张原版场景覆盖图，当前已经确认需要区分两种锚点：

```text
Pmapobj 绘制锚点：
x = (a - b) * 8 + 312
y = (a + b) * 4 - 216

Pmap 地面/脚底锚点：
x = (a - b) * 8 + 312
y = (a + b) * 4 - 193
```

二者横坐标相同，地面锚点的纵坐标比人物绘制锚点低 `23` 像素。旧覆盖图直接使用了人物绘制锚点，因此把障碍层整体画高了。

当前实现规则：

1. 人物图继续使用 Pmapobj 绘制锚点，并由该锚点计算 `48×64` 图像左上角。
2. 网格线、`0xF0FF` 障碍格、鼠标命中和状态栏坐标统一使用地面锚点。
3. `0xF0FF` 表示不可通行，`0xFFFF` 表示可通行。
4. 地面坐标正反投影已经对全部 `100×100` 格完成一致性测试。
5. 多场景覆盖对照显示，`Y+23` 后障碍格稳定落在桌椅、屏风、树木和墙体的地面占位区域。

### 4.6.4 `Face.e5` 的进一步结论

`Face.e5` 是目前已经能推进到“块级结构基本坐实”的图像资源之一。

结合 `Ls12` 解包结果与本地样本验证：

- `Face.e5` 一共可解出 `228` 个资源块
- 每个资源块解压后固定为 `5120` 字节
- 这 `228` 个块没有额外的小头部，块开头就是连续像素/索引数据
- 当前项目里人物与存档实际引用到的头像编号最大为 `218`

这 4 点组合起来，强烈说明：

- `Face.e5` 的“一个块”基本就对应“一个头像槽位”
- 目录里至少预留了少量未使用头像

### 4.6.2 单头像尺寸判断

对 `5120` 字节头像块做灰度预览后，可以直接区分出方向：

- 按 `64x80` 排列时，能渲染出可识别的人像轮廓
- 按 `80x64` 排列时，只会得到横向噪声状图

因此目前可以高可信判断：

- `Face.e5` 的单头像尺寸应为 `64x80`
- 每像素大概率是 `8-bit` 索引色值
- 当前直接渲染成灰度能看出轮廓，但颜色仍不正确，说明真正显示时还需要外部调色板或引擎内置调色逻辑

也就是说，`Face.e5` 的结构可以暂时表述为：

```text
Face.e5
└─ Ls12 container
   ├─ 228 portrait blocks
   └─ each block = 5120 bytes = 64 x 80 indexed pixels
```

### 4.6.3 与人物/存档编号的对应关系

结合当前项目数据、头像导出结果和人工校对，`Face.e5` 的槽位规则应修正为：

- `000-007`：曹操的不同表情
- `008` 开始：后续基本按“一个武将一个头像”的顺序排列

这说明 `face_id` 不能简单理解为“武将 ID 对应的一张唯一头像”，至少在前 8 个槽位上不是这样。

本地验证结果：

- [`parse.json`](https://github.com/lometsj/ccz_eex_parse/blob/master/parse.json) 中人物的 `face` 最大值为 `218`
- 原版样式存档 [`Sv00d.e5s`](https://github.com/lometsj/ccz_eex_parse/blob/master/ccz/Sv00d.e5s) 等中，人物块里的 `face_id` 最大值也是 `218`

这说明：

- `Data.e5` 和存档里的 `face_id`
- 与 `Face.e5` 的块序号

三者之间已经能建立稳定映射。

换句话说，当前完全可以把：

- `face_id = 0..7` 视为曹操表情槽位组
- `face_id = 8` 之后再按武将顺序映射

这对后续做头像导出器、头像替换器已经足够实用。

### 4.6.4 `Face.e5` 的调色板来源

继续对调色板文件做 `Ls12` 解包后，可以确认：

- `Pmpalet.e5` 含有大量 `768` 字节块
- `Spalet.e5` 也含有多个 `768` 字节块
- `768 = 256 * 3`，非常符合 `256` 色、每色 `RGB` 三字节调色板的经典结构

其中：

- `Pmpalet.e5` 解压后共有 `115` 个调色板块量级
- `Spalet.e5` 解压后共有 `58` 个调色板块量级
- `Logo_p.e5` 也含调色板块，但其颜色分布更像标题/界面通用调色板，不适合头像

把 `Face.e5` 第 0、1 块头像分别套用候选调色板后，结果非常明确：

- `Logo_p.e5` 的 768 字节块上色结果明显失真
- `Pmpalet.e5` 第 0 块可以把头像渲染出正常肤色、头发、衣物颜色
- `Spalet.e5` 第 0 块也能得到同样正确的头像颜色

进一步对比发现：

- `Pmpalet[0]` 与 `Spalet[0]` 并非整块完全相同
- 但在头像实际使用到的索引范围 `10..173` 上，`164` 个调色板三元组完全一致

这意味着：

- 头像实际显示依赖的是一套“通用旧引擎 256 色调色板”
- 对 `Face.e5` 来说，`Pmpalet.e5` 第 0 块和 `Spalet.e5` 第 0 块在有效索引区间内可以等价使用
- 头像并不需要一份单独命名为 `Face_p.e5` 的专属调色板

另外，本次还做了一个“174 色重排实验”：

- 取 `Pmpalet[0]` / `Spalet[0]`
- 把头像常用索引区间 `10..173` 压缩映射成一个“174 色实验板”
- 再重新给头像上色

实验结果是：颜色明显变坏，远不如直接使用 `Pmpalet[0]` / `Spalet[0]` 的原始索引结果。

这说明社区口中的“174 色调色板”更可能表示：

- 头像实际只使用其中一段有效颜色集合

而不是：

- 要把头像像素索引整体重排成 `0..173` 的紧凑新索引

换句话说，当前阶段最正确的做法是：

- 保持 `Face.e5` 原始索引不动
- 直接用兼容的通用调色板块上色

而不是先做一轮索引压缩重排。

再往前推进一步，本地还做了一个“调色板三元组通道顺序实验”：

- 对同一张头像，分别测试 `RGB` 的 6 种通道排列
- 结果显示默认 `RGB` 只是“勉强能看”
- `120` 排列得到的肤色、发色、服装色最接近正常人物头像

这里的 `120` 指：

```text
output_rgb = (src[1], src[2], src[0])
```

也就是把 `E5` 调色板字节三元组按：

```text
BRG 存储 -> RGB 使用
```

来理解，会比直接把它当 `RGB` 使用更合理。

因此，当前对头像调色板最实用的工程结论应更新为：

```text
Face.e5
├─ pixels: 64x80, 8-bit indexed
├─ palette source: Pmpalet[0] / Spalet[0]
└─ palette channel order: likely BRG -> RGB
```

目前最稳妥的工程结论是：

```text
Face.e5
├─ pixels: 64x80, 8-bit indexed
└─ palette: compatible with Pmpalet[0] / Spalet[0] in used index range
```

### 4.6.5 旧引擎与新引擎差异

社区资料明确提到：

- 旧版资源大量依赖 256 色调色板
- `6.1/6.2` 新引擎开始支持 JPG/PNG/真彩替换
- 一些调色板文件在新引擎下可以废弃
- `Item.e5` 与 `Itemicon.dll` 在部分引擎版本中被合并使用

因此，当前目录同时出现：

- `Item.e5`
- `Itemicon.dll`
- `Logo_p.e5`
- `Pmpalet.e5`
- `Spalet.e5`

并不矛盾，它说明这份目录更接近“兼容旧资源组织方式”的游戏目录。

## 4.7 程序与动态库

### 4.7.1 `Ekd5.exe`

`file` 命令已验证它是标准 `PE32 executable`，即主程序。

本项目中的大量资源命名都与它的加载逻辑直接相关，例如：

- `R_*.eex` / `S_*.eex`
- `HM??.e5`
- `HEXZMAP.E5`
- `Face.e5`
- `Pmapobj.e5`

### 4.7.2 `mp3serv.dat`

虽然扩展名是 `.dat`，但其文件头是标准 `MZ`，说明它本质上是一个可执行文件。  
结合社区资料，它负责 MP3 播放服务；标题文字不匹配时甚至会导致“没音乐”的经典问题。

### 4.7.3 其他 DLL

基于命名和社区长期用法，可以高可信判断：

- `Cmdicon.dll`：菜单图标
- `Itemicon.dll`：物品图标
- `Mgcicon.dll`：策略图标
- `Mapatr.dll`：地形属性图标
- `AutoDLL.dll`：与本地旧引擎固定地址绑定的自动存档、快速点击补丁
- `Koeicda.dll`：保留旧引擎 `CDAudio*` 接口并通过私有窗口消息控制
  `mp3serv.dat`；20 个导出中 4 个有实际播放控制代码，其余为成功桩

其中 [`ccz/AutoDLL.ini`](https://github.com/lometsj/ccz_eex_parse/blob/master/ccz/AutoDLL.ini) 已明确暴露出：

- `EnableAutoSave`
- `EnableFastKey`
- `AutoSaveSlot`

本地旧引擎的调用链已经进一步确认：

- `Ekd5.exe` 的 `.Silvana` 补丁节从 `AutoDLL.dll` 导入 `ASave`，
  `0x0044E1CD` 通过 IAT `0x004CD014` 跳入该函数；
- 自动存档复用 `0x0041B16A(slot_index)`，调用前临时 NOP 两个
  存档流程调用点并把一个 `jne` 改成 `jmp`，结束后恢复原字节；
- `AutoSaveSlot` 外部使用 1～10，内部减 1；越界回退为内部索引 9；
- 快速点击使用 `WH_KEYBOARD` 钩子，默认以空格切换，按
  `DelayTime` 间隔向类名为“`三国志曹操传`”的窗口发送
  `WM_LBUTTONDOWN/WM_LBUTTONUP`。

因此它不是通用资源容器，而是精确依赖这份旧 `Ekd5.exe` 地址布局的
外挂机能补丁 DLL。Star 新引擎的同名选项或版本号资料不能直接用于解释
这些地址和调用方式。

## 4.8 音频视频与文本辅助文件

这些格式最明确：

- `Se*.wav`：标准 WAV 音效，样本为 8-bit mono 22050Hz
- `SoundTrk/*.mp3`：背景音乐
- `mp3list.txt`：曲目表，逐行列出 `SoundTrk\xx-AudioTrack xx.mp3`
- `Logo.avi` / `Press.avi` / `open.avi` / `end.avi`：标准 AVI 过场视频
- `font.reg` / `font2.reg`：Windows 字体注册表脚本

## 5. 对“具体格式”的最终判断

如果按“能否直接写解析器”的标准来分，这个目录下的文件可以分为三档：

### 5.1 已经足够写稳定解析器

- `.eex`
- `Hm00.e5` ~ `Hm57.e5`（配合 `Spalet.e5`）
- `Hexzmap.e5`
- `Smlmap.e5`（配合 `Hexzmap.e5`、`Spalet.e5`）
- `Gate.e5`（配合 `Spalet.e5` 和 EEX 0x58 指令）
- `Item.e5`（配合 `Itemicon.dll` 和 Data.e5 的 icon 字段）
- `Itemicon.dll`
- `Data.e5` 的全部 10 个物理分块
- `Meff.e5`、`Mcall??.e5` 动画块
- `Mark.e5`
- `Logo.e5`、`Logo_p.e5`
- `Weather.e5`
- `Hitarea.e5`、`Effarea.e5`
- `U_select.e5`
- `Font.e5` 的物理字形格式
- `Sv??s.e5s` 的战场单位表
- `Sv??e.e5s` 的物理区段
- `Svcmn.e5s` 的结局、物品图鉴标志及 Logo 花色选择
- `.wav`
- `.mp3`
- `.avi`
- `.ini`
- `.txt`
- `.reg`

### 5.2 已完成初始格式解析、不再属于“待解包”

`Imsg.e5`、`Face.e5`、`Mmap.e5`、`Pmap.e5`、`Pmapobj.e5`、
`Unit_*.e5` 和三类调色板均已有稳定的分块、尺寸、像素或网格解释。
其中 R/S 形象和地图资源还已有实际浏览、叠加和回写工具。

### 5.3 已知的旧引擎保留位

- `Sv??s.e5s` 的 26 字节战场单位记录已经全部完成物理分段和字段级
  定名；复合运行标志已拆出 `0x02` 已行动、`0x04` 已移动、`0x08`
  形象镜像、`0x20` 动画期间抑制地图绘制、`0x80` 临时离场。其余
  三个位在本地调用点和十份存档中均未使用，按未知位保留

## 6. 完成度审计与仍可进行的运行时研究

本轮原 1～10 优先级文件及目录内全部 52 个规范化文件族，均已达到可
稳定拆包、解析或按标准媒体/PE 格式识别的程度。以下内容不再构成文件
格式缺口，只是可以继续做的运行时行为研究：

1. `Sv??s.e5s` 的 26 字节战场运行时单位记录已由本地旧引擎完成字段
   定名：原 `+0A..+0C` 是 AI 移动目标坐标与目标单位出场顺序，
   `+0D` 是 AI 行动调度序号；`+0F` 当前实际使用的五个运行标志位也已
   按本地旧引擎调用点完成定名。`Sv??e.e5s` 原先所列五个“保留字节”
   已经证实为
   `u32le` 出场标志的高三字节及两字节 AI 参数联合体，不再属于未知
   填充。`Svcmn[0x6B]`
   已由本地旧引擎调用点确定为 Logo 花色选择：0～3 对应花色 1～4，
   4 为“无”。
2. EEX 格式、命令语义与严格保真回写已经闭环。旧引擎 `0x10` 已由
   固定教学调用链定名为“战斗操作教学”；`0x28` 是写入后无消费路径的
   “设置保留脚本值”；`0x6A` 是清零战场矩形高亮标志的“清除高亮
   区域”。三者均以本地 `Ekd5.exe` 为证据，没有套用 Star 新引擎的
   重定义。
3. `Mcall00.e5` 头部 `+0x03=1` 已由旧引擎装载器确认是内嵌调色板
   数量；真实 768 字节调色板位于 A、B 两组图像之后，而不是两组之间
   的全零空槽。Meff/Mcall 步骤 `+0x02/+0x03` 也已由旧引擎混色路径
   定名为单层加深级别和 16 级透明/混合强度。

`Data.e5` 分块 6～9 已经由本地旧引擎调用图确认不会被装载，应归为
未使用的“终端”占位表，不再列为待定运行时字段。`EEX` schema 目前
106 个普通命令均为 `done`；仅 `0x00` 和 `0x03` 两个结构控制标记保留
`partial` 标签，命令表中不存在 `placeholder`。若继续研究保留位，应
优先依赖本地主程序调用点、动态调试或制作受控差分样本。

完整结论和实现入口已按类别整理到：

- [`图像资源文件说明.md`](图像资源文件说明.md)
- [`存档文件说明.md`](存档文件说明.md)
- [`数据与文本文件说明.md`](数据与文本文件说明.md)

## 7. 参考资料

### 本仓库内资料

- [`parse_eex.py`](https://github.com/lometsj/ccz_eex_parse/blob/master/parse_eex.py)
- [`parse_data.py`](https://github.com/lometsj/ccz_eex_parse/blob/master/parse_data.py)
- [`ccz_origin.json`](https://github.com/lometsj/ccz_eex_parse/blob/master/ccz_origin.json)
- [`docs/EEX二进制格式说明.md`](../EEX脚本/EEX二进制格式说明.md)
- [`docs/曹操传文件结构说明.md`](曹操传文件结构说明.md)

### 社区资料

- [曹操传存档文件深度剖析](https://www.xycq.org.cn/forum/thread-196312-1-1.html)
- [发布6.1版](https://www.xycq.org.cn/forum/thread-283417-1-1.html)
- [发布6.2版exe修正版](https://www.xycq.org.cn/forum/thread-309404-1-1.html)
- [CCZ MOD美工之无双教程](https://www.xycq.org.cn/forum/blog.php?endtime=0&starttime=0&tid=204321)
- [将剧本放入文件夹](https://www.xycq.org.cn/forum/archiver/tid-277056.html)
- [tzengyuxio/kaodata](https://github.com/tzengyuxio/kaodata)

## 8. 本次落盘文件

本文已落盘到：

- [`docs/ccz目录文件用途与格式分析.md`](ccz目录文件用途与格式分析.md)

本次还新增了一个可复跑的分析脚本：

- [`tools/ccz_probe.py`](https://github.com/lometsj/ccz_eex_parse/blob/master/tools/ccz_probe.py)
- [`tools/export_face_experiments.py`](https://github.com/lometsj/ccz_eex_parse/blob/master/tools/export_face_experiments.py)
- [`tools/export_face_png.py`](https://github.com/lometsj/ccz_eex_parse/blob/master/tools/export_face_png.py)

示例：

```bash
./venv/bin/python tools/ccz_probe.py
./venv/bin/python tools/ccz_probe.py --save Sv00d.e5s --units 8
./venv/bin/python tools/ccz_probe.py --ls12 Data.e5
./venv/bin/python tools/export_face_experiments.py --face-id 0 --count 3
./venv/bin/python tools/export_face_png.py --start 0 --count 10 --format png
```

其中 `export_face_png.py` 默认使用当前已知规则命名：

- `000-007` -> `曹操_表情00` ~ `曹操_表情07`
- `008+` -> 后续武将顺序命名
