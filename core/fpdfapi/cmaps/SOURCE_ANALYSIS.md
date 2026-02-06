# core/fpdfapi/cmaps 源码分析

## 模块概述

`cmaps` 模块包含 PDF 中 CID 字体所需的预编译字符映射数据。CMap（Character Map）定义了字符编码（如 Unicode 或传统编码）到 CID（Character Identifier）的映射关系，是处理中日韩（CJK）等大字符集语言的关键组件。

## 目录结构

```
core/fpdfapi/cmaps/
├── BUILD.gn              # 构建配置
├── fpdf_cmaps.cpp        # CMap 加载接口
├── fpdf_cmaps.h          # CMap 头文件
│
├── CNS1/                 # 繁体中文 (CNS 字符集)
│   ├── B5pc-H_0.cpp      # Big5 到 CID 映射（横排）
│   ├── B5pc-V_0.cpp      # Big5 到 CID 映射（竖排）
│   ├── HKscs-B5-H_5.cpp  # 香港扩展映射（横排）
│   ├── HKscs-B5-V_5.cpp  # 香港扩展映射（竖排）
│   ├── ETen-B5-H_0.cpp   # ETen 扩展（横排）
│   ├── ETen-B5-V_0.cpp   # ETen 扩展（竖排）
│   ├── UniCNS-UCS2-H_3.cpp  # Unicode 映射（横排）
│   ├── UniCNS-UCS2-V_3.cpp  # Unicode 映射（竖排）
│   ├── cmaps_cns1.cpp    # CNS1 CMap 注册
│   └── cmaps_cns1.h      # CNS1 CMap 头文件
│
├── GB1/                  # 简体中文 (GB 字符集)
│   ├── GB-EUC-H_0.cpp    # GB2312 映射（横排）
│   ├── GB-EUC-V_0.cpp    # GB2312 映射（竖排）
│   ├── GBpc-EUC-H_0.cpp  # Mac 平台映射
│   ├── GBpc-EUC-V_0.cpp  # Mac 平台映射（竖排）
│   ├── GBK-EUC-H_2.cpp   # GBK 映射（横排）
│   ├── GBK-EUC-V_2.cpp   # GBK 映射（竖排）
│   ├── GBKp-EUC-H_2.cpp  # GBK 比例宽度（横排）
│   ├── GBKp-EUC-V_2.cpp  # GBK 比例宽度（竖排）
│   ├── GBK2K-H_5.cpp     # GBK2000 映射（横排）
│   ├── GBK2K-V_5.cpp     # GBK2000 映射（竖排）
│   ├── UniGB-UCS2-H_4.cpp   # Unicode 映射（横排）
│   ├── UniGB-UCS2-V_4.cpp   # Unicode 映射（竖排）
│   ├── cmaps_gb1.cpp     # GB1 CMap 注册
│   └── cmaps_gb1.h       # GB1 CMap 头文件
│
├── Japan1/               # 日文 (Japan1 字符集)
│   ├── 83pv-RKSJ-H_1.cpp # Mac 日文（横排）
│   ├── 90ms-RKSJ-H_2.cpp # Windows 日文（横排）
│   ├── 90ms-RKSJ-V_2.cpp # Windows 日文（竖排）
│   ├── 90msp-RKSJ-H_2.cpp # 比例宽度（横排）
│   ├── 90msp-RKSJ-V_2.cpp # 比例宽度（竖排）
│   ├── 90pv-RKSJ-H_1.cpp # Mac 比例宽度（横排）
│   ├── Add-RKSJ-H_1.cpp  # 扩展字符（横排）
│   ├── Add-RKSJ-V_1.cpp  # 扩展字符（竖排）
│   ├── EUC-H_1.cpp       # EUC-JP（横排）
│   ├── EUC-V_1.cpp       # EUC-JP（竖排）
│   ├── Ext-RKSJ-H_2.cpp  # 扩展 RKSJ（横排）
│   ├── Ext-RKSJ-V_2.cpp  # 扩展 RKSJ（竖排）
│   ├── UniJIS-UCS2-H_4.cpp  # Unicode 映射（横排）
│   ├── UniJIS-UCS2-V_4.cpp  # Unicode 映射（竖排）
│   ├── UniJIS-UCS2-HW-H_4.cpp  # 半角（横排）
│   ├── UniJIS-UCS2-HW-V_4.cpp  # 半角（竖排）
│   ├── cmaps_japan1.cpp  # Japan1 CMap 注册
│   └── cmaps_japan1.h    # Japan1 CMap 头文件
│
└── Korea1/               # 韩文 (Korea1 字符集)
    ├── KSC-EUC-H_0.cpp   # EUC-KR（横排）
    ├── KSC-EUC-V_0.cpp   # EUC-KR（竖排）
    ├── KSCms-UHC-H_1.cpp # Windows 韩文（横排）
    ├── KSCms-UHC-V_1.cpp # Windows 韩文（竖排）
    ├── KSCms-UHC-HW-H_1.cpp # 半角（横排）
    ├── KSCms-UHC-HW-V_1.cpp # 半角（竖排）
    ├── KSCpc-EUC-H_0.cpp # Mac 韩文（横排）
    ├── UniKS-UCS2-H_1.cpp # Unicode 映射（横排）
    ├── UniKS-UCS2-V_1.cpp # Unicode 映射（竖排）
    ├── cmaps_korea1.cpp  # Korea1 CMap 注册
    └── cmaps_korea1.h    # Korea1 CMap 头文件
```

## 核心概念

### 1. CMap 概述

CMap 是字符编码到 CID 的映射表：
- **输入**: 一个或多个字节的字符编码
- **输出**: CID（字符标识符）

### 2. 字符集 (Character Collection)

| 字符集 | 描述 | 区域 |
|--------|------|------|
| Adobe-CNS1 | CNS 11643 | 繁体中文（台湾/香港） |
| Adobe-GB1 | GB 2312 / GBK | 简体中文 |
| Adobe-Japan1 | JIS | 日文 |
| Adobe-Korea1 | KS | 韩文 |

### 3. CMap 命名约定

CMap 名称格式：`<编码>-<变体>-<方向>_<版本>`

方向后缀：
- `H`: 横排书写
- `V`: 竖排书写

示例：
- `90ms-RKSJ-H`: Windows 日文 Shift-JIS，横排
- `UniGB-UCS2-V`: Unicode 到 GB1，竖排

## 数据结构

### 映射表格式

CMap 数据以压缩的二进制格式存储：

| 字段 | 描述 |
|------|------|
| 范围起始 | 起始编码值 |
| 范围数量 | 连续编码的数量 |
| CID 起始 | 对应的起始 CID |

### 代码空间

定义有效的字符编码范围：
- 单字节范围：0x00-0xFF
- 双字节范围：0x8140-0xFEFE 等

## 核心函数

### fpdf_cmaps.h/cpp

| 函数 | 功能 |
|------|------|
| `CPDF_ModuleMgr::LoadEmbeddedGB1CMaps` | 加载 GB1 CMap |
| `CPDF_ModuleMgr::LoadEmbeddedCNS1CMaps` | 加载 CNS1 CMap |
| `CPDF_ModuleMgr::LoadEmbeddedJapan1CMaps` | 加载 Japan1 CMap |
| `CPDF_ModuleMgr::LoadEmbeddedKorea1CMaps` | 加载 Korea1 CMap |

## 各字符集详解

### GB1 (简体中文)

| CMap | 编码 | 说明 |
|------|------|------|
| GB-EUC-H/V | EUC-CN | GB2312 标准编码 |
| GBpc-EUC-H/V | Mac | Mac 平台 |
| GBK-EUC-H/V | GBK | Windows 扩展 |
| GBKp-EUC-H/V | GBK | 比例宽度 |
| GBK2K-H/V | GB18030 | 完整 Unicode |
| UniGB-UCS2-H/V | Unicode | UCS-2 编码 |

GB1 版本：
- GB1-0: 基础 GB2312
- GB1-2: GBK 扩展
- GB1-4: Unicode 支持
- GB1-5: GB18030

### CNS1 (繁体中文)

| CMap | 编码 | 说明 |
|------|------|------|
| B5pc-H/V | Big5 | Mac Big5 |
| HKscs-B5-H/V | HKSCS | 香港扩展 |
| ETen-B5-H/V | ETen | ETen 扩展 |
| UniCNS-UCS2-H/V | Unicode | UCS-2 编码 |

### Japan1 (日文)

| CMap | 编码 | 说明 |
|------|------|------|
| 83pv-RKSJ-H | Shift-JIS | Mac 日文 |
| 90ms-RKSJ-H/V | Shift-JIS | Windows 日文 |
| 90msp-RKSJ-H/V | Shift-JIS | 比例宽度 |
| EUC-H/V | EUC-JP | Unix 日文 |
| UniJIS-UCS2-H/V | Unicode | UCS-2 编码 |
| UniJIS-UCS2-HW-H/V | Unicode | 半角 |

Japan1 版本：
- Japan1-0: 基础 JIS
- Japan1-1: JIS 扩展
- Japan1-2: Windows 扩展
- Japan1-4: Unicode 支持
- Japan1-6: 最新扩展

### Korea1 (韩文)

| CMap | 编码 | 说明 |
|------|------|------|
| KSC-EUC-H/V | EUC-KR | 标准 KS |
| KSCms-UHC-H/V | UHC | Windows 统一韩文 |
| KSCms-UHC-HW-H/V | UHC | 半角 |
| KSCpc-EUC-H | EUC-KR | Mac 韩文 |
| UniKS-UCS2-H/V | Unicode | UCS-2 编码 |

## CMap 查找流程

```
输入字节序列
    │
    ├── 确定代码空间
    │   ├── 单字节?
    │   └── 多字节?
    │
    ├── 在 CMap 中查找
    │   ├── 二分搜索范围
    │   └── 计算 CID
    │
    └── 返回 CID
```

## 竖排书写支持

竖排 CMap 特点：
- 使用竖排字形变体
- 标点符号位置调整
- 字符间距不同

竖排 CID 替换：
- 部分字符使用专门的竖排字形
- 通过 CMap 映射实现

## 性能优化

- 预编译为 C++ 数组
- 紧凑的二进制格式
- 按需加载
- 范围合并减少条目数

## 构建时生成

CMap 数据文件在构建时从 Adobe 标准 CMap 文件转换：
1. 解析 CMap 文本文件
2. 提取映射范围
3. 生成 C++ 数组
4. 编译进二进制

## 依赖关系

```
cmaps/
    └── fxcrt/   (基础设施)

被依赖:
    font/ → cmaps/
```

## 使用场景

1. **PDF 渲染**: 将字符编码转换为 CID 以获取字形
2. **文本提取**: 将 CID 转换为 Unicode
3. **搜索匹配**: 规范化字符编码进行比较
4. **字体替代**: 确定字符集兼容性
