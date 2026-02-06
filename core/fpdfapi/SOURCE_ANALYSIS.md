# core/fpdfapi 源码分析

## 模块概述

`fpdfapi` 是 PDF API 的核心实现模块，负责 PDF 格式的解析、页面处理、渲染、字体管理和编辑功能。它是 PDFium 中最核心和最复杂的模块。

## 目录结构

```
core/fpdfapi/
├── parser/               # PDF 解析器
├── page/                 # 页面处理
├── render/               # 渲染引擎
├── font/                 # 字体处理
├── edit/                 # 编辑功能
└── cmaps/                # 字符映射表
```

## 子模块职责

| 子模块 | 职责 |
|--------|------|
| `parser` | PDF 文件解析、对象模型、交叉引用、加密 |
| `page` | 页面内容解析、页面对象、颜色空间、图像 |
| `render` | 渲染流程、文本渲染、图像渲染、着色 |
| `font` | 字体加载、编码、CMap、字形获取 |
| `edit` | 内容生成、文档保存、页面操作 |
| `cmaps` | CJK 字符到 CID 的映射数据 |

## 核心类概览

### parser/

| 类 | 职责 |
|----|------|
| `CPDF_Document` | PDF 文档 |
| `CPDF_Parser` | 文件解析 |
| `CPDF_SyntaxParser` | 语法分析 |
| `CPDF_Object` | 对象基类 |
| `CPDF_Dictionary` | 字典对象 |
| `CPDF_Array` | 数组对象 |
| `CPDF_Stream` | 流对象 |
| `CPDF_CrossRefTable` | 交叉引用表 |
| `CPDF_SecurityHandler` | 加密处理 |

### page/

| 类 | 职责 |
|----|------|
| `CPDF_Page` | PDF 页面 |
| `CPDF_PageObject` | 页面对象基类 |
| `CPDF_TextObject` | 文本对象 |
| `CPDF_PathObject` | 路径对象 |
| `CPDF_ImageObject` | 图像对象 |
| `CPDF_ColorSpace` | 颜色空间 |
| `CPDF_ContentParser` | 内容解析 |
| `CPDF_StreamContentParser` | 流内容解析 |
| `CPDF_Image` | 图像资源 |
| `CPDF_Pattern` | 图案 |
| `CPDF_Function` | PDF 函数 |

### render/

| 类 | 职责 |
|----|------|
| `CPDF_RenderContext` | 渲染上下文 |
| `CPDF_RenderStatus` | 渲染状态 |
| `CPDF_RenderOptions` | 渲染选项 |
| `CPDF_ProgressiveRenderer` | 渐进式渲染 |
| `CPDF_TextRenderer` | 文本渲染 |
| `CPDF_ImageRenderer` | 图像渲染 |

### font/

| 类 | 职责 |
|----|------|
| `CPDF_Font` | 字体基类 |
| `CPDF_Type1Font` | Type1 字体 |
| `CPDF_TrueTypeFont` | TrueType 字体 |
| `CPDF_CIDFont` | CID 字体 |
| `CPDF_Type3Font` | Type3 字体 |
| `CPDF_CMap` | 字符映射 |
| `CPDF_ToUnicodeMap` | Unicode 映射 |
| `CPDF_FontGlobals` | 字体全局 |

### edit/

| 类 | 职责 |
|----|------|
| `CPDF_Creator` | 文档创建器 |
| `CPDF_PageContentGenerator` | 内容生成 |
| `CPDF_PageExporter` | 页面导出 |

## 核心流程

### 文档加载

```
FPDF_LoadDocument()
    │
    ├── CPDF_Document::LoadDoc()
    │   │
    │   ├── CPDF_Parser::StartParse()
    │   │   ├── 解析 PDF 头
    │   │   ├── 加载交叉引用表
    │   │   ├── 解析 trailer
    │   │   └── 处理加密
    │   │
    │   └── TryInit()
    │       ├── 获取 Catalog
    │       └── 构建页面树
    │
    └── 返回文档句柄
```

### 页面加载

```
FPDF_LoadPage()
    │
    ├── CPDF_Document::GetPage()
    │   │
    │   ├── 获取页面字典
    │   │
    │   └── 创建 CPDF_Page
    │
    └── 返回页面句柄
```

### 页面解析

```
CPDF_Page::ParseContent()
    │
    ├── 获取 Contents 流
    │
    ├── CPDF_ContentParser
    │   │
    │   ├── 读取操作符和操作数
    │   │
    │   ├── CPDF_StreamContentParser 处理
    │   │   ├── 路径操作 → CPDF_PathObject
    │   │   ├── 文本操作 → CPDF_TextObject
    │   │   ├── 图像操作 → CPDF_ImageObject
    │   │   └── 状态操作 → 更新状态
    │   │
    │   └── 添加到对象列表
    │
    └── 完成
```

### 页面渲染

```
FPDF_RenderPageBitmap()
    │
    ├── 创建 CFX_RenderDevice
    │
    ├── CPDF_RenderContext::Render()
    │   │
    │   ├── CPDF_RenderStatus 处理
    │   │   │
    │   │   ├── 遍历页面对象
    │   │   │
    │   │   └── 根据类型渲染
    │   │       ├── 路径 → 绑制路径
    │   │       ├── 文本 → TextRenderer
    │   │       ├── 图像 → ImageRenderer
    │   │       └── 着色 → RenderShading
    │   │
    │   └── 完成
    │
    └── 输出位图
```

## PDF 对象模型

### 对象层次

```
CPDF_Object (基类)
├── CPDF_Boolean    # 布尔
├── CPDF_Number     # 数字
├── CPDF_String     # 字符串
├── CPDF_Name       # 名称
├── CPDF_Array      # 数组
├── CPDF_Dictionary # 字典
├── CPDF_Stream     # 流
├── CPDF_Null       # 空
└── CPDF_Reference  # 引用
```

### 页面对象层次

```
CPDF_PageObject (基类)
├── CPDF_TextObject    # 文本
├── CPDF_PathObject    # 路径
├── CPDF_ImageObject   # 图像
├── CPDF_ShadingObject # 着色
└── CPDF_FormObject    # 表单 XObject
```

## 颜色空间

| 类型 | 类 |
|------|-----|
| 设备颜色 | `CPDF_DeviceCS` (Gray/RGB/CMYK) |
| 校准颜色 | `CPDF_CalGray`, `CPDF_CalRGB`, `CPDF_LabCS` |
| ICC | `CPDF_ICCBasedCS` |
| 索引 | `CPDF_IndexedCS` |
| 分色 | `CPDF_SeparationCS` |
| DeviceN | `CPDF_DeviceNCS` |
| 图案 | `CPDF_PatternCS` |

## PDF 函数

| 类型 | 类 |
|------|-----|
| 采样函数 | `CPDF_SampledFunc` |
| 指数函数 | `CPDF_ExpIntFunc` |
| 拼接函数 | `CPDF_StitchFunc` |
| PS 函数 | `CPDF_PSFunc` |

## 依赖关系

```
fpdfapi/
├── parser/
│   ├── fxcrt/     # 基础设施
│   └── fdrm/      # 加密
│
├── page/
│   ├── parser/    # PDF 对象
│   ├── font/      # 字体
│   └── fxcodec/   # 图像解码
│
├── render/
│   ├── page/      # 页面对象
│   ├── font/      # 字体
│   └── fxge/      # 图形引擎
│
├── font/
│   ├── parser/    # PDF 对象
│   ├── fxge/      # 字体渲染
│   └── cmaps/     # CMap 数据
│
├── edit/
│   ├── parser/    # PDF 对象
│   └── page/      # 页面对象
│
└── cmaps/
    └── fxcrt/     # 基础设施
```

## 详细分析

各子模块的详细分析：

- [parser 分析](parser/SOURCE_ANALYSIS.md)
- [page 分析](page/SOURCE_ANALYSIS.md)
- [render 分析](render/SOURCE_ANALYSIS.md)
- [font 分析](font/SOURCE_ANALYSIS.md)
- [edit 分析](edit/SOURCE_ANALYSIS.md)
- [cmaps 分析](cmaps/SOURCE_ANALYSIS.md)
