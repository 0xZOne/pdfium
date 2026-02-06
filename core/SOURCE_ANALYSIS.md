# core 源码分析

## 模块概述

`core` 目录包含 PDFium 的核心实现，是整个库的基础。它实现了 PDF 解析、渲染、文本提取、字体处理、图像编解码等核心功能。

## 目录结构

```
core/
├── fpdfapi/              # PDF API 核心实现
│   ├── parser/           # PDF 解析器
│   ├── page/             # 页面处理
│   ├── render/           # 渲染引擎
│   ├── font/             # 字体处理
│   ├── edit/             # 编辑功能
│   └── cmaps/            # 字符映射表
│
├── fpdfdoc/              # PDF 文档对象
│
├── fpdftext/             # 文本提取
│
├── fxcodec/              # 编解码器
│
├── fxcrt/                # 核心运行时
│
├── fxge/                 # 图形引擎
│
└── fdrm/                 # 数字版权管理
```

## 模块职责

| 模块 | 职责 |
|------|------|
| `fpdfapi` | PDF 格式解析、页面对象处理、渲染、字体、编辑 |
| `fpdfdoc` | 文档级结构（书签、链接、表单、注释） |
| `fpdftext` | 文本提取和搜索 |
| `fxcodec` | 图像编解码（JPEG、PNG、JBIG2 等） |
| `fxcrt` | 基础设施（字符串、容器、流、内存） |
| `fxge` | 图形引擎（绑制、字体渲染） |
| `fdrm` | 加密解密（RC4、AES、MD5、SHA） |

## 层次结构

```
┌─────────────────────────────────────────────────────────────┐
│                        fpdfdoc                               │
│               (书签、链接、表单、注释、元数据)                  │
├─────────────────────────────────────────────────────────────┤
│                        fpdfapi                               │
│  ┌──────────┬──────────┬──────────┬──────────┬────────────┐ │
│  │ parser   │   page   │  render  │   font   │   edit     │ │
│  └──────────┴──────────┴──────────┴──────────┴────────────┘ │
├───────────────────┬─────────────────────────────────────────┤
│     fpdftext      │                fxcodec                   │
│    (文本提取)     │               (编解码器)                  │
├───────────────────┴─────────────────────────────────────────┤
│                         fxge                                 │
│                      (图形引擎)                              │
├─────────────────────────────────────────────────────────────┤
│                         fxcrt                                │
│                    (核心运行时库)                             │
├─────────────────────────────────────────────────────────────┤
│                         fdrm                                 │
│                      (加密解密)                              │
└─────────────────────────────────────────────────────────────┘
```

## 模块详解

### fpdfapi - PDF API 核心

最核心的模块，包含 6 个子模块：

| 子模块 | 功能 |
|--------|------|
| `parser` | PDF 文件解析、对象模型、加密处理 |
| `page` | 页面内容解析、页面对象、颜色空间 |
| `render` | 页面渲染、文本渲染、图像渲染 |
| `font` | 字体加载、编码映射、字形获取 |
| `edit` | PDF 创建、内容生成、页面操作 |
| `cmaps` | CJK 字符映射数据 |

### fpdfdoc - 文档对象

处理 PDF 文档结构：

| 组件 | 功能 |
|------|------|
| 书签 | `CPDF_Bookmark`, `CPDF_BookmarkTree` |
| 链接 | `CPDF_Link`, `CPDF_LinkList` |
| 动作 | `CPDF_Action`, `CPDF_AAction` |
| 目标 | `CPDF_Dest` |
| 表单 | `CPDF_FormField`, `CPDF_FormControl` |
| 注释 | `CPDF_Annot`, `CPDF_AnnotList` |
| 结构 | `CPDF_StructTree`, `CPDF_StructElement` |
| 元数据 | `CPDF_Metadata` |

### fpdftext - 文本处理

文本提取和搜索：

| 类 | 功能 |
|----|------|
| `CPDF_TextPage` | 页面文本提取 |
| `CPDF_TextPageFind` | 文本搜索 |
| `CPDF_LinkExtract` | URL 提取 |

### fxcodec - 编解码器

图像格式支持：

| 格式 | 目录 |
|------|------|
| 基本编码 | `basic/` |
| Flate | `flate/` |
| JPEG | `jpeg/` |
| JPEG2000 | `jpx/` |
| JBIG2 | `jbig2/` |
| 传真 | `fax/` |
| ICC | `icc/` |
| PNG | `png/` |
| GIF | `gif/` |
| BMP | `bmp/` |
| TIFF | `tiff/` |

### fxcrt - 运行时

基础设施：

| 类别 | 内容 |
|------|------|
| 字符串 | `ByteString`, `WideString` |
| 智能指针 | `RetainPtr`, `UnownedPtr`, `ObservedPtr` |
| 容器 | `DataVector`, `span` |
| 流 | `IFX_SeekableReadStream`, `CFX_MemoryStream` |
| 坐标 | `CFX_PointF`, `CFX_FloatRect`, `CFX_Matrix` |
| 时间 | `CFX_DateTime`, `CFX_Timer` |
| 内存 | `FX_Alloc`, `FX_Free` |

### fxge - 图形引擎

图形渲染：

| 类别 | 内容 |
|------|------|
| 设备 | `CFX_RenderDevice`, `CFX_DefaultRenderDevice` |
| 位图 | `CFX_DIBitmap`, `CFX_DIBBase` |
| 路径 | `CFX_Path`, `CFX_GraphStateData` |
| 字体 | `CFX_Font`, `CFX_FontMgr`, `CFX_GlyphCache` |
| 后端 | `agg/`, `skia/`, `win32/` |

### fdrm - 加密

加密算法：

| 算法 | 文件 |
|------|------|
| RC4 | `fx_crypt.cpp` |
| MD5 | `fx_crypt.cpp` |
| AES | `fx_crypt_aes.cpp` |
| SHA | `fx_crypt_sha.cpp` |

## 关键数据流

### 文档加载

```
文件 → parser → CPDF_Document → CPDF_Page → 渲染
```

### 页面渲染

```
CPDF_Page → ContentParser → PageObjects → RenderStatus → 位图
```

### 文本提取

```
CPDF_Page → CPDF_TextPage → CharInfo 列表 → 文本
```

## 依赖关系

```
         fpdfdoc
            ↓
         fpdfapi
            ↓
    ┌───────┼───────┐
    ↓       ↓       ↓
fpdftext  fxcodec  fdrm
    ↓       ↓       ↓
    └───────┼───────┘
            ↓
          fxge
            ↓
          fxcrt
            ↓
      third_party
```

## 第三方依赖

| 库 | 用途 |
|----|------|
| FreeType | 字体渲染 |
| zlib | Flate 压缩 |
| libjpeg | JPEG 编解码 |
| libpng | PNG 编解码 |
| OpenJPEG | JPEG2000 解码 |
| libtiff | TIFF 解码 |
| lcms2 | ICC 颜色管理 |
| AGG | 软件渲染 |
| Skia | 可选渲染后端 |
| ICU | Unicode 支持 |

## 构建配置

主要构建选项：

| 选项 | 描述 |
|------|------|
| `pdf_use_skia` | 使用 Skia 后端 |
| `pdf_enable_v8` | 启用 JavaScript |
| `pdf_enable_xfa` | 启用 XFA |
| `is_component_build` | 组件构建 |

## 详细分析

各子模块的详细分析请参见各目录下的 `SOURCE_ANALYSIS.md` 文件：

- [fpdfapi 分析](fpdfapi/SOURCE_ANALYSIS.md)
- [fpdfdoc 分析](fpdfdoc/SOURCE_ANALYSIS.md)
- [fpdftext 分析](fpdftext/SOURCE_ANALYSIS.md)
- [fxcodec 分析](fxcodec/SOURCE_ANALYSIS.md)
- [fxcrt 分析](fxcrt/SOURCE_ANALYSIS.md)
- [fxge 分析](fxge/SOURCE_ANALYSIS.md)
- [fdrm 分析](fdrm/SOURCE_ANALYSIS.md)
