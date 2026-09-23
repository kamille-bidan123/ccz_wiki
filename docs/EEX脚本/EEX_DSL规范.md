# EEX DSL 规范

## 目标

本规范定义一套面向人类编辑的 EEX 文本表示，用于：

- 结构化查看 `Scene / Section / ChildBlock`
- 批量编辑事件脚本
- 支持 `AST <-> DSL` 双向转换
- 为后续 DSL 编辑器、格式化器、校验器提供统一依据

本规范的设计原则：

1. 贴近 EEX 原始层级
2. 比 JSON 更易读
3. 尽量避免上下文敏感语法
4. 第一阶段不引入完整编程语言能力

---

## 范围

本规范描述的是 **EEX DSL MVP**，优先覆盖：

- `scene`
- `section`
- `test`
- `event`
- `child`
- `comment`
- 普通命令的命名参数表示

本规范暂不包含：

- `include`
- `macro`
- `template`
- `const`
- `label/goto` 语法糖
- 条件表达式语言扩展

这些能力可以在后续版本增加，但不属于第一版解析器必需功能。

---

## 词法规则

### 空白

- 空格、Tab、换行都可作为分隔符
- 换行对语义通常不敏感
- 推荐一条命令写在一行

### 注释

支持两种注释：

- 行注释：`# comment`
- 行尾注释：`dialogue msg="hi" # note`

注释不进入 AST。

注意：`comment "..."` 是 DSL 中的一个**语义节点**，会进入 AST，对应 EEX 中的内部消息；它与 `#` 注释不是一回事。

### 标识符

标识符用于：

- 关键字
- 命令名
- 参数名
- 枚举字面量

推荐规则：

```ebnf
IDENT := [A-Za-z_][A-Za-z0-9_]*
```

例如：

- `scene`
- `dialogue2`
- `people_id`
- `is_hide`
- `up`

### 整数

第一版只要求支持十进制整数：

```ebnf
INT := "-"? [0-9]+
```

可选扩展：

- 十六进制 `0x14`

但第一版 printer 推荐统一输出十进制。

### 布尔

```ebnf
BOOL := "true" | "false" | "unknow"
```

其中：

- `true` -> `True`
- `false` -> `False`
- `unknow` -> 特殊异常布尔值，对应 EEX 字节 `ff00`

当前样本中，布尔异常值已知且稳定的只有 `ff00`。  
因此 DSL 规范直接将它收编为正式字面量 `unknow`，不再要求依赖 `bool_raw` 才能表达。

### 字符串

使用双引号字符串：

```ebnf
STRING := '"' { CHAR | ESCAPE } '"'
```

至少支持这些转义：

- `\"`
- `\\`
- `\n`
- `\t`

printer 推荐始终输出双引号形式。

### 列表

```ebnf
LIST := "[" [ value { "," value } ] "]"
```

例如：

- `[]`
- `[14]`
- `[1, 2, 3]`

---

## 结构语法

### 顶层结构

```ebnf
file        := { scene }
scene       := "scene" INT block
section     := "section" INT block
block       := "{" { statement } "}"
```

### 语句

```ebnf
statement   := section
            | test_block
            | event_block
            | child_block
            | command_stmt
            | comment_stmt
```

### 结构块

```ebnf
test_block  := "test" block
event_block := "event" block
child_block := "child" block
```

约束：

- `scene` 下只能出现 `section`
- `section` 下必须出现且只允许出现一个 `test` 和一个 `event`
- `child` 下必须出现且只允许出现一个 `test` 和一个 `event`
- `test` / `event` 下可以出现 `command_stmt`、`comment_stmt`、`child_block`

### 命令语句

```ebnf
command_stmt := IDENT { argument }
argument     := IDENT "=" value
comment_stmt := "comment" STRING
value        := INT | STRING | BOOL | LIST | IDENT
```

例如：

```txt
dialogue msg="曹操：全军前进。"
move people_id=32 x=10 y=12 dire=up
var_test false_var=[14]
comment "第3回合触发"
```

---

## AST 映射规则

### Scene

```txt
scene 0 { ... }
```

映射为：

```python
Scene(index=0, ...)
```

### Section

```txt
section 0 {
  test { ... }
  event { ... }
}
```

映射为：

```python
Section(index=0, test=[...], event=[...])
```

### ChildBlock

```txt
child {
  test { ... }
  event { ... }
}
```

映射为：

```python
ChildBlock(test=[...], event=[...])
```

### Comment

```txt
comment "第3回合触发"
```

映射为：

```python
Comment(text="第3回合触发")
```

### Command

```txt
dialogue msg="曹操：前进。"
```

映射为：

```python
Command(
    opcode=0x14,
    mnemonic="dialogue",
    args={"msg": "曹操：前进。"},
)
```

---

## 命令与参数规范

### 命令名

- 命令名必须使用 schema 中定义的 `mnemonic`
- DSL parser 应通过 `mnemonic -> opcode` 反查命令
- 不应在 DSL 中直接暴露 opcode 作为主写法

例如：

- `dialogue`
- `dialogue2`
- `move`
- `team_set`

### 参数名

- 参数名必须使用 AST / schema 的统一字段名
- 不应混用历史 JSON 字段别名

例如统一用：

- `people_id`
- `people_id2`
- `is_hide`
- `msg`

### 参数顺序

DSL parser 不依赖参数顺序。  
但 DSL printer 应按 schema 的字段顺序输出，以确保稳定 diff。

### 未写参数

- 允许省略有默认值的参数
- 不允许省略 required 参数

例如：

```txt
move people_id=32 x=10 y=12
```

如果 `dire` 在 schema 中有默认值，则合法。

### 枚举参数

枚举在 DSL 中优先使用标识符字面量，而不是裸整数。

例如：

- `dire=up`
- `operator=ge`

如果 parser 遇到未知枚举名，应报错。

---

## 字段级保真与 DSL

DSL 是“结构化文本编辑层”，不是“原始字节编辑层”。

因此：

- `*_raw` 字段不在 DSL 中暴露
- DSL 只读写结构化字段

例如 AST 中可能有：

```python
{"bool": "unknow"}
```

printer 可输出为：

```txt
team_set people_id=16 bool=unknow people_lv=0
```

如果 DSL parser 再读回：

- 生成 `bool="unknow"`
- writer 会直接编码为 `ff00`

也就是说：

- DSL 导出保留语义
- DSL 重新导入后默认走结构化重编码
- 对于 `ff00` 这一已知异常值，不再需要依赖字段级 `bool_raw`

这与 GUI 的局部字段编辑不同，是有意的设计。

---

## 语义约束

DSL parser / validator 应至少检查：

1. `scene` 必须包含 `section`
2. `section` 必须包含 `test` 和 `event`
3. `child` 必须包含 `test` 和 `event`
4. `test` / `event` 中命令必须属于 schema 允许的范围
5. required 字段必须提供
6. 字段类型必须匹配
7. 枚举值必须合法

可选进一步检查：

1. 某些命令只能出现在 `test`
2. 某些命令只能出现在 `event`
3. 人物 ID / 地图 ID / 道具 ID 合法性
4. 列表长度约束

---

## 格式化规范

为了让 diff 稳定，printer 推荐输出为统一风格：

### 缩进

- 每层 2 个空格

### 大括号

- 块起始与声明同一行
- 块结束单独一行

### 一条命令一行

例如：

```txt
event {
  dialogue msg="曹操：前进。"
  move people_id=32 x=10 y=12 dire=up
}
```

### 参数格式

- 使用 `name=value`
- 参数之间以单空格分隔
- 列表使用 `, ` 分隔

### 字段顺序

- 按 schema 顺序输出
- 未知字段放到末尾，按字段名排序

---

## 错误报告规范

DSL parser 应报告：

- 行号
- 列号
- 错误类型
- 期望内容

例如：

```txt
Line 12, Column 18: expected '=' after argument name
```

或：

```txt
Line 24, Column 7: command 'dialogue' is not allowed in test block
```

---

## MVP 支持范围

第一版建议只要求：

1. `AST -> DSL` 导出
2. `DSL -> AST` 导入
3. `scene / section / test / event / child / comment`
4. 常用命令的命名参数写法
5. 基础类型：`int / bool / string / list / ident`
6. 基础错误定位

不要求第一版支持：

1. 宏
2. include
3. 常量系统
4. 代码补全
5. 上下文相关自动推断

---

## 示例

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

      child {
        test {
          people_near_test people_id=32 people_id2=146 is_near_atk=false
        }
        event {
          dialogue2 people_id=32 people_id2=146 msg="敌将已逼近！"
          play_sound se_id=23 count=1
        }
      }
    }
  }
}
```

---

## 实现建议

建议拆成四层：

1. `DSLTokenizer`
2. `DSLParser`
3. `DSLPrinter`
4. `DSLValidator`

调用链建议为：

```text
EEX -> AST -> DSLPrinter -> text
text -> DSLParser -> AST -> DSLValidator
```

这样：

- GUI 可直接编辑 AST
- DSL 编辑器可编辑 text
- 二者共用同一 AST 和 schema
