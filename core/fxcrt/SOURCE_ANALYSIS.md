# core/fxcrt 源码分析

## 模块概述

`fxcrt` (FoxIt Core RunTime) 是 PDFium 的核心运行时库，提供跨平台的基础设施和实用工具。它是所有其他模块的基础依赖，实现了字符串、智能指针、容器、流操作、内存管理等核心功能。

## 目录结构

```
core/fxcrt/
├── BUILD.gn                    # 构建配置
├── DEPS                        # 依赖声明
│
├── 字符串类
│   ├── bytestring.cpp/h        # 字节字符串
│   ├── widestring.cpp/h        # 宽字符串
│   ├── string_view_template.h  # 字符串视图模板
│   ├── string_template.cpp/h   # 字符串模板基类
│   ├── string_data_template.cpp/h  # 字符串数据存储
│   ├── string_pool_template.h  # 字符串池
│   ├── binary_buffer.cpp/h     # 二进制缓冲区
│   └── widetext_buffer.cpp/h   # 宽文本缓冲区
│
├── 智能指针
│   ├── retain_ptr.h            # 引用计数指针
│   ├── unowned_ptr.h           # 非拥有指针
│   ├── observed_ptr.cpp/h      # 观察者指针
│   ├── weak_ptr.h              # 弱引用指针
│   └── maybe_owned.h           # 可选拥有指针
│
├── 容器和工具
│   ├── data_vector.h           # 数据向量
│   ├── fixed_size_data_vector.h  # 固定大小向量
│   ├── span.h                  # 安全数组视图
│   ├── span_util.h             # span 工具函数
│   ├── mask.h                  # 位掩码类型
│   ├── stl_util.h              # STL 工具函数
│   └── tree_node.h             # 树节点模板
│
├── 流和文件
│   ├── fx_stream.cpp/h         # 流接口
│   ├── fileaccess_iface.h      # 文件访问接口
│   ├── cfx_fileaccess_posix.cpp/h  # POSIX 文件访问
│   ├── cfx_fileaccess_windows.cpp/h  # Windows 文件访问
│   ├── cfx_memorystream.cpp/h  # 内存流
│   ├── cfx_seekablestreamproxy.cpp/h  # 可寻址流代理
│   ├── cfx_read_only_*.cpp/h   # 只读流实现
│   └── fx_folder_*.cpp         # 文件夹操作
│
├── 坐标和几何
│   ├── fx_coordinates.cpp/h    # 坐标和矩阵类型
│   └── fx_2d_size.h            # 2D 尺寸类型
│
├── 内存管理
│   ├── fx_memory.cpp/h         # 内存分配
│   ├── fx_memory_malloc.cpp    # malloc 实现
│   ├── fx_memory_pa.cpp        # PartitionAlloc 实现
│   └── fx_memory_wrappers.h    # 内存包装器
│
├── 编码和字符
│   ├── fx_codepage.cpp/h       # 代码页处理
│   ├── fx_unicode.cpp/h        # Unicode 处理
│   ├── fx_string.cpp/h         # 字符串转换
│   ├── fx_bidi.cpp/h           # 双向文本
│   ├── utf16.h                 # UTF-16 处理
│   └── code_point_view.cpp/h   # 码点视图
│
├── 时间和定时器
│   ├── cfx_datetime.cpp/h      # 日期时间
│   └── cfx_timer.cpp/h         # 定时器
│
├── 数值和位操作
│   ├── fx_number.cpp/h         # 数字解析
│   ├── cfx_bitstream.cpp/h     # 位流操作
│   ├── byteorder.h             # 字节序转换
│   └── fx_safe_types.h         # 安全类型
│
├── 工具类
│   ├── autorestorer.h          # 自动恢复器
│   ├── autonuller.h            # 自动置空器
│   ├── scoped_set_insertion.h  # 作用域集合插入
│   ├── fx_random.cpp/h         # 随机数生成
│   └── fx_extension.cpp/h      # 扩展函数
│
├── 系统相关
│   ├── fx_system.cpp/h         # 系统函数
│   ├── check.h                 # 断言检查
│   ├── notreached.h            # 不可达标记
│   └── compiler_specific.h     # 编译器特定
│
├── 子目录
│   ├── containers/             # 容器实现
│   ├── css/                    # CSS 解析
│   ├── debug/                  # 调试工具
│   ├── numerics/               # 数值运算
│   ├── win/                    # Windows 特定
│   └── xml/                    # XML 解析
```

## 核心组件分析

### 1. 字符串系统

#### ByteString 类

**位置**: bytestring.h/cpp

8位字节字符串，用于处理 ASCII 和二进制数据：

| 方法 | 功能 |
|------|------|
| `IsEmpty` | 检查是否为空 |
| `GetLength` | 获取长度 |
| `operator[]` | 字符访问 |
| `operator+` | 字符串连接 |
| `operator==` | 相等比较 |
| `c_str` | 获取 C 字符串 |
| `Left/Right/Mid` | 子串提取 |
| `Find` | 查找子串 |
| `Replace` | 替换内容 |
| `Trim` | 去除空白 |
| `Format` | 格式化 |

#### WideString 类

**位置**: widestring.h/cpp

UTF-16 宽字符串，用于 Unicode 文本：

与 ByteString 类似的接口，额外支持：
- Unicode 规范化
- 大小写转换
- UTF-8/UTF-16 转换

#### 字符串视图

`ByteStringView` 和 `WideStringView` 提供不拥有数据的字符串视图，避免不必要的拷贝。

### 2. 智能指针系统

#### RetainPtr<T>

**位置**: retain_ptr.h

引用计数智能指针，用于管理 Retainable 对象：

| 特性 | 描述 |
|------|------|
| 引用计数 | 自动管理对象生命周期 |
| 复制语义 | 增加引用计数 |
| 移动语义 | 转移所有权 |
| 空安全 | 支持空指针 |

#### UnownedPtr<T>

**位置**: unowned_ptr.h

非拥有指针，用于调试时检测悬空指针：

| 特性 | 描述 |
|------|------|
| 零开销 | Release 版本无额外开销 |
| 调试检查 | Debug 版本检测悬空访问 |
| 替代裸指针 | 明确表示非拥有语义 |

#### ObservedPtr<T>

**位置**: observed_ptr.h/cpp

观察者指针，当被观察对象销毁时自动置空：

| 特性 | 描述 |
|------|------|
| 自动失效 | 对象销毁时自动置空 |
| 安全访问 | 可检查对象是否有效 |
| 多观察者 | 支持多个观察者 |

#### WeakPtr<T>

**位置**: weak_ptr.h

弱引用指针，不阻止对象销毁。

### 3. 容器类

#### DataVector<T>

**位置**: data_vector.h

基于 `std::vector` 的数据容器，添加了安全特性。

#### FixedSizeDataVector<T, N>

**位置**: fixed_size_data_vector.h

固定容量的向量，避免动态分配。

#### pdfium::span<T>

**位置**: span.h

安全的数组视图，类似于 C++20 的 `std::span`：

| 特性 | 描述 |
|------|------|
| 边界检查 | 运行时边界检查 |
| 零开销 | 只存储指针和大小 |
| 子视图 | `first`, `last`, `subspan` |
| 兼容性 | 兼容各种容器 |

### 4. 坐标和几何

#### CFX_PointF / CFX_Point

**位置**: fx_coordinates.h

点坐标类，支持浮点和整数版本。

#### CFX_SizeF / CFX_Size

尺寸类，表示宽度和高度。

#### CFX_FloatRect / FX_RECT

矩形类，表示边界框：

| 方法 | 功能 |
|------|------|
| `Contains` | 点是否在矩形内 |
| `Intersect` | 计算交集 |
| `Union` | 计算并集 |
| `Normalize` | 规范化坐标 |
| `Translate` | 平移 |
| `Scale` | 缩放 |

#### CFX_Matrix

变换矩阵类，2D 仿射变换：

| 方法 | 功能 |
|------|------|
| `Transform` | 变换点 |
| `TransformRect` | 变换矩形 |
| `Concat` | 矩阵连接 |
| `GetInverse` | 求逆矩阵 |
| `Rotate` | 旋转 |
| `Scale` | 缩放 |
| `Translate` | 平移 |

### 5. 流和文件操作

#### IFX_SeekableReadStream

**位置**: fx_stream.h

可寻址读取流接口：

| 方法 | 功能 |
|------|------|
| `GetSize` | 获取大小 |
| `ReadBlockAtOffset` | 在偏移处读取 |

#### IFX_SeekableWriteStream

可寻址写入流接口：

| 方法 | 功能 |
|------|------|
| `WriteBlock` | 写入数据块 |

#### CFX_MemoryStream

内存流实现，在内存中读写数据。

### 6. 内存管理

#### 分配器选择

根据构建配置选择内存分配器：
- **PartitionAlloc**: Chrome 的安全分配器
- **malloc**: 标准分配器

#### 内存函数

| 函数 | 功能 |
|------|------|
| `FX_Alloc` | 分配并清零 |
| `FX_Realloc` | 重新分配 |
| `FX_Free` | 释放内存 |
| `FX_AllocOrDie` | 分配（失败则终止） |

### 7. 编码和字符处理

#### 代码页支持

| 代码页 | 描述 |
|--------|------|
| GB2312 | 简体中文 |
| Big5 | 繁体中文 |
| Shift-JIS | 日文 |
| EUC-KR | 韩文 |
| UTF-8 | Unicode |
| Latin-1 | 西欧 |

#### 双向文本 (BiDi)

处理从右到左的文本（如阿拉伯语、希伯来语）。

### 8. 时间和定时器

#### CFX_DateTime

**位置**: cfx_datetime.h/cpp

日期时间类，支持 PDF 日期格式解析。

#### CFX_Timer

**位置**: cfx_timer.h/cpp

定时器接口，用于 JavaScript 的 `setTimeout`/`setInterval`。

### 9. XML 解析 (xml/)

提供 XML 文档的解析和操作功能：
- XML 节点树
- 属性访问
- 文本内容
- XMP 元数据解析

### 10. CSS 解析 (css/)

解析和处理 CSS 样式：
- 选择器解析
- 属性值解析
- 样式计算

## 平台抽象

### Windows 特定 (win/)

- COM 接口支持
- Windows 字符串转换
- 文件系统访问

### POSIX 特定

- 文件操作
- 时间函数

## 安全特性

### 边界检查

- `span` 提供运行时边界检查
- 字符串类防止缓冲区溢出

### 安全数值运算 (numerics/)

- 溢出检测
- 安全的算术运算
- 范围检查

### 指针安全

- `UnownedPtr` 检测悬空指针
- `ObservedPtr` 自动失效
- `RetainPtr` 防止内存泄漏

## 依赖关系

`fxcrt` 是最底层的模块，只依赖：
- C++ 标准库
- 操作系统 API
- 少量第三方库（如 ICU）

所有其他 PDFium 模块都依赖 `fxcrt`。
