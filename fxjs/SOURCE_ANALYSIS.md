# fxjs 源码分析

## 模块概述

`fxjs` 模块提供 PDF JavaScript 支持，通过 V8 引擎实现 Adobe Acrobat JavaScript API 的子集。该模块使 PDFium 能够执行 PDF 文档中嵌入的 JavaScript 代码。

## 目录结构

```
fxjs/
├── BUILD.gn                          # 构建配置
├── DEPS                              # 依赖声明
├── README                            # 模块说明
│
├── V8 引擎封装
│   ├── cfx_v8.cpp/h                  # V8 基础封装
│   ├── cfx_v8_array_buffer_allocator.cpp/h  # 数组缓冲分配器
│   ├── cfxjs_engine.cpp/h            # JavaScript 引擎
│   └── fxv8.cpp/h                    # V8 工具函数
│
├── 运行时
│   ├── ijs_runtime.cpp/h             # 运行时接口
│   ├── cjs_runtime.cpp/h             # 运行时实现
│   └── cjs_runtimestub.cpp/h         # 运行时存根
│
├── 事件上下文
│   ├── ijs_event_context.h           # 事件上下文接口
│   ├── cjs_event_context.cpp/h       # 事件上下文实现
│   └── cjs_event_context_stub.cpp/h  # 事件上下文存根
│
├── JavaScript 对象
│   ├── cjs_object.cpp/h              # 对象基类
│   ├── cjs_document.cpp/h            # document 对象
│   ├── cjs_app.cpp/h                 # app 对象
│   ├── cjs_field.cpp/h               # field 对象
│   ├── cjs_event.cpp/h               # event 对象
│   ├── cjs_console.cpp/h             # console 对象
│   ├── cjs_util.cpp/h                # util 对象
│   ├── cjs_color.cpp/h               # color 对象
│   ├── cjs_global.cpp/h              # global 对象
│   ├── cjs_annot.cpp/h               # Annot 对象
│   ├── cjs_icon.cpp/h                # Icon 对象
│   └── cjs_timerobj.cpp/h            # 定时器对象
│
├── 常量和枚举
│   ├── cjs_border.cpp/h              # border 常量
│   ├── cjs_display.cpp/h             # display 常量
│   ├── cjs_font.cpp/h                # font 常量
│   ├── cjs_highlight.cpp/h           # highlight 常量
│   ├── cjs_position.cpp/h            # position 常量
│   ├── cjs_scalehow.cpp/h            # scaleHow 常量
│   ├── cjs_scalewhen.cpp/h           # scaleWhen 常量
│   ├── cjs_style.cpp/h               # style 常量
│   ├── cjs_zoomtype.cpp/h            # zoomType 常量
│   ├── cjs_globalarrays.cpp/h        # 全局数组
│   └── cjs_globalconsts.cpp/h        # 全局常量
│
├── 工具类
│   ├── cjs_publicmethods.cpp/h       # 公共方法
│   ├── cjs_result.cpp/h              # 结果封装
│   ├── cjs_delaydata.cpp/h           # 延迟数据
│   ├── cjs_keyvalue.cpp/h            # 键值对
│   ├── cfx_globaldata.cpp/h          # 全局数据
│   ├── fx_date_helpers.cpp/h         # 日期辅助函数
│   ├── js_define.cpp/h               # JS 定义宏
│   ├── js_resources.cpp/h            # JS 资源
│   └── global_timer.cpp/h            # 全局定时器
│
├── 子目录
│   ├── gc/                           # 垃圾回收支持
│   └── xfa/                          # XFA JavaScript 支持
│
└── 测试文件
    ├── *_unittest.cpp
    └── *_embeddertest.cpp
```

## 核心组件分析

### 1. V8 引擎封装

#### CFX_V8 类

**位置**: cfx_v8.h/cpp

V8 引擎的基础封装。

| 方法 | 功能 |
|------|------|
| `GetIsolate` | 获取 V8 隔离区 |
| `NewNull` | 创建 null 值 |
| `NewUndefined` | 创建 undefined 值 |
| `NewBoolean` | 创建布尔值 |
| `NewNumber` | 创建数字 |
| `NewString` | 创建字符串 |
| `NewObject` | 创建对象 |
| `NewArray` | 创建数组 |
| `ToBoolean` | 转换为布尔 |
| `ToInt` | 转换为整数 |
| `ToDouble` | 转换为浮点 |
| `ToWideString` | 转换为宽字符串 |

#### CFXJS_Engine 类

**位置**: cfxjs_engine.h/cpp

JavaScript 引擎实现。

| 方法 | 功能 |
|------|------|
| `InitializeEngine` | 初始化引擎 |
| `ReleaseEngine` | 释放引擎 |
| `DefineObj` | 定义对象类型 |
| `DefineProperty` | 定义属性 |
| `DefineMethod` | 定义方法 |
| `DefineConst` | 定义常量 |
| `GetGlobalObject` | 获取全局对象 |
| `NewObject` | 创建对象实例 |
| `Execute` | 执行脚本 |

### 2. 运行时系统

#### IJS_Runtime 接口

**位置**: ijs_runtime.h

运行时抽象接口。

| 方法 | 功能 |
|------|------|
| `NewEventContext` | 创建事件上下文 |
| `ReleaseEventContext` | 释放事件上下文 |
| `GetCurrentEventContext` | 获取当前上下文 |
| `SetFormFillEnvToDocument` | 设置表单环境 |
| `FieldEvent_Delay` | 延迟字段事件 |

#### CJS_Runtime 类

**位置**: cjs_runtime.h/cpp

运行时实现。

| 方法 | 功能 |
|------|------|
| `DefineJSObjects` | 定义 JS 对象 |
| `NewJSObject` | 创建 JS 对象 |
| `GetFormFillEnv` | 获取表单环境 |
| `BeginBlock` | 开始执行块 |
| `EndBlock` | 结束执行块 |

### 3. 事件上下文

#### CJS_EventContext 类

**位置**: cjs_event_context.h/cpp

管理 JavaScript 事件的执行上下文。

| 方法 | 功能 |
|------|------|
| `RunScript` | 执行脚本 |
| `OnDoc_Open` | 文档打开事件 |
| `OnDoc_WillClose` | 文档关闭前事件 |
| `OnDoc_WillSave` | 保存前事件 |
| `OnDoc_DidSave` | 保存后事件 |
| `OnDoc_WillPrint` | 打印前事件 |
| `OnDoc_DidPrint` | 打印后事件 |
| `OnPage_Open` | 页面打开事件 |
| `OnPage_Close` | 页面关闭事件 |
| `OnField_Focus` | 字段获得焦点 |
| `OnField_Blur` | 字段失去焦点 |
| `OnField_MouseDown` | 字段鼠标按下 |
| `OnField_MouseUp` | 字段鼠标释放 |
| `OnField_MouseEnter` | 字段鼠标进入 |
| `OnField_MouseExit` | 字段鼠标离开 |
| `OnField_Calculate` | 字段计算 |
| `OnField_Format` | 字段格式化 |
| `OnField_KeyStroke` | 字段按键 |
| `OnField_Validate` | 字段验证 |

### 4. JavaScript 对象

#### CJS_Document 类

**位置**: cjs_document.h/cpp

实现 document 对象。

| 属性 | 描述 |
|------|------|
| `numPages` | 页面数 |
| `title` | 标题 |
| `author` | 作者 |
| `subject` | 主题 |
| `keywords` | 关键词 |
| `creationDate` | 创建日期 |
| `modDate` | 修改日期 |
| `URL` | 文档 URL |
| `pageNum` | 当前页号 |

| 方法 | 功能 |
|------|------|
| `getField` | 获取字段 |
| `getNthFieldName` | 获取第 N 个字段名 |
| `print` | 打印文档 |
| `submitForm` | 提交表单 |
| `resetForm` | 重置表单 |
| `mailDoc` | 邮件文档 |
| `calculateNow` | 立即计算 |

#### CJS_App 类

**位置**: cjs_app.h/cpp

实现 app 对象。

| 属性 | 描述 |
|------|------|
| `platform` | 平台 |
| `language` | 语言 |
| `viewerVersion` | 查看器版本 |
| `viewerType` | 查看器类型 |

| 方法 | 功能 |
|------|------|
| `alert` | 显示警告 |
| `beep` | 蜂鸣 |
| `setInterval` | 设置定时器 |
| `setTimeOut` | 设置超时 |
| `clearInterval` | 清除定时器 |
| `clearTimeOut` | 清除超时 |
| `response` | 显示输入对话框 |
| `launchURL` | 打开 URL |
| `popUpMenu` | 弹出菜单 |

#### CJS_Field 类

**位置**: cjs_field.h/cpp

实现 field 对象。

| 属性 | 描述 |
|------|------|
| `value` | 字段值 |
| `name` | 字段名 |
| `type` | 字段类型 |
| `page` | 所在页面 |
| `rect` | 边界矩形 |
| `display` | 显示状态 |
| `fillColor` | 填充色 |
| `textColor` | 文本色 |
| `alignment` | 对齐方式 |
| `readonly` | 只读 |
| `required` | 必填 |

| 方法 | 功能 |
|------|------|
| `setFocus` | 设置焦点 |
| `buttonSetCaption` | 设置按钮标题 |
| `setItems` | 设置选项 |
| `clearItems` | 清除选项 |

#### CJS_Event 类

**位置**: cjs_event.h/cpp

实现 event 对象。

| 属性 | 描述 |
|------|------|
| `change` | 变更内容 |
| `changeEx` | 扩展变更 |
| `value` | 当前值 |
| `selStart` | 选择起始 |
| `selEnd` | 选择结束 |
| `rc` | 返回代码 |
| `name` | 事件名 |
| `type` | 事件类型 |
| `target` | 目标对象 |
| `modifier` | 修饰键 |
| `shift` | Shift 键 |
| `commitKey` | 提交键 |

#### CJS_Util 类

**位置**: cjs_util.h/cpp

实现 util 对象。

| 方法 | 功能 |
|------|------|
| `printf` | 格式化输出 |
| `printd` | 格式化日期 |
| `printx` | 格式化字符串 |
| `scand` | 解析日期 |
| `byteToChar` | 字节转字符 |

#### CJS_Color 类

**位置**: cjs_color.h/cpp

实现 color 对象。

| 属性 | 描述 |
|------|------|
| `transparent` | 透明 |
| `black` | 黑色 |
| `white` | 白色 |
| `red` | 红色 |
| `green` | 绿色 |
| `blue` | 蓝色 |
| `cyan` | 青色 |
| `magenta` | 品红 |
| `yellow` | 黄色 |
| `gray` | 灰色 |

| 方法 | 功能 |
|------|------|
| `convert` | 颜色空间转换 |
| `equal` | 颜色相等判断 |

### 5. 公共方法

#### CJS_PublicMethods

**位置**: cjs_publicmethods.h/cpp

提供全局可用的格式化和验证方法。

| 方法 | 功能 |
|------|------|
| `AFNumber_Format` | 数字格式化 |
| `AFNumber_Keystroke` | 数字按键验证 |
| `AFDate_Format` | 日期格式化 |
| `AFDate_Keystroke` | 日期按键验证 |
| `AFTime_Format` | 时间格式化 |
| `AFTime_Keystroke` | 时间按键验证 |
| `AFPercent_Format` | 百分比格式化 |
| `AFPercent_Keystroke` | 百分比按键验证 |
| `AFSimple_Calculate` | 简单计算 |
| `AFRange_Validate` | 范围验证 |

## 脚本执行流程

```
触发事件
    │
    ├── 创建 EventContext
    │
    ├── 设置事件参数
    │
    ├── 获取关联脚本
    │
    ├── CJS_Runtime::Execute()
    │   │
    │   ├── 编译脚本
    │   │
    │   └── 执行脚本
    │
    ├── 处理返回值
    │
    └── 释放 EventContext
```

## 依赖关系

```
fxjs/
    ├── v8/              (V8 引擎)
    ├── fpdfsdk/         (表单填充环境)
    ├── core/fpdfdoc/    (表单字段)
    └── core/fxcrt/      (基础设施)
```

## 构建配置

- `pdf_enable_v8`: 启用 V8 支持
- 未启用时使用 stub 实现

## 安全考虑

- 脚本在沙箱中执行
- 限制系统访问
- 禁用危险 API
- 超时保护
