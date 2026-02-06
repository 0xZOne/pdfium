# xfa/fxfa 源码分析

## 模块概述

`fxfa` 是 XFA 的核心模块，提供 XFA 文档管理、表单控件渲染、事件处理等核心功能。

## 目录结构

```
xfa/fxfa/
├── BUILD.gn                          # 构建配置
├── DEPS                              # 依赖声明
├── README.md                         # 模块说明
│
├── 核心类
│   ├── cxfa_ffapp.cpp/h              # XFA 应用
│   ├── cxfa_ffdoc.cpp/h              # XFA 文档
│   ├── cxfa_ffdocview.cpp/h          # 文档视图
│   ├── cxfa_ffpageview.cpp/h         # 页面视图
│   ├── cxfa_ffwidget.cpp/h           # 控件基类
│   ├── cxfa_ffwidgethandler.cpp/h    # 控件处理器
│   └── cxfa_ffnotify.cpp/h           # 通知器
│
├── 表单控件
│   ├── cxfa_fftextedit.cpp/h         # 文本编辑
│   ├── cxfa_ffnumericedit.cpp/h      # 数字编辑
│   ├── cxfa_ffpasswordedit.cpp/h     # 密码编辑
│   ├── cxfa_ffdatetimeedit.cpp/h     # 日期时间
│   ├── cxfa_ffcheckbutton.cpp/h      # 复选按钮
│   ├── cxfa_ffcombobox.cpp/h         # 下拉框
│   ├── cxfa_ffdropdown.cpp/h         # 下拉基类
│   ├── cxfa_fflistbox.cpp/h          # 列表框
│   ├── cxfa_ffpushbutton.cpp/h       # 按钮
│   ├── cxfa_ffimageedit.cpp/h        # 图像编辑
│   ├── cxfa_ffsignature.cpp/h        # 签名
│   └── cxfa_ffbarcode.cpp/h          # 条形码
│
├── 静态绘制
│   ├── cxfa_fftext.cpp/h             # 文本
│   ├── cxfa_ffimage.cpp/h            # 图像
│   ├── cxfa_ffline.cpp/h             # 直线
│   ├── cxfa_ffarc.cpp/h              # 弧形
│   └── cxfa_ffrectangle.cpp/h        # 矩形
│
├── 字段和分组
│   ├── cxfa_fffield.cpp/h            # 字段基类
│   └── cxfa_ffexclgroup.cpp/h        # 互斥组
│
├── 文本处理
│   ├── cxfa_textlayout.cpp/h         # 文本布局
│   ├── cxfa_textparser.cpp/h         # 文本解析
│   ├── cxfa_textprovider.cpp/h       # 文本提供者
│   └── cxfa_texttabstopscontext.cpp/h # Tab 停止
│
├── 辅助类
│   ├── cxfa_eventparam.cpp/h         # 事件参数
│   ├── cxfa_fontmgr.cpp/h            # 字体管理
│   ├── cxfa_fwladapterwidgetmgr.cpp/h # FWL 适配器
│   ├── cxfa_fwltheme.cpp/h           # FWL 主题
│   ├── cxfa_imagerenderer.cpp/h      # 图像渲染
│   └── cxfa_readynodeiterator.cpp/h  # 节点迭代器
│
├── 头文件
│   ├── fxfa.h                        # 公共头文件
│   ├── fxfa_basic.h                  # 基础定义
│   └── cxfa_ffwidget_type.h          # 控件类型
│
├── 子目录
│   ├── parser/                       # XFA 解析器
│   ├── layout/                       # 布局引擎
│   └── formcalc/                     # FormCalc 脚本
│
└── 测试文件
    ├── fxfa_basic_unittest.cpp
    ├── cxfa_ffbarcode_unittest.cpp
    └── cxfa_textparser_unittest.cpp
```

## 核心类分析

### 1. CXFA_FFApp 类

**位置**: cxfa_ffapp.h/cpp

XFA 应用类，管理全局资源。

| 方法 | 功能 |
|------|------|
| `CreateDoc` | 创建文档 |
| `GetWidgetMgrDelegate` | 获取控件管理器代理 |
| `GetFWLAdapterWidgetMgr` | 获取 FWL 适配器 |
| `GetTimerMgr` | 获取定时器管理器 |
| `GetFWLTheme` | 获取 FWL 主题 |

### 2. CXFA_FFDoc 类

**位置**: cxfa_ffdoc.h/cpp

XFA 文档类。

| 方法 | 功能 |
|------|------|
| `OpenDoc` | 打开文档 |
| `CloseDoc` | 关闭文档 |
| `CreateDocView` | 创建文档视图 |
| `GetDocView` | 获取文档视图 |
| `GetPDFDoc` | 获取关联的 PDF 文档 |
| `GetFormType` | 获取表单类型 |
| `GetXFADoc` | 获取 XFA 文档对象 |
| `ExportData` | 导出数据 |
| `ImportData` | 导入数据 |

### 3. CXFA_FFDocView 类

**位置**: cxfa_ffdocview.h/cpp

文档视图类，管理 XFA 表单的显示和交互。

| 方法 | 功能 |
|------|------|
| `GetPageCount` | 获取页数 |
| `GetPageViewByIndex` | 获取页面视图 |
| `ResetNode` | 重置节点 |
| `RunCalculateScripts` | 运行计算脚本 |
| `RunValidateScripts` | 运行验证脚本 |
| `SetFocus` | 设置焦点 |
| `GetFocusWidget` | 获取焦点控件 |
| `GetWidgetHandler` | 获取控件处理器 |
| `OnPageViewEvent` | 页面视图事件 |

### 4. CXFA_FFPageView 类

**位置**: cxfa_ffpageview.h/cpp

页面视图类。

| 方法 | 功能 |
|------|------|
| `GetPageIndex` | 获取页面索引 |
| `GetPageSize` | 获取页面尺寸 |
| `GetDisplayMatrix` | 获取显示矩阵 |
| `GetWidgetIterator` | 获取控件迭代器 |

### 5. CXFA_FFWidget 类

**位置**: cxfa_ffwidget.h/cpp

XFA 控件基类。

| 方法 | 功能 |
|------|------|
| `GetWidgetRect` | 获取控件矩形 |
| `GetRotateMatrix` | 获取旋转矩阵 |
| `RenderWidget` | 渲染控件 |
| `HitTest` | 点击测试 |
| `OnMouseEnter` | 鼠标进入 |
| `OnMouseExit` | 鼠标离开 |
| `OnLButtonDown` | 左键按下 |
| `OnLButtonUp` | 左键释放 |
| `OnMouseMove` | 鼠标移动 |
| `OnMouseWheel` | 鼠标滚轮 |
| `OnRButtonDown` | 右键按下 |
| `OnRButtonUp` | 右键释放 |
| `OnKeyDown` | 按键按下 |
| `OnKeyUp` | 按键释放 |
| `OnChar` | 字符输入 |
| `OnSetFocus` | 获得焦点 |
| `OnKillFocus` | 失去焦点 |
| `ProcessEvent` | 处理事件 |

### 6. CXFA_FFWidgetHandler 类

**位置**: cxfa_ffwidgethandler.h/cpp

控件处理器，管理控件事件分发。

| 方法 | 功能 |
|------|------|
| `OnMouseEnter` | 分发鼠标进入 |
| `OnMouseExit` | 分发鼠标离开 |
| `OnLButtonDown` | 分发左键按下 |
| `OnLButtonUp` | 分发左键释放 |
| `OnMouseMove` | 分发鼠标移动 |
| `OnKeyDown` | 分发按键 |
| `OnChar` | 分发字符 |
| `ProcessEvent` | 处理事件 |

## 表单控件详解

### CXFA_FFTextField

**位置**: cxfa_fftextedit.h/cpp

文本编辑控件。

| 方法 | 功能 |
|------|------|
| `LoadWidget` | 加载控件 |
| `UpdateWidgetProperty` | 更新属性 |
| `CommitData` | 提交数据 |
| `UpdateFWLData` | 更新 FWL 数据 |
| `SetEditScrollOffset` | 设置滚动偏移 |

### CXFA_FFCheckButton

**位置**: cxfa_ffcheckbutton.h/cpp

复选按钮控件。

| 方法 | 功能 |
|------|------|
| `LoadWidget` | 加载控件 |
| `RenderWidget` | 渲染控件 |
| `CommitData` | 提交数据 |
| `SetFWLCheckState` | 设置选中状态 |

### CXFA_FFComboBox

**位置**: cxfa_ffcombobox.h/cpp

下拉框控件。

| 方法 | 功能 |
|------|------|
| `LoadWidget` | 加载控件 |
| `UpdateWidgetProperty` | 更新属性 |
| `InsertItem` | 插入选项 |
| `DeleteItem` | 删除选项 |
| `CommitData` | 提交数据 |

## 子目录详解

### parser/

XFA XML 解析器，解析 XFA 文档结构。

主要类：
- `CXFA_Document`: XFA 文档对象模型
- `CXFA_Node`: XFA 节点
- `CXFA_DataPacket`: 数据包

### layout/

布局引擎，计算 XFA 元素的位置和大小。

主要类：
- `CXFA_LayoutProcessor`: 布局处理器
- `CXFA_ItemLayoutProcessor`: 项目布局
- `CXFA_ContentLayoutProcessor`: 内容布局

### formcalc/

FormCalc 脚本编译器和执行器。

主要类：
- `CXFA_FMParser`: FormCalc 解析器
- `CXFA_FMAST`: 抽象语法树

## 事件处理流程

```
用户操作
    │
    ├── CXFA_FFWidgetHandler 接收
    │
    ├── 定位目标控件
    │
    ├── 调用控件事件方法
    │
    ├── 执行关联脚本
    │   ├── JavaScript
    │   └── FormCalc
    │
    └── 更新控件状态
```

## 渲染流程

```
CXFA_FFWidget::RenderWidget()
    │
    ├── 计算渲染区域
    │
    ├── 绘制边框
    │
    ├── 绘制背景
    │
    ├── 绘制内容
    │   └── 由子类实现
    │
    └── 绘制装饰
```

## 依赖关系

```
fxfa/
    ├── fwl/             (窗口部件)
    ├── fgas/            (图形和脚本)
    ├── fde/             (设备扩展)
    ├── fxjs/            (JavaScript)
    ├── fxbarcode/       (条形码)
    ├── core/fxge/       (图形引擎)
    └── core/fxcrt/      (基础设施)
```

## 与 FWL 的协作

XFA 控件使用 FWL 实现底层 UI：

```
CXFA_FFWidget
    │
    └── 包含 CFWL_Widget
        │
        ├── 事件转发
        │
        └── 渲染委托
```
