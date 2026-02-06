# xfa/fde 源码分析

## 模块概述

`fde` (FoxIt Device Extension) 模块提供设备扩展功能，主要包括文本编辑引擎和文本输出功能，为 XFA 表单的文本处理提供底层支持。

## 目录结构

```
xfa/fde/
├── BUILD.gn                          # 构建配置
│
├── 文本编辑
│   ├── cfde_texteditengine.cpp/h     # 文本编辑引擎
│   └── cfde_texteditengine_unittest.cpp  # 单元测试
│
├── 文本输出
│   ├── cfde_textout.cpp/h            # 文本输出
│   └── cfde_textout_unittest.cpp     # 单元测试
│
├── 分词
│   ├── cfde_wordbreak_data.cpp/h     # 分词数据
│   └── cfde_data.h                   # 数据定义
```

## 核心类分析

### 1. CFDE_TextEditEngine 类

**位置**: cfde_texteditengine.h/cpp

文本编辑引擎，提供完整的文本编辑功能。

#### 核心方法

| 方法类别 | 方法 | 功能 |
|----------|------|------|
| 文本操作 | `SetText` | 设置文本内容 |
| | `GetText` | 获取文本内容 |
| | `GetLength` | 获取文本长度 |
| | `Insert` | 插入文本 |
| | `Delete` | 删除文本 |
| | `Clear` | 清空文本 |
| | `Replace` | 替换文本 |
| 选择操作 | `SelectAll` | 全选 |
| | `ClearSelection` | 清除选择 |
| | `SetSelection` | 设置选择范围 |
| | `GetSelection` | 获取选择范围 |
| | `HasSelection` | 是否有选择 |
| | `GetSelectedText` | 获取选中文本 |
| 撤销/重做 | `Undo` | 撤销 |
| | `Redo` | 重做 |
| | `CanUndo` | 能否撤销 |
| | `CanRedo` | 能否重做 |
| | `ClearUndoStack` | 清空撤销栈 |
| 光标操作 | `GetCaretRect` | 获取光标矩形 |
| | `SetCaretPosition` | 设置光标位置 |
| | `GetCaretPosition` | 获取光标位置 |
| | `MoveCaretLeft` | 光标左移 |
| | `MoveCaretRight` | 光标右移 |
| | `MoveCaretUp` | 光标上移 |
| | `MoveCaretDown` | 光标下移 |
| | `MoveCaretToLineStart` | 移到行首 |
| | `MoveCaretToLineEnd` | 移到行尾 |
| 布局 | `Layout` | 执行布局 |
| | `GetIndexAtPoint` | 获取点处的索引 |
| | `GetCharRects` | 获取字符矩形 |
| | `GetLineCount` | 获取行数 |
| | `GetLineInfo` | 获取行信息 |

#### 属性设置

| 属性 | 描述 |
|------|------|
| `SetFont` | 设置字体 |
| `SetFontSize` | 设置字号 |
| `SetFontColor` | 设置字体颜色 |
| `SetAlignment` | 设置对齐方式 |
| `SetLineSpace` | 设置行距 |
| `SetMaxEditOperations` | 设置最大操作数 |
| `SetHasCharLimit` | 设置字符限制 |
| `SetCharLimit` | 设置字符上限 |
| `SetCombText` | 设置梳式文本 |
| `SetTabWidth` | 设置 Tab 宽度 |
| `SetMultiLine` | 设置多行模式 |
| `SetAutoLineWrap` | 设置自动换行 |
| `SetReadOnly` | 设置只读 |
| `SetPassword` | 设置密码模式 |
| `SetPasswordCharacter` | 设置密码字符 |
| `SetRichText` | 设置富文本 |

#### 委托接口

`CFDE_TextEditEngine::Delegate` 接口用于通知外部变化：

| 方法 | 功能 |
|------|------|
| `OnTextWillChange` | 文本即将变更 |
| `OnTextChanged` | 文本已变更 |
| `OnSelChanged` | 选择已变更 |
| `OnCaretChanged` | 光标已变更 |
| `SetScrollOffset` | 设置滚动偏移 |

### 2. CFDE_TextOut 类

**位置**: cfde_textout.h/cpp

文本输出类，负责将文本渲染到设备。

#### 核心方法

| 方法 | 功能 |
|------|------|
| `SetFont` | 设置字体 |
| `SetFontSize` | 设置字号 |
| `SetTextColor` | 设置文本颜色 |
| `SetAlignment` | 设置对齐方式 |
| `SetLineSpace` | 设置行距 |
| `SetStyles` | 设置样式 |
| `SetMatrix` | 设置变换矩阵 |
| `CalcLogicSize` | 计算逻辑尺寸 |
| `CalcSize` | 计算尺寸 |
| `DrawLogicText` | 绘制逻辑文本 |
| `DrawText` | 绘制文本 |
| `GetCharRects` | 获取字符矩形 |

#### 对齐方式

| 对齐 | 描述 |
|------|------|
| `Left` | 左对齐 |
| `Center` | 居中 |
| `Right` | 右对齐 |
| `Top` | 顶部对齐 |
| `Middle` | 垂直居中 |
| `Bottom` | 底部对齐 |

#### 样式选项

| 样式 | 描述 |
|------|------|
| `SingleLine` | 单行 |
| `MultiLine` | 多行 |
| `LastLineHeight` | 使用最后一行高度 |

### 3. 分词数据

**位置**: cfde_wordbreak_data.h/cpp

提供文本分词所需的数据表。

| 功能 | 描述 |
|------|------|
| 字符分类 | 确定字符的分类 |
| 断词规则 | 定义可断词位置 |
| 语言特定 | 支持不同语言的规则 |

## 文本编辑流程

### 插入文本

```
Insert(position, text)
    │
    ├── 验证输入
    │   ├── 检查只读状态
    │   └── 检查字符限制
    │
    ├── 调用委托 OnTextWillChange
    │   └── 允许修改或取消
    │
    ├── 执行插入
    │   ├── 更新内部缓冲区
    │   └── 添加撤销记录
    │
    ├── 重新布局
    │
    └── 调用委托 OnTextChanged
```

### 删除文本

```
Delete(start, length)
    │
    ├── 验证输入
    │
    ├── 调用委托 OnTextWillChange
    │
    ├── 执行删除
    │   ├── 保存删除内容用于撤销
    │   └── 更新内部缓冲区
    │
    ├── 重新布局
    │
    └── 调用委托 OnTextChanged
```

### 撤销/重做

```
撤销栈                     重做栈
┌─────────┐              ┌─────────┐
│ 操作 N  │  ← Undo →    │ 操作 N  │
├─────────┤              ├─────────┤
│ 操作 N-1│  ← Undo →    │ 操作 N-1│
├─────────┤              └─────────┘
│  ...    │              ← Redo ──
└─────────┘
```

## 文本布局

### 布局流程

```
Layout()
    │
    ├── 初始化断行器 (CFGAS_TxtBreak)
    │
    ├── 遍历文本
    │   ├── 添加字符到断行器
    │   ├── 检查断行点
    │   └── 记录行信息
    │
    ├── 生成行数据
    │   ├── 行起始位置
    │   ├── 行长度
    │   └── 行高度
    │
    └── 计算字符矩形
```

### 字符矩形计算

```
GetCharRects()
    │
    ├── 遍历每行
    │
    ├── 遍历行中的每个字符
    │   ├── 获取字符宽度
    │   ├── 计算 X 位置
    │   └── 计算 Y 位置
    │
    └── 返回矩形数组
```

## 光标管理

### 光标位置计算

```
GetCaretRect()
    │
    ├── 获取光标位置的字符索引
    │
    ├── 确定所在行
    │
    ├── 计算 X 坐标
    │   └── 累加前面字符的宽度
    │
    ├── 计算 Y 坐标
    │   └── 累加前面行的高度
    │
    └── 返回光标矩形
```

### 光标移动

| 操作 | 逻辑 |
|------|------|
| 左移 | 索引减 1，处理行首 |
| 右移 | 索引加 1，处理行尾 |
| 上移 | 同列上一行，或行首 |
| 下移 | 同列下一行，或行尾 |
| 行首 | 移到当前行开始 |
| 行尾 | 移到当前行结束 |
| 文档首 | 移到索引 0 |
| 文档尾 | 移到最后 |

## 密码模式

密码模式下的处理：
- 存储实际文本
- 显示时替换为密码字符
- 布局使用密码字符的宽度
- 禁用某些操作（如复制）

## 依赖关系

```
fde/
    ├── fgas/            (文本断行、字体)
    ├── core/fxge/       (图形渲染)
    └── core/fxcrt/      (基础设施)

被依赖:
    xfa/fwl/ → fde/  (编辑控件使用)
```

## 与 FWL 的协作

FWL 编辑控件使用 CFDE_TextEditEngine：

```
CFWL_Edit
    │
    ├── 包含 CFDE_TextEditEngine
    │
    ├── 实现 Delegate 接口
    │   ├── 接收变更通知
    │   └── 请求重绘
    │
    ├── 转发用户输入
    │   ├── 键盘输入 → Insert/Delete
    │   └── 鼠标点击 → SetCaretPosition
    │
    └── 渲染文本
        └── 使用引擎的布局信息
```

## 性能优化

- 增量布局：只重新布局变化的部分
- 脏区域跟踪：只重绘需要更新的区域
- 字符矩形缓存：避免重复计算
