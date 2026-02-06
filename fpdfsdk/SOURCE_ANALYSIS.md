# fpdfsdk 源码分析

## 模块概述

`fpdfsdk` 模块是 SDK 实现层，负责实现 `public/` 目录中定义的公共 API。它提供了与嵌入器交互所需的高级抽象，包括表单填充、注释管理、页面视图等功能。

## 目录结构

```
fpdfsdk/
├── BUILD.gn                          # 构建配置
├── DEPS                              # 依赖声明
├── PRESUBMIT.py                      # 预提交检查
│
├── SDK 核心类
│   ├── cpdfsdk_annot.cpp/h           # 注释基类
│   ├── cpdfsdk_baannot.cpp/h         # 基本注释
│   ├── cpdfsdk_widget.cpp/h          # 表单小部件
│   ├── cpdfsdk_pageview.cpp/h        # 页面视图
│   ├── cpdfsdk_formfillenvironment.cpp/h  # 表单填充环境
│   ├── cpdfsdk_interactiveform.cpp/h # 交互式表单
│   ├── cpdfsdk_annotiteration.cpp/h  # 注释迭代
│   ├── cpdfsdk_annotiterator.cpp/h   # 注释迭代器
│   └── cpdfsdk_appstream.cpp/h       # 外观流
│
├── 适配器和辅助类
│   ├── cpdfsdk_helpers.cpp/h         # 辅助函数
│   ├── cpdfsdk_customaccess.cpp/h    # 自定义访问
│   ├── cpdfsdk_filewriteadapter.cpp/h # 文件写入适配器
│   ├── cpdfsdk_pauseadapter.cpp/h    # 暂停适配器
│   └── cpdfsdk_renderpage.cpp/h      # 页面渲染
│
├── API 实现
│   ├── fpdf_view.cpp                 # fpdfview.h 实现
│   ├── fpdf_annot.cpp                # fpdf_annot.h 实现
│   ├── fpdf_attachment.cpp           # fpdf_attachment.h 实现
│   ├── fpdf_catalog.cpp              # fpdf_catalog.h 实现
│   ├── fpdf_dataavail.cpp            # fpdf_dataavail.h 实现
│   ├── fpdf_doc.cpp                  # fpdf_doc.h 实现
│   ├── fpdf_editimg.cpp              # 图像编辑实现
│   ├── fpdf_editpage.cpp             # 页面编辑实现
│   ├── fpdf_editpath.cpp             # 路径编辑实现
│   ├── fpdf_edittext.cpp             # 文本编辑实现
│   ├── fpdf_ext.cpp                  # 扩展功能实现
│   ├── fpdf_flatten.cpp              # 扁平化实现
│   ├── fpdf_formfill.cpp             # 表单填充实现
│   ├── fpdf_javascript.cpp           # JavaScript 实现
│   ├── fpdf_ppo.cpp                  # 页面操作实现
│   ├── fpdf_progressive.cpp          # 渐进式渲染实现
│   ├── fpdf_save.cpp                 # 保存实现
│   ├── fpdf_searchex.cpp             # 扩展搜索实现
│   ├── fpdf_signature.cpp            # 签名实现
│   ├── fpdf_structtree.cpp           # 结构树实现
│   ├── fpdf_sysfontinfo.cpp          # 系统字体实现
│   ├── fpdf_text.cpp                 # 文本 API 实现
│   ├── fpdf_thumbnail.cpp            # 缩略图实现
│   └── fpdf_transformpage.cpp        # 页面变换实现
│
├── 子目录
│   ├── formfiller/                   # 表单填充器
│   ├── pwl/                          # 轻量级窗口小部件
│   └── fpdfxfa/                      # XFA 扩展集成
│
└── 测试文件
    └── *_embeddertest.cpp            # 嵌入器测试
```

## 核心类分析

### 1. CPDFSDK_FormFillEnvironment 类

**位置**: cpdfsdk_formfillenvironment.h/cpp

表单填充环境，管理整个表单交互的核心类。

| 成员 | 描述 |
|------|------|
| `page_map_` | 页面视图映射 |
| `interactive_form_` | 交互式表单 |
| `focus_annot_` | 当前焦点注释 |
| `ijs_runtime_` | JavaScript 运行时 |

| 方法 | 功能 |
|------|------|
| `GetPageView` | 获取页面视图 |
| `GetInteractiveForm` | 获取交互式表单 |
| `SetFocusAnnot` | 设置焦点注释 |
| `KillFocusAnnot` | 取消焦点 |
| `GetJSRuntime` | 获取 JS 运行时 |
| `Invalidate` | 使区域无效 |
| `OnSetFieldInputFocus` | 设置字段输入焦点 |
| `DoURIAction` | 执行 URI 动作 |
| `DoGoToAction` | 执行跳转动作 |

### 2. CPDFSDK_PageView 类

**位置**: cpdfsdk_pageview.h/cpp

页面视图类，管理单个页面的注释和交互。

| 成员 | 描述 |
|------|------|
| `annots_` | 注释列表 |
| `page_` | 关联的页面 |
| `form_fill_env_` | 表单填充环境 |

| 方法 | 功能 |
|------|------|
| `GetAnnotByDict` | 通过字典获取注释 |
| `GetAnnotAtPoint` | 在点处获取注释 |
| `GetFocusAnnot` | 获取焦点注释 |
| `GetAnnotCount` | 获取注释数量 |
| `GetAnnot` | 获取指定注释 |
| `OnMouseDown` | 鼠标按下事件 |
| `OnMouseUp` | 鼠标释放事件 |
| `OnMouseMove` | 鼠标移动事件 |
| `OnChar` | 字符输入事件 |
| `OnKeyDown` | 按键按下事件 |

### 3. CPDFSDK_Annot 类

**位置**: cpdfsdk_annot.h/cpp

SDK 注释基类，所有 SDK 层注释类型的父类。

| 方法 | 功能 |
|------|------|
| `GetAnnotDict` | 获取注释字典 |
| `GetAnnotSubtype` | 获取注释子类型 |
| `GetRect` | 获取注释矩形 |
| `GetPageView` | 获取页面视图 |
| `SetRect` | 设置矩形 |

### 4. CPDFSDK_BAAnnot 类

**位置**: cpdfsdk_baannot.h/cpp

基本注释类（Basic Annotation），扩展了 CPDFSDK_Annot。

| 方法 | 功能 |
|------|------|
| `GetContents` | 获取注释内容 |
| `SetContents` | 设置内容 |
| `GetAnnotName` | 获取名称 |
| `SetAnnotName` | 设置名称 |
| `GetFlags` | 获取标志 |
| `SetFlags` | 设置标志 |
| `GetColor` | 获取颜色 |
| `SetColor` | 设置颜色 |

### 5. CPDFSDK_Widget 类

**位置**: cpdfsdk_widget.h/cpp

表单小部件类，处理表单字段的交互。

| 方法 | 功能 |
|------|------|
| `GetFormField` | 获取表单字段 |
| `GetFieldType` | 获取字段类型 |
| `GetValue` | 获取字段值 |
| `SetValue` | 设置字段值 |
| `GetExportValue` | 获取导出值 |
| `IsChecked` | 是否选中 |
| `SetCheck` | 设置选中状态 |
| `OnXFAChangedByStruct` | XFA 结构变化 |

### 6. CPDFSDK_InteractiveForm 类

**位置**: cpdfsdk_interactiveform.h/cpp

SDK 层的交互式表单。

| 方法 | 功能 |
|------|------|
| `GetWidget` | 获取控件 |
| `GetFormField` | 获取字段 |
| `OnCalculate` | 计算事件 |
| `OnFormat` | 格式化事件 |
| `OnValidate` | 验证事件 |
| `OnKeyStroke` | 按键事件 |
| `ResetForm` | 重置表单 |
| `ExportFormToFDFTextBuf` | 导出为 FDF |

## API 实现文件

### fpdf_view.cpp

实现核心功能：

| API | 对应实现 |
|-----|----------|
| `FPDF_InitLibrary` | 初始化模块 |
| `FPDF_LoadDocument` | 创建 CPDF_Document |
| `FPDF_LoadPage` | 创建 CPDF_Page |
| `FPDF_RenderPageBitmap` | 调用渲染流程 |
| `FPDFBitmap_Create` | 创建 CFX_DIBitmap |

### fpdf_formfill.cpp

实现表单填充：

| API | 对应实现 |
|-----|----------|
| `FPDFDOC_InitFormFillEnvironment` | 创建环境 |
| `FORM_OnMouseMove` | 转发到 PageView |
| `FORM_OnLButtonDown` | 转发到 PageView |
| `FORM_OnChar` | 转发到 PageView |

### fpdf_editpage.cpp

实现页面编辑：

| API | 对应实现 |
|-----|----------|
| `FPDFPage_New` | 创建新页面 |
| `FPDFPage_InsertObject` | 添加对象 |
| `FPDFPage_GenerateContent` | 生成内容流 |

## 辅助函数

### cpdfsdk_helpers.cpp/h

提供 API 实现中的辅助函数：

| 函数类别 | 功能 |
|----------|------|
| 类型转换 | 句柄与对象互转 |
| 参数验证 | 检查空指针 |
| 坐标变换 | 页面坐标与设备坐标 |
| 字符串处理 | UTF-16LE 转换 |

## 事件处理流程

### 鼠标事件

```
FORM_OnLButtonDown()
    │
    ├── 获取 PageView
    │
    ├── PageView::OnMouseDown()
    │   │
    │   ├── 查找目标注释
    │   │
    │   └── 注释事件处理
    │       ├── Widget → FormFiller
    │       └── 其他注释类型
    │
    └── 更新显示
```

### 键盘事件

```
FORM_OnChar()
    │
    ├── 获取焦点注释
    │
    ├── 调用注释的 OnChar
    │   │
    │   └── Widget → FormFiller → PWL
    │
    └── 触发计算和格式化
```

## JavaScript 集成

表单填充环境与 JavaScript 运行时集成：

```
表单事件
    │
    ├── 查找关联脚本
    │
    ├── 创建事件上下文
    │
    ├── 执行脚本
    │   └── 使用 fxjs 模块
    │
    └── 处理结果
```

## 渲染集成

### cpdfsdk_renderpage.cpp

封装页面渲染逻辑：

```
CPDFSDK_RenderPage()
    │
    ├── 创建渲染上下文
    │
    ├── 渲染页面内容
    │
    ├── 渲染表单
    │
    └── 渲染注释
```

## XFA 集成

### fpdfxfa/ 子目录

提供 XFA 表单的集成支持，将 XFA 功能暴露给 SDK 层。

## 依赖关系

```
fpdfsdk/
    ├── public/          (API 定义)
    ├── core/fpdfapi/    (PDF 核心)
    ├── core/fpdfdoc/    (文档对象)
    ├── core/fpdftext/   (文本提取)
    ├── core/fxge/       (图形引擎)
    ├── fxjs/            (JavaScript)
    └── xfa/             (XFA 支持，可选)
```

## 线程安全

SDK 层遵循 PDFium 的线程模型：
- 所有操作必须在同一线程
- 或通过外部同步机制保护
