# xfa 源码分析

## 模块概述

`xfa` 模块提供 XFA（XML Forms Architecture）表单支持。XFA 是 Adobe 开发的动态表单标准，允许创建复杂的交互式表单，支持动态布局、数据绑定和脚本编程。

## 目录结构

```
xfa/
├── BUILD.gn                          # 构建配置
├── DEPS                              # 依赖声明
├── README.md                         # 模块说明
│
├── fxfa/                             # XFA 核心
│   ├── parser/                       # XFA XML 解析
│   ├── layout/                       # 布局引擎
│   └── formcalc/                     # FormCalc 脚本语言
│
├── fgas/                             # 图形和脚本支持
│   ├── crt/                          # 运行时
│   ├── font/                         # 字体
│   ├── graphics/                     # 图形
│   └── layout/                       # 文本布局
│
├── fwl/                              # 窗口部件库
│   └── theme/                        # 主题
│
└── fde/                              # 设备扩展
```

## 架构概述

### XFA 分层架构

```
┌─────────────────────────────────────┐
│          fpdfsdk/fpdfxfa/           │  SDK 集成层
├─────────────────────────────────────┤
│              xfa/fxfa/              │  XFA 核心层
│  ┌───────────┬───────────────────┐  │
│  │  parser/  │      layout/      │  │
│  └───────────┴───────────────────┘  │
├─────────────────────────────────────┤
│              xfa/fwl/               │  窗口部件层
├─────────────────────────────────────┤
│    xfa/fgas/    │    xfa/fde/       │  基础服务层
└─────────────────────────────────────┘
```

## 子模块概述

### fxfa - XFA 核心

主要职责：
- XFA 文档管理
- 表单控件渲染
- 事件处理
- 脚本执行

核心类：
- `CXFA_FFDoc`: XFA 文档
- `CXFA_FFDocView`: 文档视图
- `CXFA_FFWidget`: XFA 控件
- `CXFA_FFApp`: XFA 应用

### fgas - 图形和脚本

主要职责：
- 字体管理
- 文本布局
- 图形绘制
- 字符断行

核心类：
- `CFGAS_FontMgr`: 字体管理器
- `CFGAS_GEFont`: 字体对象
- `CFGAS_TxtBreak`: 文本断行

### fwl - 窗口部件库

主要职责：
- 提供 XFA 控件的底层实现
- 事件分发
- 主题渲染

核心类：
- `CFWL_Widget`: 控件基类
- `CFWL_App`: 应用管理
- `CFWL_NoteDriver`: 事件驱动

### fde - 设备扩展

主要职责：
- 文本编辑
- 文本输出
- 分词

核心类：
- `CFDE_TextEditEngine`: 文本编辑引擎
- `CFDE_TextOut`: 文本输出

## XFA 文档结构

### XML 包

XFA 文档由多个 XML 包组成：

| 包名 | 功能 |
|------|------|
| `template` | 表单模板定义 |
| `form` | 运行时表单数据 |
| `data` | 用户数据 |
| `config` | 配置信息 |
| `connectionSet` | 数据连接 |
| `sourceSet` | 数据源 |
| `xdc` | 设备配置 |
| `xdp` | 顶层容器 |

### 节点类型

| 类型 | 描述 |
|------|------|
| `field` | 表单字段 |
| `draw` | 静态绘制 |
| `subform` | 子表单 |
| `area` | 区域容器 |
| `exclGroup` | 互斥组 |
| `pageSet` | 页面集 |
| `pageArea` | 页面区域 |
| `contentArea` | 内容区域 |

## XFA 控件类型

| 控件 | 类 | 描述 |
|------|-----|------|
| 文本框 | `CXFA_FFTextField` | 单行/多行文本 |
| 下拉框 | `CXFA_FFComboBox` | 下拉选择 |
| 列表框 | `CXFA_FFListBox` | 列表选择 |
| 复选框 | `CXFA_FFCheckButton` | 复选 |
| 按钮 | `CXFA_FFPushButton` | 按钮 |
| 日期 | `CXFA_FFDateTimeEdit` | 日期选择 |
| 数字 | `CXFA_FFNumericEdit` | 数字输入 |
| 密码 | `CXFA_FFPasswordEdit` | 密码输入 |
| 图像 | `CXFA_FFImage` | 图像显示 |
| 签名 | `CXFA_FFSignature` | 数字签名 |
| 条形码 | `CXFA_FFBarcode` | 条形码 |
| 文本 | `CXFA_FFText` | 静态文本 |
| 直线 | `CXFA_FFLine` | 直线 |
| 矩形 | `CXFA_FFRectangle` | 矩形 |
| 弧形 | `CXFA_FFArc` | 弧形 |

## 脚本支持

### JavaScript

标准 PDF JavaScript，通过 fxjs 模块执行。

### FormCalc

Adobe 专有脚本语言，语法类似 Lotus 1-2-3：

特点：
- 简化的表达式语法
- 内置日期/时间函数
- 内置金融函数
- 内置字符串函数

FormCalc 由 `formcalc/` 子目录实现。

## 布局系统

### 布局流程

```
CXFA_FFDoc
    │
    ├── 解析 XFA 模板
    │
    ├── 创建节点树
    │
    ├── 布局计算
    │   ├── 遍历节点
    │   ├── 计算尺寸
    │   ├── 分配位置
    │   └── 处理分页
    │
    └── 生成页面
```

### 布局类型

| 类型 | 描述 |
|------|------|
| `lr-tb` | 从左到右，从上到下 |
| `rl-tb` | 从右到左，从上到下 |
| `tb` | 从上到下 |
| `rl-row` | 从右到左行 |
| `row` | 行布局 |
| `table` | 表格布局 |
| `positioned` | 绝对定位 |

## 事件系统

### 支持的事件

| 事件 | 触发时机 |
|------|----------|
| `initialize` | 初始化 |
| `calculate` | 计算 |
| `validate` | 验证 |
| `enter` | 进入字段 |
| `exit` | 离开字段 |
| `change` | 值改变 |
| `click` | 点击 |
| `mouseEnter` | 鼠标进入 |
| `mouseExit` | 鼠标离开 |
| `docReady` | 文档就绪 |
| `docClose` | 文档关闭 |
| `preSubmit` | 提交前 |
| `postSubmit` | 提交后 |

## 数据绑定

XFA 支持将表单字段与 XML 数据绑定：

```
数据包 (data)
    │
    └── 数据节点
        ↓
        绑定
        ↑
    表单节点
    │
    └── 模板 (template)
```

## 依赖关系

```
xfa/
    ├── fpdfsdk/fpdfxfa/  (SDK 集成)
    ├── core/fpdfapi/     (PDF 核心)
    ├── core/fxge/        (图形引擎)
    ├── core/fxcrt/       (基础设施)
    ├── fxjs/             (JavaScript)
    └── fxbarcode/        (条形码)
```

## 条件编译

XFA 功能通过宏控制：
- `PDF_ENABLE_XFA`: 启用 XFA 支持

## 注意事项

- XFA 是 Adobe 专有技术
- PDFium 的 XFA 支持是实验性的
- Chrome 浏览器已禁用 XFA
- XFA 文档解析较复杂，可能影响性能
- 并非所有 XFA 功能都被支持
