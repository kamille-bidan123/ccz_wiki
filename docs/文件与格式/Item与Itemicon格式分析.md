# Item.e5 与 Itemicon.dll 格式分析

## 结论

原版目录同时保存两套物品图片：

- `Item.e5`：104 张 32×32、8-bit 索引图，与 `Data.e5` 的物品图标编号对应。
- `Itemicon.dll`：105 组 Windows 位图，每组各有一张 16×16 小图和一张
  32×32 大图；最后一组是空白保留槽。

两者不能被当作完全独立的两套编号：

```text
Data.e5 item.icon = n
Item.e5 分块 = n
Itemicon 16x16 资源 ID = 100 + 2*n
Itemicon 32x32 资源 ID = 101 + 2*n
```

本目录的 `Data.e5` 有 104 个物品，`icon` 恰好为互不重复的 `0..103`。
字段本身仍然是图标索引，修改版可以让多个物品复用同一图标。

社区资料也区分了两种用途：`Item.e5` 图片会用于物品升级等游戏画面，
图鉴等小图需要修改 `Itemicon.dll`；传统修改方式是用 eXeScope 导入 DLL
位图。

- https://www.xycq.org.cn/forum/thread-55018-1-1.html
- https://xycq.org.cn/forum/viewthread.php?page=1&tid=70267

## Item.e5

### 容器与分块

`Item.e5` 是标准 `Ls12` 容器：

```text
分块数：104
每块解压长度：1024 字节
图像尺寸：32 * 32
像素布局：整图从上到下、从左到右逐行排列
```

像素地址：

```text
color_index = part[y * 32 + x]
```

`Ekd5.exe` 的读取代码按物品图标编号选择 Item 分块，固定复制 `0x400`
字节并以 32×32 绘制，和文件统计相符。

### 调色板与透明色

Item 分块不携带自己的调色板。104 张图片实际只使用索引 `0..173`，
共出现 156 个颜色索引，属于曹操传在 R/S 场景间共用的前 174 色。

- 索引 0 的颜色为 `(247, 0, 255)`，即原版洋红透明背景。
- `Itemicon.dll` 每张 DIB 自带 256 色 BGRA 调色板。
- DLL 调色板的前 174 色与 `Pmpalet.e5`、`Spalet.e5` 的固定色区一致。
- 因此解析 Item.e5 时可直接采用 Itemicon 的前 174 色，也可以采用任一
  原版 Pmpalet/Spalet 的固定色区。

## Itemicon.dll

### PE 资源结构

`Itemicon.dll` 是 PE32 DLL。资源树只包含 `RT_BITMAP`（类型 2）：

```text
位图资源数：210
资源 ID：100..309
资源语言 ID：1041（0x0411）
逻辑槽位：105（0..104）
```

槽位换算：

| 槽位 | 16×16 资源 | 32×32 资源 |
| ---: | ---: | ---: |
| 0 | 100 | 101 |
| 1 | 102 | 103 |
| ... | ... | ... |
| 103 | 306 | 307 |
| 104 | 308 | 309 |

`Ekd5.exe` 初始化 DLL 图标列表时也明确使用：

```text
起始资源 ID = 100
图标数 = 105
尺寸分支 = 16x16 / 32x32
```

### DIB 格式

每个资源是没有 BMP 文件头的 DIB：

```text
BITMAPINFOHEADER：40 字节
位深：8-bit
压缩：BI_RGB（0，不压缩）
调色板：256 * 4 字节，顺序 BGRA
像素行：4 字节对齐
存储方向：正高度 DIB，从下往上保存
```

原版前 104 组资源末尾各多两个 `00` 字节，保留解析时忽略即可。
第 104 槽的 16×16 和 32×32 图片全部为索引 0，是 Data 物品范围之外的
空白保留槽。

### 16×16 不是机械缩小

把 32×32 图片用四种奇偶采样方式缩为 16×16，最佳平均索引吻合率约为
59%，没有任何一张能完全吻合。这表明 DLL 中的小图是单独缩放、量化或
人工修整的资源，编辑器不应默认用隔点采样覆盖原小图。

## 两套 32×32 图片的差异

104 个有效槽中：

- 103 个 `Item.e5` 分块与 DLL 同槽 32×32 位图逐像素完全一致；
- 槽位 75“青囊书”不同。

本目录中：

- `Item.e5[75]` 是展开的书卷图案；
- `Itemicon.dll` 资源 251 是红色封皮书本图案。

因此这两套资源曾被分别更新，不能假定永远完全相同。物品编辑器应同时
显示并分别写回 Item、DLL 16×16、DLL 32×32 三张图片，并提示差异。

## 新旧引擎差异

较新的 6.6 系引擎资料声称可以把 `Itemicon.dll` 道具图标迁移到
`Item.e5` 并删除 DLL；这只说明新引擎存在另一套资源组织方式，不能用于
反推本地格式。本地旧 `Ekd5.exe` 的实际装载代码分别读取 `Item.e5`，
并以资源 ID 100 起、105 组、16×16/32×32 两种尺寸初始化
`Itemicon.dll`，所以当前工具必须为本地旧引擎保留双资源兼容。

- https://www.xycq.org.cn/forum/viewthread.php?authoruid=0&extra=&page=1&tid=310256

## 已实现工具

- `item_resources.py`
  - 读取 PE 资源目录；
  - 解码 8-bit DIB；
  - 读取 Item.e5；
  - 执行槽位、尺寸及跨文件一致性校验。
- `tools/export_item_resources.py`
  - 导出指定或全部物品的三套图片；
  - 生成 Item.e5 总览图；
  - 报告两套 32×32 图片的差异槽位。

示例：

```bash
./venv/bin/python tools/export_item_resources.py --analyze
./venv/bin/python tools/export_item_resources.py --item 75
./venv/bin/python tools/export_item_resources.py --all --contact-sheet
```
