# core/fdrm 源码分析

## 模块概述

`fdrm` (FoxIt Digital Rights Management) 模块负责处理 PDF 文档的加密和解密功能。该模块实现了 PDF 规范中定义的多种加密算法，用于保护文档内容和控制访问权限。

## 目录结构

```
core/fdrm/
├── BUILD.gn              # 构建配置文件
├── fx_crypt.cpp          # 核心加密算法实现（RC4, MD5）
├── fx_crypt.h            # 加密算法头文件
├── fx_crypt_aes.cpp      # AES 加密算法实现
├── fx_crypt_aes.h        # AES 加密头文件
├── fx_crypt_sha.cpp      # SHA 哈希算法实现
├── fx_crypt_sha.h        # SHA 哈希头文件
└── fx_crypt_unittest.cpp # 单元测试
```

## 核心组件分析

### 1. RC4 加密 (fx_crypt.cpp)

#### CRYPT_rc4_context 结构体

**位置**: fx_crypt.h 第 17-23 行

该结构体维护 RC4 流密码的内部状态：
- `x`, `y`: 两个状态索引变量，用于遍历置换表
- `m`: 长度为 256 的置换数组，存储 0-255 的排列

#### CRYPT_ArcFourSetup 函数

**位置**: fx_crypt.cpp 第 131-145 行

**功能**: 使用密钥初始化 RC4 上下文

**实现逻辑**:
1. 将置换数组初始化为恒等排列 (0, 1, 2, ..., 255)
2. 使用密钥对置换数组进行 Key Scheduling Algorithm (KSA) 处理
3. 通过交换操作打乱置换数组的顺序

#### CRYPT_ArcFourCrypt 函数

**位置**: fx_crypt.cpp 第 147-156 行

**功能**: 对数据进行 RC4 加密/解密

**实现逻辑**:
1. 遍历每个数据字节
2. 更新状态索引 x 和 y
3. 交换置换数组中的元素
4. 使用置换数组生成伪随机字节与数据进行异或

#### CRYPT_ArcFourCryptBlock 函数

**位置**: fx_crypt.cpp 第 158-163 行

**功能**: 一次性加密整个数据块的便捷函数

**实现逻辑**: 创建临时上下文，调用 Setup 和 Crypt 函数

### 2. MD5 哈希 (fx_crypt.cpp)

#### CRYPT_md5_context 结构体

**位置**: fx_crypt.h 第 25-29 行

该结构体维护 MD5 计算的中间状态：
- `total`: 已处理数据的总比特数
- `state`: 4 个 32 位状态变量 (A, B, C, D)
- `buffer`: 64 字节的输入缓冲区

#### CRYPT_MD5Start 函数

**位置**: fx_crypt.cpp 第 165-174 行

**功能**: 初始化 MD5 上下文

**实现逻辑**: 
- 将计数器清零
- 使用 MD5 规范定义的魔数初始化状态变量

#### CRYPT_MD5Update 函数

**位置**: fx_crypt.cpp 第 176-203 行

**功能**: 处理输入数据块

**实现逻辑**:
1. 更新已处理数据的计数
2. 如果缓冲区有剩余数据，先填满并处理
3. 处理所有完整的 64 字节块
4. 将剩余数据存入缓冲区

#### md5_process 函数

**位置**: fx_crypt.cpp 第 22-127 行

**功能**: 处理单个 64 字节的数据块

**实现逻辑**:
- 将输入数据转换为 16 个 32 位字
- 执行 4 轮共 64 步的转换操作
- 每轮使用不同的非线性函数 (F, G, H, I)
- 更新状态变量

#### CRYPT_MD5Finish 函数

**位置**: fx_crypt.cpp 第 205-219 行

**功能**: 完成 MD5 计算并输出摘要

**实现逻辑**:
1. 添加填充数据（以 0x80 开始，后跟若干 0x00）
2. 添加 64 位消息长度
3. 输出最终的 128 位摘要

### 3. AES 加密 (fx_crypt_aes.cpp)

#### CRYPT_aes_context 结构体

**位置**: fx_crypt_aes.h

维护 AES 加密状态，包括：
- 轮密钥数组
- 加密轮数
- 初始化向量 (IV)

#### 主要函数

| 函数名 | 功能 |
|--------|------|
| `CRYPT_AESSetKey` | 设置加密密钥并生成轮密钥 |
| `CRYPT_AESSetIV` | 设置初始化向量 |
| `CRYPT_AESEncrypt` | 执行 AES 加密 |
| `CRYPT_AESDecrypt` | 执行 AES 解密 |

### 4. SHA 哈希 (fx_crypt_sha.cpp)

支持多种 SHA 变体：
- SHA-1 (160 位摘要)
- SHA-256 (256 位摘要)
- SHA-384 (384 位摘要)
- SHA-512 (512 位摘要)

#### 主要函数

| 函数名 | 功能 |
|--------|------|
| `CRYPT_SHA1Start` | 初始化 SHA-1 上下文 |
| `CRYPT_SHA1Update` | 处理输入数据 |
| `CRYPT_SHA1Finish` | 输出 SHA-1 摘要 |
| `CRYPT_SHA256Generate` | 一次性计算 SHA-256 |
| `CRYPT_SHA512Generate` | 一次性计算 SHA-512 |

## PDF 加密中的应用

### 密码验证流程

1. 使用用户输入的密码和文档加密参数计算密钥
2. 使用 MD5 或 SHA-256 进行密钥派生
3. 验证计算出的值是否与文档中存储的值匹配

### 内容解密流程

1. 从文档中获取加密字典
2. 根据加密版本选择算法（RC4 或 AES）
3. 使用派生密钥解密流对象和字符串

## 安全性考虑

- RC4 已被认为不够安全，PDF 2.0 推荐使用 AES-256
- 密钥长度支持 40 位到 256 位
- AES 使用 CBC 模式，需要正确处理 IV

## 依赖关系

```
fdrm/
  └── fxcrt/  (基础运行时库)
        ├── span.h      (安全的数组视图)
        ├── byteorder.h (字节序转换)
        └── stl_util.h  (STL 工具函数)
```

## 测试覆盖

`fx_crypt_unittest.cpp` 包含以下测试：
- RC4 加密/解密正确性测试
- MD5 哈希测试向量验证
- AES 加密/解密测试
- SHA 系列哈希测试

## 使用示例

### MD5 哈希计算

```cpp
// 1. 创建上下文
auto ctx = CRYPT_MD5Start();

// 2. 添加数据
CRYPT_MD5Update(&ctx, data_span);

// 3. 获取结果
uint8_t digest[16];
CRYPT_MD5Finish(&ctx, digest);
```

### RC4 加密

```cpp
// 一次性加密
CRYPT_ArcFourCryptBlock(data_span, key_span);

// 或分步处理
CRYPT_rc4_context ctx;
CRYPT_ArcFourSetup(&ctx, key_span);
CRYPT_ArcFourCrypt(&ctx, data_span);
```
