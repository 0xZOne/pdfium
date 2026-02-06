# xfa/fwl 源码分析

## 模块概述

`fwl` (FoxIt Widget Library) 模块提供 XFA 控件的底层实现。它是一个跨平台的窗口部件库，为 XFA 表单控件提供基础 UI 组件。

## 目录结构

```
xfa/fwl/
├── BUILD.gn                          # 构建配置
├── DEPS                              # 依赖声明
├── README.md                         # 模块说明
│
├── 核心类
│   ├── cfwl_app.cpp/h                # 应用管理
│   ├── cfwl_widget.cpp/h             # 控件基类
│   ├── cfwl_widgetmgr.cpp/h          # 控件管理器
│   ├── cfwl_notedriver.cpp/h         # 事件驱动
│   └── ifwl_widgetdelegate.h         # 控件代理接口
│
├── 表单控件
│   ├── cfwl_edit.cpp/h               # 编辑框
│   ├── cfwl_combobox.cpp/h           # 下拉框
│   ├── cfwl_comboedit.cpp/h          # 下拉编辑
│   ├── cfwl_combolist.cpp/h          # 下拉列表
│   ├── cfwl_listbox.cpp/h            # 列表框
│   ├── cfwl_checkbox.cpp/h           # 复选框
│   ├── cfwl_pushbutton.cpp/h         # 按钮
│   ├── cfwl_scrollbar.cpp/h          # 滚动条
│   ├── cfwl_caret.cpp/h              # 光标
│   ├── cfwl_barcode.cpp/h            # 条形码
│   ├── cfwl_datetimepicker.cpp/h     # 日期选择器
│   ├── cfwl_datetimeedit.cpp/h       # 日期编辑
│   ├── cfwl_monthcalendar.cpp/h      # 月历
│   └── cfwl_picturebox.cpp/h         # 图片框
│
├── 事件类
│   ├── cfwl_event.cpp/h              # 事件基类
│   ├── cfwl_eventmouse.cpp/h         # 鼠标事件
│   ├── cfwl_eventscroll.cpp/h        # 滚动事件
│   ├── cfwl_eventselectchanged.cpp/h # 选择变更事件
│   ├── cfwl_eventtextwillchange.cpp/h # 文本变更事件
│   └── cfwl_eventvalidate.cpp/h      # 验证事件
│
├── 消息类
│   ├── cfwl_message.cpp/h            # 消息基类
│   ├── cfwl_messagekey.cpp/h         # 按键消息
│   ├── cfwl_messagemouse.cpp/h       # 鼠标消息
│   ├── cfwl_messagemousewheel.cpp/h  # 滚轮消息
│   ├── cfwl_messagesetfocus.cpp/h    # 设置焦点消息
│   └── cfwl_messagekillfocus.cpp/h   # 失去焦点消息
│
├── 主题类
│   ├── cfwl_themepart.cpp/h          # 主题部件
│   ├── cfwl_themetext.cpp/h          # 主题文本
│   ├── cfwl_themebackground.cpp/h    # 主题背景
│   └── ifwl_themeprovider.cpp/h      # 主题提供者接口
│
├── 定义
│   ├── fwl_widgetdef.cpp/h           # 控件定义
│   └── fwl_widgethit.h               # 命中测试定义
│
├── 子目录
│   └── theme/                        # 主题实现
│
└── 测试
    └── cfwl_edit_embeddertest.cpp
```

## 核心类分析

### 1. CFWL_App 类

**位置**: cfwl_app.h/cpp

FWL 应用管理类。

| 方法 | 功能 |
|------|------|
| `GetWidgetMgr` | 获取控件管理器 |
| `GetNoteDriver` | 获取事件驱动器 |
| `GetThemeProvider` | 获取主题提供者 |

### 2. CFWL_Widget 类

**位置**: cfwl_widget.h/cpp

所有 FWL 控件的基类。

| 属性 | 描述 |
|------|------|
| `properties_` | 控件属性 |
| `widget_rect_` | 控件矩形 |
| `parent_` | 父控件 |
| `delegate_` | 控件代理 |

| 方法 | 功能 |
|------|------|
| `GetWidgetRect` | 获取控件矩形 |
| `SetWidgetRect` | 设置控件矩形 |
| `GetClientRect` | 获取客户区矩形 |
| `IsVisible` | 是否可见 |
| `IsEnabled` | 是否启用 |
| `HasBorder` | 是否有边框 |
| `Update` | 更新控件 |
| `DrawWidget` | 绘制控件 |
| `HitTest` | 命中测试 |
| `OnProcessMessage` | 处理消息 |
| `OnProcessEvent` | 处理事件 |
| `OnDrawWidget` | 绘制回调 |
| `SetFocus` | 设置焦点 |
| `GetFocus` | 获取焦点 |
| `Repaint` | 重绘 |
| `ThrowScroll` | 滚动通知 |
| `DispatchEvent` | 分发事件 |

### 3. CFWL_WidgetMgr 类

**位置**: cfwl_widgetmgr.h/cpp

控件管理器，管理控件层次结构。

| 方法 | 功能 |
|------|------|
| `GetParentWidget` | 获取父控件 |
| `GetFirstChildWidget` | 获取第一个子控件 |
| `GetNextSiblingWidget` | 获取下一个兄弟控件 |
| `InsertWidget` | 插入控件 |
| `RemoveWidget` | 移除控件 |
| `GetWidgetAtPoint` | 获取点处的控件 |
| `RepaintWidget` | 重绘控件 |

### 4. CFWL_NoteDriver 类

**位置**: cfwl_notedriver.h/cpp

事件驱动器，处理消息分发。

| 方法 | 功能 |
|------|------|
| `SendEvent` | 发送事件 |
| `ProcessMessage` | 处理消息 |
| `SetFocus` | 设置焦点控件 |
| `GetFocus` | 获取焦点控件 |
| `SetGrab` | 设置捕获控件 |
| `GetGrab` | 获取捕获控件 |
| `NotifyTargetHide` | 通知目标隐藏 |
| `NotifyTargetDestroy` | 通知目标销毁 |

## 表单控件详解

### CFWL_Edit 类

**位置**: cfwl_edit.h/cpp

编辑框控件。

| 方法 | 功能 |
|------|------|
| `SetText` | 设置文本 |
| `GetText` | 获取文本 |
| `InsertText` | 插入文本 |
| `DeleteText` | 删除文本 |
| `AddSelRange` | 添加选择范围 |
| `ClearSelection` | 清除选择 |
| `SelectAll` | 全选 |
| `GetCaret` | 获取光标位置 |
| `Undo` | 撤销 |
| `Redo` | 重做 |
| `CanUndo` | 能否撤销 |
| `CanRedo` | 能否重做 |
| `Copy` | 复制 |
| `Cut` | 剪切 |
| `Paste` | 粘贴 |
| `SetScrollOffset` | 设置滚动偏移 |

### CFWL_ComboBox 类

**位置**: cfwl_combobox.h/cpp

下拉框控件。

| 方法 | 功能 |
|------|------|
| `AddString` | 添加选项 |
| `RemoveAt` | 移除选项 |
| `RemoveAll` | 移除所有 |
| `GetCount` | 获取选项数 |
| `GetTextByIndex` | 获取选项文本 |
| `GetCurSel` | 获取当前选择 |
| `SetCurSel` | 设置当前选择 |
| `GetEditText` | 获取编辑文本 |
| `SetEditText` | 设置编辑文本 |
| `OpenDropDownList` | 打开下拉列表 |
| `CloseDropDownList` | 关闭下拉列表 |
| `IsDropListVisible` | 下拉列表是否可见 |

### CFWL_ListBox 类

**位置**: cfwl_listbox.h/cpp

列表框控件。

| 方法 | 功能 |
|------|------|
| `AddString` | 添加项 |
| `RemoveAt` | 移除项 |
| `DeleteAll` | 删除所有 |
| `GetCount` | 获取项数 |
| `GetItem` | 获取项 |
| `GetItemText` | 获取项文本 |
| `GetSelIndex` | 获取选择索引 |
| `SetSelItem` | 设置选择项 |
| `GetSelItem` | 获取选择项 |
| `ScrollToVisible` | 滚动到可见 |

### CFWL_CheckBox 类

**位置**: cfwl_checkbox.h/cpp

复选框控件。

| 方法 | 功能 |
|------|------|
| `SetCheckState` | 设置选中状态 |
| `GetCheckState` | 获取选中状态 |

选中状态：
- `kUnchecked`: 未选中
- `kChecked`: 选中
- `kNeutral`: 中间状态

### CFWL_DateTimePicker 类

**位置**: cfwl_datetimepicker.h/cpp

日期时间选择器。

| 方法 | 功能 |
|------|------|
| `GetCurSel` | 获取当前选择日期 |
| `SetCurSel` | 设置当前选择日期 |
| `GetEditText` | 获取编辑文本 |
| `SetEditText` | 设置编辑文本 |

## 事件系统

### 事件类型

| 事件类 | 描述 |
|--------|------|
| `CFWL_EventMouse` | 鼠标事件 |
| `CFWL_EventScroll` | 滚动事件 |
| `CFWL_EventSelectChanged` | 选择变更 |
| `CFWL_EventTextWillChange` | 文本即将变更 |
| `CFWL_EventValidate` | 验证事件 |

### 消息类型

| 消息类 | 描述 |
|--------|------|
| `CFWL_MessageKey` | 按键消息 |
| `CFWL_MessageMouse` | 鼠标消息 |
| `CFWL_MessageMouseWheel` | 滚轮消息 |
| `CFWL_MessageSetFocus` | 设置焦点 |
| `CFWL_MessageKillFocus` | 失去焦点 |

### 事件分发流程

```
外部输入
    │
    ├── CFWL_NoteDriver 接收
    │
    ├── 确定目标控件
    │   └── 命中测试或焦点控件
    │
    ├── 创建消息对象
    │
    ├── 调用 ProcessMessage
    │
    ├── 控件处理消息
    │
    └── 生成事件通知
```

## 主题系统

### IFWL_ThemeProvider 接口

**位置**: ifwl_themeprovider.h

定义主题绘制接口。

| 方法 | 功能 |
|------|------|
| `DrawBackground` | 绘制背景 |
| `DrawText` | 绘制文本 |

### CFWL_ThemePart 类

**位置**: cfwl_themepart.h/cpp

主题部件信息。

| 属性 | 描述 |
|------|------|
| `part_` | 部件类型 |
| `states_` | 状态标志 |
| `rect_` | 部件矩形 |
| `matrix_` | 变换矩阵 |

## 依赖关系

```
fwl/
    ├── fgas/            (图形和字体)
    ├── fde/             (文本编辑)
    ├── core/fxge/       (图形引擎)
    └── core/fxcrt/      (基础设施)

被依赖:
    xfa/fxfa/ → fwl/
```

## 与 XFA 的协作

FWL 作为 XFA 控件的底层实现：

```
CXFA_FFWidget
    │
    ├── 创建对应的 CFWL_Widget
    │
    ├── 转发事件
    │   ├── 鼠标事件
    │   ├── 键盘事件
    │   └── 焦点事件
    │
    └── 请求重绘
```
