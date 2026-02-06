# fpdfsdk/formfiller 源码分析

## 模块概述

`formfiller` 模块是表单填充子系统，处理用户与 PDF 表单控件的交互。它管理各种类型的表单字段（文本框、复选框、下拉列表等）的用户输入和显示。

## 目录结构

```
fpdfsdk/formfiller/
├── BUILD.gn                          # 构建配置
│
├── 核心类
│   ├── cffl_interactiveformfiller.cpp/h  # 交互式表单填充器
│   ├── cffl_formfield.cpp/h          # 表单字段基类
│   └── cffl_fieldaction.cpp/h        # 字段动作
│
├── 字段类型实现
│   ├── cffl_textfield.cpp/h          # 文本字段
│   ├── cffl_checkbox.cpp/h           # 复选框
│   ├── cffl_radiobutton.cpp/h        # 单选按钮
│   ├── cffl_combobox.cpp/h           # 下拉框
│   ├── cffl_listbox.cpp/h            # 列表框
│   ├── cffl_pushbutton.cpp/h         # 按钮
│   └── cffl_button.cpp/h             # 按钮基类
│
├── 辅助类
│   ├── cffl_textobject.cpp/h         # 文本对象
│   └── cffl_perwindowdata.cpp/h      # 每窗口数据
│
└── 测试
    └── cffl_combobox_embeddertest.cpp # 下拉框测试
```

## 核心类分析

### 1. CFFL_InteractiveFormFiller 类

**位置**: cffl_interactiveformfiller.h/cpp

表单填充器的主类，协调所有表单字段的交互。

| 方法 | 功能 |
|------|------|
| `OnLButtonDown` | 处理鼠标左键按下 |
| `OnLButtonUp` | 处理鼠标左键释放 |
| `OnMouseMove` | 处理鼠标移动 |
| `OnChar` | 处理字符输入 |
| `OnKeyDown` | 处理按键按下 |
| `OnSetFocus` | 处理获得焦点 |
| `OnKillFocus` | 处理失去焦点 |
| `GetFormFieldAtPoint` | 获取点处的表单字段 |
| `CommitData` | 提交数据 |
| `ResetPWLWindow` | 重置 PWL 窗口 |

### 2. CFFL_FormField 类

**位置**: cffl_formfield.h/cpp

表单字段的抽象基类，所有字段类型都继承自它。

| 方法 | 功能 |
|------|------|
| `GetPWLWindow` | 获取 PWL 窗口 |
| `CreatePWLWindow` | 创建 PWL 窗口 |
| `DestroyPWLWindow` | 销毁 PWL 窗口 |
| `OnDraw` | 绘制字段 |
| `OnMouseEnter` | 鼠标进入 |
| `OnMouseExit` | 鼠标离开 |
| `OnLButtonDown` | 鼠标按下 |
| `OnLButtonUp` | 鼠标释放 |
| `OnChar` | 字符输入 |
| `CommitData` | 提交数据 |
| `GetValue` | 获取值 |
| `SetValue` | 设置值 |

### 3. CFFL_TextField 类

**位置**: cffl_textfield.h/cpp

文本字段实现。

| 特性 | 支持 |
|------|------|
| 单行文本 | ✓ |
| 多行文本 | ✓ |
| 密码模式 | ✓ |
| 富文本 | ✓ |
| 最大长度 | ✓ |
| 对齐方式 | ✓ |

| 方法 | 功能 |
|------|------|
| `GetText` | 获取文本 |
| `SetText` | 设置文本 |
| `GetSelectedText` | 获取选中文本 |
| `ReplaceSelection` | 替换选中 |

### 4. CFFL_CheckBox 类

**位置**: cffl_checkbox.h/cpp

复选框实现。

| 方法 | 功能 |
|------|------|
| `IsChecked` | 是否选中 |
| `SetCheck` | 设置选中状态 |
| `OnLButtonUp` | 切换状态 |

### 5. CFFL_RadioButton 类

**位置**: cffl_radiobutton.h/cpp

单选按钮实现。

| 特性 | 描述 |
|------|------|
| 互斥选择 | 同组只能选一个 |
| 组管理 | 自动处理组内状态 |

### 6. CFFL_ComboBox 类

**位置**: cffl_combobox.h/cpp

下拉框实现。

| 方法 | 功能 |
|------|------|
| `GetCurSel` | 获取当前选择 |
| `SetCurSel` | 设置当前选择 |
| `GetText` | 获取文本 |
| `SetText` | 设置文本 |
| `GetEditText` | 获取编辑文本 |

### 7. CFFL_ListBox 类

**位置**: cffl_listbox.h/cpp

列表框实现。

| 方法 | 功能 |
|------|------|
| `GetTopVisibleIndex` | 获取顶部可见索引 |
| `GetSelectedIndices` | 获取选中索引 |
| `IsMultipleSel` | 是否多选 |

### 8. CFFL_PushButton 类

**位置**: cffl_pushbutton.h/cpp

按钮实现。

| 方法 | 功能 |
|------|------|
| `OnMouseEnter` | 高亮效果 |
| `OnMouseExit` | 移除高亮 |
| `OnLButtonUp` | 触发动作 |

### 9. CFFL_FieldAction 类

**位置**: cffl_fieldaction.h/cpp

字段动作参数，用于传递 JavaScript 事件信息。

| 字段 | 描述 |
|------|------|
| `sChange` | 变更文本 |
| `sChangeEx` | 扩展变更 |
| `sValue` | 当前值 |
| `nSelStart` | 选择起始 |
| `nSelEnd` | 选择结束 |
| `bModifier` | 修饰键状态 |
| `bShift` | Shift 键状态 |
| `bRC` | 返回代码 |

## 事件处理流程

### 鼠标点击处理

```
OnLButtonDown()
    │
    ├── 查找目标字段
    │
    ├── 设置焦点
    │
    ├── 创建/显示 PWL 窗口
    │
    └── 转发到 PWL 控件
```

### 键盘输入处理

```
OnChar()
    │
    ├── 获取焦点字段
    │
    ├── 检查格式脚本
    │
    ├── 转发到 PWL 编辑器
    │
    ├── 执行按键脚本
    │
    └── 更新字段值
```

### 值变更处理

```
CommitData()
    │
    ├── 获取 PWL 窗口值
    │
    ├── 触发验证脚本
    │
    ├── 如果验证通过
    │   ├── 更新字段值
    │   └── 触发计算脚本
    │
    └── 刷新显示
```

## JavaScript 集成

表单字段支持以下 JavaScript 事件：

| 事件 | 触发时机 |
|------|----------|
| `OnFocus` | 获得焦点 |
| `OnBlur` | 失去焦点 |
| `OnMouseDown` | 鼠标按下 |
| `OnMouseUp` | 鼠标释放 |
| `OnMouseEnter` | 鼠标进入 |
| `OnMouseExit` | 鼠标离开 |
| `OnKeyStroke` | 按键（可验证/修改输入） |
| `OnValidate` | 值验证 |
| `OnCalculate` | 计算相关字段 |
| `OnFormat` | 格式化显示 |

## PWL 窗口管理

FormFiller 与 PWL 模块协作：

```
CFFL_FormField
    │
    ├── 创建 CPWL_Wnd（通过 CreatePWLWindow）
    │   ├── 文本框 → CPWL_Edit
    │   ├── 下拉框 → CPWL_ComboBox
    │   ├── 列表框 → CPWL_ListBox
    │   └── 按钮 → CPWL_Button
    │
    ├── 事件转发
    │
    └── 销毁窗口（失去焦点时）
```

## 外观更新

字段值变化时更新外观：

1. 获取新值
2. 生成外观流
3. 更新注释的 AP 字典
4. 请求重绘区域

## 依赖关系

```
formfiller/
    ├── pwl/             (窗口小部件)
    ├── cpdfsdk_widget   (小部件类)
    ├── core/fpdfdoc/    (表单字段)
    └── fxjs/            (JavaScript)
```

## 与 PWL 的协作

FormFiller 负责：
- 字段逻辑
- 值管理
- 脚本执行

PWL 负责：
- UI 渲染
- 用户输入
- 光标管理
