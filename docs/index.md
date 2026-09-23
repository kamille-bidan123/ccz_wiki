# 魏武 · 曹操传旧引擎 MOD 工坊

面向本地 1999 年旧引擎《三国志曹操传》的统一 MOD 制作环境。编辑器以
人物、物品、R 场景和 S 关卡为中心，把 Data、Imsg、EEX、地图、地形和
形象资源放进同一个项目模型；Star 等新引擎资料不会自动套用。

旧的单文件 EEX、战场地图、R 场景和形象编辑器仍保留，统一工作台是推荐
入口。

## 启动统一 MOD 工坊

直接打开游戏目录：

```bash
./venv/bin/python run_mod_studio.py /path/to/ccz
```

也可以先启动，再选择目录：

```bash
./venv/bin/python run_mod_studio.py
```

当前统一工作台已经支持：

- 扫描 1,985 个逻辑实体和 29,843 条跨文件引用；
- 战役总览按旧引擎编号配对 R/S 关卡并汇总引用与问题；
- `Data.e5` Ls12/42079 字节解压负载双形态读取、可撤销转换、字段修改、分块重建和回读；
- 旧引擎 AI 目标解释与带证据等级的物理候选评分实验器；
- 校验旧引擎哈希的 Windows/Wine 测试运行，以及 31 个存档文件的字段级前后差分；
- 兼容配置门控的插件 SDK（格式、实体、验证、动态编辑器和预览扩展点），
  支持不删除文件的项目级停用；
- `Imsg.e5` 人物列传、物品/策略/兵种说明和关卡名联动编辑；
- 商店人物、32 个物品槽、兵种许可、策略学习等级及地形适应/移动表可
  等长编辑并进入引用验证；
- EEX AST 事件参数编辑、复制、删除、同块排序、重建和再次解析；
- EEX 事件单步/继续预览、断点、局部/全局变量条件、状态观察与快照导出；
- S 战场真实底图、Hexzmap 地形、Smlmap 小地图和 Gate 动态图层；
- Gate 对象新增、选择、拖动、属性修改和删除；
- Hitarea/Effarea 精确逻辑范围、移动/攻击/策略候选及 Weather 四帧天气叠加预览；
- `0x46/0x47` 固定出场槽显示、拖动、增删、阵营转换、属性编辑和旧引擎 AI 解释；
- R 场景背景、Pmap 障碍、人物显示/移动命令及路径节点编辑；
- Face/Pmapobj/Unit 形象、物品图、策略图标及 Meff/Mcall A/B 图组预览、
  导入与量化；
- Logo、Weather、Mark、Hitarea、Effarea、U_select 共 116 张固定资源图
  已进入统一导航，可按各自尺寸和调色板预览、导入、回读与撤销；
- 数据库多选批量修改，以及全项目撤销/重做、引用和业务差异预览；
- 源目录只读、`.cczmod/working` 覆盖层保存和测试副本发布。

项目扫描命令：

```bash
./venv/bin/python cczstudio_cli.py /path/to/ccz --problems
```

运行测试：

```bash
./venv/bin/python -m unittest discover -s tests -v
```

## 功能特性

- **场景/章节浏览**: 树状结构查看和管理场景 (Scene) 和章节 (Section)
- **事件编辑**: 添加、删除、移动事件命令
- **属性编辑**: 可视化编辑事件参数
- **搜索功能**: 支持中文、拼音、ID 搜索事件
- **JSON 导入导出**: 支持 JSON 格式的导入和导出
- **撤销/重做**: 基本编辑操作支持
- **事件摘要显示**: 自动显示事件关键信息（对话内容、人物 ID、坐标等）

## 界面说明

### 场景树（左侧）
- 第一列：场景/章节名称
- 第二列：章节数/事件数统计

### 事件树（中间）
- 第一列：`[序号] 0x 类型 事件名称`
- 第二列：事件摘要
  - 对话事件：显示对话内容（最长 30 字符）
  - 人物事件：显示人物 ID
  - 坐标事件：显示 (x,y) 坐标
  - 变量事件：显示变量索引或真假列表
  - 布尔事件：显示"是"或"否"

## 安装

### 1. 创建虚拟环境

```bash
python3 -m venv venv
```

### 2. 激活虚拟环境

**macOS/Linux:**
```bash
source venv/bin/activate
```

**Windows:**
```bash
venv\Scripts\activate
```

### 3. 安装依赖

```bash
pip install -r requirements.txt
```

## 使用方法

### 启动编辑器

```bash
python run_editor.py
```

或者

```bash
python eex_editor.py
```

### 启动战场地图地形编辑器

直接传入曹操传游戏目录：

```bash
./venv/bin/python run_battle_map_editor.py /path/to/ccz
```

也可以不传路径，启动后在界面中选择：

```bash
./venv/bin/python run_battle_map_editor.py
```

战场地图编辑器支持：

- 读取 `HmXX.e5`、`Spalet.e5` 和 `Hexzmap.e5`
- 显示战场地图总数、格数和像素尺寸
- 显示地形网格、半透明分类色层和地形名称
- 左键绘制、拖动连续绘制、右键吸取地形
- 撤销、重做及跨地图修改
- 保存前重建并校验 `Hexzmap.e5`
- 首次保存时自动生成 `Hexzmap.e5.bak`

### 启动 R / S 人物形象编辑器

直接传入曹操传游戏目录：

```bash
./venv/bin/python run_sprite_editor.py /path/to/ccz
```

也可以启动后选择目录：

```bash
./venv/bin/python run_sprite_editor.py
```

形象编辑器支持：

- 浏览 151 组 `Unit_spc.e5`、`Unit_mov.e5`、`Unit_atk.e5` 战场形象
- 浏览 206 组 `Pmapobj.e5` R 形象及其正面、反面动作
- R 形象 20 帧动作名称和人物引用名称联动
- 单帧或整张动作条的 PNG/BMP 导入与导出
- 导入时自动量化到原版 256 色调色板
- 透明色索引 0（`#F700FF`）预览和动画播放
- 跨文件撤销、重做和修改状态提示
- 保存前重建并重新解包校验所有修改过的 LS12 文件
- 首次保存时为每个资源生成对应的 `.e5.bak`

### 启动 R 场景地图编辑器

直接传入曹操传游戏目录：

```bash
./venv/bin/python run_r_scene_editor.py /path/to/ccz
```

也可以启动后选择目录：

```bash
./venv/bin/python run_r_scene_editor.py
```

R 场景编辑器支持：

- 浏览全部 74 张 `Pmap.e5` 内场景，并自动对应 `Mmap.e5` 的底图
- 使用游戏坐标投影显示完整菱形网格和 `Pmap.e5` 障碍格
- 绘制可通行/不可通行格，支持拖动连续绘制、撤销和重做
- 导出 640×400 场景底图，或导入同尺寸图片并量化到原版调色板
- 打开游戏目录内的 `R_*.eex`，按场景和章节浏览人物显示/移动命令
- 使用 `Pmapobj.e5` 实际形象预览人物，并编辑人物编号、坐标、方向和动作
- 在“放置人物”工具下直接点击地图修改所选人物的 R 场景坐标
- 保存前重新解析校验 `Pmap.e5`、`Mmap.e5` 和 `R_*.eex`
- 首次保存时自动生成对应的 `.bak` 备份

### 导出其余资源的分析结果

下面的命令会解析并导出 `Hitarea`、`Effarea`、`Weather`、`U_select`、
`Meff`、`Mcall`、`Mark`、`Logo`、`Font`、存档区段和 `Data.e5`
全部十个物理分块：

```bash
./venv/bin/python tools/export_remaining_resources.py \
  /path/to/ccz /path/to/output
```

输出目录包含各图像资源的联系表、天气逐帧 PNG 和保存完整结构数据的
`analysis.json`。格式结论和仍未命名字段已按类别整理到：

- [`docs/图像资源文件说明.md`](文件与格式/图像资源文件说明.md)
- [`docs/存档文件说明.md`](文件与格式/存档文件说明.md)
- [`docs/数据与文本文件说明.md`](文件与格式/数据与文本文件说明.md)

三个图标 DLL 可单独审计和导出：

```bash
QT_QPA_PLATFORM=offscreen ./venv/bin/python \
  tools/export_icon_dll_resources.py /path/to/ccz /path/to/output
```

原版 `Imsg.e5` 的 921 条定长文本可导出为 JSON：

```bash
./venv/bin/python tools/export_imsg.py \
  /path/to/ccz /path/to/imsg-analysis.json
```

### 操作步骤

1. **打开文件**:
   - 菜单：文件 > 打开 (Ctrl+O)
   - 选择 .eex 文件或 JSON 文件

2. **浏览场景**:
   - 左侧面板显示场景和章节树
   - 点击章节查看事件列表

3. **添加事件**:
   - 选择一个章节
   - 点击"添加事件"按钮或使用菜单 (Ctrl+N)
   - 在搜索框中输入事件名/拼音/ID 查找
   - 双击或点击"插入"添加事件

4. **编辑事件属性**:
   - 在事件树中选择事件
   - 右侧面板编辑属性
   - 修改自动保存

5. **删除事件**:
   - 选择事件后按 Delete 键或点击"删除事件"按钮

6. **移动事件**:
   - 使用"上移"/"下移"按钮调整事件顺序

7. **保存文件**:
   - 菜单：文件 > 保存 (Ctrl+S)
   - 导出为 JSON 格式

## 事件命令列表

| ID | 名称 | 说明 |
|----|------|------|
| 0x00 | 事件结束 | 结束当前事件/章节 |
| 0x02 | 内部消息 | 显示内部消息 |
| 0x14 | 对话 | 单人对话 |
| 0x15 | 双人对话 | 两人对话 |
| 0x30 | 人物显示 | 显示人物 |
| 0x31 | 人物消失 | 隐藏人物 |
| 0x32 | 人物移动 | 移动人物 |
| 0x69 | 旁白 | 旁白叙述 |
| ... | ... | ... |

完整列表参见代码中的 `EVENT_COMMANDS` 字典。

## 快捷键

| 快捷键 | 功能 |
|--------|------|
| Ctrl+O | 打开文件 |
| Ctrl+S | 保存文件 |
| Ctrl+N | 添加事件 |
| Delete | 删除事件 |
| Ctrl+Z | 撤销 |
| Ctrl+Y | 重做 |
| Ctrl+F | 搜索事件 |
| F3 | 查找下一个 |

## 文件结构

```
ccz_eex_parse/
├── eex_editor.py      # 编辑器主程序
├── run_editor.py      # 启动脚本
├── init.ui            # 主界面 UI 文件
├── init_ui.py         # 生成的 UI 代码
├── search.ui          # 搜索界面 UI 文件
├── search_ui.py       # 生成的 UI 代码
├── parse_eex.py       # EEX 解析器
├── until.py           # 工具函数
├── requirements.txt   # Python 依赖
└── venv/              # 虚拟环境
```

## 注意事项

1. **EEX 二进制格式**：统一工作台使用 AST 写回器，可以重建并回读旧引擎
   `0x00..0x6B` 命令。旧的字典式编辑窗口仍保留其原有兼容行为。

2. **编码**: 游戏文本使用 GBK 编码，JSON 导出使用 UTF-8 编码。

3. **安全保存**：统一工作台默认不修改源游戏目录。保存写入项目覆盖层；
   发布时请选择单独的测试游戏副本。

## 已知问题

- 事件 VM 不会猜测依赖实时部队状态的测试命令，遇到未模拟条件会暂停；
- 静态战斗估算尚未覆盖全部装备、特技及原游戏运行态分支；
- Python 插件停用会撤销全部扩展注册，彻底释放已导入模块需重启编辑器。

## 开发

使用 PyCharm 或其他 IDE 打开项目即可开发。

确保使用虚拟环境：
```bash
source venv/bin/activate  # macOS/Linux
```

## 许可证

本工具仅供学习和研究使用。

## 致谢

感谢所有为曹操传修改工具做出贡献的开发者。
