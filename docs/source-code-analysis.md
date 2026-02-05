# PDFium 源码分析文档

## 目录

1. [概述](#概述)
2. [整体架构](#整体架构)
3. [目录结构](#目录结构)
4. [核心模块详解](#核心模块详解)
   - [公共 API 层 (public/)](#公共-api-层-public)
   - [SDK 层 (fpdfsdk/)](#sdk-层-fpdfsdk)
   - [核心层 (core/)](#核心层-core)
   - [XFA 支持 (xfa/)](#xfa-支持-xfa)
   - [JavaScript 引擎 (fxjs/)](#javascript-引擎-fxjs)
   - [条形码支持 (fxbarcode/)](#条形码支持-fxbarcode)
5. [核心数据结构](#核心数据结构)
6. [关键流程分析](#关键流程分析)
7. [模块依赖关系](#模块依赖关系)

---

## 概述

PDFium 是由 Google 开发和维护的开源 PDF 渲染引擎，最初基于 Foxit Software 的代码。它被用于 Chrome 浏览器中渲染 PDF 文件，也可以作为独立库嵌入到其他应用程序中。

### 主要特性

- **PDF 解析与渲染**：完整支持 PDF 1.7 标准
- **文本提取**：从 PDF 中提取文本内容
- **表单支持**：支持 AcroForm 和 XFA 表单
- **JavaScript 支持**：通过 V8 引擎支持 PDF JavaScript
- **编辑功能**：支持创建和修改 PDF 文档
- **跨平台**：支持 Windows、Linux、macOS、Android 等平台

### 构建系统

PDFium 使用与 Chromium 相同的构建工具链：
- **GN**：用于生成构建文件
- **Ninja**：用于执行构建

---

## 整体架构

PDFium 采用分层架构设计，从上到下可分为以下几层：

```
┌─────────────────────────────────────────────────────────────────┐
│                    嵌入器应用 (Embedder)                         │
├─────────────────────────────────────────────────────────────────┤
│                    公共 API 层 (public/)                         │
│  fpdfview.h, fpdf_edit.h, fpdf_text.h, fpdf_formfill.h, ...     │
├─────────────────────────────────────────────────────────────────┤
│                    SDK 层 (fpdfsdk/)                             │
│  CPDFSDK_*, 表单填充, 注释, 页面视图管理                          │
├───────────────────────┬─────────────────────────────────────────┤
│   XFA 支持 (xfa/)     │        JavaScript (fxjs/)                │
│   XFA 表单处理        │        V8 集成, JS 对象绑定               │
├───────────────────────┴─────────────────────────────────────────┤
│                    核心层 (core/)                                │
│  ┌─────────────┬──────────────┬────────────┬──────────────────┐ │
│  │  fpdfapi/   │   fpdfdoc/   │  fpdftext/ │    fxcodec/      │ │
│  │  PDF 解析   │   文档对象   │  文本提取  │    编解码器       │ │
│  ├─────────────┼──────────────┼────────────┼──────────────────┤ │
│  │   fxge/     │    fxcrt/    │   fdrm/    │                  │ │
│  │  图形引擎   │   运行时库   │  数字版权  │                  │ │
│  └─────────────┴──────────────┴────────────┴──────────────────┘ │
├─────────────────────────────────────────────────────────────────┤
│                    第三方库 (third_party/)                       │
│  FreeType, libjpeg, zlib, libpng, ICU, V8, Skia, ...            │
└─────────────────────────────────────────────────────────────────┘
```

---

## 目录结构

```
pdfium/
├── public/              # 公共 API 头文件（嵌入器使用的接口）
├── fpdfsdk/             # SDK 实现层
│   ├── formfiller/      # 表单填充器
│   ├── pwl/             # 轻量级窗口小部件
│   └── fpdfxfa/         # XFA 扩展集成
├── core/                # 核心实现
│   ├── fpdfapi/         # PDF API 核心实现
│   │   ├── parser/      # PDF 解析器
│   │   ├── page/        # 页面处理
│   │   ├── render/      # 渲染引擎
│   │   ├── font/        # 字体处理
│   │   ├── edit/        # 编辑功能
│   │   └── cmaps/       # 字符映射表
│   ├── fpdfdoc/         # PDF 文档对象
│   ├── fpdftext/        # 文本提取
│   ├── fxcodec/         # 编解码器
│   ├── fxcrt/           # 核心运行时
│   ├── fxge/            # 图形引擎
│   └── fdrm/            # 数字版权管理
├── xfa/                 # XFA 表单支持
│   ├── fxfa/            # XFA 核心
│   ├── fgas/            # 图形和脚本
│   ├── fwl/             # 窗口部件库
│   └── fde/             # 设备扩展
├── fxjs/                # JavaScript 支持
├── fxbarcode/           # 条形码支持
├── testing/             # 测试相关
├── samples/             # 示例代码
├── third_party/         # 第三方依赖
├── build_overrides/     # 构建配置覆盖
├── skia/                # Skia 后端支持
└── docs/                # 文档
```

---

## 核心模块详解

### 公共 API 层 (public/)

公共 API 层是嵌入器与 PDFium 交互的唯一接口。所有 API 都是 C 语言接口，确保二进制兼容性。

#### 主要头文件

| 头文件 | 功能描述 |
|--------|----------|
| `fpdfview.h` | 核心 API：初始化、文档加载、页面渲染 |
| `fpdf_edit.h` | 编辑 API：创建和修改 PDF 文档 |
| `fpdf_text.h` | 文本 API：文本提取和搜索 |
| `fpdf_formfill.h` | 表单 API：交互式表单处理 |
| `fpdf_annot.h` | 注释 API：注释读取和创建 |
| `fpdf_doc.h` | 文档 API：书签、链接、元数据 |
| `fpdf_save.h` | 保存 API：保存修改后的 PDF |
| `fpdf_progressive.h` | 渐进式渲染 API |
| `fpdf_signature.h` | 数字签名 API |
| `fpdf_structtree.h` | 结构树 API：辅助功能支持 |

#### 核心类型定义

```c
// 文档句柄
typedef struct fpdf_document_t__* FPDF_DOCUMENT;

// 页面句柄
typedef struct fpdf_page_t__* FPDF_PAGE;

// 位图句柄
typedef struct fpdf_bitmap_t__* FPDF_BITMAP;

// 表单句柄
typedef struct fpdf_form_handle_t__* FPDF_FORMHANDLE;

// 文本页面句柄
typedef struct fpdf_textpage_t__* FPDF_TEXTPAGE;

// 页面对象句柄 (文本、路径、图像等)
typedef struct fpdf_pageobject_t__* FPDF_PAGEOBJECT;
```

#### 核心 API 示例

```c
// 初始化库
FPDF_InitLibrary();

// 加载文档
FPDF_DOCUMENT doc = FPDF_LoadDocument("file.pdf", "password");

// 获取页面数量
int page_count = FPDF_GetPageCount(doc);

// 加载页面
FPDF_PAGE page = FPDF_LoadPage(doc, 0);

// 创建位图
FPDF_BITMAP bitmap = FPDFBitmap_Create(width, height, 0);

// 渲染页面到位图
FPDF_RenderPageBitmap(bitmap, page, 0, 0, width, height, 0, 0);

// 清理资源
FPDF_ClosePage(page);
FPDF_CloseDocument(doc);
FPDF_DestroyLibrary();
```

---

### SDK 层 (fpdfsdk/)

SDK 层实现了公共 API 的功能，并提供了与嵌入器交互所需的高级抽象。

#### 主要组件

##### CPDFSDK_FormFillEnvironment
表单填充环境，管理表单交互的核心类。

```cpp
class CPDFSDK_FormFillEnvironment {
  // 管理页面视图
  std::map<IPDF_Page*, std::unique_ptr<CPDFSDK_PageView>> page_map_;
  
  // 交互式表单
  std::unique_ptr<CPDFSDK_InteractiveForm> interactive_form_;
  
  // 当前焦点注释
  ObservedPtr<CPDFSDK_Annot> focus_annot_;
  
  // JavaScript 运行时
  std::unique_ptr<IJS_Runtime> ijs_runtime_;
};
```

##### CPDFSDK_PageView
页面视图类，管理单个页面的注释和交互。

```cpp
class CPDFSDK_PageView {
  // 页面上的注释列表
  std::vector<std::unique_ptr<CPDFSDK_Annot>> annots_;
  
  // 关联的页面对象
  IPDF_Page* page_;
};
```

##### CPDFSDK_Annot
注释基类，是所有注释类型的父类。

##### CPDFSDK_Widget
表单小部件类，处理表单字段的交互。

#### 表单填充 (formfiller/)

表单填充子系统处理用户与 PDF 表单的交互：

- `CFFL_InteractiveFormFiller`：表单填充器主类
- `CFFL_FormField`：表单字段基类
- `CFFL_TextField`：文本字段
- `CFFL_CheckBox`：复选框
- `CFFL_ComboBox`：下拉列表
- `CFFL_ListBox`：列表框
- `CFFL_PushButton`：按钮

#### 轻量级窗口 (pwl/)

PWL (Portable Widget Library) 提供跨平台的 UI 组件：

- `CPWL_Wnd`：窗口基类
- `CPWL_Edit`：编辑框
- `CPWL_Button`：按钮
- `CPWL_ListBox`：列表框
- `CPWL_ComboBox`：下拉框

---

### 核心层 (core/)

核心层包含 PDFium 的主要功能实现。

#### fpdfapi/ - PDF API 核心

##### parser/ - PDF 解析器

PDF 解析器负责读取和解释 PDF 文件结构。

**CPDF_Parser**：PDF 文件解析器
```cpp
class CPDF_Parser {
  // 解析错误类型
  enum Error {
    SUCCESS = 0,
    FILE_ERROR,
    FORMAT_ERROR,
    PASSWORD_ERROR,
    HANDLER_ERROR
  };
  
  // 开始解析
  Error StartParse(RetainPtr<IFX_SeekableReadStream> pFile,
                   const ByteString& password);
  
  // 解析间接对象
  RetainPtr<CPDF_Object> ParseIndirectObject(uint32_t objnum);
  
private:
  std::unique_ptr<CPDF_SyntaxParser> syntax_;       // 语法解析器
  std::unique_ptr<CPDF_CrossRefTable> cross_ref_table_;  // 交叉引用表
  RetainPtr<CPDF_SecurityHandler> security_handler_;     // 安全处理器
};
```

**CPDF_Document**：PDF 文档对象
```cpp
class CPDF_Document {
  // 获取页面数量
  int GetPageCount() const;
  
  // 获取页面字典
  RetainPtr<CPDF_Dictionary> GetMutablePageDictionary(int iPage);
  
  // 加载文档
  CPDF_Parser::Error LoadDoc(RetainPtr<IFX_SeekableReadStream> pFileAccess,
                             const ByteString& password);

private:
  std::unique_ptr<CPDF_Parser> parser_;          // 解析器
  RetainPtr<CPDF_Dictionary> root_dict_;         // 根字典
  std::vector<uint32_t> page_list_;              // 页面对象号列表
};
```

**PDF 对象类型**：
- `CPDF_Object`：所有 PDF 对象的基类
- `CPDF_Boolean`：布尔对象
- `CPDF_Number`：数字对象
- `CPDF_String`：字符串对象
- `CPDF_Name`：名称对象
- `CPDF_Array`：数组对象
- `CPDF_Dictionary`：字典对象
- `CPDF_Stream`：流对象
- `CPDF_Reference`：引用对象

##### page/ - 页面处理

**CPDF_Page**：PDF 页面对象
```cpp
class CPDF_Page : public IPDF_Page, public CPDF_PageObjectHolder {
  // 解析页面内容
  void ParseContent();
  
  // 获取页面尺寸
  float GetPageWidth() const;
  float GetPageHeight() const;
  
  // 坐标转换
  std::optional<CFX_PointF> DeviceToPage(const FX_RECT& rect,
                                          int rotation,
                                          const CFX_PointF& device_point) const;

private:
  CFX_SizeF page_size_;                                    // 页面尺寸
  CFX_Matrix page_matrix_;                                 // 页面变换矩阵
  std::unique_ptr<CPDF_PageImageCache> page_image_cache_;  // 图像缓存
};
```

**CPDF_PageObjectHolder**：页面对象容器
管理页面上的各种对象（文本、图像、路径、表单等）。

**页面对象类型**：
- `CPDF_TextObject`：文本对象
- `CPDF_PathObject`：路径对象
- `CPDF_ImageObject`：图像对象
- `CPDF_ShadingObject`：着色对象
- `CPDF_FormObject`：表单 XObject

**CPDF_ContentParser**：内容流解析器
解析页面内容流，生成页面对象。

**CPDF_StreamContentParser**：流内容解析器
处理 PDF 操作符和操作数。

##### render/ - 渲染引擎

**CPDF_RenderContext**：渲染上下文
```cpp
class CPDF_RenderContext {
  // 渲染页面对象
  void Render(CFX_RenderDevice* pDevice,
              const CPDF_PageObject* pStopObj,
              const CPDF_RenderOptions* pOptions,
              const CFX_Matrix* pLastMatrix);
};
```

**CPDF_RenderStatus**：渲染状态
管理渲染过程中的状态信息。

**CPDF_ProgressiveRenderer**：渐进式渲染器
支持分块渲染大型页面。

**CPDF_ImageRenderer**：图像渲染器
处理图像对象的渲染。

**CPDF_TextRenderer**：文本渲染器
处理文本对象的渲染。

##### font/ - 字体处理

**CPDF_Font**：字体基类
```cpp
class CPDF_Font {
  // 获取字符宽度
  virtual int GetCharWidthF(uint32_t charcode);
  
  // 编码转换
  virtual WideString UnicodeFromCharCode(uint32_t charcode) const;
  virtual uint32_t CharCodeFromUnicode(wchar_t unicode) const;
};
```

**字体类型**：
- `CPDF_Type1Font`：Type1 字体
- `CPDF_TrueTypeFont`：TrueType 字体
- `CPDF_CIDFont`：CID 字体
- `CPDF_Type3Font`：Type3 字体（用户定义字形）

**CPDF_FontGlobals**：字体全局管理
管理字体缓存和 CMap 数据。

##### edit/ - 编辑功能

提供 PDF 创建和修改功能：
- 创建新页面
- 添加/删除页面对象
- 修改对象属性

#### fpdfdoc/ - PDF 文档对象

处理 PDF 文档级别的结构和功能。

**主要类**：
- `CPDF_Action`：动作对象
- `CPDF_Bookmark`：书签
- `CPDF_BookmarkTree`：书签树
- `CPDF_Dest`：目标（跳转位置）
- `CPDF_Link`：链接
- `CPDF_FormField`：表单字段
- `CPDF_InteractiveForm`：交互式表单
- `CPDF_Annot`：注释
- `CPDF_AnnotList`：注释列表
- `CPDF_Metadata`：元数据
- `CPDF_StructTree`：结构树（辅助功能）
- `CPDF_ViewerPreferences`：查看器首选项

#### fpdftext/ - 文本提取

**CPDF_TextPage**：文本页面
从 PDF 页面提取文本内容，支持：
- 获取字符 Unicode
- 获取字符位置
- 文本搜索
- 文本选择

**CPDF_TextPageFind**：文本搜索
实现文本查找功能。

**CPDF_LinkExtract**：链接提取
从文本中提取 URL 链接。

#### fxcodec/ - 编解码器

处理 PDF 中使用的各种编码格式。

**支持的编解码器**：

| 模块 | 功能 |
|------|------|
| `basic/` | 基本解码器（ASCII85、ASCIIHex、RunLength） |
| `flate/` | Flate/Deflate 压缩 |
| `jpeg/` | JPEG 图像 |
| `jpx/` | JPEG2000 图像 |
| `jbig2/` | JBIG2 图像压缩 |
| `fax/` | CCITT 传真压缩 |
| `icc/` | ICC 颜色配置文件 |
| `png/` | PNG 图像 |
| `gif/` | GIF 图像 |
| `bmp/` | BMP 图像 |
| `tiff/` | TIFF 图像 |

**CPDF_DIB**：设备无关位图
处理 PDF 图像对象的解码。

**ProgressiveDecoder**：渐进式解码器
支持大图像的渐进式解码。

#### fxge/ - 图形引擎

图形引擎负责所有的绘制操作。

**CFX_RenderDevice**：渲染设备基类
```cpp
class CFX_RenderDevice {
  // 绘制路径
  bool DrawPath(const CFX_Path& path,
                const CFX_Matrix* pObject2Device,
                const CFX_GraphStateData* pGraphState,
                uint32_t fill_color,
                uint32_t stroke_color,
                const CFX_FillRenderOptions& fill_options);
  
  // 绘制文本
  bool DrawNormalText(pdfium::span<const TextCharPos> pCharPos,
                      CFX_Font* pFont,
                      float font_size,
                      const CFX_Matrix& mtText2Device,
                      uint32_t fill_color,
                      const CFX_TextRenderOptions& options);
  
  // 绘制位图
  bool SetDIBits(RetainPtr<const CFX_DIBBase> bitmap,
                 int left,
                 int top);
};
```

**CFX_DIBitmap**：设备无关位图
```cpp
class CFX_DIBitmap : public CFX_DIBBase {
  // 创建位图
  bool Create(int width, int height, FXDIB_Format format);
  
  // 获取像素数据
  pdfium::span<uint8_t> GetWritableBuffer();
};
```

**CFX_Font**：字体对象
封装 FreeType 字体处理。

**CFX_FontMgr**：字体管理器
管理字体加载和缓存。

**CFX_GEModule**：图形引擎模块
图形引擎的全局初始化和清理。

**渲染后端**：
- `agg/`：AGG（Anti-Grain Geometry）软件渲染
- `skia/`：Skia 图形库后端（实验性）
- `win32/`：Windows GDI 渲染

**dib/ - 设备无关位图**：
- `CFX_DIBBase`：位图基类
- `CFX_DIBitmap`：可写位图
- `CFX_ScanlineCompositor`：扫描线合成器
- `CFX_ImageStretcher`：图像拉伸
- `CFX_ImageTransformer`：图像变换

#### fxcrt/ - 核心运行时

提供跨平台的基础设施和实用工具。

**字符串类**：
- `ByteString`：字节字符串
- `WideString`：宽字符串（UTF-16）
- `ByteStringView`：字节字符串视图
- `WideStringView`：宽字符串视图

**智能指针**：
- `RetainPtr<T>`：引用计数智能指针
- `UnownedPtr<T>`：非拥有指针（调试用）
- `ObservedPtr<T>`：观察者指针

**容器和工具**：
- `DataVector<T>`：数据向量
- `FixedSizeDataVector<T, N>`：固定大小向量
- `BinaryBuffer`：二进制缓冲区

**流接口**：
- `IFX_SeekableReadStream`：可寻址读取流
- `IFX_SeekableWriteStream`：可寻址写入流
- `CFX_MemoryStream`：内存流

**其他工具**：
- `CFX_DateTime`：日期时间
- `CFX_Timer`：定时器
- `fx_coordinates.h`：坐标和矩阵类型
- `fx_memory.h`：内存分配

#### fdrm/ - 数字版权管理

处理 PDF 加密和权限。

- `fx_crypt.h`：加密算法
- `fx_crypt_aes.h`：AES 加密
- `fx_crypt_sha.h`：SHA 哈希

---

### XFA 支持 (xfa/)

XFA (XML Forms Architecture) 是 Adobe 的动态表单标准。

#### 模块结构

```
xfa/
├── fxfa/          # XFA 核心
│   ├── parser/    # XFA XML 解析
│   ├── layout/    # 布局引擎
│   └── formcalc/  # FormCalc 脚本语言
├── fgas/          # 图形和脚本支持
├── fwl/           # 窗口部件库
└── fde/           # 设备扩展
```

#### 主要类

**CXFA_FFDoc**：XFA 表单文档
```cpp
class CXFA_FFDoc {
  // 获取表单视图
  CXFA_FFDocView* CreateDocView();
};
```

**CXFA_FFDocView**：文档视图
管理 XFA 表单的显示和交互。

**CXFA_FFWidget**：XFA 控件基类
所有 XFA 表单控件的基类。

**XFA 控件类型**：
- `CXFA_FFText`：文本
- `CXFA_FFImage`：图像
- `CXFA_FFTextField`：文本字段
- `CXFA_FFCheckButton`：复选框
- `CXFA_FFComboBox`：下拉框
- `CXFA_FFListBox`：列表框
- `CXFA_FFPushButton`：按钮
- `CXFA_FFSignature`：签名
- `CXFA_FFBarcode`：条形码

---

### JavaScript 引擎 (fxjs/)

PDFium 通过 V8 引擎支持 PDF JavaScript。

#### 架构

```cpp
// JavaScript 运行时接口
class IJS_Runtime {
  virtual IJS_EventContext* NewEventContext() = 0;
  virtual void ReleaseEventContext(IJS_EventContext* context) = 0;
};

// JavaScript 事件上下文
class IJS_EventContext {
  virtual bool RunScript(const WideString& script) = 0;
};

// V8 引擎封装
class CFX_V8 {
  // V8 隔离区管理
  v8::Isolate* GetIsolate() const;
};

// JavaScript 引擎
class CJS_Runtime : public IJS_Runtime {
  // 创建 JavaScript 对象
  v8::Local<v8::Object> NewJSObject(IJS_Constructor* pJS);
};
```

#### JavaScript 对象绑定

PDFium 实现了 Adobe Acrobat JavaScript API 的子集：

| 类 | 描述 |
|----|------|
| `CJS_Document` | document 对象 |
| `CJS_App` | app 对象 |
| `CJS_Field` | 表单字段对象 |
| `CJS_Event` | event 对象 |
| `CJS_Console` | console 对象 |
| `CJS_Util` | 工具函数 |
| `CJS_Color` | 颜色对象 |
| `CJS_Global` | 全局对象 |

---

### 条形码支持 (fxbarcode/)

支持生成多种条形码格式。

#### 支持的条形码类型

**一维码 (oned/)**：
- Code 39
- Code 128
- Codabar
- EAN-8
- EAN-13
- UPC-A

**二维码**：
- QR Code (`qrcode/`)
- PDF417 (`pdf417/`)
- Data Matrix (`datamatrix/`)

#### 主要类

```cpp
class CFX_Barcode {
  // 创建条形码
  bool Create(BC_TYPE type);
  
  // 编码数据
  bool Encode(WideStringView contents);
  
  // 渲染到设备
  bool RenderDevice(CFX_RenderDevice* device,
                    const CFX_Matrix& matrix);
};
```

---

## 核心数据结构

### PDF 对象模型

```
CPDF_Object (基类)
├── CPDF_Boolean
├── CPDF_Number
├── CPDF_String
├── CPDF_Name
├── CPDF_Array
├── CPDF_Dictionary
├── CPDF_Stream
├── CPDF_Null
└── CPDF_Reference
```

### 页面对象模型

```
CPDF_PageObject (基类)
├── CPDF_TextObject    # 文本
├── CPDF_PathObject    # 路径
├── CPDF_ImageObject   # 图像
├── CPDF_ShadingObject # 着色
└── CPDF_FormObject    # 表单 XObject
```

### 注释模型

```
CPDFSDK_Annot (基类)
└── CPDFSDK_BAAnnot (基本注释)
    └── CPDFSDK_Widget (表单小部件)
```

### 坐标系统

PDF 使用笛卡尔坐标系，原点在页面左下角：
- X 轴向右为正
- Y 轴向上为正

设备坐标系原点通常在左上角：
- X 轴向右为正
- Y 轴向下为正

坐标转换通过 `CFX_Matrix` 变换矩阵实现。

---

## 关键流程分析

### 1. 文档加载流程

```
FPDF_LoadDocument()
    │
    ├── 创建 CPDF_Document
    │
    ├── CPDF_Parser::StartParse()
    │   ├── 解析文件头
    │   ├── 查找 startxref
    │   ├── 加载交叉引用表
    │   ├── 解析 trailer
    │   └── 处理加密（如果有）
    │
    ├── CPDF_Document::TryInit()
    │   ├── 获取 Root 字典
    │   └── 加载页面信息
    │
    └── 返回文档句柄
```

### 2. 页面渲染流程

```
FPDF_RenderPageBitmap()
    │
    ├── FPDF_LoadPage() (如果未加载)
    │
    ├── CPDF_Page::ParseContent()
    │   ├── 获取内容流
    │   ├── CPDF_ContentParser 解析操作符
    │   └── 生成 CPDF_PageObject 列表
    │
    ├── 创建 CFX_RenderDevice
    │
    ├── CPDF_RenderContext::Render()
    │   ├── 遍历页面对象
    │   ├── CPDF_RenderStatus 处理每个对象
    │   │   ├── 文本 → CPDF_TextRenderer
    │   │   ├── 路径 → 直接绘制
    │   │   ├── 图像 → CPDF_ImageRenderer
    │   │   └── 着色 → CPDF_RenderShading
    │   └── 应用透明度和混合模式
    │
    └── 输出到位图
```

### 3. 文本提取流程

```
FPDFText_LoadPage()
    │
    ├── 创建 CPDF_TextPage
    │
    ├── 解析页面内容
    │
    ├── 遍历文本对象
    │   ├── 提取字符
    │   ├── 计算字符位置
    │   └── 处理文本顺序
    │
    └── 构建字符信息数组

FPDFText_GetText()
    │
    └── 返回指定范围的文本
```

### 4. 表单交互流程

```
FPDFDOC_InitFormFillEnvironment()
    │
    └── 创建 CPDFSDK_FormFillEnvironment
        ├── 初始化交互式表单
        └── 创建 JavaScript 运行时

用户交互事件
    │
    ├── CPDFSDK_PageView 接收事件
    │
    ├── 查找目标注释 (CPDFSDK_Widget)
    │
    ├── CFFL_InteractiveFormFiller 处理
    │   ├── 显示/隐藏编辑控件
    │   ├── 处理输入
    │   └── 触发 JavaScript
    │
    └── 更新显示
```

---

## 模块依赖关系

```
┌─────────────────────────────────────────────────────────────────┐
│                        public/ (API)                             │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                        fpdfsdk/                                  │
└─────────────────────────────────────────────────────────────────┘
                              │
           ┌──────────────────┼──────────────────┐
           ▼                  ▼                  ▼
    ┌───────────┐      ┌───────────┐      ┌───────────┐
    │   xfa/    │      │  fxjs/    │      │fxbarcode/ │
    └───────────┘      └───────────┘      └───────────┘
           │                  │                  │
           └──────────────────┼──────────────────┘
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                          core/                                   │
│  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐   │
│  │fpdfapi/ │ │fpdfdoc/ │ │fpdftext/│ │ fxcodec/│ │  fdrm/  │   │
│  └────┬────┘ └────┬────┘ └────┬────┘ └────┬────┘ └────┬────┘   │
│       └───────────┴───────────┴───────────┴───────────┘         │
│                              │                                   │
│                              ▼                                   │
│                    ┌─────────────────┐                          │
│                    │     fxge/       │                          │
│                    └────────┬────────┘                          │
│                              ▼                                   │
│                    ┌─────────────────┐                          │
│                    │     fxcrt/      │                          │
│                    └─────────────────┘                          │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                      third_party/                                │
│  FreeType, libjpeg, zlib, libpng, ICU, V8, Skia, OpenJPEG, ...  │
└─────────────────────────────────────────────────────────────────┘
```

### 模块职责总结

| 层级 | 模块 | 职责 |
|------|------|------|
| API | `public/` | 对外接口，C 语言 API |
| SDK | `fpdfsdk/` | API 实现，表单交互 |
| 扩展 | `xfa/` | XFA 动态表单 |
| 扩展 | `fxjs/` | JavaScript 支持 |
| 扩展 | `fxbarcode/` | 条形码生成 |
| 核心 | `core/fpdfapi/` | PDF 解析、页面、渲染 |
| 核心 | `core/fpdfdoc/` | 文档对象（书签、链接等） |
| 核心 | `core/fpdftext/` | 文本提取 |
| 核心 | `core/fxcodec/` | 图像编解码 |
| 核心 | `core/fdrm/` | 加密 |
| 基础 | `core/fxge/` | 图形引擎 |
| 基础 | `core/fxcrt/` | 运行时库 |
| 外部 | `third_party/` | 第三方依赖 |

---

## 附录

### 构建配置选项

| 选项 | 描述 |
|------|------|
| `pdf_enable_v8` | 启用 V8 JavaScript 引擎 |
| `pdf_enable_xfa` | 启用 XFA 表单支持 |
| `pdf_use_skia` | 使用 Skia 渲染后端 |
| `pdf_is_standalone` | 构建独立测试程序 |
| `is_debug` | 调试构建 |
| `is_component_build` | 组件构建 |

### 关键宏定义

| 宏 | 描述 |
|----|------|
| `PDF_ENABLE_V8` | V8 支持已启用 |
| `PDF_ENABLE_XFA` | XFA 支持已启用 |
| `PDF_USE_SKIA` | 使用 Skia 后端 |
| `BUILDFLAG(IS_WIN)` | Windows 平台 |
| `BUILDFLAG(IS_LINUX)` | Linux 平台 |
| `BUILDFLAG(IS_APPLE)` | macOS/iOS 平台 |
| `BUILDFLAG(IS_ANDROID)` | Android 平台 |

### 参考资源

- [PDFium 官方文档](https://pdfium.googlesource.com/pdfium/)
- [PDF Reference 1.7](https://opensource.adobe.com/dc-acrobat-sdk-docs/pdfstandards/pdfreference1.7old.pdf)
- [XFA Specification](https://www.pdfa.org/resource/xfa-specification/)
- [Adobe JavaScript for Acrobat API Reference](https://opensource.adobe.com/dc-acrobat-sdk-docs/acrobatsdk/pdfs/acrobatsdk_jsapiref.pdf)
