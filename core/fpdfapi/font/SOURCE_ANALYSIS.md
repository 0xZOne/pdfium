# core/fpdfapi/font 源码分析

## 模块概述

`font` 模块负责 PDF 字体处理，包括字体解析、字符编码转换、字形获取等功能。PDF 支持多种字体类型，该模块实现了对它们的统一处理。

## 目录结构

```
core/fpdfapi/font/
├── BUILD.gn                        # 构建配置
│
├── 字体基类
│   ├── cpdf_font.cpp/h             # PDF 字体基类
│   └── cpdf_simplefont.cpp/h       # 简单字体基类
│
├── 字体类型
│   ├── cpdf_type1font.cpp/h        # Type1 字体
│   ├── cpdf_truetypefont.cpp/h     # TrueType 字体
│   ├── cpdf_cidfont.cpp/h          # CID 字体
│   ├── cpdf_type3font.cpp/h        # Type3 字体
│   └── cpdf_type3char.cpp/h        # Type3 字符
│
├── 编码和映射
│   ├── cpdf_fontencoding.cpp/h     # 字体编码
│   ├── cpdf_cmap.cpp/h             # CMap 字符映射
│   ├── cpdf_cmapparser.cpp/h       # CMap 解析器
│   ├── cpdf_tounicodemap.cpp/h     # ToUnicode 映射
│   └── cpdf_cid2unicodemap.cpp/h   # CID 到 Unicode 映射
│
├── 字体管理
│   ├── cpdf_fontglobals.cpp/h      # 字体全局数据
│   └── cfx_stockfontarray.cpp/h    # 标准字体数组
│
└── OpenType 支持
    └── cfx_cttgsubtable.cpp/h      # GSUB 表处理
```

## 核心类分析

### 1. CPDF_Font 基类

**位置**: cpdf_font.h/cpp

所有 PDF 字体的抽象基类。

| 方法 | 功能 |
|------|------|
| `Create` | 工厂方法创建字体 |
| `GetFontTypeName` | 获取字体类型名 |
| `GetBaseFontName` | 获取基础字体名 |
| `GetCharWidthF` | 获取字符宽度 |
| `GetCharBBox` | 获取字符边界框 |
| `GlyphFromCharCode` | 字符编码到字形索引 |
| `UnicodeFromCharCode` | 字符编码到 Unicode |
| `CharCodeFromUnicode` | Unicode 到字符编码 |
| `GetFace` | 获取 FreeType 字体面 |
| `IsEmbedded` | 是否嵌入字体 |

字体类型标识：

| 类型 | 描述 |
|------|------|
| `kType1` | Type1 字体 |
| `kTrueType` | TrueType 字体 |
| `kType3` | Type3 用户定义字体 |
| `kCIDFont` | CID 字体 |

### 2. CPDF_SimpleFont 类

**位置**: cpdf_simplefont.h/cpp

简单字体基类（Type1 和 TrueType 的共同基类）。

特点：
- 最多 256 个字符
- 单字节编码
- 固定宽度表

| 方法 | 功能 |
|------|------|
| `GetCharWidth` | 获取字符宽度 |
| `LoadCharMetrics` | 加载字符度量 |
| `GetCharCode` | 获取字符编码 |

### 3. CPDF_Type1Font 类

**位置**: cpdf_type1font.h/cpp

Type1 字体实现（Adobe 的 PostScript 字体格式）。

特点：
- 矢量轮廓描述
- 紧凑的字符描述
- 广泛用于专业排版

标准 14 字体：
- Courier 系列（4种）
- Helvetica 系列（4种）
- Times 系列（4种）
- Symbol
- ZapfDingbats

### 4. CPDF_TrueTypeFont 类

**位置**: cpdf_truetypefont.h/cpp

TrueType 字体实现。

特点：
- 二次贝塞尔曲线轮廓
- 高质量提示指令
- 跨平台兼容

### 5. CPDF_CIDFont 类

**位置**: cpdf_cidfont.h/cpp

CID（Character Identifier）字体，用于大字符集语言。

特点：
- 支持数万个字符
- 用于中日韩文字
- 两字节或多字节编码

| 方法 | 功能 |
|------|------|
| `GetCIDTransform` | 获取 CID 变换 |
| `GetVertWidth` | 获取垂直书写宽度 |
| `GetVertOrigin` | 获取垂直书写原点 |
| `IsVertWriting` | 是否垂直书写 |

CID 字体类型：
- CIDFontType0（基于 Type1）
- CIDFontType2（基于 TrueType）

### 6. CPDF_Type3Font 类

**位置**: cpdf_type3font.h/cpp

Type3 字体，用户定义的字形。

特点：
- 字形由 PDF 内容流定义
- 可包含任意图形
- 用于特殊符号和装饰

| 方法 | 功能 |
|------|------|
| `LoadChar` | 加载字符内容 |
| `GetFontMatrix` | 获取字体矩阵 |
| `GetCharProc` | 获取字符过程 |

#### CPDF_Type3Char

**位置**: cpdf_type3char.h/cpp

单个 Type3 字符的表示：
- 字符宽度
- 边界框
- 内容流

### 7. 编码系统

#### CPDF_FontEncoding 类

**位置**: cpdf_fontencoding.h/cpp

字体编码处理。

标准编码：
- StandardEncoding
- MacRomanEncoding
- WinAnsiEncoding
- MacExpertEncoding
- PDFDocEncoding

#### CPDF_CMap 类

**位置**: cpdf_cmap.h/cpp

CMap（字符映射）处理。

功能：
- 字节序列到 CID 的映射
- 代码空间范围定义
- 支持预定义和自定义 CMap

预定义 CMap 示例：
- GB-EUC-H/V（简体中文）
- B5pc-H/V（繁体中文）
- 90ms-RKSJ-H/V（日文）
- KSCms-UHC-H/V（韩文）

#### CPDF_CMapParser 类

**位置**: cpdf_cmapparser.h/cpp

CMap 文件解析器：
- 解析代码空间范围
- 解析字符映射
- 处理嵌套 CMap

#### CPDF_ToUnicodeMap 类

**位置**: cpdf_tounicodemap.h/cpp

ToUnicode 映射，将字符编码转换为 Unicode：

| 方法 | 功能 |
|------|------|
| `Lookup` | 查找 Unicode 值 |
| `ReverseLookup` | 反向查找字符编码 |

#### CPDF_CID2UnicodeMap 类

**位置**: cpdf_cid2unicodemap.h/cpp

CID 到 Unicode 的预定义映射。

### 8. 字体全局管理

#### CPDF_FontGlobals 类

**位置**: cpdf_fontglobals.h/cpp

全局字体数据管理：

| 方法 | 功能 |
|------|------|
| `GetStockFont` | 获取标准字体 |
| `Find` | 查找字体 |
| `GetEmbeddedCharset` | 获取嵌入字符集 |
| `LoadEmbeddedGb1CMaps` | 加载 GB1 CMap |
| `LoadEmbeddedCns1CMaps` | 加载 CNS1 CMap |
| `LoadEmbeddedJapan1CMaps` | 加载 Japan1 CMap |
| `LoadEmbeddedKorea1CMaps` | 加载 Korea1 CMap |

### 9. OpenType 支持

#### CFX_CTTGSUBTable 类

**位置**: cfx_cttgsubtable.h/cpp

处理 OpenType GSUB（Glyph Substitution）表：
- 字形替换
- 连字处理
- 上下文相关替换

## 字体加载流程

```
PDF 字体字典
    │
    ├── 确定字体类型
    │   ├── /Type1 → CPDF_Type1Font
    │   ├── /TrueType → CPDF_TrueTypeFont
    │   ├── /Type3 → CPDF_Type3Font
    │   └── /CIDFontType0/2 → CPDF_CIDFont
    │
    ├── 加载字体描述符
    │   ├── 获取字体名称
    │   ├── 获取字体标志
    │   └── 获取字体度量
    │
    ├── 加载嵌入字体数据
    │   ├── 从 /FontFile 加载 Type1
    │   ├── 从 /FontFile2 加载 TrueType
    │   └── 从 /FontFile3 加载 CFF
    │
    ├── 设置编码
    │   ├── 加载 /Encoding
    │   └── 加载 /ToUnicode
    │
    └── 初始化 FreeType 面
```

## 字符渲染流程

```
输入: 字符编码
    │
    ├── 编码到字形索引
    │   ├── 简单字体: 直接映射
    │   └── CID 字体: CMap 查找
    │
    ├── 获取字形数据
    │   ├── 嵌入字体: 从字体文件
    │   └── 替代字体: 从系统字体
    │
    ├── 渲染字形
    │   ├── FreeType 光栅化
    │   └── Type3: 执行内容流
    │
    └── 输出: 字形位图/路径
```

## Unicode 转换流程

```
输入: 字符编码
    │
    ├── 检查 ToUnicode 映射
    │   └── 存在 → 直接返回
    │
    ├── 检查标准编码
    │   └── 标准编码 → 使用预定义映射
    │
    ├── CID 字体
    │   └── 使用 CID2Unicode 映射
    │
    └── 返回 Unicode 码点
```

## 字体替代

当嵌入字体不可用时，进行字体替代：
1. 匹配字体名称
2. 检查字体属性（粗细、斜体）
3. 考虑字符集需求
4. 选择最接近的系统字体

## 依赖关系

```
font/
    ├── parser/      (PDF 对象)
    ├── fxge/        (FreeType 封装)
    ├── fxcrt/       (基础设施)
    └── cmaps/       (预编译 CMap 数据)
```

## 性能优化

- 字体缓存避免重复加载
- 字形缓存减少光栅化
- 预编译的 CMap 数据
- 延迟加载 Unicode 映射
