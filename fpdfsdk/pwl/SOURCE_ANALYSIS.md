# fpdfsdk/pwl 源码分析

## 模块概述

`pwl` (Portable Widget Library) 模块提供跨平台的轻量级 UI 组件，用于在 PDF 页面上渲染表单控件。这些控件与操作系统无关，完全由 PDFium 自行绘制。

## 目录结构

```
fpdfsdk/pwl/
├── BUILD.gn                          # 构建配置
├── README.md                         # 模块说明
│
├── 核心窗口
│   ├── cpwl_wnd.cpp/h                # 窗口基类
│   └── ipwl_fillernotify.h           # 填充器通知接口
│
├── 文本编辑
│   ├── cpwl_edit.cpp/h               # 编辑框
│   ├── cpwl_edit_impl.cpp/h          # 编辑框实现
│   └── cpwl_caret.cpp/h              # 光标
│
├── 按钮控件
│   ├── cpwl_button.cpp/h             # 按钮基类
│   ├── cpwl_special_button.cpp/h     # 特殊按钮
│   ├── cpwl_sbbutton.cpp/h           # 滚动条按钮
│   └── cpwl_cbbutton.cpp/h           # 下拉框按钮
│
├── 列表控件
│   ├── cpwl_list_box.cpp/h           # 列表框
│   ├── cpwl_list_ctrl.cpp/h          # 列表控制器
│   └── cpwl_cblistbox.cpp/h          # 下拉列表框
│
├── 组合控件
│   └── cpwl_combo_box.cpp/h          # 下拉框
│
├── 滚动控件
│   └── cpwl_scroll_bar.cpp/h         # 滚动条
│
└── 测试
    ├── cpwl_edit_embeddertest.cpp
    ├── cpwl_combo_box_embeddertest.cpp
    └── cpwl_special_button_embeddertest.cpp
```

## 核心类分析

### 1. CPWL_Wnd 类

**位置**: cpwl_wnd.h/cpp

所有 PWL 窗口的基类。

| 成员 | 描述 |
|------|------|
| `rcWindow_` | 窗口矩形 |
| `visible_` | 是否可见 |
| `enabled_` | 是否启用 |
| `children_` | 子窗口列表 |
| `parent_` | 父窗口 |

| 方法 | 功能 |
|------|------|
| `Create` | 创建窗口 |
| `Destroy` | 销毁窗口 |
| `Move` | 移动窗口 |
| `SetVisible` | 设置可见性 |
| `GetWindowRect` | 获取窗口矩形 |
| `GetClientRect` | 获取客户区矩形 |
| `InvalidateRect` | 使区域无效 |
| `DrawThisAppearance` | 绘制外观 |
| `OnKeyDown` | 按键按下 |
| `OnChar` | 字符输入 |
| `OnLButtonDown` | 鼠标左键按下 |
| `OnLButtonUp` | 鼠标左键释放 |
| `OnMouseMove` | 鼠标移动 |
| `OnMouseWheel` | 鼠标滚轮 |
| `SetFocus` | 设置焦点 |
| `KillFocus` | 取消焦点 |

### 2. CPWL_Edit 类

**位置**: cpwl_edit.h/cpp

文本编辑控件。

| 方法 | 功能 |
|------|------|
| `SetText` | 设置文本 |
| `GetText` | 获取文本 |
| `GetSelectedText` | 获取选中文本 |
| `ReplaceSelection` | 替换选中 |
| `SelectAll` | 全选 |
| `ClearSelection` | 清除选择 |
| `SetCaret` | 设置光标位置 |
| `GetCaret` | 获取光标位置 |
| `SetScrollPos` | 设置滚动位置 |
| `CanUndo` | 能否撤销 |
| `CanRedo` | 能否重做 |
| `Undo` | 撤销 |
| `Redo` | 重做 |
| `Cut` | 剪切 |
| `Copy` | 复制 |
| `Paste` | 粘贴 |

编辑框属性：

| 属性 | 描述 |
|------|------|
| `IsMultiLine` | 多行模式 |
| `IsPassword` | 密码模式 |
| `IsReadOnly` | 只读模式 |
| `GetMaxLen` | 最大长度 |
| `GetCharLimit` | 字符限制 |
| `GetAlignment` | 对齐方式 |

### 3. CPWL_EditImpl 类

**位置**: cpwl_edit_impl.h/cpp

编辑框的内部实现，处理文本编辑逻辑。

| 功能 | 描述 |
|------|------|
| 光标管理 | 跟踪光标位置和选择范围 |
| 文本布局 | 计算行和字符位置 |
| 撤销/重做 | 维护编辑历史 |
| 滚动 | 处理内容滚动 |

### 4. CPWL_Caret 类

**位置**: cpwl_caret.h/cpp

文本光标（插入点）。

| 方法 | 功能 |
|------|------|
| `SetCaret` | 设置光标位置 |
| `ShowCaret` | 显示光标 |
| `HideCaret` | 隐藏光标 |

光标闪烁通过定时器实现。

### 5. CPWL_Button 类

**位置**: cpwl_button.h/cpp

按钮基类。

| 方法 | 功能 |
|------|------|
| `OnLButtonDown` | 按钮按下效果 |
| `OnLButtonUp` | 按钮释放效果 |
| `OnMouseEnter` | 悬停效果 |
| `OnMouseExit` | 移除悬停 |

### 6. CPWL_SpecialButton 类

**位置**: cpwl_special_button.h/cpp

特殊按钮（复选框、单选按钮）。

| 方法 | 功能 |
|------|------|
| `IsChecked` | 是否选中 |
| `SetCheck` | 设置选中状态 |
| `DrawThisAppearance` | 绘制选中标记 |

### 7. CPWL_ListBox 类

**位置**: cpwl_list_box.h/cpp

列表框控件。

| 方法 | 功能 |
|------|------|
| `AddString` | 添加项 |
| `SetTopVisibleIndex` | 设置顶部可见项 |
| `ScrollToListItem` | 滚动到项 |
| `Select` | 选择项 |
| `SetCaret` | 设置当前项 |
| `GetCurSel` | 获取当前选择 |
| `GetSelectedIndices` | 获取多选索引 |
| `IsMultipleSel` | 是否多选模式 |

### 8. CPWL_ListCtrl 类

**位置**: cpwl_list_ctrl.h/cpp

列表控制器，管理列表项的布局和选择。

| 方法 | 功能 |
|------|------|
| `SetNotify` | 设置通知回调 |
| `SetScrollPos` | 设置滚动位置 |
| `GetContentRect` | 获取内容矩形 |
| `GetFirstSelected` | 获取首选项 |

### 9. CPWL_ComboBox 类

**位置**: cpwl_combo_box.h/cpp

下拉框控件（编辑框 + 按钮 + 列表框）。

| 组件 | 类型 |
|------|------|
| 编辑框 | CPWL_Edit |
| 下拉按钮 | CPWL_CBButton |
| 下拉列表 | CPWL_CBListBox |

| 方法 | 功能 |
|------|------|
| `GetText` | 获取文本 |
| `SetText` | 设置文本 |
| `GetSelect` | 获取选择索引 |
| `SetSelect` | 设置选择索引 |
| `ShowPopup` | 显示下拉列表 |
| `HidePopup` | 隐藏下拉列表 |

### 10. CPWL_ScrollBar 类

**位置**: cpwl_scroll_bar.h/cpp

滚动条控件。

| 组件 | 描述 |
|------|------|
| 上/左按钮 | 减少滚动位置 |
| 下/右按钮 | 增加滚动位置 |
| 滑块 | 拖动定位 |
| 轨道 | 点击跳转 |

| 方法 | 功能 |
|------|------|
| `SetScrollRange` | 设置范围 |
| `SetScrollPos` | 设置位置 |
| `GetScrollPos` | 获取位置 |
| `SetScrollStep` | 设置步长 |

## 绘制系统

### 绘制流程

```
DrawAppearance()
    │
    ├── 设置裁剪区域
    │
    ├── 绘制背景
    │
    ├── DrawThisAppearance()
    │   └── 子类实现具体绘制
    │
    ├── 绘制边框
    │
    └── 绘制子窗口
```

### 颜色和样式

| 元素 | 配置 |
|------|------|
| 背景色 | 可设置 |
| 边框色 | 可设置 |
| 文本色 | 可设置 |
| 边框样式 | 实线/虚线/下划线 |
| 边框宽度 | 可设置 |

## 事件处理

### 事件分发

```
外部事件
    │
    ├── CPWL_Wnd 接收
    │
    ├── 检查子窗口
    │   └── 命中测试确定目标
    │
    └── 调用目标窗口的事件处理方法
```

### 焦点管理

- 使用焦点链管理焦点顺序
- Tab 键在控件间导航
- 焦点变化触发重绘

## 通知接口

### IPWL_FillerNotify

**位置**: ipwl_fillernotify.h

定义 PWL 控件到 FormFiller 的回调：

| 方法 | 功能 |
|------|------|
| `InvalidateRect` | 请求重绘区域 |
| `OnSetScrollInfo` | 滚动信息变化 |
| `OnSetScrollPos` | 滚动位置变化 |
| `OnSetCaret` | 光标变化 |
| `OnCaretChange` | 光标改变 |
| `SetFocusAnnot` | 设置焦点注释 |
| `GetSysFontPath` | 获取系统字体路径 |

## 依赖关系

```
pwl/
    ├── core/fxge/       (图形绑制)
    ├── core/fpdfdoc/    (可变文本)
    └── core/fxcrt/      (基础设施)

被依赖:
    formfiller/ → pwl/
```

## 坐标系统

PWL 使用 PDF 页面坐标系：
- 原点在左下角
- X 向右为正
- Y 向上为正

所有坐标在页面空间中，渲染时转换到设备空间。

## 特点

- **跨平台**: 不依赖系统 UI
- **轻量级**: 最小化资源占用
- **一致性**: 各平台外观相同
- **可定制**: 支持样式配置
