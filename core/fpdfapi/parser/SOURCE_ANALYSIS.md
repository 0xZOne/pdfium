# core/fpdfapi/parser 源码分析

## 模块概述

`parser` 模块是 PDF 文件解析的核心，负责读取和解释 PDF 文件结构。它实现了 PDF 规范中定义的语法分析、对象解析、交叉引用表处理、加密处理等功能。

## 目录结构

```
core/fpdfapi/parser/
├── BUILD.gn                          # 构建配置
│
├── 文档和解析器
│   ├── cpdf_document.cpp/h           # PDF 文档类
│   ├── cpdf_parser.cpp/h             # PDF 解析器
│   ├── cpdf_syntax_parser.cpp/h      # 语法解析器
│   └── cpdf_simple_parser.cpp/h      # 简单解析器
│
├── PDF 对象类型
│   ├── cpdf_object.cpp/h             # 对象基类
│   ├── cpdf_boolean.cpp/h            # 布尔对象
│   ├── cpdf_number.cpp/h             # 数字对象
│   ├── cpdf_string.cpp/h             # 字符串对象
│   ├── cpdf_name.cpp/h               # 名称对象
│   ├── cpdf_array.cpp/h              # 数组对象
│   ├── cpdf_dictionary.cpp/h         # 字典对象
│   ├── cpdf_stream.cpp/h             # 流对象
│   ├── cpdf_null.cpp/h               # 空对象
│   └── cpdf_reference.cpp/h          # 引用对象
│
├── 交叉引用
│   ├── cpdf_cross_ref_table.cpp/h    # 交叉引用表
│   ├── cpdf_cross_ref_avail.cpp/h    # 交叉引用可用性
│   └── cpdf_hint_tables.cpp/h        # 提示表（线性化）
│
├── 加密和安全
│   ├── cpdf_security_handler.cpp/h   # 安全处理器
│   ├── cpdf_crypto_handler.cpp/h     # 加密处理器
│   └── cpdf_encryptor.cpp/h          # 加密器
│
├── 流处理
│   ├── cpdf_stream_acc.cpp/h         # 流访问器
│   ├── cpdf_seekablemultistream.cpp/h # 多流寻址
│   ├── cpdf_flateencoder.cpp/h       # Flate 编码器
│   └── fpdf_parser_decode.cpp/h      # 解码工具
│
├── 对象管理
│   ├── cpdf_indirect_object_holder.cpp/h # 间接对象持有者
│   ├── cpdf_object_stream.cpp/h      # 对象流
│   ├── cpdf_object_walker.cpp/h      # 对象遍历器
│   └── cpdf_object_avail.cpp/h       # 对象可用性
│
├── 线性化支持
│   ├── cpdf_linearized_header.cpp/h  # 线性化头
│   ├── cpdf_data_avail.cpp/h         # 数据可用性
│   └── cpdf_page_object_avail.cpp/h  # 页面对象可用性
│
├── 工具
│   ├── fpdf_parser_utility.cpp/h     # 解析工具函数
│   ├── cpdf_read_validator.cpp/h     # 读取验证器
│   └── object_tree_traversal_util.cpp/h # 对象树遍历
│
└── FDF 支持
    └── cfdf_document.cpp/h           # FDF 文档
```

## 核心类分析

### 1. CPDF_Document 类

**位置**: cpdf_document.h/cpp

PDF 文档的顶层表示，管理整个文档结构。

| 方法 | 功能 |
|------|------|
| `LoadDoc` | 加载文档 |
| `GetPageCount` | 获取页面数 |
| `GetPageDictionary` | 获取页面字典 |
| `GetRoot` | 获取根字典 |
| `GetInfo` | 获取文档信息 |
| `CreateNewPage` | 创建新页面 |
| `DeletePage` | 删除页面 |

内部结构：
- 解析器实例
- 页面索引表
- 文档目录（Catalog）
- 信息字典

### 2. CPDF_Parser 类

**位置**: cpdf_parser.h/cpp

PDF 文件解析器，负责读取和解释文件结构。

#### 解析错误类型

| 错误码 | 描述 |
|--------|------|
| `SUCCESS` | 成功 |
| `FILE_ERROR` | 文件错误 |
| `FORMAT_ERROR` | 格式错误 |
| `PASSWORD_ERROR` | 密码错误 |
| `HANDLER_ERROR` | 处理器错误 |

#### 主要方法

| 方法 | 功能 |
|------|------|
| `StartParse` | 开始解析 |
| `SetPassword` | 设置密码 |
| `GetDocument` | 获取文档 |
| `ParseIndirectObject` | 解析间接对象 |
| `LoadCrossRefV4` | 加载 v4 交叉引用 |
| `RebuildCrossRef` | 重建交叉引用 |

#### 解析流程

```
StartParse()
    │
    ├── 检查 PDF 头（%PDF-x.x）
    │
    ├── 查找 startxref
    │
    ├── 解析交叉引用表
    │   ├── 传统格式（xref 表）
    │   └── 流格式（xref 流）
    │
    ├── 解析 trailer 字典
    │
    ├── 处理增量更新
    │
    └── 处理加密
```

### 3. CPDF_SyntaxParser 类

**位置**: cpdf_syntax_parser.h/cpp

底层语法解析器，处理 PDF 词法分析。

| 方法 | 功能 |
|------|------|
| `GetNextWord` | 获取下一个词 |
| `GetObject` | 解析对象 |
| `GetKeyword` | 获取关键字 |
| `GetNumber` | 解析数字 |
| `GetString` | 解析字符串 |

### 4. PDF 对象层次

#### CPDF_Object 基类

所有 PDF 对象的抽象基类。

| 方法 | 功能 |
|------|------|
| `GetType` | 获取对象类型 |
| `IsBoolean/IsNumber/...` | 类型检查 |
| `AsBoolean/AsNumber/...` | 类型转换 |
| `Clone` | 克隆对象 |
| `MakeReference` | 创建引用 |

#### 对象类型

| 类 | PDF 类型 | 描述 |
|----|----------|------|
| `CPDF_Boolean` | boolean | true/false |
| `CPDF_Number` | number | 整数或浮点数 |
| `CPDF_String` | string | 字面/十六进制字符串 |
| `CPDF_Name` | name | /Name 格式 |
| `CPDF_Array` | array | [a b c] 格式 |
| `CPDF_Dictionary` | dictionary | << /Key Value >> 格式 |
| `CPDF_Stream` | stream | 字典 + 二进制数据 |
| `CPDF_Null` | null | null |
| `CPDF_Reference` | reference | n 0 R 格式 |

#### CPDF_Dictionary

字典对象，PDF 中最常用的复合类型：

| 方法 | 功能 |
|------|------|
| `GetCount` | 获取条目数 |
| `KeyExist` | 检查键是否存在 |
| `GetDirectObjectFor` | 获取直接值 |
| `GetIndirectObjectFor` | 获取间接值 |
| `SetNewFor` | 设置新值 |
| `RemoveFor` | 删除条目 |

#### CPDF_Stream

流对象，包含字典和二进制数据：

| 方法 | 功能 |
|------|------|
| `GetDict` | 获取关联字典 |
| `GetRawSize` | 获取原始大小 |
| `GetDecodedData` | 获取解码数据 |
| `SetData` | 设置数据 |

### 5. 交叉引用系统

#### CPDF_CrossRefTable 类

**位置**: cpdf_cross_ref_table.h/cpp

管理对象偏移量和状态：

| 条目类型 | 描述 |
|----------|------|
| `kFree` | 空闲对象 |
| `kNormal` | 普通对象 |
| `kCompressed` | 压缩对象（在对象流中） |

### 6. 加密系统

#### CPDF_SecurityHandler 类

**位置**: cpdf_security_handler.h/cpp

处理 PDF 加密和权限：

| 方法 | 功能 |
|------|------|
| `OnInit` | 初始化加密处理 |
| `CheckPassword` | 验证密码 |
| `GetPermissions` | 获取权限标志 |
| `IsMetadataEncrypted` | 元数据是否加密 |

支持的加密版本：
- RC4 40位
- RC4 128位
- AES 128位
- AES 256位

#### CPDF_CryptoHandler 类

**位置**: cpdf_crypto_handler.h/cpp

执行实际的加解密操作。

### 7. 流处理

#### CPDF_StreamAcc 类

**位置**: cpdf_stream_acc.h/cpp

流访问器，处理流的解码：

| 方法 | 功能 |
|------|------|
| `LoadAllData` | 加载所有数据 |
| `LoadAllDataFiltered` | 加载过滤后的数据 |
| `GetSpan` | 获取数据视图 |

支持的过滤器：
- FlateDecode
- ASCIIHexDecode
- ASCII85Decode
- LZWDecode
- RunLengthDecode
- CCITTFaxDecode
- JBIG2Decode
- DCTDecode
- JPXDecode
- Crypt

### 8. 线性化支持

#### CPDF_DataAvail 类

**位置**: cpdf_data_avail.h/cpp

支持渐进式加载：

| 状态 | 描述 |
|------|------|
| `kDataNotAvailable` | 数据不可用 |
| `kDataAvailable` | 数据可用 |
| `kDataError` | 数据错误 |

| 方法 | 功能 |
|------|------|
| `IsDocAvail` | 文档是否可用 |
| `IsPageAvail` | 页面是否可用 |
| `IsFormAvail` | 表单是否可用 |

## 解析流程详解

### 文档加载流程

```
1. 打开文件/内存流
2. 验证 PDF 头
3. 定位 startxref
4. 解析交叉引用表
5. 解析 trailer
6. 处理加密（如需要）
7. 加载 Catalog
8. 构建页面树
```

### 对象解析流程

```
请求对象 (obj_num)
    │
    ├── 查询交叉引用表
    │   ├── 普通对象 → 读取偏移量
    │   └── 压缩对象 → 定位对象流
    │
    ├── 定位并读取数据
    │
    ├── 词法分析
    │
    ├── 构建对象
    │
    └── 缓存对象
```

## 错误处理

解析器具有容错能力：
- 自动修复常见格式错误
- 支持重建损坏的交叉引用
- 跳过无法解析的对象

## 性能优化

- 延迟解析：按需加载对象
- 对象缓存：避免重复解析
- 流式读取：减少内存占用
- 线性化支持：快速首页显示

## 依赖关系

```
parser/
    ├── fxcrt/     (基础设施)
    └── fdrm/      (加密解密)
```
