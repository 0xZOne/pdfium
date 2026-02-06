# fxbarcode 源码分析

## 模块概述

`fxbarcode` 模块提供条形码生成功能，支持多种一维和二维条形码格式。该模块主要用于 XFA 表单中的条形码字段渲染。

## 目录结构

```
fxbarcode/
├── BUILD.gn                          # 构建配置
├── DEPS                              # 依赖声明
│
├── 核心类
│   ├── cfx_barcode.cpp/h             # 条形码主类
│   ├── BC_Library.cpp/h              # 条形码库初始化
│   ├── BC_Writer.cpp/h               # 条形码写入器基类
│   └── BC_TwoDimWriter.cpp/h         # 二维码写入器基类
│
├── 一维码封装
│   ├── cbc_codebase.cpp/h            # 一维码基类
│   ├── cbc_onecode.cpp/h             # 单编码条码基类
│   ├── cbc_eancode.cpp/h             # EAN 条码基类
│   ├── cbc_code39.cpp/h              # Code 39
│   ├── cbc_code128.cpp/h             # Code 128
│   ├── cbc_codabar.cpp/h             # Codabar
│   ├── cbc_ean8.cpp/h                # EAN-8
│   ├── cbc_ean13.cpp/h               # EAN-13
│   └── cbc_upca.cpp/h                # UPC-A
│
├── 二维码封装
│   ├── cbc_qrcode.cpp/h              # QR Code
│   ├── cbc_pdf417i.cpp/h             # PDF417
│   └── cbc_datamatrix.cpp/h          # Data Matrix
│
├── 子目录
│   ├── oned/                         # 一维码实现
│   ├── qrcode/                       # QR Code 实现
│   ├── pdf417/                       # PDF417 实现
│   ├── datamatrix/                   # Data Matrix 实现
│   └── common/                       # 公共工具
│
└── 测试文件
    ├── cfx_barcode_unittest.cpp
    └── cbc_pdf417i_unittest.cpp
```

## 核心类分析

### 1. CFX_Barcode 类

**位置**: cfx_barcode.h/cpp

条形码的主要接口类。

| 方法 | 功能 |
|------|------|
| `Create` | 创建条形码实例 |
| `SetType` | 设置条形码类型 |
| `SetModuleWidth` | 设置模块宽度 |
| `SetModuleHeight` | 设置模块高度 |
| `SetHeight` | 设置高度 |
| `SetWidth` | 设置宽度 |
| `SetPrintChecksum` | 设置打印校验和 |
| `SetDataLength` | 设置数据长度 |
| `SetCalChecksum` | 设置计算校验和 |
| `SetFont` | 设置字体 |
| `SetFontSize` | 设置字体大小 |
| `SetFontColor` | 设置字体颜色 |
| `SetTextLocation` | 设置文本位置 |
| `SetWideNarrowRatio` | 设置宽窄比 |
| `SetStartChar` | 设置起始字符 |
| `SetEndChar` | 设置结束字符 |
| `SetErrorCorrectionLevel` | 设置纠错级别 |
| `Encode` | 编码数据 |
| `RenderDevice` | 渲染到设备 |

### 2. BC_Library

**位置**: BC_Library.h/cpp

条形码库的初始化和清理。

| 函数 | 功能 |
|------|------|
| `BC_Library_Init` | 初始化库 |
| `BC_Library_Destroy` | 销毁库 |

### 3. CBC_CodeBase 类

**位置**: cbc_codebase.h/cpp

所有条形码类型的基类。

| 方法 | 功能 |
|------|------|
| `Encode` | 编码数据 |
| `RenderDevice` | 渲染到设备 |
| `SetTextLocation` | 设置文本位置 |

## 一维码实现 (oned/)

### 支持的格式

| 格式 | 类 | 描述 |
|------|-----|------|
| Code 39 | CBC_Code39 | 字母数字条码 |
| Code 128 | CBC_Code128 | 高密度字母数字 |
| Codabar | CBC_Codabar | 数字条码 |
| EAN-8 | CBC_EAN8 | 8位商品条码 |
| EAN-13 | CBC_EAN13 | 13位商品条码 |
| UPC-A | CBC_UPCA | 美国商品条码 |

### Code 39

**位置**: oned/BC_OnedCode39Writer.cpp

特点：
- 支持 43 个字符（0-9, A-Z, -, ., $, /, +, %, 空格）
- 可选校验位
- 自定界

### Code 128

**位置**: oned/BC_OnedCode128Writer.cpp

特点：
- 支持所有 128 个 ASCII 字符
- 三种字符集（A, B, C）
- 自动切换模式
- 强制校验位

### EAN/UPC

**位置**: oned/BC_OnedEAN*Writer.cpp

特点：
- 标准零售条码
- 固定长度
- 内置校验位
- 支持附加码

## 二维码实现

### QR Code (qrcode/)

**位置**: qrcode/

| 类 | 功能 |
|----|------|
| `CBC_QRCodeWriter` | QR 码生成器 |
| `CBC_QREncoder` | QR 编码器 |
| `CBC_QRBitMatrixParser` | 位矩阵解析 |
| `CBC_QRDataMask` | 数据掩码 |

纠错级别：
- L (7%)
- M (15%)
- Q (25%)
- H (30%)

版本支持：1-40

### PDF417 (pdf417/)

**位置**: pdf417/

| 类 | 功能 |
|----|------|
| `CBC_PDF417Writer` | PDF417 生成器 |
| `CBC_PDF417` | PDF417 核心 |
| `CBC_PDF417HighLevelEncoder` | 高级编码器 |
| `CBC_PDF417BarcodeMatrix` | 条码矩阵 |

特点：
- 可存储大量数据
- 可配置行列数
- 可配置纠错级别
- 支持宏 PDF417

### Data Matrix (datamatrix/)

**位置**: datamatrix/

| 类 | 功能 |
|----|------|
| `CBC_DataMatrixWriter` | Data Matrix 生成器 |
| `CBC_HighLevelEncoder` | 高级编码器 |
| `CBC_SymbolInfo` | 符号信息 |
| `CBC_DefaultPlacement` | 默认放置 |

特点：
- 方形或矩形
- 多种尺寸选择
- Reed-Solomon 纠错

## 公共工具 (common/)

| 类 | 功能 |
|----|------|
| `CBC_CommonBitMatrix` | 位矩阵 |
| `CBC_CommonBitArray` | 位数组 |
| `CBC_CommonByteMatrix` | 字节矩阵 |

## 编码流程

### 一维码编码

```
输入数据
    │
    ├── 验证字符合法性
    │
    ├── 计算校验位（如需要）
    │
    ├── 转换为条空模式
    │   └── 每个字符 → 条/空序列
    │
    ├── 添加起始/结束符
    │
    └── 生成位数组
```

### 二维码编码

```
输入数据
    │
    ├── 选择编码模式
    │   ├── 数字模式
    │   ├── 字母数字模式
    │   ├── 字节模式
    │   └── 汉字模式
    │
    ├── 数据编码
    │
    ├── 生成纠错码
    │
    ├── 结构填充
    │   ├── 添加功能图案
    │   ├── 添加数据和纠错
    │   └── 应用掩码
    │
    └── 生成位矩阵
```

## 渲染流程

```
RenderDevice()
    │
    ├── 计算缩放比例
    │
    ├── 遍历位矩阵
    │
    ├── 绘制模块
    │   ├── 一维码：绘制条/空
    │   └── 二维码：绘制方块
    │
    └── 绘制文本标签（如需要）
```

## 依赖关系

```
fxbarcode/
    ├── core/fxge/       (图形渲染)
    └── core/fxcrt/      (基础设施)

被依赖:
    xfa/fxfa/ → fxbarcode/
```

## XFA 集成

XFA 表单中的条形码字段通过此模块渲染：

```
CXFA_FFBarcode
    │
    ├── 获取条形码类型和数据
    │
    ├── 创建 CFX_Barcode
    │
    ├── 设置属性
    │
    ├── 编码数据
    │
    └── 渲染到页面
```

## 支持的条形码类型枚举

| 类型 | 值 | 格式 |
|------|-----|------|
| `BC_CODE39` | 0 | Code 39 |
| `BC_CODABAR` | 1 | Codabar |
| `BC_CODE128` | 2 | Code 128 |
| `BC_CODE128_B` | 3 | Code 128 B |
| `BC_CODE128_C` | 4 | Code 128 C |
| `BC_EAN8` | 5 | EAN-8 |
| `BC_UPCA` | 6 | UPC-A |
| `BC_EAN13` | 7 | EAN-13 |
| `BC_QR_CODE` | 8 | QR Code |
| `BC_PDF417` | 9 | PDF417 |
| `BC_DATAMATRIX` | 10 | Data Matrix |

## 使用示例

```cpp
// 创建条形码
auto barcode = std::make_unique<CFX_Barcode>();

// 设置类型
barcode->SetType(BC_QR_CODE);

// 设置属性
barcode->SetErrorCorrectionLevel(2);  // M 级别
barcode->SetModuleWidth(3);
barcode->SetModuleHeight(3);

// 编码数据
bool success = barcode->Encode(L"Hello World");

// 渲染
if (success) {
    barcode->RenderDevice(pDevice, matrix);
}
```
