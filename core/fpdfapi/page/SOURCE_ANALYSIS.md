# core/fpdfapi/page 源码分析

## 模块概述

`page` 模块负责 PDF 页面的处理，包括页面内容解析、页面对象管理、颜色空间处理、图像处理、图案处理等。它是连接解析层和渲染层的桥梁。

## 目录结构

```
core/fpdfapi/page/
├── BUILD.gn                        # 构建配置
├── DEPS                            # 依赖声明
│
├── 页面核心
│   ├── cpdf_page.cpp/h             # PDF 页面
│   ├── cpdf_pageobjectholder.cpp/h # 页面对象持有者
│   ├── cpdf_docpagedata.cpp/h      # 文档页面数据
│   ├── cpdf_pagemodule.cpp/h       # 页面模块
│   └── ipdf_page.h                 # 页面接口
│
├── 页面对象
│   ├── cpdf_pageobject.cpp/h       # 页面对象基类
│   ├── cpdf_textobject.cpp/h       # 文本对象
│   ├── cpdf_pathobject.cpp/h       # 路径对象
│   ├── cpdf_imageobject.cpp/h      # 图像对象
│   ├── cpdf_shadingobject.cpp/h    # 着色对象
│   └── cpdf_formobject.cpp/h       # 表单 XObject
│
├── 内容解析
│   ├── cpdf_contentparser.cpp/h    # 内容解析器
│   ├── cpdf_streamcontentparser.cpp/h # 流内容解析器
│   └── cpdf_streamparser.cpp/h     # 流解析器
│
├── 颜色空间
│   ├── cpdf_colorspace.cpp/h       # 颜色空间基类
│   ├── cpdf_devicecs.cpp/h         # 设备颜色空间
│   ├── cpdf_basedcs.cpp/h          # 基于其他颜色空间
│   ├── cpdf_indexedcs.cpp/h        # 索引颜色空间
│   ├── cpdf_patterncs.cpp/h        # 图案颜色空间
│   ├── cpdf_color.cpp/h            # 颜色值
│   ├── cpdf_colorstate.cpp/h       # 颜色状态
│   └── cpdf_iccprofile.cpp/h       # ICC 配置文件
│
├── 图像处理
│   ├── cpdf_image.cpp/h            # 图像对象
│   ├── cpdf_imageloader.cpp/h      # 图像加载器
│   ├── cpdf_pageimagecache.cpp/h   # 页面图像缓存
│   ├── cpdf_dib.cpp/h              # 设备无关位图
│   └── jpx_decode_conversion.cpp/h # JPEG2000 转换
│
├── 图案和着色
│   ├── cpdf_pattern.cpp/h          # 图案基类
│   ├── cpdf_tilingpattern.cpp/h    # 平铺图案
│   ├── cpdf_shadingpattern.cpp/h   # 着色图案
│   └── cpdf_meshstream.cpp/h       # 网格流
│
├── 图形状态
│   ├── cpdf_graphicstates.cpp/h    # 图形状态集合
│   ├── cpdf_generalstate.cpp/h     # 通用状态
│   ├── cpdf_textstate.cpp/h        # 文本状态
│   ├── cpdf_allstates.cpp/h        # 所有状态
│   ├── cpdf_clippath.cpp/h         # 裁剪路径
│   ├── cpdf_path.cpp/h             # 路径
│   └── cpdf_transparency.cpp/h     # 透明度
│
├── 函数
│   ├── cpdf_function.cpp/h         # 函数基类
│   ├── cpdf_sampledfunc.cpp/h      # 采样函数
│   ├── cpdf_expintfunc.cpp/h       # 指数插值函数
│   ├── cpdf_stitchfunc.cpp/h       # 拼接函数
│   ├── cpdf_psfunc.cpp/h           # PostScript 函数
│   └── cpdf_psengine.cpp/h         # PS 引擎
│
├── 其他
│   ├── cpdf_form.cpp/h             # 表单 XObject
│   ├── cpdf_annotcontext.cpp/h     # 注释上下文
│   ├── cpdf_occontext.cpp/h        # 可选内容上下文
│   ├── cpdf_contentmarkitem.cpp/h  # 内容标记项
│   ├── cpdf_contentmarks.cpp/h     # 内容标记
│   └── cpdf_transferfunc.cpp/h     # 传递函数
```

## 核心类分析

### 1. CPDF_Page 类

**位置**: cpdf_page.h/cpp

PDF 页面的核心表示。

| 方法 | 功能 |
|------|------|
| `ParseContent` | 解析页面内容 |
| `GetPageWidth` | 获取页面宽度 |
| `GetPageHeight` | 获取页面高度 |
| `GetPageRotation` | 获取页面旋转 |
| `DeviceToPage` | 设备坐标到页面坐标 |
| `PageToDevice` | 页面坐标到设备坐标 |
| `GetDisplayMatrix` | 获取显示矩阵 |

页面属性：
- MediaBox: 媒体框
- CropBox: 裁剪框
- BleedBox: 出血框
- TrimBox: 裁切框
- ArtBox: 内容框
- Rotate: 旋转角度

### 2. CPDF_PageObjectHolder 类

**位置**: cpdf_pageobjectholder.h/cpp

页面对象的容器，管理页面上的所有对象。

| 方法 | 功能 |
|------|------|
| `GetPageObjectCount` | 获取对象数量 |
| `GetPageObjectByIndex` | 按索引获取对象 |
| `AppendPageObject` | 添加对象 |
| `RemovePageObject` | 删除对象 |
| `GetResources` | 获取资源字典 |

### 3. 页面对象系统

#### CPDF_PageObject 基类

**位置**: cpdf_pageobject.h/cpp

所有页面对象的抽象基类。

| 类型 | 描述 |
|------|------|
| `kText` | 文本对象 |
| `kPath` | 路径对象 |
| `kImage` | 图像对象 |
| `kShading` | 着色对象 |
| `kForm` | 表单 XObject |

公共属性：
- 变换矩阵
- 裁剪路径
- 图形状态
- 内容标记

#### CPDF_TextObject

**位置**: cpdf_textobject.h/cpp

文本对象，包含文字和定位信息：

| 方法 | 功能 |
|------|------|
| `CountItems` | 字符数量 |
| `GetCharInfo` | 获取字符信息 |
| `GetFont` | 获取字体 |
| `GetFontSize` | 获取字体大小 |
| `GetTextMatrix` | 获取文本矩阵 |
| `GetCharCode` | 获取字符编码 |

#### CPDF_PathObject

**位置**: cpdf_pathobject.h/cpp

路径对象，包含矢量图形：

| 属性 | 描述 |
|------|------|
| 路径数据 | 点和类型序列 |
| 填充类型 | 无/非零/奇偶 |
| 笔画 | 是否描边 |
| 线宽 | 笔画宽度 |
| 线帽/连接 | 样式设置 |

#### CPDF_ImageObject

**位置**: cpdf_imageobject.h/cpp

图像对象：

| 方法 | 功能 |
|------|------|
| `GetImage` | 获取图像资源 |
| `GetImageMatrix` | 获取图像矩阵 |
| `SetImage` | 设置图像 |

#### CPDF_FormObject

**位置**: cpdf_formobject.h/cpp

表单 XObject，嵌套的页面内容。

### 4. 内容解析

#### CPDF_ContentParser 类

**位置**: cpdf_contentparser.h/cpp

页面内容解析器，将内容流转换为页面对象。

解析状态：
- `kNotStarted`: 未开始
- `kParsing`: 解析中
- `kDone`: 完成

#### CPDF_StreamContentParser 类

**位置**: cpdf_streamcontentparser.h/cpp

处理 PDF 操作符和操作数。

PDF 操作符类别：

| 类别 | 操作符示例 |
|------|------------|
| 路径构建 | m, l, c, v, y, h, re |
| 路径绑制 | S, s, f, F, f*, B, B*, b, b*, n |
| 裁剪 | W, W* |
| 文本 | BT, ET, Tf, Td, Tm, T*, Tj, TJ |
| 图形状态 | q, Q, cm, w, J, j, M, d, gs |
| 颜色 | CS, cs, SC, sc, G, g, RG, rg, K, k |
| 图像 | BI, ID, EI, Do |
| 着色 | sh |

### 5. 颜色空间系统

#### CPDF_ColorSpace 基类

**位置**: cpdf_colorspace.h/cpp

颜色空间抽象类。

| 类型 | 描述 |
|------|------|
| `kDeviceGray` | 设备灰度 |
| `kDeviceRGB` | 设备 RGB |
| `kDeviceCMYK` | 设备 CMYK |
| `kCalGray` | 校准灰度 |
| `kCalRGB` | 校准 RGB |
| `kLab` | CIE Lab |
| `kICCBased` | ICC 配置 |
| `kIndexed` | 索引颜色 |
| `kSeparation` | 分色 |
| `kDeviceN` | DeviceN |
| `kPattern` | 图案 |

#### CPDF_DeviceCS

设备颜色空间实现（Gray、RGB、CMYK）。

#### CPDF_ICCProfile

**位置**: cpdf_iccprofile.h/cpp

ICC 颜色配置文件处理。

### 6. 图像处理

#### CPDF_Image 类

**位置**: cpdf_image.h/cpp

图像资源：

| 方法 | 功能 |
|------|------|
| `GetWidth` | 图像宽度 |
| `GetHeight` | 图像高度 |
| `GetPixelWidth` | 像素宽度 |
| `GetPixelHeight` | 像素高度 |
| `GetColorSpace` | 颜色空间 |
| `GetBPP` | 每像素位数 |
| `LoadDIBBase` | 加载位图 |

#### CPDF_DIB 类

**位置**: cpdf_dib.h/cpp

从 PDF 图像创建设备无关位图。

#### CPDF_PageImageCache

**位置**: cpdf_pageimagecache.h/cpp

缓存已解码的图像以提高性能。

### 7. 图案和着色

#### CPDF_Pattern 基类

**位置**: cpdf_pattern.h/cpp

图案抽象类。

| 类型 | 描述 |
|------|------|
| `kTiling` | 平铺图案 |
| `kShading` | 着色图案 |

#### CPDF_TilingPattern

**位置**: cpdf_tilingpattern.h/cpp

平铺图案，重复绘制内容。

#### CPDF_ShadingPattern

**位置**: cpdf_shadingpattern.h/cpp

着色图案，支持多种着色类型：
- 函数着色
- 轴向渐变
- 径向渐变
- 自由形式渐变
- 晶格形式渐变
- Coons 色块
- 张量积色块

### 8. 函数系统

#### CPDF_Function 基类

**位置**: cpdf_function.h/cpp

PDF 函数抽象类。

| 类型 | 描述 |
|------|------|
| `kType0SampledFunction` | 采样函数 |
| `kType2ExponentialInterpolation` | 指数插值 |
| `kType3Stitching` | 拼接函数 |
| `kType4PostScript` | PostScript 计算器 |

#### CPDF_PSEngine

**位置**: cpdf_psengine.h/cpp

PostScript 计算器引擎，执行 PS 代码。

### 9. 图形状态

#### CPDF_GraphicStates 类

**位置**: cpdf_graphicstates.h/cpp

保存完整的图形状态。

#### CPDF_GeneralState

**位置**: cpdf_generalstate.h/cpp

通用图形状态：
- 混合模式
- 透明度
- 软蒙版
- 传递函数

#### CPDF_TextState

**位置**: cpdf_textstate.h/cpp

文本状态：
- 字符间距
- 词间距
- 水平缩放
- 行距
- 渲染模式

## 内容解析流程

```
CPDF_Page::ParseContent()
    │
    ├── 获取 Contents 流
    │
    ├── 创建 CPDF_ContentParser
    │
    ├── 解析循环
    │   ├── 读取操作符和操作数
    │   ├── 根据操作符类型处理
    │   ├── 生成页面对象
    │   └── 添加到对象列表
    │
    └── 完成
```

## 依赖关系

```
page/
    ├── parser/      (PDF 对象)
    ├── font/        (字体处理)
    ├── fxcodec/     (图像解码)
    ├── fxge/        (图形基础)
    └── fxcrt/       (基础设施)
```

## 性能优化

- 懒加载页面内容
- 图像缓存
- 字体缓存
- 资源复用
