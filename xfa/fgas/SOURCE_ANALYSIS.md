# xfa/fgas 源码分析

## 模块概述

`fgas` (FoxIt Graphics and Script) 模块提供 XFA 所需的图形和脚本支持服务，包括字体管理、文本布局、图形绘制等功能。

## 目录结构

```
xfa/fgas/
├── README.md                         # 模块说明
│
├── crt/                              # 运行时工具
│   └── cfgas_stringformatter.cpp/h   # 字符串格式化
│
├── font/                             # 字体管理
│   ├── cfgas_fontmgr.cpp/h           # 字体管理器
│   ├── cfgas_gefont.cpp/h            # 字体对象
│   ├── cfgas_pdffontmgr.cpp/h        # PDF 字体管理
│   └── cfgas_defaultfontmanager.cpp/h # 默认字体管理
│
├── graphics/                         # 图形绘制
│   ├── cfgas_gegraphics.cpp/h        # 图形对象
│   ├── cfgas_gepath.cpp/h            # 路径对象
│   └── cfgas_gecolor.cpp/h           # 颜色对象
│
└── layout/                           # 文本布局
    ├── cfgas_txtbreak.cpp/h          # 文本断行
    ├── cfgas_break.cpp/h             # 断行基类
    ├── cfgas_char.cpp/h              # 字符信息
    ├── cfgas_linebreak.cpp/h         # 行断点
    └── cfgas_textuserdata.cpp/h      # 文本用户数据
```

## 核心组件分析

### 1. 字体管理 (font/)

#### CFGAS_FontMgr 类

**位置**: cfgas_fontmgr.h/cpp

XFA 字体管理器。

| 方法 | 功能 |
|------|------|
| `GetFontByCodePage` | 按代码页获取字体 |
| `GetFontByUnicode` | 按 Unicode 获取字体 |
| `GetDefFontByLanguage` | 按语言获取默认字体 |
| `GetFontByName` | 按名称获取字体 |
| `GetFontByLanguage` | 按语言获取字体 |
| `MatchFont` | 匹配字体 |

#### CFGAS_GEFont 类

**位置**: cfgas_gefont.h/cpp

XFA 字体对象。

| 方法 | 功能 |
|------|------|
| `GetFamilyName` | 获取家族名 |
| `GetCharWidth` | 获取字符宽度 |
| `GetCharBBox` | 获取字符边界框 |
| `GetGlyphIndex` | 获取字形索引 |
| `GetAscent` | 获取上升高度 |
| `GetDescent` | 获取下降高度 |
| `GetDevFont` | 获取设备字体 |
| `HasCharacter` | 是否包含字符 |

#### CFGAS_PDFFontMgr 类

**位置**: cfgas_pdffontmgr.h/cpp

PDF 嵌入字体管理。

| 方法 | 功能 |
|------|------|
| `FindFont` | 查找字体 |
| `GetFont` | 获取字体 |

### 2. 图形绘制 (graphics/)

#### CFGAS_GEGraphics 类

**位置**: cfgas_gegraphics.h/cpp

XFA 图形绑制对象。

| 方法 | 功能 |
|------|------|
| `SaveGraphState` | 保存图形状态 |
| `RestoreGraphState` | 恢复图形状态 |
| `SetLineCap` | 设置线帽 |
| `SetLineDash` | 设置虚线 |
| `SetLineWidth` | 设置线宽 |
| `SetFillColor` | 设置填充色 |
| `SetStrokeColor` | 设置描边色 |
| `FillPath` | 填充路径 |
| `StrokePath` | 描边路径 |
| `FillRect` | 填充矩形 |
| `StrokeRect` | 描边矩形 |
| `SetClipRect` | 设置裁剪矩形 |
| `GetClipRect` | 获取裁剪矩形 |
| `ConcatMatrix` | 连接矩阵 |

#### CFGAS_GEPath 类

**位置**: cfgas_gepath.h/cpp

XFA 路径对象。

| 方法 | 功能 |
|------|------|
| `Clear` | 清除路径 |
| `MoveTo` | 移动到点 |
| `LineTo` | 画线到点 |
| `BezierTo` | 贝塞尔曲线 |
| `ArcTo` | 弧线 |
| `Close` | 闭合路径 |
| `AddLine` | 添加线段 |
| `AddRectangle` | 添加矩形 |
| `AddEllipse` | 添加椭圆 |
| `AddArc` | 添加弧 |
| `AddPath` | 添加路径 |
| `TransformBy` | 变换路径 |

#### CFGAS_GEColor 类

**位置**: cfgas_gecolor.h/cpp

XFA 颜色对象。

颜色类型：
- 纯色
- 着色图案
- 线性渐变
- 径向渐变

| 方法 | 功能 |
|------|------|
| `GetType` | 获取类型 |
| `GetArgb` | 获取 ARGB |
| `GetPattern` | 获取图案 |
| `GetShading` | 获取着色 |

### 3. 文本布局 (layout/)

#### CFGAS_TxtBreak 类

**位置**: cfgas_txtbreak.h/cpp

文本断行器，负责文本的换行和排版。

| 方法 | 功能 |
|------|------|
| `SetLineWidth` | 设置行宽 |
| `SetFont` | 设置字体 |
| `SetFontSize` | 设置字号 |
| `SetParagraphBreakChar` | 设置段落分隔符 |
| `AppendChar` | 添加字符 |
| `EndBreak` | 结束断行 |
| `GetCharRects` | 获取字符矩形 |
| `GetCurrentLineForTesting` | 获取当前行 |

断行状态：

| 状态 | 描述 |
|------|------|
| `None` | 无断行 |
| `Piece` | 片段断行 |
| `Line` | 行断行 |
| `Paragraph` | 段落断行 |
| `Page` | 页面断行 |

#### CFGAS_Char 类

**位置**: cfgas_char.h/cpp

字符信息类。

| 属性 | 描述 |
|------|------|
| `char_code_` | 字符编码 |
| `char_width_` | 字符宽度 |
| `char_props_` | 字符属性 |
| `horizontal_scale_` | 水平缩放 |
| `vertical_scale_` | 垂直缩放 |

### 4. 运行时工具 (crt/)

#### CFGAS_StringFormatter 类

**位置**: cfgas_stringformatter.h/cpp

字符串格式化器，用于 XFA 表单的数据格式化。

| 方法 | 功能 |
|------|------|
| `FormatText` | 格式化文本 |
| `FormatNum` | 格式化数字 |
| `FormatDateTime` | 格式化日期时间 |
| `FormatZero` | 格式化零值 |
| `FormatNull` | 格式化空值 |
| `ParseText` | 解析文本 |
| `ParseNum` | 解析数字 |
| `ParseDateTime` | 解析日期时间 |

格式化模式类型：

| 类型 | 描述 |
|------|------|
| `text` | 文本格式 |
| `num` | 数字格式 |
| `date` | 日期格式 |
| `time` | 时间格式 |
| `dateTime` | 日期时间格式 |
| `null` | 空值格式 |
| `zero` | 零值格式 |

## 字体匹配算法

```
请求字体
    │
    ├── 按名称精确匹配
    │   └── 找到 → 返回
    │
    ├── 按字符支持匹配
    │   ├── 检查 Unicode 范围
    │   └── 找到 → 返回
    │
    ├── 按语言/代码页匹配
    │   └── 找到 → 返回
    │
    └── 返回默认字体
```

## 文本断行算法

```
输入文本
    │
    ├── 遍历字符
    │
    ├── 确定字符属性
    │   ├── 字符类型
    │   ├── 断行类型
    │   └── 方向
    │
    ├── 检查断行条件
    │   ├── 行宽超限
    │   ├── 强制断行符
    │   └── 分词边界
    │
    └── 生成断行结果
```

## 依赖关系

```
fgas/
    ├── core/fxge/       (图形引擎)
    │   └── CFX_Font
    ├── core/fxcrt/      (基础设施)
    └── third_party/
        └── icu/         (Unicode)

被依赖:
    xfa/fxfa/ → fgas/
    xfa/fwl/  → fgas/
```

## 与 fxge 的关系

fgas 建立在 fxge 之上，提供 XFA 专用的抽象：

| fgas 类 | fxge 基础 |
|---------|-----------|
| CFGAS_GEGraphics | CFX_RenderDevice |
| CFGAS_GEPath | CFX_Path |
| CFGAS_GEFont | CFX_Font |

## 国际化支持

通过 ICU 库支持：
- 双向文本 (BiDi)
- 复杂脚本
- 字符分类
- 断行规则
