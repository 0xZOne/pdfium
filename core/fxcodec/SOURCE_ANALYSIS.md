# core/fxcodec 源码分析

## 模块概述

`fxcodec` 模块负责处理 PDF 文档中使用的各种图像编解码格式。该模块封装了多种第三方编解码库，提供统一的接口用于图像数据的编码和解码。

## 目录结构

```
core/fxcodec/
├── BUILD.gn                    # 构建配置
├── fx_codec.cpp/h              # 编解码器通用接口
├── fx_codec_def.h              # 编解码器定义
├── cfx_codec_memory.cpp/h      # 编解码内存管理
├── data_and_bytes_consumed.cpp/h  # 数据消费跟踪
├── progressive_decoder.cpp/h   # 渐进式解码器
├── progressive_decoder_context.h  # 渐进式解码上下文
├── scanlinedecoder.cpp/h       # 扫描线解码器基类
├── basic/                      # 基本编解码器 (ASCII85, ASCIIHex, RunLength)
├── flate/                      # Flate/Deflate 压缩
├── jpeg/                       # JPEG 图像编解码
├── jpx/                        # JPEG2000 图像解码
├── jbig2/                      # JBIG2 图像解码
├── fax/                        # CCITT 传真编解码
├── icc/                        # ICC 颜色配置文件
├── png/                        # PNG 图像编解码
├── gif/                        # GIF 图像解码
├── bmp/                        # BMP 图像解码
└── tiff/                       # TIFF 图像解码
```

## 核心组件分析

### 1. 编解码器基础设施

#### ScanlineDecoder 类

**位置**: scanlinedecoder.h/cpp

扫描线解码器的抽象基类，定义了按行解码图像的接口：

| 方法 | 功能 |
|------|------|
| `GetWidth` | 获取图像宽度 |
| `GetHeight` | 获取图像高度 |
| `CountComps` | 获取颜色分量数 |
| `GetBPC` | 获取每分量位数 |
| `GetNextLine` | 获取下一扫描行 |
| `GetSrcLine` | 获取指定行数据 |
| `Rewind` | 重置到起始位置 |

#### ProgressiveDecoder 类

**位置**: progressive_decoder.h/cpp

渐进式解码器，支持分块加载大型图像：

| 方法 | 功能 |
|------|------|
| `LoadImageInfo` | 加载图像基本信息 |
| `ContinueDecode` | 继续解码过程 |
| `GetImageInfo` | 获取图像信息 |
| `GetWidth/Height` | 获取图像尺寸 |
| `GetType` | 获取图像类型 |

支持的图像类型：
- BMP
- GIF  
- JPEG
- PNG
- TIFF

#### CFX_CodecMemory 类

**位置**: cfx_codec_memory.h/cpp

管理编解码过程中的内存分配和数据缓冲。

### 2. 基本编解码器 (basic/)

处理 PDF 中常用的简单编码格式：

#### ASCII85Decoder

将二进制数据编码为可打印 ASCII 字符（每4字节变为5字符）。

#### ASCIIHexDecoder

将二进制数据编码为十六进制字符串。

#### RunLengthDecoder

行程长度编码，用于简单的数据压缩。

### 3. Flate 编解码 (flate/)

#### FlateDecoder

**功能**: 实现 Deflate 解压缩算法

基于 zlib 库，处理：
- PDF 流对象的压缩
- PNG 图像数据
- 预测器过滤

#### FlateEncoder

**功能**: 实现 Deflate 压缩算法

用于：
- 压缩 PDF 流对象
- 减小文件大小

#### 预测器支持

支持 PNG 预测器类型：
- None（无预测）
- Sub（左侧差分）
- Up（上方差分）
- Average（平均）
- Paeth（Paeth 预测）

### 4. JPEG 编解码 (jpeg/)

#### JpegDecoder

**依赖**: libjpeg-turbo

**功能**:
- 解码 JPEG 图像
- 支持 DCT 解压缩
- 处理颜色空间转换
- 支持渐进式 JPEG

#### JpegEncoder

**功能**:
- 编码 JPEG 图像
- 可配置质量参数

#### 颜色空间处理

| 源格式 | 目标格式 |
|--------|----------|
| RGB | RGB |
| CMYK | CMYK (需反转) |
| YCbCr | RGB (自动转换) |
| Grayscale | Grayscale |

### 5. JPEG2000 解码 (jpx/)

#### JpxDecoder

**依赖**: OpenJPEG 库

**功能**:
- 解码 JPEG2000 (JPX) 图像
- 支持无损和有损压缩
- 处理多分量图像

**特性**:
- 分辨率缩放
- 区域解码
- 多线程支持

### 6. JBIG2 解码 (jbig2/)

#### JBig2Decoder

**功能**: 解码 JBIG2 压缩图像

JBIG2 特性：
- 专门用于二值图像
- 高压缩比
- 支持符号字典
- 用于扫描文档

#### 主要组件：

| 组件 | 功能 |
|------|------|
| `CJBig2_Context` | 解码上下文 |
| `CJBig2_Segment` | JBIG2 段处理 |
| `CJBig2_SymbolDict` | 符号字典 |
| `CJBig2_ArithDecoder` | 算术解码器 |

### 7. 传真编解码 (fax/)

#### FaxDecoder

**功能**: 解码 CCITT Group 3/4 传真压缩

支持的格式：
- Group 3 一维编码
- Group 3 二维编码
- Group 4 二维编码

**PDF 中的应用**: 用于压缩黑白图像

#### FaxEncoder

**功能**: 编码传真格式图像

### 8. ICC 颜色管理 (icc/)

#### IccTransform

**依赖**: Little-CMS (lcms2) 库

**功能**:
- 解析 ICC 颜色配置文件
- 颜色空间转换
- 支持多种颜色模型

支持的颜色空间：
- RGB
- CMYK
- Lab
- Grayscale

### 9. PNG 编解码 (png/)

#### PngDecoder

**依赖**: libpng

**功能**:
- 解码 PNG 图像
- 支持透明度 (Alpha)
- 支持调色板模式
- 支持交错扫描

#### PngEncoder

**功能**: 编码 PNG 图像

### 10. GIF 解码 (gif/)

#### GifDecoder

**功能**:
- 解码 GIF 图像
- 支持动画 GIF
- 处理调色板
- 支持透明色

### 11. BMP 解码 (bmp/)

#### BmpDecoder

**功能**:
- 解码 BMP 图像
- 支持多种位深度
- 处理 RLE 压缩

### 12. TIFF 解码 (tiff/)

#### TiffDecoder

**依赖**: libtiff

**功能**:
- 解码 TIFF 图像
- 支持多种压缩格式
- 处理多页 TIFF

## 编解码流程

### 图像解码流程

```
PDF 流对象
    │
    ├── 读取 /Filter 参数
    │
    ├── 选择解码器
    │   ├── /FlateDecode → FlateDecoder
    │   ├── /DCTDecode → JpegDecoder
    │   ├── /JPXDecode → JpxDecoder
    │   ├── /JBIG2Decode → JBig2Decoder
    │   └── /CCITTFaxDecode → FaxDecoder
    │
    ├── 解码数据
    │
    └── 输出原始像素数据
```

### 渐进式解码流程

```
LoadImageInfo()
    │
    ├── 读取图像头信息
    │
    └── 返回尺寸和格式

ContinueDecode()
    │
    ├── 解码部分数据
    │
    ├── 更新进度
    │
    └── 返回状态 (继续/完成/错误)
```

## 错误处理

所有解码器统一的错误状态：

| 状态 | 描述 |
|------|------|
| `kSuccess` | 解码成功 |
| `kDataNotAvailable` | 数据不足 |
| `kError` | 解码错误 |
| `kInvalidFormat` | 格式无效 |

## 内存管理

- 使用 `pdfium::span` 进行安全的内存访问
- 大图像采用流式处理减少内存占用
- 支持内存池复用

## 依赖关系

```
fxcodec/
    ├── fxcrt/           (基础设施)
    ├── third_party/
    │   ├── zlib/        (Flate)
    │   ├── libjpeg/     (JPEG)
    │   ├── libopenjpeg/ (JPEG2000)
    │   ├── libpng/      (PNG)
    │   ├── libtiff/     (TIFF)
    │   └── lcms2/       (ICC)
    └── 内置实现
        ├── JBIG2 解码器
        ├── 传真编解码器
        └── 基本编解码器
```

## 性能优化

- JPEG 解码使用 SIMD 加速
- 大图像支持分块处理
- 缓存常用的颜色转换表
- 避免不必要的内存拷贝
