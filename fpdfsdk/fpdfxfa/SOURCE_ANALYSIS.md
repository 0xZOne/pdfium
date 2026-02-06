# fpdfsdk/fpdfxfa 源码分析

## 模块概述

`fpdfxfa` 模块提供 XFA（XML Forms Architecture）表单与 SDK 层的集成。它桥接了 PDFium 的标准 PDF 处理和 XFA 动态表单功能。

## 目录结构

```
fpdfsdk/fpdfxfa/
├── BUILD.gn                          # 构建配置
├── DEPS                              # 依赖声明
│
├── 核心类
│   ├── cpdfxfa_context.cpp/h         # XFA 上下文
│   ├── cpdfxfa_page.cpp/h            # XFA 页面
│   ├── cpdfxfa_widget.cpp/h          # XFA 小部件
│   └── cpdfxfa_docenvironment.cpp/h  # XFA 文档环境
│
└── 测试
    ├── cpdfxfa_context_embeddertest.cpp
    └── cpdfxfa_docenvironment_embeddertest.cpp
```

## 核心类分析

### 1. CPDFXFA_Context 类

**位置**: cpdfxfa_context.h/cpp

XFA 上下文类，管理 XFA 文档的生命周期和状态。

| 方法 | 功能 |
|------|------|
| `GetFormType` | 获取表单类型 |
| `GetXFADoc` | 获取 XFA 文档 |
| `GetXFADocView` | 获取 XFA 文档视图 |
| `GetPDFDoc` | 获取 PDF 文档 |
| `LoadXFADoc` | 加载 XFA 文档 |
| `GetPageCount` | 获取页数 |
| `GetXFAPage` | 获取 XFA 页面 |
| `DeletePage` | 删除页面 |
| `GetFormFillEnv` | 获取表单填充环境 |

表单类型：

| 类型 | 描述 |
|------|------|
| `kNone` | 无表单 |
| `kAcroForm` | 传统 AcroForm |
| `kXFAFull` | 完整 XFA |
| `kXFAForeground` | XFA 前景 |

### 2. CPDFXFA_Page 类

**位置**: cpdfxfa_page.h/cpp

XFA 页面类，表示包含 XFA 内容的页面。

| 方法 | 功能 |
|------|------|
| `GetXFAPageView` | 获取 XFA 页面视图 |
| `GetDocument` | 获取文档 |
| `GetPageIndex` | 获取页面索引 |
| `GetPageWidth` | 获取页面宽度 |
| `GetPageHeight` | 获取页面高度 |
| `GetDisplayMatrix` | 获取显示矩阵 |
| `DeviceToPage` | 设备到页面坐标 |
| `PageToDevice` | 页面到设备坐标 |
| `GetXFAWidgetHandler` | 获取控件处理器 |
| `GetXFAWidgetIterator` | 获取控件迭代器 |

接口实现：
- `IPDF_Page`: PDF 页面接口

### 3. CPDFXFA_Widget 类

**位置**: cpdfxfa_widget.h/cpp

XFA 小部件类，封装 XFA 控件为 SDK 层注释。

| 方法 | 功能 |
|------|------|
| `GetXFAFFWidget` | 获取 XFA FF 控件 |
| `GetAnnotSubtype` | 获取注释子类型 |
| `GetRect` | 获取矩形 |
| `GetLayoutItem` | 获取布局项 |

继承关系：
- 继承自 `CPDFSDK_Annot`
- 封装 `CXFA_FFWidget`

### 4. CPDFXFA_DocEnvironment 类

**位置**: cpdfxfa_docenvironment.h/cpp

XFA 文档环境，实现 XFA 与嵌入器之间的回调接口。

| 方法类别 | 方法 |
|----------|------|
| 文档操作 | `SetChangeMark`, `GetTitle`, `SetTitle` |
| 页面操作 | `PageViewEvent`, `WidgetPostAdd`, `WidgetPreRemove` |
| 弹出窗口 | `PopupMenu`, `Alert`, `Confirm` |
| 文件操作 | `OpenFile`, `GotoURL`, `GetPlatform` |
| 打印 | `Print`, `GetPrintRanges` |
| 表单 | `ExportData`, `ImportData`, `SubmitData` |

实现接口：
- `IXFA_DocEnvironment`

## XFA 文档加载流程

```
CPDFXFA_Context::LoadXFADoc()
    │
    ├── 检查文档类型
    │   └── 确认包含 XFA 流
    │
    ├── 创建 CXFA_FFApp
    │
    ├── 解析 XFA XML
    │   ├── 读取 XFA 包数据
    │   └── 构建 XFA DOM 树
    │
    ├── 创建 CXFA_FFDoc
    │
    ├── 创建 CXFA_FFDocView
    │
    └── 执行初始化脚本
```

## XFA 页面渲染流程

```
CPDFXFA_Page 渲染
    │
    ├── 获取 XFA 页面视图
    │
    ├── 枚举 XFA 控件
    │
    ├── 渲染每个控件
    │   └── CXFA_FFWidget::RenderWidget()
    │
    └── 合成到输出
```

## XFA 事件处理

### 用户交互

```
用户事件（鼠标/键盘）
    │
    ├── FormFillEnvironment 接收
    │
    ├── 定位 XFA 控件
    │   └── CPDFXFA_Widget
    │
    ├── 转发到 CXFA_FFWidget
    │
    └── XFA 脚本处理
```

### 脚本执行

XFA 支持两种脚本：
- **JavaScript**: 标准 PDF JavaScript
- **FormCalc**: Adobe 专有脚本语言

## 表单类型判断

```
加载文档
    │
    ├── 检查 AcroForm 字典
    │   └── 存在 → kAcroForm
    │
    ├── 检查 XFA 流
    │   ├── 完整 XFA → kXFAFull
    │   └── 前景 XFA → kXFAForeground
    │
    └── 无表单 → kNone
```

## 混合模式

XFA 前景模式：
- PDF 内容作为背景
- XFA 控件覆盖在上层
- 支持部分 XFA 功能

完整 XFA 模式：
- 页面完全由 XFA 控制
- 动态布局
- 完整 XFA 功能

## 依赖关系

```
fpdfxfa/
    ├── fpdfsdk/             (SDK 核心)
    ├── xfa/fxfa/            (XFA 核心)
    ├── core/fpdfapi/        (PDF 核心)
    └── fxjs/                (JavaScript)

被依赖:
    fpdfsdk/ → fpdfxfa/ (可选)
```

## 条件编译

XFA 功能通过宏控制：
- `PDF_ENABLE_XFA`: 启用 XFA 支持

未启用时，fpdfxfa 模块不会编译。

## 回调示例

### Alert 对话框

```
用户调用 xfa.host.messageBox()
    │
    ├── CXFA_FFDoc 调用 DocEnvironment
    │
    ├── CPDFXFA_DocEnvironment::Alert()
    │
    ├── 调用 FormFillInfo 回调
    │   └── FFI_Alert()
    │
    └── 返回用户选择
```

### 提交表单

```
用户调用 submit
    │
    ├── CPDFXFA_DocEnvironment::SubmitData()
    │
    ├── 收集表单数据
    │   ├── XDP 格式
    │   ├── XML 格式
    │   └── PDF 格式
    │
    └── 调用嵌入器提交回调
```

## 注意事项

- XFA 是 Adobe 专有技术
- Chrome 已禁用 XFA 支持
- PDFium 的 XFA 支持是可选的
- XFA 文档比较复杂，解析较慢
