# core/fxge 源码分析

## 模块概述

`fxge` (FoxIt Graphics Engine) 是 PDFium 的图形引擎模块，负责所有绑制操作。它提供了跨平台的图形抽象层，支持路径绘制、文本渲染、图像处理、字体管理等功能。

## 目录结构

```
core/fxge/
├── BUILD.gn                        # 构建配置
├── DEPS                            # 依赖声明
│
├── 核心渲染
│   ├── cfx_renderdevice.cpp/h      # 渲染设备基类
│   ├── cfx_defaultrenderdevice.cpp/h  # 默认渲染设备
│   ├── renderdevicedriver_iface.cpp/h # 渲染驱动接口
│   └── render_defines.h            # 渲染定义
│
├── 路径和绘图
│   ├── cfx_path.cpp/h              # 路径类
│   ├── cfx_graphstate.cpp/h        # 图形状态
│   ├── cfx_graphstatedata.cpp/h    # 图形状态数据
│   ├── cfx_fillrenderoptions.h     # 填充渲染选项
│   ├── cfx_color.cpp/h             # 颜色处理
│   ├── cfx_drawutils.cpp/h         # 绘图工具
│   └── calculate_pitch.cpp/h       # 行距计算
│
├── 字体管理
│   ├── cfx_font.cpp/h              # 字体类
│   ├── cfx_fontmgr.cpp/h           # 字体管理器
│   ├── cfx_fontmapper.cpp/h        # 字体映射器
│   ├── cfx_face.cpp/h              # FreeType 字体面
│   ├── cfx_substfont.cpp/h         # 替代字体
│   ├── cfx_folderfontinfo.cpp/h    # 文件夹字体信息
│   ├── cfx_unicodeencoding.cpp/h   # Unicode 编码
│   ├── cfx_unicodeencodingex.cpp/h # 扩展 Unicode 编码
│   ├── fx_font.cpp/h               # 字体工具函数
│   ├── fx_fontencoding.h           # 字体编码定义
│   └── systemfontinfo_iface.h      # 系统字体接口
│
├── 字形处理
│   ├── cfx_glyphcache.cpp/h        # 字形缓存
│   ├── cfx_glyphbitmap.cpp/h       # 字形位图
│   ├── text_char_pos.cpp/h         # 文本字符位置
│   ├── text_glyph_pos.cpp/h        # 文本字形位置
│   └── cfx_textrenderoptions.h     # 文本渲染选项
│
├── 模块管理
│   └── cfx_gemodule.cpp/h          # 图形引擎模块
│
├── 子目录
│   ├── dib/                        # 设备无关位图
│   ├── agg/                        # AGG 软件渲染器
│   ├── skia/                       # Skia 渲染后端
│   ├── freetype/                   # FreeType 封装
│   ├── fontdata/                   # 内置字体数据
│   ├── win32/                      # Windows 渲染
│   ├── apple/                      # macOS/iOS 渲染
│   ├── android/                    # Android 渲染
│   └── linux/                      # Linux 字体
```

## 核心组件分析

### 1. 渲染设备系统

#### CFX_RenderDevice 类

**位置**: cfx_renderdevice.h/cpp

渲染设备的核心抽象类，定义了所有绑制操作的接口：

| 方法类别 | 主要方法 |
|----------|----------|
| 路径绑制 | `DrawPath`, `FillRect`, `DrawCosmeticLine` |
| 文本绑制 | `DrawNormalText`, `DrawTextPath` |
| 图像绑制 | `SetDIBits`, `StretchDIBits`, `SetBitMask` |
| 状态管理 | `SaveState`, `RestoreState` |
| 裁剪 | `SetClip_Rect`, `SetClip_PathFill` |
| 属性 | `GetWidth`, `GetHeight`, `GetBPP` |

#### RenderDeviceDriverIface 类

**位置**: renderdevicedriver_iface.h/cpp

渲染驱动接口，具体的渲染后端需要实现此接口：

| 方法 | 功能 |
|------|------|
| `GetDeviceType` | 获取设备类型 |
| `StartRendering` | 开始渲染 |
| `EndRendering` | 结束渲染 |
| `DrawPath` | 绘制路径 |
| `SetDIBits` | 设置位图 |
| `StretchDIBits` | 拉伸位图 |
| `DrawCosmeticLine` | 绘制细线 |

#### CFX_DefaultRenderDevice 类

**位置**: cfx_defaultrenderdevice.h/cpp

默认渲染设备实现，使用 AGG 或 Skia 后端。

### 2. 路径系统

#### CFX_Path 类

**位置**: cfx_path.h/cpp

表示矢量路径，由点和路径类型组成：

| 路径点类型 | 描述 |
|------------|------|
| `kMoveTo` | 移动到点 |
| `kLineTo` | 直线到点 |
| `kBezierTo` | 贝塞尔曲线控制点/终点 |
| `kClose` | 闭合路径 |

主要方法：

| 方法 | 功能 |
|------|------|
| `AppendPoint` | 添加点 |
| `AppendLine` | 添加直线 |
| `AppendRect` | 添加矩形 |
| `AppendEllipse` | 添加椭圆 |
| `ClosePath` | 闭合当前子路径 |
| `Transform` | 应用变换矩阵 |
| `GetBoundingBox` | 获取边界框 |

#### CFX_GraphState 类

**位置**: cfx_graphstate.h/cpp

图形状态，包含绑制属性：

| 属性 | 描述 |
|------|------|
| 线宽 | 笔画宽度 |
| 线帽样式 | 线端处理（平头/圆头/方头） |
| 线连接样式 | 线连接处理（尖角/圆角/斜角） |
| 虚线模式 | 虚线间隔数组 |
| 斜接限制 | 尖角限制值 |

### 3. 字体管理系统

#### CFX_FontMgr 类

**位置**: cfx_fontmgr.h/cpp

字体管理器，管理所有字体资源：

| 方法 | 功能 |
|------|------|
| `GetFace` | 获取 FreeType 字体面 |
| `GetBuiltinFont` | 获取内置字体 |
| `AddInstalledFont` | 添加已安装字体 |
| `MatchInstalledFonts` | 匹配系统字体 |

#### CFX_FontMapper 类

**位置**: cfx_fontmapper.h/cpp

字体映射器，将 PDF 字体名映射到系统字体：

| 方法 | 功能 |
|------|------|
| `FindSubstFont` | 查找替代字体 |
| `GetCachedTTCFace` | 获取缓存的 TTC 字体 |
| `GetCachedFace` | 获取缓存的字体 |

字体匹配考虑因素：
- 字体名称
- 字体家族
- 字重（粗细）
- 斜体
- 字符集

#### CFX_Font 类

**位置**: cfx_font.h/cpp

字体类，封装单个字体：

| 方法 | 功能 |
|------|------|
| `LoadSubst` | 加载替代字体 |
| `LoadEmbedded` | 加载嵌入字体 |
| `GetFace` | 获取 FreeType 面 |
| `GetGlyphWidth` | 获取字形宽度 |
| `GetCharBBox` | 获取字符边界框 |
| `GetGlyphBitmap` | 获取字形位图 |

#### CFX_Face 类

**位置**: cfx_face.h/cpp

封装 FreeType 的 `FT_Face`，管理字体面生命周期。

### 4. 字形缓存系统

#### CFX_GlyphCache 类

**位置**: cfx_glyphcache.h/cpp

缓存已渲染的字形位图，提高文本渲染性能：

| 方法 | 功能 |
|------|------|
| `LoadGlyphBitmap` | 加载字形位图 |
| `LoadGlyphPath` | 加载字形路径 |
| `GetGlyphBitmap` | 获取缓存的位图 |

缓存键包括：
- 字形索引
- 字体大小
- 变换矩阵
- 渲染选项

#### CFX_GlyphBitmap 类

**位置**: cfx_glyphbitmap.h/cpp

字形位图，包含渲染后的字形图像和位置信息。

### 5. 设备无关位图 (dib/)

#### CFX_DIBBase 类

位图基类，定义位图的基本属性和操作：

| 属性 | 描述 |
|------|------|
| 宽度 | 像素宽度 |
| 高度 | 像素高度 |
| 格式 | 像素格式（RGB/ARGB/灰度等） |
| 调色板 | 可选的调色板 |

#### CFX_DIBitmap 类

可写位图，支持像素操作：

| 方法 | 功能 |
|------|------|
| `Create` | 创建位图 |
| `GetBuffer` | 获取像素缓冲区 |
| `GetWritableScanline` | 获取可写扫描行 |
| `Clear` | 清除为指定颜色 |
| `CompositeBitmap` | 合成另一位图 |
| `TransferBitmap` | 传输位图 |

#### 其他 DIB 类

| 类 | 功能 |
|----|------|
| `CFX_ImageRenderer` | 图像渲染 |
| `CFX_ImageStretcher` | 图像拉伸 |
| `CFX_ImageTransformer` | 图像变换 |
| `CFX_ScanlineCompositor` | 扫描行合成 |

### 6. 渲染后端

#### AGG 后端 (agg/)

基于 Anti-Grain Geometry 的软件渲染：
- 纯软件实现
- 高质量抗锯齿
- 跨平台

#### Skia 后端 (skia/)

基于 Skia 图形库：
- 硬件加速支持
- 与 Chrome 共享
- 更好的性能

#### Windows 后端 (win32/)

使用 Windows GDI/GDI+：
- 原生文本渲染
- 打印支持
- DirectWrite 集成

#### Apple 后端 (apple/)

使用 Core Graphics：
- 原生 macOS/iOS 渲染
- 高 DPI 支持

### 7. 颜色处理

#### CFX_Color 类

**位置**: cfx_color.h/cpp

颜色类，支持多种颜色模型：

| 颜色类型 | 描述 |
|----------|------|
| `kTransparent` | 透明 |
| `kGray` | 灰度 |
| `kRGB` | RGB 颜色 |
| `kCMYK` | CMYK 颜色 |

### 8. 图形引擎模块

#### CFX_GEModule 类

**位置**: cfx_gemodule.h/cpp

图形引擎的全局初始化和清理：

| 方法 | 功能 |
|------|------|
| `Create` | 创建模块实例 |
| `Destroy` | 销毁模块 |
| `Get` | 获取单例 |
| `GetFontMgr` | 获取字体管理器 |

## 渲染流程

### 页面渲染流程

```
渲染请求
    │
    ├── 创建 CFX_RenderDevice
    │
    ├── 设置裁剪区域和变换
    │
    ├── 遍历页面对象
    │   ├── 路径对象 → DrawPath()
    │   ├── 文本对象 → DrawNormalText()
    │   ├── 图像对象 → SetDIBits()
    │   └── 着色对象 → 特殊处理
    │
    └── 完成渲染
```

### 文本渲染流程

```
DrawNormalText()
    │
    ├── 获取字体和字形
    │
    ├── 查询字形缓存
    │   ├── 缓存命中 → 使用缓存位图
    │   └── 缓存未命中 → 渲染字形
    │
    ├── 确定字形位置
    │
    └── 合成到目标
```

## 依赖关系

```
fxge/
    ├── fxcrt/           (基础设施)
    ├── third_party/
    │   ├── freetype/    (字体渲染)
    │   ├── agg/         (软件渲染)
    │   └── skia/        (可选渲染后端)
    └── 平台 API
        ├── Windows GDI
        ├── Core Graphics
        └── Android Canvas
```

## 性能优化

- 字形缓存减少重复渲染
- 路径裁剪避免无效绘制
- 位图缓存减少解码开销
- 分块渲染支持大页面
