# core/fpdfapi/render 源码分析

## 模块概述

`render` 模块负责将 PDF 页面内容渲染到目标设备。它处理所有类型的页面对象（文本、路径、图像、着色等）的绘制，支持渐进式渲染和各种渲染选项。

## 目录结构

```
core/fpdfapi/render/
├── BUILD.gn                          # 构建配置
│
├── 渲染核心
│   ├── cpdf_rendercontext.cpp/h      # 渲染上下文
│   ├── cpdf_renderstatus.cpp/h       # 渲染状态
│   ├── cpdf_renderoptions.cpp/h      # 渲染选项
│   └── cpdf_pagerendercontext.cpp/h  # 页面渲染上下文
│
├── 渐进式渲染
│   └── cpdf_progressiverenderer.cpp/h # 渐进式渲染器
│
├── 对象渲染器
│   ├── cpdf_textrenderer.cpp/h       # 文本渲染器
│   ├── cpdf_imagerenderer.cpp/h      # 图像渲染器
│   ├── cpdf_rendershading.cpp/h      # 着色渲染
│   └── cpdf_rendertiling.cpp/h       # 平铺图案渲染
│
├── 辅助类
│   ├── charposlist.cpp/h             # 字符位置列表
│   ├── cpdf_devicebuffer.cpp/h       # 设备缓冲区
│   ├── cpdf_scaledrenderbuffer.cpp/h # 缩放渲染缓冲
│   └── cpdf_docrenderdata.cpp/h      # 文档渲染数据
│
├── Type3 字体
│   ├── cpdf_type3cache.cpp/h         # Type3 缓存
│   └── cpdf_type3glyphmap.cpp/h      # Type3 字形映射
│
└── 平台特定
    └── cpdf_windowsrenderdevice.cpp/h # Windows 渲染设备
```

## 核心类分析

### 1. CPDF_RenderContext 类

**位置**: cpdf_rendercontext.h/cpp

渲染上下文，管理渲染过程的顶层类。

| 方法 | 功能 |
|------|------|
| `AppendLayer` | 添加渲染层 |
| `Render` | 执行渲染 |
| `GetBackgroundToBitmap` | 获取背景位图 |

渲染层包括：
- 页面对象
- 变换矩阵
- 裁剪区域

### 2. CPDF_RenderStatus 类

**位置**: cpdf_renderstatus.h/cpp

渲染状态管理，是渲染的核心工作类。

| 方法 | 功能 |
|------|------|
| `Initialize` | 初始化渲染状态 |
| `RenderObjectList` | 渲染对象列表 |
| `RenderSingleObject` | 渲染单个对象 |
| `ProcessPath` | 处理路径对象 |
| `ProcessImage` | 处理图像对象 |
| `ProcessShading` | 处理着色对象 |
| `ProcessForm` | 处理表单 XObject |
| `DrawTextPathWithPattern` | 带图案的文本路径 |

内部状态：
- 当前设备
- 变换矩阵
- 渲染选项
- 图形状态栈
- 透明度组信息

### 3. CPDF_RenderOptions 类

**位置**: cpdf_renderoptions.h/cpp

渲染选项配置。

| 选项类型 | 描述 |
|----------|------|
| `kClearType` | ClearType 文本渲染 |
| `kNoTextSmooth` | 禁用文本平滑 |
| `kNoPathSmooth` | 禁用路径平滑 |
| `kNoImageSmooth` | 禁用图像平滑 |
| `kLimitedImageCache` | 限制图像缓存 |
| `kForceHalftone` | 强制半色调 |
| `kBreakForMasks` | 为蒙版中断 |
| `kConvertFillToStroke` | 填充转描边 |

颜色模式：
- 普通模式
- 灰度模式
- Alpha 模式

### 4. CPDF_ProgressiveRenderer 类

**位置**: cpdf_progressiverenderer.h/cpp

渐进式渲染器，支持分步渲染大页面。

| 状态 | 描述 |
|------|------|
| `kReady` | 就绪 |
| `kToBeContinued` | 待继续 |
| `kDone` | 完成 |
| `kFailed` | 失败 |

| 方法 | 功能 |
|------|------|
| `Start` | 开始渲染 |
| `Continue` | 继续渲染 |
| `GetStatus` | 获取状态 |

### 5. CPDF_TextRenderer 类

**位置**: cpdf_textrenderer.h/cpp

文本渲染器，处理文本对象的绘制。

| 方法 | 功能 |
|------|------|
| `DrawTextString` | 绘制文本字符串 |
| `DrawNormalText` | 绘制普通文本 |
| `DrawTextPath` | 绘制文本路径 |

文本渲染流程：
1. 获取字体和字形
2. 计算字符位置
3. 应用文本矩阵
4. 渲染字形（填充/描边）
5. 应用透明度

### 6. CPDF_ImageRenderer 类

**位置**: cpdf_imagerenderer.h/cpp

图像渲染器。

| 方法 | 功能 |
|------|------|
| `Start` | 开始图像渲染 |
| `Continue` | 继续渲染 |
| `StartDIBBase` | 开始 DIB 渲染 |
| `StartRenderDIBBase` | 开始渲染 DIB |

图像渲染流程：
1. 加载图像数据
2. 解码图像
3. 应用颜色空间转换
4. 计算目标区域
5. 拉伸/变换图像
6. 合成到目标

### 7. 着色渲染

#### CPDF_RenderShading

**位置**: cpdf_rendershading.h/cpp

着色渲染函数，处理各种类型的着色：

| 着色类型 | 描述 |
|----------|------|
| 函数着色 | 基于数学函数的着色 |
| 轴向渐变 | 沿直线的渐变 |
| 径向渐变 | 圆形渐变 |
| 自由形式 | 三角形网格 |
| 晶格形式 | 网格着色 |
| Coons 色块 | 贝塞尔曲线边界 |
| 张量积色块 | 高级曲线着色 |

### 8. 平铺图案渲染

#### CPDF_RenderTiling

**位置**: cpdf_rendertiling.h/cpp

处理平铺图案的渲染：
- 计算平铺范围
- 渲染单个平铺
- 复制平铺到目标区域

### 9. 辅助类

#### CharPosList

**位置**: charposlist.h/cpp

字符位置列表，用于文本渲染：
- 字符编码
- 字形索引
- 位置信息
- 字体引用

#### CPDF_DeviceBuffer

**位置**: cpdf_devicebuffer.h/cpp

设备缓冲区，用于隔离渲染：
- 创建临时位图
- 渲染到缓冲区
- 合成到目标

#### CPDF_ScaledRenderBuffer

**位置**: cpdf_scaledrenderbuffer.h/cpp

缩放渲染缓冲，处理高分辨率渲染。

#### CPDF_DocRenderData

**位置**: cpdf_docrenderdata.h/cpp

文档级渲染数据缓存：
- 传递函数缓存
- Type3 字体缓存
- 共享资源

### 10. Type3 字体支持

#### CPDF_Type3Cache

**位置**: cpdf_type3cache.h/cpp

Type3 字体缓存，存储渲染后的 Type3 字形。

#### CPDF_Type3GlyphMap

**位置**: cpdf_type3glyphmap.h/cpp

Type3 字形映射，管理字形位图。

## 渲染流程

### 完整页面渲染

```
FPDF_RenderPageBitmap()
    │
    ├── 创建渲染上下文
    │
    ├── 设置渲染选项
    │
    ├── 添加页面层
    │
    ├── 创建渲染设备
    │
    ├── CPDF_RenderContext::Render()
    │   │
    │   ├── 遍历每个层
    │   │
    │   ├── CPDF_RenderStatus::RenderObjectList()
    │   │   │
    │   │   ├── 遍历页面对象
    │   │   │
    │   │   ├── 检查裁剪
    │   │   │
    │   │   └── RenderSingleObject()
    │   │       ├── 路径 → ProcessPath()
    │   │       ├── 文本 → ProcessText()
    │   │       ├── 图像 → ProcessImage()
    │   │       ├── 着色 → ProcessShading()
    │   │       └── 表单 → ProcessForm()
    │   │
    │   └── 完成
    │
    └── 返回
```

### 渐进式渲染流程

```
Start()
    │
    ├── 初始化渲染器
    │
    └── 渲染第一批对象

Continue()  [可调用多次]
    │
    ├── 检查是否完成
    │
    ├── 渲染下一批对象
    │
    └── 返回状态
```

## 透明度处理

### 透明度组

处理带有透明度的对象组：
1. 创建透明度组缓冲区
2. 渲染组内对象
3. 应用混合模式
4. 合成到目标

### 软蒙版

处理软蒙版效果：
1. 渲染蒙版内容
2. 转换为 Alpha 通道
3. 应用蒙版

## 颜色空间转换

渲染时自动进行颜色空间转换：
- PDF 颜色空间 → 设备颜色空间
- ICC 配置文件处理
- CMYK → RGB 转换

## 性能优化

- 对象裁剪减少无效渲染
- 图像缓存避免重复解码
- 字形缓存提高文本性能
- 分层渲染支持局部更新
- 渐进式渲染改善用户体验

## 依赖关系

```
render/
    ├── page/        (页面对象)
    ├── font/        (字体)
    ├── fxge/        (图形引擎)
    ├── fxcodec/     (图像解码)
    └── fxcrt/       (基础设施)
```
