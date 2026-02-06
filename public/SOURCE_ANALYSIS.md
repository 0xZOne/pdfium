# public 源码分析

## 模块概述

`public` 目录包含 PDFium 的公共 API 头文件。这些头文件定义了嵌入器（embedder）与 PDFium 库交互的所有接口。所有 API 都是 C 语言接口，以确保二进制兼容性和跨语言调用能力。

## 目录结构

```
public/
├── DEPS                      # 依赖声明
├── PRESUBMIT.py              # 预提交检查脚本
├── README                    # 说明文件
│
├── 核心 API
│   └── fpdfview.h            # 核心功能：初始化、文档加载、页面渲染
│
├── 编辑 API
│   └── fpdf_edit.h           # 创建和修改 PDF 文档
│
├── 文本 API
│   └── fpdf_text.h           # 文本提取和搜索
│
├── 表单 API
│   ├── fpdf_formfill.h       # 交互式表单处理
│   └── fpdf_fwlevent.h       # 表单事件定义
│
├── 注释 API
│   └── fpdf_annot.h          # 注释读取和创建
│
├── 文档 API
│   ├── fpdf_doc.h            # 书签、链接、元数据
│   ├── fpdf_catalog.h        # 文档目录访问
│   └── fpdf_attachment.h     # 文件附件
│
├── 保存 API
│   └── fpdf_save.h           # 保存修改后的 PDF
│
├── 渲染 API
│   ├── fpdf_progressive.h    # 渐进式渲染
│   └── fpdf_transformpage.h  # 页面变换
│
├── 签名 API
│   └── fpdf_signature.h      # 数字签名
│
├── 辅助功能 API
│   └── fpdf_structtree.h     # 结构树（无障碍支持）
│
├── 其他 API
│   ├── fpdf_dataavail.h      # 数据可用性（线性化）
│   ├── fpdf_ext.h            # 扩展功能
│   ├── fpdf_flatten.h        # 表单扁平化
│   ├── fpdf_javascript.h     # JavaScript 动作
│   ├── fpdf_ppo.h            # 页面操作
│   ├── fpdf_searchex.h       # 扩展搜索
│   ├── fpdf_sysfontinfo.h    # 系统字体
│   └── fpdf_thumbnail.h      # 缩略图
│
└── cpp/                      # C++ 封装（可选）
    └── fpdf_scopers.h        # RAII 封装
```

## 核心类型定义

### 句柄类型

所有 PDFium 对象通过不透明句柄暴露：

| 句柄类型 | 描述 |
|----------|------|
| `FPDF_DOCUMENT` | 文档句柄 |
| `FPDF_PAGE` | 页面句柄 |
| `FPDF_BITMAP` | 位图句柄 |
| `FPDF_FORMHANDLE` | 表单句柄 |
| `FPDF_TEXTPAGE` | 文本页面句柄 |
| `FPDF_PAGEOBJECT` | 页面对象句柄 |
| `FPDF_FONT` | 字体句柄 |
| `FPDF_ANNOTATION` | 注释句柄 |
| `FPDF_BOOKMARK` | 书签句柄 |
| `FPDF_LINK` | 链接句柄 |
| `FPDF_ACTION` | 动作句柄 |
| `FPDF_DEST` | 目标句柄 |
| `FPDF_STRUCTTREE` | 结构树句柄 |
| `FPDF_STRUCTELEMENT` | 结构元素句柄 |
| `FPDF_SIGNATURE` | 签名句柄 |

### 基本数据类型

| 类型 | 定义 |
|------|------|
| `FPDF_BOOL` | 布尔值（int） |
| `FPDF_DWORD` | 32位无符号整数 |
| `FS_FLOAT` | 浮点数 |
| `FPDF_WCHAR` | 宽字符 |
| `FPDF_BYTESTRING` | 字节字符串 |
| `FPDF_WIDESTRING` | UTF-16LE 宽字符串 |

### 几何类型

| 类型 | 成员 |
|------|------|
| `FS_MATRIX` | a, b, c, d, e, f (变换矩阵) |
| `FS_RECTF` | left, top, right, bottom |
| `FS_SIZEF` | width, height |
| `FS_POINTF` | x, y |
| `FS_QUADPOINTSF` | x1, y1, x2, y2, x3, y3, x4, y4 |

## API 分类详解

### 1. 核心 API (fpdfview.h)

#### 库初始化

| 函数 | 功能 |
|------|------|
| `FPDF_InitLibrary` | 初始化库 |
| `FPDF_InitLibraryWithConfig` | 带配置初始化 |
| `FPDF_DestroyLibrary` | 销毁库 |
| `FPDF_SetSandBoxPolicy` | 设置沙箱策略 |

#### 文档操作

| 函数 | 功能 |
|------|------|
| `FPDF_LoadDocument` | 从文件加载 |
| `FPDF_LoadMemDocument` | 从内存加载 |
| `FPDF_LoadCustomDocument` | 自定义加载 |
| `FPDF_CloseDocument` | 关闭文档 |
| `FPDF_GetPageCount` | 获取页数 |
| `FPDF_GetDocPermissions` | 获取权限 |
| `FPDF_GetSecurityHandlerRevision` | 获取安全版本 |

#### 页面操作

| 函数 | 功能 |
|------|------|
| `FPDF_LoadPage` | 加载页面 |
| `FPDF_ClosePage` | 关闭页面 |
| `FPDF_GetPageWidth` | 获取宽度 |
| `FPDF_GetPageHeight` | 获取高度 |
| `FPDF_GetPageSizeByIndex` | 按索引获取尺寸 |

#### 渲染操作

| 函数 | 功能 |
|------|------|
| `FPDF_RenderPageBitmap` | 渲染到位图 |
| `FPDF_RenderPageBitmapWithMatrix` | 带矩阵渲染 |
| `FPDFBitmap_Create` | 创建位图 |
| `FPDFBitmap_CreateEx` | 创建扩展位图 |
| `FPDFBitmap_Destroy` | 销毁位图 |
| `FPDFBitmap_GetBuffer` | 获取缓冲区 |
| `FPDFBitmap_GetWidth` | 获取宽度 |
| `FPDFBitmap_GetHeight` | 获取高度 |
| `FPDFBitmap_GetStride` | 获取步长 |
| `FPDFBitmap_GetFormat` | 获取格式 |
| `FPDFBitmap_FillRect` | 填充矩形 |

### 2. 编辑 API (fpdf_edit.h)

#### 页面对象

| 函数 | 功能 |
|------|------|
| `FPDFPage_New` | 创建新页面 |
| `FPDFPage_Delete` | 删除页面 |
| `FPDFPage_CountObjects` | 计数对象 |
| `FPDFPage_GetObject` | 获取对象 |
| `FPDFPage_InsertObject` | 插入对象 |
| `FPDFPage_RemoveObject` | 移除对象 |
| `FPDFPage_GenerateContent` | 生成内容 |

#### 文本对象

| 函数 | 功能 |
|------|------|
| `FPDFPageObj_NewTextObj` | 创建文本对象 |
| `FPDFText_SetText` | 设置文本 |
| `FPDFText_LoadFont` | 加载字体 |

#### 路径对象

| 函数 | 功能 |
|------|------|
| `FPDFPageObj_CreateNewPath` | 创建路径 |
| `FPDFPath_MoveTo` | 移动到 |
| `FPDFPath_LineTo` | 画线到 |
| `FPDFPath_BezierTo` | 贝塞尔到 |
| `FPDFPath_Close` | 闭合路径 |
| `FPDFPath_SetDrawMode` | 设置绘制模式 |

#### 图像对象

| 函数 | 功能 |
|------|------|
| `FPDFPageObj_NewImageObj` | 创建图像对象 |
| `FPDFImageObj_LoadJpegFile` | 加载 JPEG |
| `FPDFImageObj_SetBitmap` | 设置位图 |
| `FPDFImageObj_GetBitmap` | 获取位图 |

### 3. 文本 API (fpdf_text.h)

| 函数 | 功能 |
|------|------|
| `FPDFText_LoadPage` | 加载文本页面 |
| `FPDFText_ClosePage` | 关闭文本页面 |
| `FPDFText_CountChars` | 计数字符 |
| `FPDFText_GetUnicode` | 获取 Unicode |
| `FPDFText_GetCharBox` | 获取字符框 |
| `FPDFText_GetCharIndexAtPos` | 按位置获取索引 |
| `FPDFText_GetText` | 获取文本 |
| `FPDFText_CountRects` | 计数矩形 |
| `FPDFText_GetRect` | 获取矩形 |
| `FPDFText_FindStart` | 开始搜索 |
| `FPDFText_FindNext` | 查找下一个 |
| `FPDFText_FindPrev` | 查找上一个 |
| `FPDFText_FindClose` | 关闭搜索 |
| `FPDFLink_LoadWebLinks` | 加载网页链接 |
| `FPDFLink_CountWebLinks` | 计数链接 |
| `FPDFLink_GetURL` | 获取 URL |

### 4. 表单 API (fpdf_formfill.h)

| 函数 | 功能 |
|------|------|
| `FPDFDOC_InitFormFillEnvironment` | 初始化表单环境 |
| `FPDFDOC_ExitFormFillEnvironment` | 退出表单环境 |
| `FORM_OnAfterLoadPage` | 页面加载后回调 |
| `FORM_OnBeforeClosePage` | 页面关闭前回调 |
| `FORM_DoDocumentJSAction` | 执行文档 JS 动作 |
| `FORM_DoDocumentOpenAction` | 执行文档打开动作 |
| `FORM_DoPageAAction` | 执行页面附加动作 |
| `FORM_OnMouseMove` | 鼠标移动 |
| `FORM_OnLButtonDown` | 鼠标左键按下 |
| `FORM_OnLButtonUp` | 鼠标左键释放 |
| `FORM_OnChar` | 字符输入 |
| `FORM_OnKeyDown` | 按键按下 |
| `FORM_OnKeyUp` | 按键释放 |
| `FORM_OnFocus` | 获得焦点 |
| `FORM_GetFocusedText` | 获取焦点文本 |
| `FORM_GetSelectedText` | 获取选中文本 |
| `FORM_ReplaceSelection` | 替换选中 |

### 5. 注释 API (fpdf_annot.h)

| 函数 | 功能 |
|------|------|
| `FPDFPage_GetAnnotCount` | 获取注释数量 |
| `FPDFPage_GetAnnot` | 获取注释 |
| `FPDFPage_CreateAnnot` | 创建注释 |
| `FPDFPage_RemoveAnnot` | 移除注释 |
| `FPDFAnnot_GetSubtype` | 获取子类型 |
| `FPDFAnnot_GetRect` | 获取矩形 |
| `FPDFAnnot_SetRect` | 设置矩形 |
| `FPDFAnnot_GetColor` | 获取颜色 |
| `FPDFAnnot_SetColor` | 设置颜色 |
| `FPDFAnnot_GetContents` | 获取内容 |
| `FPDFAnnot_SetContents` | 设置内容 |

### 6. 保存 API (fpdf_save.h)

| 函数 | 功能 |
|------|------|
| `FPDF_SaveAsCopy` | 另存为副本 |
| `FPDF_SaveWithVersion` | 指定版本保存 |

## 回调接口

### 文件访问

定义自定义文件访问：
- `FPDF_FILEACCESS`: 读取接口
- `FPDF_FILEWRITE`: 写入接口

### 表单填充

定义表单交互回调：
- `FPDF_FORMFILLINFO`: 表单填充回调集合
- `IPDF_JSPLATFORM`: JavaScript 平台回调

## 错误处理

| 错误码 | 含义 |
|--------|------|
| `FPDF_ERR_SUCCESS` | 成功 |
| `FPDF_ERR_UNKNOWN` | 未知错误 |
| `FPDF_ERR_FILE` | 文件访问错误 |
| `FPDF_ERR_FORMAT` | 格式错误 |
| `FPDF_ERR_PASSWORD` | 密码错误 |
| `FPDF_ERR_SECURITY` | 安全错误 |
| `FPDF_ERR_PAGE` | 页面错误 |

获取错误码：`FPDF_GetLastError()`

## 线程安全

**重要提示**: PDFium API 不是线程安全的。所有调用必须：
- 从单一线程进行，或
- 通过互斥锁进行同步

## 使用示例

### 基本渲染流程

```c
// 1. 初始化
FPDF_InitLibrary();

// 2. 加载文档
FPDF_DOCUMENT doc = FPDF_LoadDocument("file.pdf", NULL);

// 3. 获取页面数
int pages = FPDF_GetPageCount(doc);

// 4. 加载页面
FPDF_PAGE page = FPDF_LoadPage(doc, 0);

// 5. 创建位图
int width = (int)FPDF_GetPageWidth(page);
int height = (int)FPDF_GetPageHeight(page);
FPDF_BITMAP bitmap = FPDFBitmap_Create(width, height, 0);
FPDFBitmap_FillRect(bitmap, 0, 0, width, height, 0xFFFFFFFF);

// 6. 渲染
FPDF_RenderPageBitmap(bitmap, page, 0, 0, width, height, 0, 0);

// 7. 获取数据
void* buffer = FPDFBitmap_GetBuffer(bitmap);

// 8. 清理
FPDFBitmap_Destroy(bitmap);
FPDF_ClosePage(page);
FPDF_CloseDocument(doc);
FPDF_DestroyLibrary();
```

## C++ 封装 (cpp/)

`fpdf_scopers.h` 提供 RAII 风格的智能指针：
- `ScopedFPDFDocument`
- `ScopedFPDFPage`
- `ScopedFPDFBitmap`
- `ScopedFPDFTextPage`
- 等等

这些封装自动管理资源生命周期，防止内存泄漏。
