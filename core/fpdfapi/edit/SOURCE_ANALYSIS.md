# core/fpdfapi/edit 源码分析

## 模块概述

`edit` 模块提供 PDF 文档的创建和编辑功能，包括生成新的 PDF 内容、保存文档、页面操作等。它是实现 PDF 编辑 API 的核心模块。

## 目录结构

```
core/fpdfapi/edit/
├── BUILD.gn                          # 构建配置
│
├── 文档创建和保存
│   ├── cpdf_creator.cpp/h            # PDF 创建器
│   └── cpdf_stringarchivestream.cpp/h # 字符串归档流
│
├── 页面内容生成
│   ├── cpdf_pagecontentgenerator.cpp/h # 页面内容生成器
│   ├── cpdf_pagecontentmanager.cpp/h   # 页面内容管理器
│   └── cpdf_contentstream_write_utils.cpp/h # 内容流写入工具
│
└── 页面操作
    ├── cpdf_pageorganizer.cpp/h      # 页面组织器
    ├── cpdf_pageexporter.cpp/h       # 页面导出器
    └── cpdf_npagetooneexporter.cpp/h # 多页合一导出器
```

## 核心类分析

### 1. CPDF_Creator 类

**位置**: cpdf_creator.h/cpp

PDF 文档创建器，负责将 PDF 文档写入输出流。

| 方法 | 功能 |
|------|------|
| `Create` | 开始创建过程 |
| `Continue` | 继续创建（增量） |
| `SetFileVersion` | 设置 PDF 版本 |

创建流程阶段：

| 阶段 | 描述 |
|------|------|
| 写入头部 | %PDF-x.x 头 |
| 写入对象 | 所有间接对象 |
| 写入交叉引用 | xref 表或 xref 流 |
| 写入 trailer | trailer 字典 |
| 写入 startxref | 交叉引用偏移 |
| 写入 %%EOF | 文件结束标记 |

保存模式：
- 完整保存：重写整个文件
- 增量保存：追加修改到文件末尾

### 2. CPDF_PageContentGenerator 类

**位置**: cpdf_pagecontentgenerator.h/cpp

页面内容生成器，将页面对象转换为 PDF 内容流。

| 方法 | 功能 |
|------|------|
| `GenerateContent` | 生成内容流 |
| `ProcessPageObjects` | 处理页面对象 |

处理的对象类型：

| 对象类型 | 生成的操作符 |
|----------|--------------|
| 路径对象 | m, l, c, re, S, f 等 |
| 文本对象 | BT, Tf, Td, Tj, ET 等 |
| 图像对象 | q, cm, Do, Q |
| 表单对象 | q, cm, Do, Q |

#### 路径对象生成

```
路径对象 → 内容流
    │
    ├── 输出图形状态 (gs, w, J, j, M, d)
    │
    ├── 输出颜色状态 (g, rg, k, G, RG, K)
    │
    ├── 输出路径构建命令
    │   ├── MoveTo → m
    │   ├── LineTo → l
    │   ├── BezierTo → c
    │   └── ClosePath → h
    │
    └── 输出绑制命令 (S, f, B, n 等)
```

#### 文本对象生成

```
文本对象 → 内容流
    │
    ├── BT (开始文本块)
    │
    ├── 设置字体 (Tf)
    │
    ├── 设置文本矩阵 (Tm)
    │
    ├── 输出文本 (Tj 或 TJ)
    │
    └── ET (结束文本块)
```

#### 图像对象生成

```
图像对象 → 内容流
    │
    ├── q (保存状态)
    │
    ├── cm (变换矩阵)
    │
    ├── /ImageName Do (绘制图像)
    │
    └── Q (恢复状态)
```

### 3. CPDF_PageContentManager 类

**位置**: cpdf_pagecontentmanager.h/cpp

页面内容管理器，管理页面的内容流数组。

| 方法 | 功能 |
|------|------|
| `AddStream` | 添加内容流 |
| `ScheduleRemoveStream` | 调度移除流 |
| `GetContentStream` | 获取内容流 |

页面内容组织：
- 单一内容流：直接引用
- 多内容流：Contents 数组

### 4. 内容流写入工具

#### cpdf_contentstream_write_utils

**位置**: cpdf_contentstream_write_utils.h/cpp

提供写入 PDF 内容流的工具函数：

| 函数 | 功能 |
|------|------|
| `WriteFloat` | 写入浮点数 |
| `WriteMatrix` | 写入变换矩阵 |
| `WritePoint` | 写入点坐标 |
| `WriteRect` | 写入矩形 |

数值格式化规则：
- 浮点数精度优化
- 避免不必要的小数位
- 紧凑表示

### 5. CPDF_PageOrganizer 类

**位置**: cpdf_pageorganizer.h/cpp

页面组织器基类，用于页面操作。

派生类：
- CPDF_PageExporter
- CPDF_NPageToOneExporter

### 6. CPDF_PageExporter 类

**位置**: cpdf_pageexporter.h/cpp

页面导出器，从一个文档复制页面到另一个文档。

| 方法 | 功能 |
|------|------|
| `ExportPage` | 导出页面 |

页面导出流程：
1. 复制页面字典
2. 复制引用的资源
3. 处理对象号映射
4. 更新引用

### 7. CPDF_NPageToOneExporter 类

**位置**: cpdf_npagetooneexporter.h/cpp

多页合一导出器，将多个页面合并到一个页面。

| 方法 | 功能 |
|------|------|
| `ExportNPagesToOne` | 执行多页合一 |

功能特点：
- 支持任意页面数量
- 自动计算布局
- 保持页面比例
- 处理旋转

### 8. CPDF_StringArchiveStream 类

**位置**: cpdf_stringarchivestream.h/cpp

字符串归档流，用于将 PDF 内容写入字符串缓冲区。

| 方法 | 功能 |
|------|------|
| `WriteBlock` | 写入数据块 |
| `GetResult` | 获取结果字符串 |

## 文档保存流程

### 完整保存

```
FPDF_SaveAsCopy()
    │
    ├── 创建 CPDF_Creator
    │
    ├── 设置保存选项
    │
    ├── 写入 PDF 头
    │   └── %PDF-1.7
    │
    ├── 写入所有对象
    │   ├── 遍历间接对象
    │   ├── 序列化每个对象
    │   └── 记录偏移量
    │
    ├── 写入交叉引用表
    │   ├── xref 关键字
    │   ├── 对象偏移条目
    │   └── trailer 字典
    │
    ├── 写入 startxref
    │
    └── 写入 %%EOF
```

### 增量保存

```
FPDF_SaveWithVersion() [增量模式]
    │
    ├── 保留原始内容
    │
    ├── 追加修改的对象
    │
    ├── 写入新的交叉引用
    │   └── 指向前一个 xref
    │
    └── 写入新的 trailer
        └── /Prev 指向旧 xref
```

## 页面内容生成流程

```
CPDF_PageContentGenerator::GenerateContent()
    │
    ├── 获取页面对象列表
    │
    ├── 初始化内容流缓冲区
    │
    ├── 遍历对象
    │   ├── 处理路径对象
    │   │   ├── 输出状态
    │   │   ├── 输出路径
    │   │   └── 输出绘制命令
    │   │
    │   ├── 处理文本对象
    │   │   ├── BT
    │   │   ├── 字体和文本
    │   │   └── ET
    │   │
    │   ├── 处理图像对象
    │   │   ├── 添加图像资源
    │   │   └── 输出 Do 命令
    │   │
    │   └── 处理表单对象
    │       ├── 添加 XObject 资源
    │       └── 输出 Do 命令
    │
    └── 创建内容流对象
```

## 资源管理

生成内容时自动管理资源字典：

| 资源类型 | 说明 |
|----------|------|
| `/Font` | 字体资源 |
| `/XObject` | 图像和表单 XObject |
| `/ExtGState` | 扩展图形状态 |
| `/ColorSpace` | 颜色空间 |
| `/Pattern` | 图案 |
| `/Shading` | 着色 |

## 对象序列化

### PDF 对象写入格式

| 对象类型 | 格式 |
|----------|------|
| Boolean | `true` / `false` |
| Number | `123` / `3.14159` |
| String | `(text)` / `<hex>` |
| Name | `/Name` |
| Array | `[a b c]` |
| Dictionary | `<< /Key Value >>` |
| Stream | `<< ... >> stream ... endstream` |
| Reference | `n 0 R` |

### 间接对象写入

```
obj_num 0 obj
  << 对象内容 >>
endobj
```

## 依赖关系

```
edit/
    ├── parser/      (PDF 对象)
    ├── page/        (页面对象)
    └── fxcrt/       (基础设施)
```

## 注意事项

- 保存时自动压缩流对象
- 增量保存保留原始签名
- 对象重新编号优化文件大小
- 未使用对象可被清理
