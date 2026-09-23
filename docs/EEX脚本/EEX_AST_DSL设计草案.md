# EEX AST / DSL 设计草案

## 目标

为当前 EEX 编辑器补齐一层稳定的中间表示，使以下几种编辑方式共用同一套核心模型：

- EEX 二进制读写
- GUI 三栏编辑器
- 文本脚本编辑
- 校验、重构、批量改写

推荐总架构：

```text
EEX Binary <-> AST <-> GUI
EEX Binary <-> AST <-> DSL
```

这里：

- **AST** 是程序内部的结构化表示
- **DSL** 是给人类编写和审阅的文本表示

AST 是核心，DSL 和 GUI 都应建立在 AST 之上，而不是分别直接操作 EEX 二进制。

---

## 当前编辑器现状

当前 `eex_editor.py` 的界面骨架已经接近合理：

- 左侧：Scene / Section 导航
- 中间：事件列表
- 右侧：属性编辑

但当前数据结构仍有明显限制：

1. `Section` 的 `test` 段和 `event` 段没有被建模成独立结构，而是混在一个列表里，通过特殊标记 `_section_end` 分隔。
2. 子事件块只是裸 `list`，不是明确节点类型。
3. 属性编辑器依赖“常见字段表”，属于弱语义强类型，而不是基于命令 schema 的强语义编辑。
4. GUI、解析器、写回器之间共享的是半结构化 `list/dict`，不利于校验、扩展和脚本化。

因此，当前问题的根源不是“三栏界面是否成立”，而是：

- 缺少统一 AST
- 缺少命令 schema
- 缺少适合人写的 DSL

---

## 设计原则

### 1. AST 先行

先把 EEX 解析结果变成稳定 AST，再让 GUI 和 DSL 建立在 AST 上。

### 2. 结构显式化

不要再依赖：

- `_section_end`
- `isinstance(node, list)` 推断子块
- 不同命令临时约定字段

而应将这些结构显式建模。

### 3. 保真优先

即使某些命令还未完全理解，也要能保留下来并重新写回，避免“读得进、写不回”。

### 4. 渐进增强

先支持 AST + 常用命令 schema + DSL 导出，再逐步增加 DSL 导入、静态检查、宏系统。

---

## AST 总体结构

建议的 AST 顶层结构：

```python
from dataclasses import dataclass, field
from typing import Any


@dataclass
class EexFile:
    scenes: list["Scene"] = field(default_factory=list)


@dataclass
class Scene:
    index: int
    sections: list["Section"] = field(default_factory=list)


@dataclass
class Section:
    index: int
    test: list["Node"] = field(default_factory=list)
    event: list["Node"] = field(default_factory=list)


class Node:
    pass


@dataclass
class Command(Node):
    opcode: int
    mnemonic: str
    args: dict[str, Any] = field(default_factory=dict)
    raw_meta: dict[str, Any] = field(default_factory=dict)


@dataclass
class ChildBlock(Node):
    test: list["Node"] = field(default_factory=list)
    event: list["Node"] = field(default_factory=list)


@dataclass
class Comment(Node):
    text: str


@dataclass
class UnknownCommand(Node):
    opcode: int
    payload: bytes
```

---

## AST 节点说明

### EexFile

代表整个 `.eex` 文件。

- 包含多个 `Scene`

### Scene

对应 EEX 文件中的一个场景。

- `index`: 场景索引
- `sections`: 当前场景下的章节列表

### Section

对应文档中的一个 Section。

根据 `docs/EEX二进制格式说明.md`，每个 Section 天然分为两段：

- `test`: 测试条件段
- `event`: 事件执行段

这是 AST 相对于当前实现最重要的结构修正。

### Command

表示普通事件命令。

- `opcode`: 原始命令码
- `mnemonic`: DSL 和 GUI 使用的人类可读助记名
- `args`: 参数字典
- `raw_meta`: 为保真和调试保留的额外元信息

当前项目中的 AST 实现已经进一步细化为“字段级保真”：

- 普通结构化字段保存在 `args[name]`
- 若该字段在原文件里存在需要保真的原始字节，则保存在 `args[f"{name}_raw"]`

例如：

```python
Command(
    opcode=0x3B,
    mnemonic="team_set",
    args={
        "people_id": 16,
        "bool": "unknow",
        "people_lv": 0,
    },
)
```

这里：

- `bool` 是语义值
- 在当前 DSL 规范中，`"unknow"` 直接表示 EEX 中的 `ff00`

写回时：

- `True -> 0100`
- `False -> 0000`
- `"unknow" -> ff00`

因此 `ff00` 已不再需要依赖字段级 `bool_raw` 才能表达。

### ChildBlock

对应 `0x01` 子事件设定。

它本身不是普通命令，而是一个嵌套结构节点，也应和 `Section` 一样拆分为：

- `test`
- `event`

### Comment

建议将 `0x02 内部消息` 在 AST 层视作注释型节点，便于：

- GUI 中以辅助说明显示
- DSL 中自然写成 `comment "..."`

如果需要严格保留原始语义，也可同时保留原 `opcode` 于 `raw_meta`。

当前实现中，`Comment` 节点使用：

- `text`
- `text_raw`

来实现同样的字段级保真。

### UnknownCommand

用于兜底。

当某条命令尚未完成语义解析时：

- 不阻塞整个文件解析
- 不直接丢弃
- 保留原始 payload，尽量支持回写

---

## 命令 Schema 层

为了让 GUI、DSL、校验器复用一套规则，建议在 AST 之上再定义命令 schema 表。

示例：

```python
COMMAND_SPECS = {
    0x14: {
        "mnemonic": "dialogue",
        "title": "对话",
        "fields": [
            {"name": "msg", "type": "string", "required": True},
        ],
    },
    0x15: {
        "mnemonic": "dialogue2",
        "title": "双人对话",
        "fields": [
            {"name": "people_id", "type": "person", "required": True},
            {"name": "people_id2", "type": "person", "required": True},
            {"name": "msg", "type": "string", "required": True},
        ],
    },
    0x32: {
        "mnemonic": "move",
        "title": "人物移动",
        "fields": [
            {"name": "is_battle", "type": "bool", "default": False},
            {"name": "people_id", "type": "person", "required": True},
            {"name": "battle_id", "type": "int", "default": 0},
            {"name": "x", "type": "int", "required": True},
            {"name": "y", "type": "int", "required": True},
            {"name": "dire", "type": "direction", "default": "down"},
        ],
    },
}
```

---

## 字段级保真约定

为了避免重新编码时丢失一些“语义上可理解、字节上不标准”的数据，当前 AST / Writer 采用字段级保真，而不是节点级原样回放。

### 原则

1. 不使用整条命令的 `raw_bytes` 直接写回
2. 优先保留“字段级”的原始字节
3. 只有该字段没有被修改时，才优先使用 `*_raw`
4. 一旦用户明确修改了该字段，就应清理对应的 `*_raw`

### 当前已覆盖的典型字段

- `bool_raw`
- `msg_raw`
- `dialog_raw`
- `text_raw`

说明：

- `bool_raw` 仍作为兜底保真机制保留
- 但对于目前已收敛的异常布尔值 `ff00`，DSL 与 AST 统一使用 `"unknow"` 表达
- 因此常规 DSL 路径下，不再需要为 `ff00` 特地依赖 `bool_raw`

### 为什么不用节点级 raw 回放

节点级原样回放虽然最容易实现无损 roundtrip，但会带来几个问题：

- 无法知道用户究竟改了哪个字段
- 很难做 GUI 局部编辑
- 难以支持 DSL 导入后结构化重编码
- 会把“AST 可编辑”退化回“AST 只是包装过的字节块”

因此，更合适的策略是：

- 节点级 `raw_bytes` 只用于调试和定位
- 写回保真以字段级 `*_raw` 为准

### 编辑规则

如果 GUI / DSL / 批处理代码修改了某个字段：

- 改 `msg` 时，应清理 `msg_raw`
- 改 `dialog` 时，应清理 `dialog_raw`
- 改 `bool` 时，应清理 `bool_raw`
- 如果要显式表达 `ff00`，应把布尔值设为 `"unknow"`
- 改 `Comment.text` 时，应清理 `text_raw`

项目中已经补了一个辅助模块：

- [eex_ast_edit.py](https://github.com/lometsj/ccz_eex_parse/blob/master/eex_ast_edit.py#L1)

用于统一处理这些修改行为，避免出现：

- 结构化值已改
- 对应 `*_raw` 还残留旧字节

这样的状态不一致问题。

### 推荐修改入口

推荐优先使用：

- `set_command_arg(node, name, value)`
- `set_bool_arg(node, name, value)`
- `set_text_arg(node, name, value)`
- `set_comment_text(node, value)`
- `clear_all_raw_args(node)`

这样 GUI、DSL、批量脚本都能遵守同一套约定。

### Schema 的职责

1. 为 opcode 提供统一命名
2. 定义字段列表
3. 定义字段类型
4. 定义默认值
5. 指定字段是否必填
6. 提供 GUI 渲染方式
7. 提供 DSL 输出顺序
8. 提供静态校验规则

### 推荐字段类型

- `int`
- `bool`
- `string`
- `list[int]`
- `direction`
- `operator`
- `person`
- `item`
- `battle_map`
- `music`
- `sound`
- `video`
- `enum:<name>`

### 推荐扩展字段

除了基础字段，还可以给 schema 加这些元信息：

- `allowed_in`: `["test"]`, `["event"]`, `["test", "event"]`
- `summary_fields`: 用于中间列表摘要显示
- `group`: `dialogue`, `battle`, `map`, `var`, `audio`
- `lossless`: 当前实现是否支持无损回写

---

## DSL 设计目标

DSL 不是为了替代 GUI，而是为了补足 GUI 不擅长的工作：

- 批量改写
- 快速录入
- 模板复用
- 版本管理和 diff
- 搜索替换
- 审查脚本结构

因此 DSL 应满足：

1. 尽量贴近 EEX 结构
2. 比 JSON 更简洁
3. 比通用编程语言更易学
4. 容易格式化和解析

详细 DSL 语法、约束、字段规则、格式化规范与错误报告规则，已经单独整理到：

- [EEX_DSL规范.md](EEX_DSL规范.md)

这里保留摘要：

- 顶层结构：`scene -> section -> test/event`
- 嵌套结构：`child { test { ... } event { ... } }`
- 普通命令：`mnemonic name=value name=value`
- 语义注释：`comment "..."`，进入 AST
- 普通源码注释：`# ...`，不进入 AST
- 布尔文本：`true / false / unknow`
- 参数顺序：parser 不敏感，printer 按 schema 顺序输出
- DSL 不暴露 `*_raw`；重新导入后按结构化值重编码

示例：

```txt
scene 0 {
  section 0 {
    test {
      comment "第3回合触发"
      round_num_test round=3 operator=ge
      var_test false_var=[14]
    }

    event {
      event_name msg="颖川之战"
      dialogue msg="曹操：全军前进。"
      move people_id=32 x=10 y=12 dire=up
    }
  }
}
```

---

## DSL 与 AST 的关系

两者不是竞争关系，而是“外部表示”和“内部表示”的关系。

### 外部表示：DSL

面向人类：

- 易读
- 易写
- 易 diff
- 易模板化

### 内部表示：AST

面向程序：

- 易校验
- 易在 GUI 中编辑
- 易写回二进制
- 易做自动化变换

推荐链路：

```text
EEX Binary -> Parser -> AST -> GUI
                        -> DSL Printer -> DSL Text

DSL Text -> DSL Parser -> AST -> Writer -> EEX Binary
```

---

## 从当前实现到目标架构的映射

### 当前实现

当前结构本质更接近：

```python
[
  [
    {"type": 0x02, "msg": ...},
    {"type": 0x00, "_section_end": True},
    {"type": 0x14, "msg": ...},
    [
      {"type": 0x02, "msg": ...},
      {"type": 0x14, "msg": ...},
    ]
  ]
]
```

问题：

- `Section` 的两段被混在同一个 list 中
- `ChildBlock` 只是裸 list
- 需要借助 `_section_end` 才能知道边界

### 目标 AST

应变成：

```python
EexFile(
    scenes=[
        Scene(
            index=0,
            sections=[
                Section(
                    index=0,
                    test=[
                        Comment("第3回合触发"),
                        Command(opcode=0x05, mnemonic="var_test", args={"false_var": [14]}),
                    ],
                    event=[
                        Command(opcode=0x14, mnemonic="dialogue", args={"msg": "曹操：前进"}),
                        ChildBlock(
                            test=[],
                            event=[
                                Command(opcode=0x15, mnemonic="dialogue2", args={...}),
                            ],
                        ),
                    ],
                )
            ],
        )
    ]
)
```

---

## GUI 如何接入 AST

现有三栏 GUI 可以保留，只需要把编辑对象从 `list/dict` 替换为 AST。

### 左侧

显示：

- `Scene`
- `Section`

### 中间

显示：

- `Section.test`
- `Section.event`
- `ChildBlock`

建议中间列表明确分区，而不是混在一起：

```text
Section 0
├─ Test
│  ├─ [00] var_test ...
│  └─ [01] round_num_test ...
└─ Event
   ├─ [00] dialogue ...
   ├─ [01] move ...
   └─ [02] child ...
```

### 右侧

根据 `Command` 对应 schema 渲染真正强语义表单：

- `direction` -> 下拉框
- `person` -> 人物选择器
- `operator` -> 运算符枚举
- `list[int]` -> 列表编辑器
- `string` -> 单行/多行文本框

这样当前 `common_fields` 式的通用表单可以逐步被 schema 驱动表单替代。

---

## 解析与写回策略

### 解析器

建议将解析器改为分层输出：

1. 二进制层：读 opcode 和原始参数
2. 语义层：映射为 `Command / ChildBlock / Comment / UnknownCommand`
3. AST 层：组织成 `Scene / Section`

### 写回器

写回器只接受 AST，不再接受半结构化 JSON。

写回过程：

1. 写文件头
2. 写 Scene 偏移表
3. 写每个 Scene
4. 写每个 Section
5. 对 `Section.test` 和 `Section.event` 分段编码
6. 对 `ChildBlock` 递归编码

这样可以自然替代当前依赖 `_section_end` 的实现。

---

## 保真策略

如果目标是安全编辑原版脚本，必须考虑“未完全理解命令”的处理方式。

推荐分级：

### Level 1: 结构保真

至少保证：

- Scene 数量不变
- Section 数量不变
- test/event 分段不乱

### Level 2: 已知命令语义保真

对已建模命令，保证：

- 参数解析正确
- 参数写回正确

### Level 3: 未知命令原样保真

对未知命令：

- 保存原始 payload
- 不因 GUI/DSL 导出而丢失

如果暂时做不到 Level 3，也应在系统中明确标记“该文件不是无损回写”。

---

## 静态校验建议

有了 AST 和 schema 后，可以自然加入校验器。

### 结构校验

- `Section` 必须包含 `test` 和 `event`
- `ChildBlock` 必须合法结束
- test 段和 event 段不能混淆

### 命令位置校验

- 条件命令应出现在 `test`
- 执行动作应出现在 `event`

### 参数校验

- 必填参数缺失
- 枚举值非法
- 坐标越界
- 人物 ID 不存在

### 交叉引用校验

- 脚本跳转目标不存在
- 地图资源 ID 无效
- 音视频资源未找到

---

## DSL 后续扩展方向

第一版 DSL 不建议做成完整编程语言。

推荐只加入低风险高收益能力：

- `const`
- `alias`
- `include`
- `template`
- `macro`

不建议一开始就加入：

- 通用循环
- 自定义函数
- 动态表达式解释器
- 复杂模块系统

因为 EEX 本质更像事件指令描述，而不是通用程序。

### 可接受的后续语法示例

```txt
const BOSS = 146

template warn_dialog(speaker, target, text) {
  dialogue2 people_id=speaker people_id2=target msg=text
  play_sound se_id=23 count=1
}
```

这类能力对剧本批量编写已经非常有帮助。

---

## 推荐实现顺序

### 第一阶段：重建中间层

1. 定义 AST 类
2. 定义 `COMMAND_SPECS`
3. 改 `parse_eex.py` 输出 AST
4. 改 `write_eex.py` 从 AST 写回

### 第二阶段：GUI 对接 AST

1. 左侧导航绑定 `Scene/Section`
2. 中间列表显示 `test/event/child`
3. 右侧属性面板改为 schema 驱动

### 第三阶段：只读 DSL

1. 实现 `AST -> DSL`
2. 在 GUI 中增加脚本预览视图
3. 支持导出 `.eexdsl` 或 `.txt`

### 第四阶段：可编辑 DSL

1. 实现 `DSL -> AST`
2. 加入语法错误提示
3. 支持脚本导入和保存

### 第五阶段：增强能力

1. 静态校验
2. 批量重构工具
3. 模板/宏
4. 资源引用检查

---

## MVP 范围建议

为了尽快落地，建议首版只覆盖：

- AST 核心结构
- 常用命令 schema
- GUI 基于 AST 的浏览与编辑
- `AST -> DSL` 导出

不必首版就实现：

- 全量命令 schema
- 无损未知命令回写
- DSL 宏系统
- 完整脚本导入

只要第一阶段做对，后面都可以自然扩展。

---

## 一句话结论

对于这个项目，最合适的长期架构不是“树控件 vs 文本语言”的二选一，而是：

- 用 **AST** 作为唯一真实数据模型
- 用 **GUI** 提供可视化点改
- 用 **DSL** 提供高效率文本编辑

三者共同服务于 EEX 的读写、校验和自动化处理。
