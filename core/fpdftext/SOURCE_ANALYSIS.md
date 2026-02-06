# core/fpdftext 源码分析

## 模块概述

`fpdftext` 模块负责从 PDF 页面中提取文本内容。它提供了文本提取、文本搜索和链接提取等功能，是实现 PDF 文本处理的核心组件。

## 目录结构

```
core/fpdftext/
├── BUILD.gn                      # 构建配置
├── cpdf_textpage.cpp             # 文本页面核心实现
├── cpdf_textpage.h               # 文本页面头文件
├── cpdf_textpagefind.cpp         # 文本搜索实现
├── cpdf_textpagefind.h           # 文本搜索头文件
├── cpdf_linkextract.cpp          # 链接提取实现
├── cpdf_linkextract.h            # 链接提取头文件
├── cpdf_linkextract_unittest.cpp # 链接提取单元测试
├── unicodenormalizationdata.cpp  # Unicode 规范化数据
└── unicodenormalizationdata.h    # Unicode 规范化头文件
```

## 核心类分析

### 1. CPDF_TextPage 类

**位置**: cpdf_textpage.h/cpp

#### 类概述

`CPDF_TextPage` 是文本提取的核心类，负责解析 PDF 页面中的所有文本对象，并将它们组织成可供查询的字符序列。

#### CharType 枚举

定义字符的类型分类：
- `kNormal`: 普通字符
- `kGenerated`: 生成的字符（如添加的空格）
- `kNotUnicode`: 无法映射到 Unicode 的字符
- `kHyphen`: 连字符
- `kPiece`: 片段字符

#### CharInfo 嵌套类

**位置**: cpdf_textpage.h 第 46-86 行

存储单个字符的详细信息：

| 成员 | 类型 | 描述 |
|------|------|------|
| `char_type_` | CharType | 字符类型 |
| `unicode_` | wchar_t | Unicode 码点 |
| `char_code_` | uint32_t | 原始字符编码 |
| `origin_` | CFX_PointF | 字符原点坐标 |
| `char_box_` | CFX_FloatRect | 字符边界框 |
| `loose_char_box_` | CFX_FloatRect | 宽松边界框 |
| `matrix_` | CFX_Matrix | 变换矩阵 |
| `text_object_` | CPDF_TextObject* | 关联的文本对象 |

#### 主要公共方法

| 方法 | 功能描述 |
|------|----------|
| `CharIndexFromTextIndex` | 从文本索引转换为字符索引 |
| `TextIndexFromCharIndex` | 从字符索引转换为文本索引 |
| `CountChars` | 获取总字符数 |
| `GetCharInfo` | 获取指定索引的字符信息 |
| `GetCharFontSize` | 获取字符字体大小 |
| `GetCharLooseBounds` | 获取字符的宽松边界 |
| `GetPageText` | 获取页面的完整文本 |
| `GetTextByRect` | 获取指定矩形区域内的文本 |
| `GetCharIndexAtPos` | 根据坐标获取字符索引 |
| `GetRectArray` | 获取指定范围文本的矩形数组 |

#### 文本提取算法

**位置**: cpdf_textpage.cpp

文本提取的主要流程：

1. **遍历页面对象**: 递归处理页面中的所有对象
2. **处理文本对象**: 对每个 CPDF_TextObject 提取字符
3. **字符排序**: 根据阅读顺序排列字符
4. **空格推断**: 根据字符间距插入空格
5. **换行处理**: 识别换行位置并插入换行符

### 2. CPDF_TextPageFind 类

**位置**: cpdf_textpagefind.h/cpp

#### 类概述

实现在文本页面中搜索指定文本的功能。

#### 主要功能

| 方法 | 功能描述 |
|------|----------|
| `FindFirst` | 开始搜索，定位第一个匹配 |
| `FindNext` | 查找下一个匹配 |
| `FindPrev` | 查找上一个匹配 |
| `GetCurOrder` | 获取当前匹配的位置 |
| `GetMatchedCount` | 获取当前匹配的字符数 |

#### 搜索选项

- 区分大小写
- 全词匹配
- 连续搜索

#### 搜索算法

1. 将搜索文本规范化
2. 遍历文本页面内容
3. 使用字符串匹配算法查找
4. 处理 Unicode 规范化差异
5. 返回匹配位置和范围

### 3. CPDF_LinkExtract 类

**位置**: cpdf_linkextract.h/cpp

#### 类概述

从文本页面中自动识别和提取 URL 链接。

#### 主要方法

| 方法 | 功能描述 |
|------|----------|
| `ExtractLinks` | 执行链接提取 |
| `CountLinks` | 获取提取到的链接数量 |
| `GetURL` | 获取指定索引的 URL |
| `GetBoundedSegment` | 获取链接的边界区域 |

#### 链接识别规则

支持识别以下模式：
- `http://` 和 `https://` 开头的 URL
- `www.` 开头的网址
- 电子邮件地址（包含 `@` 符号）

## Unicode 规范化

### unicodenormalizationdata 模块

**位置**: unicodenormalizationdata.cpp/h

提供 Unicode 规范化数据，用于：
- 处理组合字符
- 规范化等价字符
- 支持文本搜索时的字符匹配

## 文本提取流程详解

### 第一阶段：收集文本对象

```
CPDF_Page
    └── ParseContent()
            └── 遍历所有 CPDF_PageObject
                    └── 筛选 CPDF_TextObject
```

### 第二阶段：字符提取

对每个文本对象：
1. 获取字体信息
2. 解码字符编码
3. 转换为 Unicode
4. 计算字符位置和边界

### 第三阶段：文本组织

1. 根据 Y 坐标分组（识别文本行）
2. 在每行内按 X 坐标排序
3. 处理 RTL（从右到左）文本
4. 插入逻辑空格和换行符

### 第四阶段：结果存储

将处理后的字符信息存储在 `char_list_` 中，支持快速查询。

## 坐标系统

### 页面坐标到字符位置

- PDF 页面使用笛卡尔坐标（左下角为原点）
- 每个字符都有精确的边界框
- 支持坐标到字符的双向映射

### 矩形区域选择

`GetTextByRect` 方法实现：
1. 遍历所有字符
2. 检查字符边界与选择矩形的交集
3. 收集符合条件的字符
4. 按阅读顺序返回文本

## 性能优化

- 懒加载：只在需要时解析文本
- 字符缓存：避免重复解析
- 空间索引：支持快速区域查询

## 依赖关系

```
fpdftext/
    ├── fpdfapi/page/     (页面和文本对象)
    ├── fpdfapi/font/     (字体和字符编码)
    └── fxcrt/            (基础设施)
```

## 公共 API 映射

| fpdftext 类/方法 | 公共 API |
|------------------|----------|
| CPDF_TextPage | FPDFText_LoadPage |
| GetPageText | FPDFText_GetText |
| GetCharInfo | FPDFText_GetCharBox |
| GetCharIndexAtPos | FPDFText_GetCharIndexAtPos |
| CPDF_TextPageFind | FPDFText_FindStart |
| CPDF_LinkExtract | FPDFLink_LoadWebLinks |

## 使用注意事项

1. 文本提取结果取决于 PDF 的创建方式
2. 某些 PDF 可能使用自定义编码，导致提取结果不准确
3. 扫描版 PDF 需要 OCR，本模块不支持
4. RTL 文本需要特殊处理
