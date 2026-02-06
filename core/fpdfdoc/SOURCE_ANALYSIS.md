# core/fpdfdoc 源码分析

## 模块概述

`fpdfdoc` 模块处理 PDF 文档级别的结构和功能对象，包括书签、链接、注释、表单、元数据等高级文档特性。该模块是 SDK 层与核心解析层之间的桥梁。

## 目录结构

```
core/fpdfdoc/
├── BUILD.gn                          # 构建配置
├── cpdf_aaction.cpp/h                # 附加动作
├── cpdf_action.cpp/h                 # PDF 动作
├── cpdf_annot.cpp/h                  # 注释基类
├── cpdf_annotlist.cpp/h              # 注释列表
├── cpdf_apsettings.cpp/h             # 外观设置
├── cpdf_bafontmap.cpp/h              # 基本注释字体映射
├── cpdf_bookmark.cpp/h               # 书签
├── cpdf_bookmarktree.cpp/h           # 书签树
├── cpdf_dest.cpp/h                   # 目标位置
├── cpdf_filespec.cpp/h               # 文件规范
├── cpdf_formcontrol.cpp/h            # 表单控件
├── cpdf_formfield.cpp/h              # 表单字段
├── cpdf_generateap.cpp/h             # 外观流生成
├── cpdf_icon.cpp/h                   # 图标
├── cpdf_iconfit.cpp/h                # 图标适配
├── cpdf_interactiveform.cpp/h        # 交互式表单
├── cpdf_link.cpp/h                   # 链接
├── cpdf_linklist.cpp/h               # 链接列表
├── cpdf_metadata.cpp/h               # 元数据
├── cpdf_nametree.cpp/h               # 名称树
├── cpdf_numbertree.cpp/h             # 编号树
├── cpdf_pagelabel.cpp/h              # 页面标签
├── cpdf_structelement.cpp/h          # 结构元素
├── cpdf_structtree.cpp/h             # 结构树
├── cpdf_viewerpreferences.cpp/h      # 查看器首选项
└── cpvt_*.cpp/h                      # 可变文本相关类
```

## 核心类分析

### 1. 动作系统

#### CPDF_Action 类

**位置**: cpdf_action.h/cpp

表示 PDF 文档中的动作对象，支持多种动作类型：

| 动作类型 | 描述 |
|----------|------|
| `GoTo` | 跳转到文档内位置 |
| `GoToR` | 跳转到远程文档 |
| `GoToE` | 跳转到嵌入文档 |
| `Launch` | 启动应用程序 |
| `URI` | 打开 URI |
| `Hide` | 隐藏/显示注释 |
| `Named` | 执行命名动作 |
| `SubmitForm` | 提交表单 |
| `ResetForm` | 重置表单 |
| `JavaScript` | 执行 JavaScript |

#### CPDF_AAction 类

**位置**: cpdf_aaction.h/cpp

附加动作类，定义在特定事件触发时执行的动作：

| 事件类型 | 触发条件 |
|----------|----------|
| `kCursorEnter` | 鼠标进入区域 |
| `kCursorExit` | 鼠标离开区域 |
| `kButtonDown` | 鼠标按下 |
| `kButtonUp` | 鼠标释放 |
| `kGetFocus` | 获得焦点 |
| `kLoseFocus` | 失去焦点 |
| `kPageOpen` | 页面打开 |
| `kPageClose` | 页面关闭 |

### 2. 书签系统

#### CPDF_Bookmark 类

**位置**: cpdf_bookmark.h/cpp

表示 PDF 文档的书签（大纲项）：

| 方法 | 功能 |
|------|------|
| `GetTitle` | 获取书签标题 |
| `GetDest` | 获取目标位置 |
| `GetAction` | 获取关联动作 |
| `GetFirstChild` | 获取第一个子书签 |
| `GetNextSibling` | 获取下一个兄弟书签 |

#### CPDF_BookmarkTree 类

**位置**: cpdf_bookmarktree.h/cpp

管理整个书签树结构，提供遍历接口。

### 3. 注释系统

#### CPDF_Annot 类

**位置**: cpdf_annot.h/cpp

PDF 注释基类，支持多种注释类型：

| 子类型 | 描述 |
|--------|------|
| `Text` | 文本注释 |
| `Link` | 链接注释 |
| `FreeText` | 自由文本 |
| `Line` | 线条 |
| `Square` | 矩形 |
| `Circle` | 椭圆 |
| `Highlight` | 高亮 |
| `Underline` | 下划线 |
| `StrikeOut` | 删除线 |
| `Stamp` | 印章 |
| `Ink` | 墨迹 |
| `Popup` | 弹出窗口 |
| `FileAttachment` | 文件附件 |
| `Widget` | 表单控件 |

主要方法：

| 方法 | 功能 |
|------|------|
| `GetSubtype` | 获取注释子类型 |
| `GetRect` | 获取注释矩形 |
| `GetFlags` | 获取注释标志 |
| `DrawAppearance` | 绘制注释外观 |

#### CPDF_AnnotList 类

**位置**: cpdf_annotlist.h/cpp

管理页面上的所有注释，按 Z 顺序组织。

### 4. 表单系统

#### CPDF_InteractiveForm 类

**位置**: cpdf_interactiveform.h/cpp

交互式表单（AcroForm）的核心类：

| 方法 | 功能 |
|------|------|
| `CountFields` | 获取字段数量 |
| `GetField` | 获取指定字段 |
| `FindFieldByName` | 按名称查找字段 |
| `GetFormFont` | 获取表单字体 |
| `ExportToFDF` | 导出为 FDF 格式 |
| `ImportFromFDF` | 从 FDF 导入 |

#### CPDF_FormField 类

**位置**: cpdf_formfield.h/cpp

表单字段类，支持以下字段类型：

| 类型 | 描述 |
|------|------|
| `kPushButton` | 按钮 |
| `kCheckBox` | 复选框 |
| `kRadioButton` | 单选按钮 |
| `kComboBox` | 下拉框 |
| `kListBox` | 列表框 |
| `kTextField` | 文本框 |
| `kSignature` | 签名域 |

主要方法：

| 方法 | 功能 |
|------|------|
| `GetType` | 获取字段类型 |
| `GetFullName` | 获取完整字段名 |
| `GetValue` | 获取字段值 |
| `SetValue` | 设置字段值 |
| `GetMaxLen` | 获取最大长度 |
| `GetOptions` | 获取选项列表 |
| `IsRequired` | 是否必填 |
| `IsReadOnly` | 是否只读 |

#### CPDF_FormControl 类

**位置**: cpdf_formcontrol.h/cpp

表单控件类，表示字段在页面上的具体呈现：

| 方法 | 功能 |
|------|------|
| `GetWidget` | 获取关联的注释 |
| `GetRect` | 获取控件矩形 |
| `GetExportValue` | 获取导出值 |
| `IsChecked` | 是否选中 |
| `GetNormalIcon` | 获取正常状态图标 |
| `GetHighlightingMode` | 获取高亮模式 |

### 5. 导航系统

#### CPDF_Dest 类

**位置**: cpdf_dest.h/cpp

目标位置类，定义文档内的跳转目标：

| 目标类型 | 描述 |
|----------|------|
| `XYZ` | 指定坐标和缩放 |
| `Fit` | 适合整页 |
| `FitH` | 适合宽度 |
| `FitV` | 适合高度 |
| `FitR` | 适合矩形区域 |
| `FitB` | 适合边界框 |

#### CPDF_Link 类

**位置**: cpdf_link.h/cpp

链接注释的封装，提供目标和动作访问。

#### CPDF_LinkList 类

**位置**: cpdf_linklist.h/cpp

管理页面上的所有链接，支持点击测试。

### 6. 文档结构

#### CPDF_StructTree 类

**位置**: cpdf_structtree.h/cpp

结构树类，用于辅助功能（Accessibility）支持：

| 方法 | 功能 |
|------|------|
| `CountTopElements` | 获取顶级元素数量 |
| `GetTopElement` | 获取顶级元素 |

#### CPDF_StructElement 类

**位置**: cpdf_structelement.h/cpp

结构元素类，表示文档的逻辑结构：

| 方法 | 功能 |
|------|------|
| `GetType` | 获取结构类型（如 P, H1, Table） |
| `GetTitle` | 获取标题 |
| `GetAltText` | 获取替代文本 |
| `CountKids` | 获取子元素数量 |

### 7. 元数据

#### CPDF_Metadata 类

**位置**: cpdf_metadata.h/cpp

处理 PDF 文档的 XMP 元数据。

#### CPDF_ViewerPreferences 类

**位置**: cpdf_viewerpreferences.h/cpp

查看器首选项，控制 PDF 阅读器的行为：

| 首选项 | 描述 |
|--------|------|
| `HideToolbar` | 隐藏工具栏 |
| `HideMenubar` | 隐藏菜单栏 |
| `HideWindowUI` | 隐藏窗口 UI |
| `FitWindow` | 调整窗口大小 |
| `CenterWindow` | 居中窗口 |
| `DisplayDocTitle` | 显示文档标题 |
| `Direction` | 阅读方向 |

### 8. 名称和编号树

#### CPDF_NameTree 类

**位置**: cpdf_nametree.h/cpp

名称树实现，用于：
- 命名目标
- JavaScript 动作
- 嵌入文件

#### CPDF_NumberTree 类

**位置**: cpdf_numbertree.h/cpp

编号树实现，用于：
- 页面标签
- 结构树父元素

### 9. 可变文本系统

#### CPVT_VariableText 类

**位置**: cpvt_variabletext.h/cpp

可变文本编辑核心，用于表单字段文本编辑：

| 功能 | 描述 |
|------|------|
| 文本输入 | 处理用户输入 |
| 光标管理 | 跟踪光标位置 |
| 选择处理 | 文本选择 |
| 自动换行 | 多行文本支持 |
| 对齐 | 左/中/右对齐 |

相关辅助类：
- `CPVT_Section`: 文本段落
- `CPVT_Line`: 文本行
- `CPVT_Word`: 单个字符
- `CPVT_WordPlace`: 字符位置
- `CPVT_WordRange`: 字符范围

## 外观生成

### CPDF_GenerateAP 命名空间

**位置**: cpdf_generateap.h/cpp

为表单字段和注释生成外观流：

| 函数 | 功能 |
|------|------|
| `GenerateFormAP` | 生成表单外观 |
| `GenerateEmptyAP` | 生成空外观 |

## 依赖关系

```
fpdfdoc/
    ├── fpdfapi/parser/   (PDF 对象解析)
    ├── fpdfapi/page/     (页面对象)
    ├── fpdfapi/font/     (字体处理)
    └── fxcrt/            (基础设施)
```

## 公共 API 映射

| fpdfdoc 类 | 公共 API 文件 |
|------------|---------------|
| CPDF_Bookmark | fpdf_doc.h |
| CPDF_Action | fpdf_doc.h |
| CPDF_Dest | fpdf_doc.h |
| CPDF_Link | fpdf_doc.h |
| CPDF_Annot | fpdf_annot.h |
| CPDF_FormField | fpdf_formfill.h |
| CPDF_StructTree | fpdf_structtree.h |
| CPDF_Metadata | fpdf_doc.h |
