# CPDF_TextPage 函数详细分析

本文档对 `core/fpdftext/cpdf_textpage.cpp` 文件中的每个函数进行详细的逐行分析。

## 目录

1. [匿名命名空间辅助函数](#匿名命名空间辅助函数)
2. [类构造和析构函数](#类构造和析构函数)
3. [公共接口方法](#公共接口方法)
4. [私有处理方法](#私有处理方法)

---

## 匿名命名空间辅助函数

这些函数定义在匿名命名空间中，仅供本文件内部使用。

### 1. NormalizeThreshold (第47-60行)

```cpp
float NormalizeThreshold(float threshold, int t1, int t2, int t3)
```

**功能**: 根据阈值大小返回不同的缩放比例。

**逐行分析**:
- 第48行: `DCHECK(t1 < t2)` - 断言 t1 小于 t2，确保参数有效
- 第49行: `DCHECK(t2 < t3)` - 断言 t2 小于 t3
- 第50-52行: 如果 threshold < t1，返回 threshold/2.0
- 第53-55行: 如果 threshold < t2，返回 threshold/4.0
- 第56-58行: 如果 threshold < t3，返回 threshold/5.0
- 第59行: 否则返回 threshold/6.0

**用途**: 用于计算字符间空格检测的阈值，阈值越大，缩放比例越小。

---

### 2. CalculateBaseSpace (第62-87行)

```cpp
float CalculateBaseSpace(const CPDF_TextObject* pTextObj, const CFX_Matrix& matrix)
```

**功能**: 计算文本对象中字符的基础间距。

**逐行分析**:
- 第64行: 获取文本对象中的项目数量
- 第65行: 获取字符间距设置
- 第66-68行: 如果字符间距为0或项目少于3个，返回0
- 第70行: 初始化 bAllChar 为 true
- 第71行: 使用矩阵变换字符间距
- 第72行: 获取水平方向的字体大小
- 第73行: 初始化 base_space 为变换后的间距
- 第74-81行: 遍历所有项目
  - 第76行: 检查是否为特殊字符码 0xffffffff（kerning 调整）
  - 第77行: 计算 kerning 值
  - 第78行: 更新 base_space 为最小值
  - 第79行: 设置 bAllChar 为 false
- 第82-84行: 如果 base_space 为负或者只有3个项目且不全是字符，返回0
- 第86行: 返回计算的 base_space

**用途**: 用于确定文本对象中字符之间的基础间距，以便正确识别单词边界。

---

### 3. CalculateBaseSpaceAdjustment (第89-99行)

```cpp
float CalculateBaseSpaceAdjustment(const CPDF_TextObject* pTextObj, const CFX_Matrix& matrix)
```

**功能**: 计算基础间距的调整值。

**逐行分析**:
- 第91行: 获取字符间距
- 第92-94行: 如果间距大于0.001，返回负的变换距离
- 第95-97行: 如果间距小于-0.001，返回正的变换距离的绝对值
- 第98行: 否则返回0

**用途**: 根据字符间距的正负值进行相应的调整。

---

### 4. GetUnicodeNormalization (第101-121行)

```cpp
DataVector<wchar_t> GetUnicodeNormalization(wchar_t wch)
```

**功能**: 获取字符的 Unicode 规范化形式。

**逐行分析**:
- 第102行: 将字符限制在16位范围内
- 第103行: 从规范化表中查找字符
- 第104-106行: 如果没找到，返回原字符
- 第107-110行: 如果值 >= 0x8000，从 Map1 中获取映射
- 第111行: 提取低12位作为索引
- 第112行: 右移12位获取映射表索引
- 第113-114行: 从对应的映射表中获取子范围
- 第115-118行: 如果是 Map4，需要额外处理
- 第119-120行: 返回规范化后的字符序列

**用途**: 将连字等特殊字符分解为基本字符序列，用于文本搜索匹配。

---

### 5. MaskPercentFilled (第123-132行)

```cpp
float MaskPercentFilled(const std::vector<bool>& mask, int32_t start, int32_t end)
```

**功能**: 计算掩码数组中指定范围内 true 值的百分比。

**逐行分析**:
- 第126-128行: 如果 start >= end，返回0
- 第129-130行: 使用 count_if 统计 true 的数量
- 第131行: 返回百分比（count / 范围长度）

**用途**: 用于判断文本方向（水平或垂直）。

---

### 6. IsControlChar (第134-148行)

```cpp
bool IsControlChar(const CPDF_TextPage::CharInfo& char_info)
```

**功能**: 判断字符是否为控制字符。

**逐行分析**:
- 第135-144行: 检查字符的 Unicode 值是否匹配特定的控制字符码
  - 0x2, 0x3: 控制字符
  - 0x93, 0x94, 0x96, 0x97, 0x98: 特殊控制字符
  - 0xfffe: 非字符
- 第144行: 返回 true 条件是字符类型不是连字符

**用途**: 识别需要特殊处理的控制字符。

---

### 7. IsHyphenCode (第150-152行)

```cpp
bool IsHyphenCode(wchar_t c)
```

**功能**: 判断字符是否为连字符。

**逐行分析**:
- 第151行: 检查字符是否为 0x2D（普通连字符）或 0xAD（软连字符）

**用途**: 识别连字符以处理跨行单词。

---

### 8. IsNormalCharacter (第154-157行)

```cpp
bool IsNormalCharacter(const CPDF_TextPage::CharInfo& char_info)
```

**功能**: 判断是否为正常可显示字符。

**逐行分析**:
- 第155-156行: 如果 unicode 非零，返回 !IsControlChar；否则检查 char_code 是否非零

**用途**: 过滤掉控制字符和空字符。

---

### 9. IsRectIntersect (第159-163行)

```cpp
bool IsRectIntersect(const CFX_FloatRect& rect1, const CFX_FloatRect& rect2)
```

**功能**: 判断两个矩形是否相交。

**逐行分析**:
- 第160行: 复制 rect1
- 第161行: 计算与 rect2 的交集
- 第162行: 返回交集是否非空

**用途**: 用于选择区域内的文本。

---

### 10. IsRightToLeft (第165-186行)

```cpp
bool IsRightToLeft(const CPDF_TextObject& text_obj)
```

**功能**: 判断文本对象是否为从右到左的书写方向。

**逐行分析**:
- 第166行: 获取字体
- 第167行: 获取项目数量
- 第168-169行: 预分配字符串空间
- 第170-183行: 遍历所有项目，收集 Unicode 字符
  - 第172-174行: 跳过特殊字符码
  - 第175-179行: 获取 Unicode 字符
  - 第180-182行: 添加到字符串
- 第184-185行: 使用 CFX_BidiString 判断整体方向

**用途**: 识别阿拉伯语、希伯来语等 RTL 文本。

---

### 11. GetCharWidth (第188-211行)

```cpp
int GetCharWidth(uint32_t charCode, CPDF_Font* font)
```

**功能**: 获取字符的宽度。

**逐行分析**:
- 第189-191行: 如果是无效字符码，返回0
- 第193-196行: 尝试直接获取字符宽度
- 第198-203行: 如果失败，尝试通过字符串获取宽度
- 第205-210行: 如果仍失败，使用字符边界框的宽度

**用途**: 计算字符间距和空格检测。

---

### 12. CalculateSpaceThreshold (第213-232行)

```cpp
float CalculateSpaceThreshold(CPDF_Font* font, float fontsize_h, uint32_t char_code)
```

**功能**: 计算空格检测的阈值。

**逐行分析**:
- 第216行: 获取空格字符的编码
- 第217行: 初始化阈值为0
- 第218-220行: 如果空格有效，计算基于空格宽度的阈值
- 第221-225行: 如果阈值过大或过小，进行调整
- 第226-230行: 如果阈值仍为0，基于字符宽度计算

**用途**: 确定字符间距多大时应该插入空格。

---

### 13. GenerateSpace (第234-252行)

```cpp
bool GenerateSpace(const CFX_PointF& pos, float last_pos, float this_width, 
                   float last_width, float threshold)
```

**功能**: 判断是否应在两个字符之间生成空格。

**逐行分析**:
- 第239-241行: 如果字符紧邻，不生成空格
- 第243-244行: 计算阈值位置和位置差
- 第245-247行: 如果位置差大于阈值，生成空格
- 第248-250行: 处理负坐标情况
- 第251行: 如果位置差大于两个字符宽度之和，生成空格

**用途**: 在单词之间自动插入空格。

---

### 14. EndHorizontalLine (第254-263行)

```cpp
bool EndHorizontalLine(const CFX_FloatRect& this_rect, const CFX_FloatRect& prev_rect)
```

**功能**: 判断水平文本行是否结束。

**逐行分析**:
- 第256-258行: 如果任一矩形高度 <= 4.5，返回 false
- 第260行: 计算两个矩形顶部的较小值
- 第261行: 计算两个矩形底部的较大值
- 第262行: 如果 bottom >= top，说明不在同一行

**用途**: 检测换行位置。

---

### 15. EndVerticalLine (第265-278行)

```cpp
bool EndVerticalLine(const CFX_FloatRect& this_rect, const CFX_FloatRect& prev_rect,
                     const CFX_FloatRect& curline_rect, float this_fontsize, float prev_fontsize)
```

**功能**: 判断垂直文本列是否结束。

**逐行分析**:
- 第270-273行: 如果任一矩形宽度过小，返回 false
- 第275行: 计算当前矩形和行矩形左边的较大值
- 第276行: 计算右边的较小值
- 第277行: 如果 right <= left，说明不在同一列

**用途**: 处理垂直排版的日文、中文等文本。

---

### 16. GetFontSize (第280-283行)

```cpp
float GetFontSize(const CPDF_TextObject* text_object)
```

**功能**: 获取文本对象的字体大小。

**逐行分析**:
- 第281行: 检查文本对象和字体是否有效
- 第282行: 有效则返回字体大小，否则返回默认值 1.0

**用途**: 安全地获取字体大小。

---

### 17. GetLooseBounds (第285-339行)

```cpp
CFX_FloatRect GetLooseBounds(const CPDF_TextPage::CharInfo& charinfo)
```

**功能**: 获取字符的宽松边界框（包含变音符号等）。

**逐行分析**:
- 第286-288行: 如果字符框为空，直接返回
- 第290-293行: 获取文本对象和字体大小
- 第294-335行: 根据字体类型计算宽松边界
  - 第295-314行: 处理 CID 字体的垂直书写
  - 第316-334行: 处理普通字体，使用字体边界框
- 第337-338行: 回退到紧凑边界

**用途**: 提供更准确的字符选择区域。

---

## 类构造和析构函数

### 18. TransformedTextObject 构造/析构 (第343-348行)

```cpp
CPDF_TextPage::TransformedTextObject::TransformedTextObject() = default;
CPDF_TextPage::TransformedTextObject::TransformedTextObject(const TransformedTextObject& that) = default;
CPDF_TextPage::TransformedTextObject::~TransformedTextObject() = default;
```

**功能**: TransformedTextObject 的默认构造、拷贝构造和析构函数。

---

### 19. CharInfo 构造函数 (第350-371行)

```cpp
CPDF_TextPage::CharInfo::CharInfo()
CPDF_TextPage::CharInfo::CharInfo(CharType, uint32_t, wchar_t, CFX_PointF, 
                                   CFX_FloatRect, CFX_Matrix, CPDF_TextObject*)
CPDF_TextPage::CharInfo::CharInfo(const CharInfo&)
CPDF_TextPage::CharInfo::~CharInfo()
```

**功能**: CharInfo 的各种构造函数。

**逐行分析** (完整构造函数):
- 第352-358行: 参数列表包含字符类型、编码、Unicode、原点、边界框、矩阵、文本对象
- 第359-366行: 初始化列表设置所有成员
- 第366行: 计算宽松边界框

---

### 20. CPDF_TextPage 构造函数 (第373-376行)

```cpp
CPDF_TextPage::CPDF_TextPage(const CPDF_Page* pPage, bool rtl)
```

**功能**: 初始化文本页面对象。

**逐行分析**:
- 第374行: 初始化页面指针、RTL 标志、显示矩阵
- 第375行: 调用 Init() 开始文本提取

---

### 21. Init (第380-405行)

```cpp
void CPDF_TextPage::Init()
```

**功能**: 初始化文本页面，执行文本提取。

**逐行分析**:
- 第381行: 设置文本缓冲区的分配步长为 10240
- 第382行: 调用 ProcessObject() 处理页面对象
- 第384行: 获取字符数量
- 第385-387行: 如果有字符，初始化索引数组
- 第389行: 初始化跳过标志
- 第390-404行: 遍历所有字符，构建索引映射
  - 第392-395行: 如果是生成字符或正常字符，增加计数
  - 第396-403行: 否则处理非正常字符的索引

**用途**: 建立文本索引到字符索引的映射关系。

---

## 公共接口方法

### 22. CountChars (第407-409行)

```cpp
int CPDF_TextPage::CountChars() const
```

**功能**: 返回字符列表的大小。

---

### 23. CharIndexFromTextIndex (第411-420行)

```cpp
int CPDF_TextPage::CharIndexFromTextIndex(int text_index) const
```

**功能**: 将文本索引转换为字符索引。

**逐行分析**:
- 第412行: 初始化计数器
- 第413-418行: 遍历索引段
  - 第414行: 累加每段的计数
  - 第415-417行: 如果累计超过目标索引，计算并返回字符索引
- 第419行: 未找到返回 -1

**用途**: 在选择文本时将文本位置映射到字符位置。

---

### 24. TextIndexFromCharIndex (第422-433行)

```cpp
int CPDF_TextPage::TextIndexFromCharIndex(int char_index) const
```

**功能**: 将字符索引转换为文本索引。

**逐行分析**:
- 第423行: 初始化计数器
- 第424-431行: 遍历索引段
  - 第425行: 计算相对于段起始的文本索引
  - 第426-428行: 如果在当前段内，返回文本索引
  - 第430行: 累加段计数
- 第432行: 未找到返回 -1

---

### 25. GetRectArray (第435-483行)

```cpp
std::vector<CFX_FloatRect> CPDF_TextPage::GetRectArray(int start, int count) const
```

**功能**: 获取指定字符范围的矩形数组。

**逐行分析**:
- 第437行: 创建结果向量
- 第438-440行: 参数验证
- 第442-445行: 检查起始位置
- 第447-450行: 调整计数范围
- 第452-455行: 初始化变量
- 第456-480行: 遍历字符
  - 第458-460行: 跳过生成字符
  - 第461-464行: 跳过过小的字符框
  - 第465-472行: 检测文本对象变化，开始新矩形
  - 第473-479行: 合并矩形或初始化新矩形
- 第481行: 添加最后的矩形
- 第482行: 返回结果

**用途**: 用于文本选择高亮显示。

---

### 26. GetIndexAtPos (第485-523行)

```cpp
int CPDF_TextPage::GetIndexAtPos(const CFX_PointF& point, const CFX_SizeF& tolerance) const
```

**功能**: 获取指定坐标位置的字符索引。

**逐行分析**:
- 第487-491行: 初始化变量
- 第492-521行: 遍历所有字符
  - 第494-496行: 如果点在字符框内，直接返回
  - 第498-500行: 如果没有容差，继续下一个
  - 第502-510行: 扩展字符框并检查
  - 第512-520行: 计算距离，更新最近位置
- 第522行: 返回精确匹配或最近位置

---

### 27. GetTextByPredicate (第525-557行)

```cpp
WideString CPDF_TextPage::GetTextByPredicate(
    const std::function<bool(const CharInfo&)>& predicate) const
```

**功能**: 根据谓词条件获取文本。

**逐行分析**:
- 第527-530行: 初始化变量
- 第531-555行: 遍历字符列表
  - 第532行: 检查谓词条件
  - 第533-544行: 如果满足条件
    - 处理换行
    - 添加字符到结果
  - 第545-554行: 处理空格和其他字符

**用途**: 通用的文本提取框架。

---

### 28. GetTextByRect (第559-563行)

```cpp
WideString CPDF_TextPage::GetTextByRect(const CFX_FloatRect& rect) const
```

**功能**: 获取矩形区域内的文本。

**逐行分析**:
- 第560-562行: 调用 GetTextByPredicate，使用矩形相交检测作为谓词

---

### 29. GetTextByObject (第565-570行)

```cpp
WideString CPDF_TextPage::GetTextByObject(const CPDF_TextObject* pTextObj) const
```

**功能**: 获取特定文本对象的文本。

**逐行分析**:
- 第567-569行: 调用 GetTextByPredicate，使用对象匹配作为谓词

---

### 30. GetCharInfo (第572-580行)

```cpp
const CPDF_TextPage::CharInfo& CPDF_TextPage::GetCharInfo(size_t index) const
CPDF_TextPage::CharInfo& CPDF_TextPage::GetCharInfo(size_t index)
```

**功能**: 获取指定索引的字符信息（const 和非 const 版本）。

**逐行分析**:
- 第573/578行: CHECK 验证索引有效
- 第574/579行: 返回字符信息引用

---

### 31. GetCharFontSize (第582-585行)

```cpp
float CPDF_TextPage::GetCharFontSize(size_t index) const
```

**功能**: 获取指定索引字符的字体大小。

---

### 32. GetCharLooseBounds (第587-590行)

```cpp
CFX_FloatRect CPDF_TextPage::GetCharLooseBounds(size_t index) const
```

**功能**: 获取指定索引字符的宽松边界。

---

### 33. GetPageText (第592-636行)

```cpp
WideString CPDF_TextPage::GetPageText(int start, int count) const
```

**功能**: 获取页面指定范围的文本。

**逐行分析**:
- 第593-596行: 参数验证
- 第598-610行: 处理起始位置的非打印字符
- 第612行: 调整计数
- 第614-627行: 处理结束位置的非打印字符
- 第629-631行: 验证范围
- 第633-635行: 返回文本子串

---

### 34. CountRects (第638-645行)

```cpp
int CPDF_TextPage::CountRects(int start, int nCount)
```

**功能**: 计算指定范围的矩形数量。

---

### 35. GetRect (第647-654行)

```cpp
bool CPDF_TextPage::GetRect(int rectIndex, CFX_FloatRect* pRect) const
```

**功能**: 获取指定索引的矩形。

---

## 私有处理方法

### 36. FindTextlineFlowOrientation (第656-725行)

```cpp
CPDF_TextPage::TextOrientation CPDF_TextPage::FindTextlineFlowOrientation() const
```

**功能**: 检测页面文本的主要流向（水平或垂直）。

**逐行分析**:
- 第658-662行: 获取页面尺寸，无效则返回未知
- 第664-670行: 初始化掩码和边界变量
- 第671-703行: 遍历页面对象
  - 第672-674行: 跳过非活动和非文本对象
  - 第676-686行: 计算边界并更新掩码
  - 第688-698行: 更新范围边界
  - 第700-702行: 获取行高
- 第704-710行: 基于范围判断方向
- 第712-723行: 基于填充百分比判断方向

**用途**: 确定文本是水平还是垂直排列。

---

### 37. AppendGeneratedCharacter (第727-742行)

```cpp
void CPDF_TextPage::AppendGeneratedCharacter(wchar_t unicode, 
    const CFX_Matrix& form_matrix, bool use_temp_buffer)
```

**功能**: 添加生成的字符（如空格、换行）。

**逐行分析**:
- 第730-733行: 生成字符信息，失败则返回
- 第735-741行: 根据 use_temp_buffer 添加到相应的缓冲区

---

### 38. ProcessObject (第744-768行)

```cpp
void CPDF_TextPage::ProcessObject()
```

**功能**: 处理页面中的所有对象。

**逐行分析**:
- 第745-747行: 如果没有活动对象，返回
- 第749行: 检测文本方向
- 第750-761行: 遍历页面对象
  - 第752-754行: 跳过非活动对象
  - 第756-760行: 分别处理文本和表单对象
- 第762-764行: 处理收集的文本对象
- 第766-767行: 清理并关闭临时行

---

### 39. ProcessFormObject (第770-786行)

```cpp
void CPDF_TextPage::ProcessFormObject(CPDF_FormObject* pFormObj, 
    const CFX_Matrix& form_matrix)
```

**功能**: 递归处理表单对象中的文本。

**逐行分析**:
- 第772行: 计算组合矩阵
- 第773行: 获取表单持有者
- 第774-785行: 遍历表单中的对象
  - 递归处理文本和嵌套表单

---

### 40. AddCharInfoByLRDirection (第788-811行)

```cpp
void CPDF_TextPage::AddCharInfoByLRDirection(wchar_t wChar, const CharInfo& info)
```

**功能**: 按从左到右的方向添加字符。

**逐行分析**:
- 第790-793行: 非正常字符直接添加
- 第795-798行: 检查是否需要 Unicode 规范化（连字）
- 第799-803行: 无需规范化，直接添加
- 第804-810行: 添加规范化后的字符序列

**用途**: 处理 LTR 文本的字符添加。

---

### 41. AddCharInfoByRLDirection (第813-835行)

```cpp
void CPDF_TextPage::AddCharInfoByRLDirection(wchar_t wChar, const CharInfo& info)
```

**功能**: 按从右到左的方向添加字符。

**逐行分析**:
- 第815-817行: 非正常字符直接添加
- 第820-821行: 创建修改后的信息并获取镜像字符
- 第822-828行: 规范化并添加
- 第829-834行: 添加规范化后的字符序列

**用途**: 处理 RTL 文本（阿拉伯语、希伯来语）。

---

### 42. CloseTempLine (第837-890行)

```cpp
void CPDF_TextPage::CloseTempLine()
```

**功能**: 关闭临时文本行，进行双向文本处理。

**逐行分析**:
- 第838-840行: 如果临时列表为空，返回
- 第842行: 获取临时缓冲区字符串
- 第843-865行: 删除连续空格
- 第866-869行: 设置双向文本方向
- 第870行: 获取整体方向
- 第871-887行: 根据段方向添加字符
  - RTL 段：从后向前添加
  - LTR 段：从前向后添加
- 第888-889行: 清空临时缓冲区

**用途**: 处理混合方向的文本。

---

### 43. ProcessTextObject (重载1) (第892-956行)

```cpp
void CPDF_TextPage::ProcessTextObject(CPDF_TextObject* pTextObj,
    const CFX_Matrix& form_matrix, const CPDF_PageObjectHolder* pObjList,
    CPDF_PageObjectHolder::const_iterator ObjPos)
```

**功能**: 处理文本对象，按位置排序插入。

**逐行分析**:
- 第897-904行: 首个对象直接添加
- 第905-907行: 检查是否与前一个相同
- 第909-926行: 计算位置和宽度阈值
- 第936-943行: 如果 Y 位置差异大，处理已收集的对象
- 第945-956行: 按 X 位置排序插入

**用途**: 收集并排序同一行的文本对象。

---

### 44. PreMarkedContent (第958-1010行)

```cpp
CPDF_TextPage::MarkedContentState CPDF_TextPage::PreMarkedContent(
    const CPDF_TextObject* pTextObj)
```

**功能**: 预处理标记内容（ActualText）。

**逐行分析**:
- 第960-964行: 检查内容标记
- 第966-983行: 查找 ActualText 属性
- 第985-991行: 检查是否与前一个对象相同
- 第993-1009行: 验证 ActualText 内容

**返回值**:
- `kPass`: 正常处理
- `kDone`: 已处理过
- `kDelay`: 延迟处理

---

### 45. ProcessMarkedContent (第1012-1058行)

```cpp
void CPDF_TextPage::ProcessMarkedContent(const TransformedTextObject& obj)
```

**功能**: 处理标记内容的实际文本。

**逐行分析**:
- 第1013-1026行: 获取 ActualText
- 第1028-1039行: 计算字符框和步长
- 第1041-1057行: 为每个字符创建 CharInfo

---

### 46. FindPreviousTextObject (第1060-1069行)

```cpp
void CPDF_TextPage::FindPreviousTextObject()
```

**功能**: 查找前一个文本对象。

---

### 47. SwapTempTextBuf (第1071-1087行)

```cpp
void CPDF_TextPage::SwapTempTextBuf(size_t iCharListStartAppend, size_t iBufStartAppend)
```

**功能**: 反转临时缓冲区的一部分（用于 RTL 处理）。

---

### 48. ProcessTextObject (重载2) (第1089-1136行)

```cpp
void CPDF_TextPage::ProcessTextObject(const TransformedTextObject& obj)
```

**功能**: 处理已排序的文本对象。

**逐行分析**:
- 第1090-1098行: 处理标记内容
- 第1100-1113行: 处理与前一对象的关系
- 第1115-1120行: 处理延迟标记内容
- 第1122-1135行: 处理文本项目

---

### 49. GetTextObjectWritingMode (第1138-1166行)

```cpp
CPDF_TextPage::TextOrientation CPDF_TextPage::GetTextObjectWritingMode(
    const CPDF_TextObject* pTextObj) const
```

**功能**: 检测单个文本对象的书写模式。

**逐行分析**:
- 第1140-1143行: 单字符返回默认方向
- 第1145-1149行: 获取首尾字符位置
- 第1151-1165行: 根据位移比例判断方向

---

### 50. IsHyphen (第1168-1197行)

```cpp
bool CPDF_TextPage::IsHyphen(wchar_t curChar) const
```

**功能**: 判断当前位置是否为连字符断行。

**逐行分析**:
- 第1169-1176行: 获取当前文本视图
- 第1178-1181行: 跳过尾部空格
- 第1183-1185行: 检查是否为连字符
- 第1187-1196行: 检查上下文是否符合连字符条件

---

### 51. GetPrevCharInfo (第1199-1204行)

```cpp
const CPDF_TextPage::CharInfo* CPDF_TextPage::GetPrevCharInfo() const
```

**功能**: 获取前一个字符的信息。

---

### 52. ProcessInsertObject (第1206-1330行)

```cpp
CPDF_TextPage::GenerateCharacter CPDF_TextPage::ProcessInsertObject(
    const CPDF_TextObject* pObj, const CFX_Matrix& form_matrix)
```

**功能**: 处理新插入的文本对象，决定生成什么字符。

**逐行分析**:
- 第1209-1218行: 获取书写模式和前一对象信息
- 第1220-1241行: 检测行结束（水平或垂直）
- 第1243-1258行: 计算位置和阈值
- 第1261-1295行: 检测换行条件
- 第1297-1329行: 判断是否需要生成空格

**返回值**: `kNone`, `kSpace`, `kLineBreak`, `kHyphen`

---

### 53. ProcessGenerateCharacter (第1332-1375行)

```cpp
bool CPDF_TextPage::ProcessGenerateCharacter(GenerateCharacter type,
    const CPDF_TextObject* text_object, const CFX_Matrix& form_matrix)
```

**功能**: 根据类型生成相应字符。

**逐行分析**:
- 第1335-1337行: `kNone` - 不生成
- 第1338-1341行: `kSpace` - 生成空格
- 第1342-1348行: `kLineBreak` - 生成换行
- 第1349-1373行: `kHyphen` - 处理连字符

---

### 54. ProcessTextObjectItems (第1377-1481行)

```cpp
void CPDF_TextPage::ProcessTextObjectItems(CPDF_TextObject* text_object,
    const CFX_Matrix& form_matrix, const CFX_Matrix& matrix)
```

**功能**: 处理文本对象中的所有项目（字符）。

**逐行分析**:
- 第1380-1382行: 计算基础间距
- 第1384-1385行: 初始化变量
- 第1386-1480行: 遍历所有项目
  - 第1388-1397行: 处理 kerning 调整
  - 第1400-1416行: 检查是否需要插入空格
  - 第1418-1424行: 获取 Unicode
  - 第1426-1439行: 计算字符边界框
  - 第1441-1479行: 添加字符，处理重复检测

---

### 55. IsSameTextObject (第1483-1542行)

```cpp
bool CPDF_TextPage::IsSameTextObject(CPDF_TextObject* pTextObj1,
    CPDF_TextObject* pTextObj2) const
```

**功能**: 判断两个文本对象是否相同（重复检测）。

**逐行分析**:
- 第1485-1487行: 空指针检查
- 第1489-1512行: 检查边界框重叠和字体大小
- 第1514-1522行: 检查项目数量
- 第1524-1541行: 比较字符和位置

---

### 56. IsSameAsPreTextObject (第1544-1561行)

```cpp
bool CPDF_TextPage::IsSameAsPreTextObject(CPDF_TextObject* pTextObj,
    const CPDF_PageObjectHolder* pObjList,
    CPDF_PageObjectHolder::const_iterator iter) const
```

**功能**: 检查是否与前5个文本对象之一相同。

---

### 57. GenerateCharInfo (第1563-1590行)

```cpp
std::optional<CPDF_TextPage::CharInfo> CPDF_TextPage::GenerateCharInfo(
    wchar_t unicode, const CFX_Matrix& form_matrix)
```

**功能**: 为生成的字符创建 CharInfo。

**逐行分析**:
- 第1566-1568行: 获取前一字符信息
- 第1571-1576行: 计算字符宽度
- 第1578-1583行: 获取字体大小
- 第1585-1589行: 创建并返回 CharInfo

---

## 总结

### 文本提取流程

1. **初始化**: `CPDF_TextPage()` → `Init()` → `ProcessObject()`
2. **对象收集**: `ProcessFormObject()` → `ProcessTextObject()`（排序版本）
3. **文本处理**: `ProcessTextObject()`（处理版本）→ `ProcessTextObjectItems()`
4. **字符添加**: `AddCharInfoByLRDirection()` / `AddCharInfoByRLDirection()`
5. **行结束**: `CloseTempLine()` → 双向文本处理

### 关键数据结构

- `char_list_`: 最终的字符列表
- `temp_char_list_`: 临时字符列表（用于双向处理）
- `text_buf_`: 文本缓冲区
- `temp_text_buf_`: 临时文本缓冲区
- `text_objects_`: 待处理的文本对象列表

### 设计模式

- **模板方法**: `GetTextByPredicate()` 提供通用框架
- **迭代器**: 遍历字符和文本对象
- **工厂方法**: `GenerateCharInfo()` 创建字符信息
