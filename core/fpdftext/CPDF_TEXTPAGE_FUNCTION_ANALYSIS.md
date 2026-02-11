# CPDF_TextPage 函数详细分析

本文档对 `core/fpdftext/cpdf_textpage.cpp` 文件中的每个函数进行详细的逐行分析，每个函数都包含完整的源代码。

## 目录

1. [专业术语解释](#专业术语解释)
2. [匿名命名空间辅助函数](#匿名命名空间辅助函数)
3. [类构造和析构函数](#类构造和析构函数)
4. [公共接口方法](#公共接口方法)
5. [私有处理方法](#私有处理方法)

---

## 专业术语解释

在阅读本文档之前，了解以下专业术语有助于理解代码的功能和目的：

### Kerning（字距调整）

**定义**: Kerning 是指调整特定字符对之间间距的排版技术。

**目的**: 
- 改善文本的视觉外观和可读性
- 消除某些字符组合之间过大或过小的间距
- 例如，字母组合 "AV"、"To"、"We" 通常需要减小间距使其看起来更紧凑

**在 PDF 中的表现**:
- 当 `char_code_ == 0xffffffff` 时，表示这是一个 kerning 调整项而非实际字符
- `item.origin_.x` 的值表示水平方向的位移量（以千分之一字体大小为单位）
- 负值表示向左移动（字符更紧凑），正值表示向右移动（字符更分散）

**计算公式**: `kerning = -fontsize_h * item.origin_.x / 1000`

---

### BiDi（双向文本）

**定义**: BiDi 是 Bidirectional（双向）的缩写，指文本可以同时包含从左到右（LTR）和从右到左（RTL）两种书写方向的文字。

**目的**:
- 正确渲染混合方向的文本，如英文和阿拉伯文混排
- 处理 RTL 语言（阿拉伯语、希伯来语）与 LTR 语言（英语、中文）的混合显示

**相关类**: `CFX_BidiString`、`CFX_BidiChar`

---

### CID 字体（CID Font）

**定义**: CID（Character Identifier）字体是一种用于表示大字符集（如中日韩文字）的字体格式。

**特点**:
- 使用 CID 号码而非字符编码来标识字形
- 支持垂直书写模式
- 常用于中文、日文、韩文等东亚语言

**相关方法**: `CIDFromCharCode()`、`GetVertOrigin()`、`GetVertWidth()`

---

### 变换矩阵（Transformation Matrix）

**定义**: 2D 变换矩阵用于将坐标从一个空间变换到另一个空间。

**在 PDF 中的用途**:
- 字体坐标到页面坐标的转换
- 处理文本的旋转、缩放、倾斜
- `matrix.TransformDistance()` 用于变换距离值
- `matrix.Transform()` 用于变换点坐标

**矩阵结构**: `CFX_Matrix` 包含 a, b, c, d, e, f 六个分量

---

### Unicode 规范化（Unicode Normalization）

**定义**: 将 Unicode 字符转换为标准形式的过程。

**目的**:
- 将连字（Ligature，如 ﬁ、ﬂ）分解为基本字符序列（fi、fl）
- 使文本搜索和比较更加准确
- 处理组合字符和预组合字符的等价性

**示例**: 字符 U+FB01 (ﬁ) 规范化为 'f' + 'i'

---

### 字符边界框（Character Bounding Box / CharBox）

**定义**: 包围字符可见部分的最小矩形区域。

**类型**:
- **紧凑边界框（Tight Bounds）**: 仅包含字符主体
- **宽松边界框（Loose Bounds）**: 包含变音符号、上标、下标等扩展区域

**用途**: 文本选择、点击检测、布局计算

---

### 连字符断行（Hyphenation）

**定义**: 在单词中间使用连字符（-）将其分成两行的排版技术。

**相关字符**:
- `0x2D`: 普通连字符（Hyphen-Minus）
- `0xAD`: 软连字符（Soft Hyphen）- 仅在需要断行时显示

**处理方式**: 检测行尾连字符，将跨行的单词连接起来

---

### 文本方向（Text Orientation）

**类型**:
- **水平（Horizontal）**: 从左到右或从右到左的横向文本
- **垂直（Vertical）**: 从上到下的纵向文本（常见于日文、古典中文）

**检测方法**: 分析首尾字符的位置关系确定书写方向

---

### 标记内容（Marked Content）

**定义**: PDF 中用于标识特殊内容区域的结构元素。

**ActualText 属性**: 
- 标记内容可以包含 `ActualText` 属性
- 用于提供替代文本（如图形中的等价文本表示）
- 在文本提取时优先使用 ActualText 的内容

---

### 阈值（Threshold）

**在空格检测中的用途**:
- 判断两个字符之间是否应该插入空格
- 如果字符间距大于阈值，则认为存在单词边界
- 阈值根据字体大小和字符宽度动态计算

---

## 匿名命名空间辅助函数

这些函数定义在匿名命名空间中，仅供本文件内部使用。

---

### 1. NormalizeThreshold

**功能**: 根据阈值大小返回不同的缩放比例，用于计算字符间空格检测的阈值。

**源代码**:
```cpp
float NormalizeThreshold(float threshold, int t1, int t2, int t3) {
  DCHECK(t1 < t2);                    // 断言 t1 < t2，确保参数有效性
  DCHECK(t2 < t3);                    // 断言 t2 < t3，确保参数递增
  if (threshold < t1) {               // 如果阈值小于第一个边界 t1
    return threshold / 2.0f;          // 返回阈值的一半
  }
  if (threshold < t2) {               // 如果阈值在 t1 和 t2 之间
    return threshold / 4.0f;          // 返回阈值的四分之一
  }
  if (threshold < t3) {               // 如果阈值在 t2 和 t3 之间
    return threshold / 5.0f;          // 返回阈值的五分之一
  }
  return threshold / 6.0f;            // 阈值大于等于 t3 时，返回六分之一
}
```

**逐行解释**:
| 行 | 代码 | 解释 |
|----|------|------|
| 1 | `DCHECK(t1 < t2)` | 调试时检查参数t1必须小于t2 |
| 2 | `DCHECK(t2 < t3)` | 调试时检查参数t2必须小于t3 |
| 3-4 | `if (threshold < t1) return threshold / 2.0f` | threshold < t1: 返回 threshold/2.0 |
| 5-6 | `if (threshold < t2) return threshold / 4.0f` | t1 ≤ threshold < t2: 返回 threshold/4.0 |
| 7-8 | `if (threshold < t3) return threshold / 5.0f` | t2 ≤ threshold < t3: 返回 threshold/5.0 |
| 9 | `return threshold / 6.0f` | threshold ≥ t3: 返回 threshold/6.0 |

**用途**: 阈值越大，缩放比例越小，用于空格检测算法中根据字符宽度动态调整阈值。

---

### 2. CalculateBaseSpace

**功能**: 计算文本对象中字符的基础间距，用于确定单词边界。

**源代码**:
```cpp
float CalculateBaseSpace(const CPDF_TextObject* pTextObj,
                         const CFX_Matrix& matrix) {
  const size_t nItems = pTextObj->CountItems();           // 获取文本对象中的项目数量
  const float char_space = pTextObj->text_state().GetCharSpace();  // 获取字符间距设置
  if (char_space == 0.0f || nItems < 3) {                 // 如果字符间距为0或项目少于3个
    return 0.0f;                                          // 直接返回0，不需要计算
  }

  bool bAllChar = true;                                   // 标记是否所有项目都是字符
  const float spacing = matrix.TransformDistance(char_space);  // 使用矩阵变换字符间距
  const float fontsize_h = pTextObj->text_state().GetFontSizeH();  // 获取水平方向字体大小
  float base_space = spacing;                             // 初始化基础间距为变换后的间距
  for (size_t i = 0; i < nItems; ++i) {                   // 遍历所有项目
    CPDF_TextObject::Item item = pTextObj->GetItemInfo(i);  // 获取当前项目信息
    if (item.char_code_ == 0xffffffff) {                  // 如果是特殊字符码（kerning调整）
      float kerning = -fontsize_h * item.origin_.x / 1000;  // 计算kerning值
      base_space = std::min(base_space, kerning + spacing);  // 更新base_space为最小值
      bAllChar = false;                                   // 标记不全是字符
    }
  }
  if (base_space < 0.0 || (nItems == 3 && !bAllChar)) {   // 如果base_space为负或特殊情况
    return 0.0f;                                          // 返回0
  }

  return base_space;                                      // 返回计算的基础间距
}
```

**逐行解释**:
| 行 | 代码 | 解释 |
|----|------|------|
| 1 | `const size_t nItems = pTextObj->CountItems()` | 获取文本对象的项目数量 |
| 2 | `const float char_space = pTextObj->text_state().GetCharSpace()` | 获取字符间距设置 |
| 3-4 | `if (char_space == 0.0f \|\| nItems < 3) return 0.0f` | 间距为0或项目太少时直接返回0 |
| 5 | `bool bAllChar = true` | 初始化标志位，假设全是字符 |
| 6 | `const float spacing = matrix.TransformDistance(char_space)` | 使用矩阵变换间距到页面坐标 |
| 7 | `const float fontsize_h = ...` | 获取水平方向字体大小 |
| 8 | `float base_space = spacing` | 初始化基础间距 |
| 9-14 | `for循环` | 遍历所有项目，检测kerning调整 |
| 10 | `if (item.char_code_ == 0xffffffff)` | 检测特殊字符码（kerning） |
| 11 | `float kerning = -fontsize_h * item.origin_.x / 1000` | kerning值计算公式 |
| 12 | `base_space = std::min(...)` | 取最小间距 |
| 15-16 | `if (base_space < 0.0 ...)` | 处理边界情况 |
| 17 | `return base_space` | 返回结果 |

---

### 3. CalculateBaseSpaceAdjustment

**功能**: 计算基础间距的调整值。

**源代码**:
```cpp
float CalculateBaseSpaceAdjustment(const CPDF_TextObject* pTextObj,
                                   const CFX_Matrix& matrix) {
  float char_space = pTextObj->text_state().GetCharSpace();  // 获取字符间距
  if (char_space > 0.001f) {                              // 如果间距是正值（大于0.001）
    return -matrix.TransformDistance(char_space);         // 返回负的变换距离
  }
  if (char_space < -0.001f) {                             // 如果间距是负值（小于-0.001）
    return matrix.TransformDistance(fabs(char_space));    // 返回正的变换距离绝对值
  }
  return 0.0f;                                            // 间距接近0时返回0
}
```

**逐行解释**:
| 行 | 代码 | 解释 |
|----|------|------|
| 1 | `float char_space = ...GetCharSpace()` | 获取文本状态中的字符间距值 |
| 2-3 | `if (char_space > 0.001f)` | 正间距处理 |
| 3 | `return -matrix.TransformDistance(char_space)` | 返回负的变换后距离（减少间距） |
| 4-5 | `if (char_space < -0.001f)` | 负间距处理 |
| 5 | `return matrix.TransformDistance(fabs(char_space))` | 返回正的变换后距离（增加间距） |
| 6 | `return 0.0f` | 间距接近0时返回0 |

---

### 4. GetUnicodeNormalization

**功能**: 获取字符的Unicode规范化形式，将连字等特殊字符分解为基本字符序列。

**源代码**:
```cpp
DataVector<wchar_t> GetUnicodeNormalization(wchar_t wch) {
  wch = wch & 0xFFFF;                                     // 限制字符在16位范围内
  wchar_t wFind = kUnicodeDataNormalization[wch];         // 从规范化表中查找字符
  if (!wFind) {                                           // 如果没找到映射
    return DataVector<wchar_t>(1, wch);                   // 返回原字符
  }
  if (wFind >= 0x8000) {                                  // 如果值大于等于0x8000
    return DataVector<wchar_t>(1,                         // 从Map1中获取单个映射字符
                               kUnicodeDataNormalizationMap1[wFind - 0x8000]);
  }
  wch = wFind & 0x0FFF;                                   // 提取低12位作为偏移量
  wFind >>= 12;                                           // 右移12位获取映射表索引（2,3,4）
  auto maps = kUnicodeDataNormalizationMaps[wFind - 2].subspan(  // 获取对应映射表的子范围
      static_cast<size_t>(wch));
  if (wFind == 4) {                                       // 如果是Map4
    wFind = maps.front();                                 // 第一个元素是长度
    maps = maps.subspan<1u>();                            // 跳过长度元素
  }
  const auto range = maps.first(static_cast<size_t>(wFind));  // 获取指定长度的范围
  return DataVector<wchar_t>(range.begin(), range.end()); // 返回规范化后的字符序列
}
```

**逐行解释**:
| 行 | 代码 | 解释 |
|----|------|------|
| 1 | `wch = wch & 0xFFFF` | 将字符限制在16位Unicode范围内 |
| 2 | `wchar_t wFind = kUnicodeDataNormalization[wch]` | 查询预定义的规范化表 |
| 3-4 | `if (!wFind)` | 如果无映射，返回原字符 |
| 5-7 | `if (wFind >= 0x8000)` | 如果映射值≥0x8000，从Map1获取单字符映射 |
| 8 | `wch = wFind & 0x0FFF` | 低12位是映射表中的偏移量 |
| 9 | `wFind >>= 12` | 高4位确定使用哪个映射表（Map2/3/4） |
| 10-11 | `auto maps = ...` | 获取对应映射表的子范围 |
| 12-14 | `if (wFind == 4)` | Map4特殊处理：第一个元素表示后续字符数量 |
| 15 | `const auto range = maps.first(...)` | 获取指定长度的范围 |
| 16 | `return DataVector<wchar_t>(...)` | 返回规范化后的字符向量 |

---

### 5. MaskPercentFilled

**功能**: 计算布尔掩码数组中指定范围内true值的百分比。

**源代码**:
```cpp
float MaskPercentFilled(const std::vector<bool>& mask,
                        int32_t start,
                        int32_t end) {
  if (start >= end) {                                     // 如果范围无效
    return 0;                                             // 返回0
  }
  float count = std::count_if(mask.begin() + start, mask.begin() + end,
                              [](bool r) { return r; });  // 统计true的数量
  return count / (end - start);                           // 返回百分比
}
```

**逐行解释**:
| 行 | 代码 | 解释 |
|----|------|------|
| 1-2 | `if (start >= end) return 0` | 检查范围有效性，start必须小于end |
| 3-4 | `float count = std::count_if(...)` | 使用count_if算法统计范围内true的数量 |
| 5 | `return count / (end - start)` | 返回true数量除以范围长度，得到填充百分比 |

**用途**: 用于判断文本方向（水平或垂直）时分析页面区域的填充情况。

---

### 6. IsControlChar

**功能**: 判断字符是否为控制字符。

**源代码**:
```cpp
bool IsControlChar(const CPDF_TextPage::CharInfo& char_info) {
  switch (char_info.unicode()) {                          // 根据unicode值判断
    case 0x2:                                             // STX (Start of Text)
    case 0x3:                                             // ETX (End of Text)
    case 0x93:                                            // 特殊控制字符
    case 0x94:                                            // 特殊控制字符
    case 0x96:                                            // 特殊控制字符
    case 0x97:                                            // 特殊控制字符
    case 0x98:                                            // 特殊控制字符
    case 0xfffe:                                          // 非字符标记
      return char_info.char_type() != CPDF_TextPage::CharType::kHyphen;  // 非连字符时返回true
    default:
      return false;                                       // 其他字符返回false
  }
}
```

**逐行解释**:
| 行 | 代码 | 解释 |
|----|------|------|
| 1 | `switch (char_info.unicode())` | 检查字符的unicode值 |
| 2-8 | `case 0x2: case 0x3: ...` | 匹配特定的控制字符码 |
| 9 | `return char_info.char_type() != ...kHyphen` | 特殊情况：连字符不算控制字符 |
| 10 | `default: return false` | 其他字符返回false |

---

### 7. IsHyphenCode

**功能**: 判断字符是否为连字符。

**源代码**:
```cpp
bool IsHyphenCode(wchar_t c) {
  return c == 0x2D || c == 0xAD;                          // 0x2D是普通连字符，0xAD是软连字符
}
```

**逐行解释**:
| 字符码 | 说明 |
|--------|------|
| `0x2D` | ASCII连字符 '-' |
| `0xAD` | Unicode软连字符（Soft Hyphen） |

---

### 8. IsNormalCharacter

**功能**: 判断是否为正常可显示字符。

**源代码**:
```cpp
bool IsNormalCharacter(const CPDF_TextPage::CharInfo& char_info) {
  return char_info.unicode() != 0 ? !IsControlChar(char_info)  // unicode非零时检查是否为控制字符
                                  : char_info.char_code() != 0;  // unicode为零时检查char_code
}
```

**逐行解释**:
| 条件 | 判断逻辑 |
|------|----------|
| `unicode != 0` | 正常字符 = 非控制字符 |
| `unicode == 0` | 正常字符 = char_code不为0 |

---

### 9. IsRectIntersect

**功能**: 判断两个矩形是否相交。

**源代码**:
```cpp
bool IsRectIntersect(const CFX_FloatRect& rect1, const CFX_FloatRect& rect2) {
  CFX_FloatRect rect = rect1;                             // 复制rect1
  rect.Intersect(rect2);                                  // 计算与rect2的交集
  return !rect.IsEmpty();                                 // 交集非空则相交
}
```

---

### 10. IsRightToLeft

**功能**: 判断文本对象是否为从右到左（RTL）的书写方向。

**源代码**:
```cpp
bool IsRightToLeft(const CPDF_TextObject& text_obj) {
  RetainPtr<const CPDF_Font> font = text_obj.GetFont();   // 获取字体
  const size_t nItems = text_obj.CountItems();            // 获取项目数量
  WideString str;                                         // 创建字符串
  str.Reserve(nItems);                                    // 预分配空间
  for (size_t i = 0; i < nItems; ++i) {                   // 遍历所有项目
    CPDF_TextObject::Item item = text_obj.GetItemInfo(i); // 获取项目信息
    if (item.char_code_ == 0xffffffff) {                  // 跳过特殊字符码
      continue;
    }
    WideString unicode = font->UnicodeFromCharCode(item.char_code_);  // 转换为unicode
    wchar_t wChar = !unicode.IsEmpty() ? unicode[0] : 0;  // 获取第一个字符
    if (wChar == 0) {                                     // 如果转换失败
      wChar = item.char_code_;                            // 直接使用char_code
    }
    if (wChar) {                                          // 如果字符有效
      str += wChar;                                       // 添加到字符串
    }
  }
  return CFX_BidiString(str).OverallDirection() ==        // 使用BiDi算法判断方向
         CFX_BidiChar::Direction::kRight;
}
```

---

### 11. GetCharWidth

**功能**: 获取字符的宽度。

**源代码**:
```cpp
int GetCharWidth(uint32_t charCode, CPDF_Font* font) {
  if (charCode == CPDF_Font::kInvalidCharCode) {          // 如果是无效字符码
    return 0;                                             // 返回0
  }

  int w = font->GetCharWidthF(charCode);                  // 尝试直接获取字符宽度
  if (w > 0) {                                            // 如果获取成功
    return w;                                             // 返回宽度
  }

  ByteString str;                                         // 创建字节字符串
  font->AppendChar(&str, charCode);                       // 将字符追加到字符串
  w = font->GetStringWidth(str.AsStringView());           // 获取字符串宽度
  if (w > 0) {                                            // 如果获取成功
    return w;                                             // 返回宽度
  }

  FX_RECT rect = font->GetCharBBox(charCode);             // 获取字符边界框
  if (!rect.Valid()) {                                    // 如果边界框无效
    return 0;                                             // 返回0
  }

  return std::max(rect.Width(), 0);                       // 返回边界框宽度（非负）
}
```

**尝试顺序**:
1. 直接调用GetCharWidthF获取宽度
2. 构建字符串后获取字符串宽度
3. 使用字符边界框的宽度

---

### 12. CalculateSpaceThreshold

**功能**: 计算空格检测的阈值。

**源代码**:
```cpp
float CalculateSpaceThreshold(CPDF_Font* font,
                              float fontsize_h,
                              uint32_t char_code) {
  const uint32_t space_charcode = font->CharCodeFromUnicode(' ');  // 获取空格的字符码
  float threshold = 0;                                    // 初始化阈值
  if (space_charcode != CPDF_Font::kInvalidCharCode) {    // 如果空格字符有效
    threshold = fontsize_h * font->GetCharWidthF(space_charcode) / 1000;  // 计算阈值
  }
  if (threshold > fontsize_h / 3) {                       // 如果阈值过大
    threshold = 0;                                        // 重置为0
  } else {
    threshold /= 2;                                       // 否则减半
  }
  if (threshold == 0) {                                   // 如果阈值仍为0
    threshold = GetCharWidth(char_code, font);            // 使用字符宽度
    threshold = NormalizeThreshold(threshold, 300, 500, 700);  // 规范化阈值
    threshold = fontsize_h * threshold / 1000;            // 转换为页面单位
  }
  return threshold;                                       // 返回阈值
}
```

---

### 13. GenerateSpace

**功能**: 判断是否应在两个字符之间生成空格。

**源代码**:
```cpp
bool GenerateSpace(const CFX_PointF& pos,
                   float last_pos,
                   float this_width,
                   float last_width,
                   float threshold) {
  if (fabs(last_pos + last_width - pos.x) <= threshold) { // 如果字符紧邻
    return false;                                         // 不生成空格
  }

  float threshold_pos = threshold + last_width;           // 计算位置阈值
  float pos_difference = pos.x - last_pos;                // 计算位置差
  if (fabs(pos_difference) > threshold_pos) {             // 如果位置差大于阈值
    return true;                                          // 生成空格
  }
  if (pos.x < 0 && -threshold_pos > pos_difference) {     // 处理负坐标情况
    return true;                                          // 生成空格
  }
  return pos_difference > this_width + last_width;        // 位置差大于两字符宽度和时生成空格
}
```

---

### 14. EndHorizontalLine

**功能**: 判断水平文本行是否结束。

**源代码**:
```cpp
bool EndHorizontalLine(const CFX_FloatRect& this_rect,
                       const CFX_FloatRect& prev_rect) {
  if (this_rect.Height() <= 4.5 || prev_rect.Height() <= 4.5) {  // 如果矩形太小
    return false;                                         // 不判断为行结束
  }

  float top = std::min(this_rect.top, prev_rect.top);     // 取两个矩形顶部的较小值
  float bottom = std::max(this_rect.bottom, prev_rect.bottom);  // 取底部的较大值
  return bottom >= top;                                   // 如果不重叠则行结束
}
```

---

### 15. EndVerticalLine

**功能**: 判断垂直文本列是否结束。

**源代码**:
```cpp
bool EndVerticalLine(const CFX_FloatRect& this_rect,
                     const CFX_FloatRect& prev_rect,
                     const CFX_FloatRect& curline_rect,
                     float this_fontsize,
                     float prev_fontsize) {
  if (this_rect.Width() <= this_fontsize * 0.1f ||        // 如果当前矩形宽度过小
      prev_rect.Width() <= prev_fontsize * 0.1f) {        // 或前一个矩形宽度过小
    return false;                                         // 不判断为列结束
  }

  float left = std::max(this_rect.left, curline_rect.left);    // 取左边界的较大值
  float right = std::min(this_rect.right, curline_rect.right); // 取右边界的较小值
  return right <= left;                                   // 如果不重叠则列结束
}
```

---

### 16. GetFontSize

**功能**: 安全地获取文本对象的字体大小。

**源代码**:
```cpp
float GetFontSize(const CPDF_TextObject* text_object) {
  bool has_font = text_object && text_object->GetFont();  // 检查文本对象和字体是否有效
  return has_font ? text_object->GetFontSize() : kDefaultFontSize;  // 返回字体大小或默认值1.0
}
```

---

### 17. GetLooseBounds

**功能**: 获取字符的宽松边界框，包含变音符号等扩展区域。

**源代码**:
```cpp
CFX_FloatRect GetLooseBounds(const CPDF_TextPage::CharInfo& charinfo) {
  if (charinfo.char_box().IsEmpty()) {                    // 如果字符框为空
    return charinfo.char_box();                           // 直接返回空框
  }

  const CPDF_TextObject* text_object = charinfo.text_object();  // 获取文本对象
  float font_size = GetFontSize(text_object);             // 获取字体大小
  if (text_object && !FXSYS_IsFloatZero(font_size) &&     // 如果文本对象有效
      charinfo.char_code() != CPDF_Font::kInvalidCharCode) {  // 且字符码有效
    RetainPtr<CPDF_Font> font = text_object->GetFont();   // 获取字体
    bool is_vert_writing = font->IsVertWriting();         // 是否垂直书写
    
    // CID字体垂直书写的特殊处理
    if (is_vert_writing && font->IsCIDFont()) {
      CPDF_CIDFont* pCIDFont = font->AsCIDFont();         // 转换为CID字体
      uint16_t cid = pCIDFont->CIDFromCharCode(charinfo.char_code());  // 获取CID

      CFX_Point16 vertical_origin = pCIDFont->GetVertOrigin(cid);  // 获取垂直原点
      double offsetx = (vertical_origin.x - 500) * font_size / 1000.0;  // 计算X偏移
      double offsety = vertical_origin.y * font_size / 1000.0;  // 计算Y偏移
      int16_t vert_width = pCIDFont->GetVertWidth(cid);   // 获取垂直宽度（通常为负值）
      double height = vert_width * font_size / 1000.0;    // 计算高度

      float left = charinfo.origin().x + offsetx;         // 计算左边界
      float right = left + font_size;                     // 计算右边界
      float top = charinfo.origin().y + offsety;          // 计算顶边界
      float bottom = top + height;                        // 计算底边界
      CFX_FloatRect char_box(left, bottom, right, top);   // 创建边界框
      char_box.Union(charinfo.char_box());                // 与原边界框合并
      return char_box;                                    // 返回合并后的边界框
    }

    // 普通字体处理
    FX_RECT font_bbox = font->GetFontBBox();              // 获取字体边界框
    if (font_bbox.Valid() && font_bbox.Height() != 0) {   // 如果字体边界框有效
      float width = text_object->GetCharWidth(charinfo.char_code());  // 获取字符宽度
      CFX_Matrix inverse_matrix = charinfo.matrix().GetInverse();  // 获取逆矩阵
      CFX_PointF original_origin = inverse_matrix.Transform(charinfo.origin());  // 变换原点

      float left = original_origin.x;                     // 左边界为原点X
      float right = original_origin.x + (is_vert_writing ? -width : width);  // 右边界

      // 使用字体边界框计算顶部和底部（包含变音符号）
      float bottom = font_bbox.bottom * font_size / 1000; // 底部
      float top = font_bbox.top * font_size / 1000;       // 顶部
      CFX_FloatRect char_box = charinfo.matrix().TransformRect(
          CFX_FloatRect(left, bottom, right, top));       // 变换边界框
      char_box.Union(charinfo.char_box());                // 与原边界框合并
      return char_box;                                    // 返回合并后的边界框
    }
  }

  // 回退到紧凑边界
  return charinfo.char_box();
}
```

---

## 类构造和析构函数

### 18. TransformedTextObject 构造/析构函数

**源代码**:
```cpp
CPDF_TextPage::TransformedTextObject::TransformedTextObject() = default;

CPDF_TextPage::TransformedTextObject::TransformedTextObject(
    const TransformedTextObject& that) = default;

CPDF_TextPage::TransformedTextObject::~TransformedTextObject() = default;
```

**说明**: 使用编译器生成的默认实现。

---

### 19. CharInfo 构造函数

**源代码**:
```cpp
CPDF_TextPage::CharInfo::CharInfo() = default;            // 默认构造函数

CPDF_TextPage::CharInfo::CharInfo(CharType char_type,     // 完整构造函数
                                  uint32_t char_code,
                                  wchar_t unicode,
                                  CFX_PointF origin,
                                  CFX_FloatRect char_box,
                                  CFX_Matrix matrix,
                                  CPDF_TextObject* text_object)
    : char_type_(char_type),                              // 初始化字符类型
      unicode_(unicode),                                  // 初始化Unicode
      char_code_(char_code),                              // 初始化字符码
      origin_(origin),                                    // 初始化原点
      char_box_(char_box),                                // 初始化字符框
      matrix_(matrix),                                    // 初始化变换矩阵
      text_object_(text_object) {                         // 初始化文本对象指针
  loose_char_box_ = GetLooseBounds(*this);                // 计算宽松边界框
}

CPDF_TextPage::CharInfo::CharInfo(const CharInfo&) = default;  // 拷贝构造函数

CPDF_TextPage::CharInfo::~CharInfo() = default;           // 析构函数
```

---

### 20. CPDF_TextPage 构造函数

**源代码**:
```cpp
CPDF_TextPage::CPDF_TextPage(const CPDF_Page* pPage, bool rtl)
    : page_(pPage),                                       // 初始化页面指针
      rtl_(rtl),                                          // 初始化RTL标志
      display_matrix_(page_->GetDisplayMatrix()) {        // 初始化显示矩阵
  Init();                                                 // 调用初始化方法
}

CPDF_TextPage::~CPDF_TextPage() = default;                // 析构函数
```

---

### 21. Init

**功能**: 初始化文本页面，执行文本提取。

**源代码**:
```cpp
void CPDF_TextPage::Init() {
  text_buf_.SetAllocStep(10240);                          // 设置文本缓冲区分配步长为10KB
  ProcessObject();                                        // 处理页面对象，提取文本

  const int nCount = CountChars();                        // 获取字符数量
  if (nCount) {                                           // 如果有字符
    char_indices_.push_back({0, 0});                      // 初始化索引数组
  }

  bool skipped = false;                                   // 标记是否跳过了字符
  for (int i = 0; i < nCount; ++i) {                      // 遍历所有字符
    const CharInfo& charinfo = char_list_[i];             // 获取字符信息
    if (charinfo.char_type() == CharType::kGenerated ||   // 如果是生成字符
        IsNormalCharacter(charinfo)) {                    // 或正常字符
      char_indices_.back().count++;                       // 增加当前段的计数
      skipped = true;                                     // 标记为已处理
    } else {                                              // 否则是非正常字符
      if (skipped) {                                      // 如果之前有处理过的字符
        char_indices_.push_back({i + 1, 0});              // 开始新的索引段
        skipped = false;                                  // 重置标记
      } else {                                            // 如果连续的非正常字符
        char_indices_.back().index = i + 1;               // 更新当前段的起始索引
      }
    }
  }
}
```

---

## 公共接口方法

### 22. CountChars

**源代码**:
```cpp
int CPDF_TextPage::CountChars() const {
  return fxcrt::CollectionSize<int>(char_list_);          // 返回字符列表大小
}
```

---

### 23. CharIndexFromTextIndex

**功能**: 将文本索引转换为字符索引。

**源代码**:
```cpp
int CPDF_TextPage::CharIndexFromTextIndex(int text_index) const {
  int count = 0;                                          // 累计计数
  for (const auto& info : char_indices_) {                // 遍历索引段
    count += info.count;                                  // 累加每段的计数
    if (count > text_index) {                             // 如果累计超过目标索引
      return text_index - count + info.count + info.index;  // 计算字符索引
    }
  }
  return -1;                                              // 未找到返回-1
}
```

---

### 24. TextIndexFromCharIndex

**功能**: 将字符索引转换为文本索引。

**源代码**:
```cpp
int CPDF_TextPage::TextIndexFromCharIndex(int char_index) const {
  int count = 0;                                          // 累计计数
  for (const auto& info : char_indices_) {                // 遍历索引段
    int text_index = char_index - info.index;             // 计算相对索引
    if (text_index < info.count) {                        // 如果在当前段内
      return text_index >= 0 ? text_index + count : -1;   // 返回文本索引
    }

    count += info.count;                                  // 累加计数
  }
  return -1;                                              // 未找到返回-1
}
```

---

### 25-35. 其他公共方法

**GetRectArray**: 获取指定字符范围的矩形数组
**GetIndexAtPos**: 获取指定坐标位置的字符索引
**GetTextByPredicate**: 根据谓词条件获取文本
**GetTextByRect**: 获取矩形区域内的文本
**GetTextByObject**: 获取特定文本对象的文本
**GetCharInfo**: 获取指定索引的字符信息
**GetCharFontSize**: 获取字符字体大小
**GetCharLooseBounds**: 获取字符的宽松边界
**GetPageText**: 获取页面指定范围的文本
**CountRects/GetRect**: 矩形选择相关方法

---

## 私有处理方法

### 36. ProcessObject

**功能**: 遍历页面中的所有对象。

**源代码**:
```cpp
void CPDF_TextPage::ProcessObject() {
  if (page_->GetActivePageObjectCount() == 0) {           // 如果没有活动对象
    return;                                               // 直接返回
  }

  textline_dir_ = FindTextlineFlowOrientation();          // 检测文本方向
  for (auto it = page_->begin(); it != page_->end(); ++it) {  // 遍历页面对象
    CPDF_PageObject* pObj = it->get();                    // 获取对象
    if (!pObj->IsActive()) {                              // 跳过非活动对象
      continue;
    }

    if (pObj->IsText()) {                                 // 如果是文本对象
      ProcessTextObject(pObj->AsText(), CFX_Matrix(), page_, it);
    } else if (pObj->IsForm()) {                          // 如果是表单对象
      ProcessFormObject(pObj->AsForm(), CFX_Matrix());
    }
  }
  for (const auto& obj : text_objects_) {                 // 处理收集的文本对象
    ProcessTextObject(obj);
  }

  text_objects_.clear();                                  // 清空列表
  CloseTempLine();                                        // 关闭临时行
}
```

---

### 37-57. 其他私有方法概述

| 方法 | 功能 |
|------|------|
| `ProcessFormObject` | 递归处理表单对象中的文本 |
| `ProcessTextObject` (排序版本) | 收集文本对象并按位置排序 |
| `ProcessTextObject` (处理版本) | 处理已排序的文本对象，提取字符 |
| `ProcessTextObjectItems` | 处理文本对象中的所有字符项目 |
| `CloseTempLine` | 关闭临时文本行，进行双向文本处理 |
| `AddCharInfoByLRDirection` | 按从左到右方向添加字符 |
| `AddCharInfoByRLDirection` | 按从右到左方向添加字符 |
| `FindTextlineFlowOrientation` | 检测页面文本的主要流向 |
| `GetTextObjectWritingMode` | 检测单个文本对象的书写模式 |
| `IsHyphen` | 判断当前位置是否为连字符断行 |
| `GetPrevCharInfo` | 获取前一个字符的信息 |
| `ProcessInsertObject` | 处理新插入的文本对象 |
| `ProcessGenerateCharacter` | 根据类型生成相应字符 |
| `PreMarkedContent` | 预处理标记内容 |
| `ProcessMarkedContent` | 处理标记内容的实际文本 |
| `IsSameTextObject` | 判断两个文本对象是否相同 |
| `IsSameAsPreTextObject` | 检查是否与前几个文本对象相同 |
| `GenerateCharInfo` | 为生成的字符创建CharInfo |
| `SwapTempTextBuf` | 反转临时缓冲区（用于RTL处理） |
| `FindPreviousTextObject` | 查找前一个文本对象 |
| `AppendGeneratedCharacter` | 添加生成的字符 |

---

## 总结

### 文本提取流程

```
CPDF_TextPage() 构造函数
    │
    ▼
Init() 初始化
    │
    ▼
ProcessObject() 处理页面对象
    │
    ├──► ProcessFormObject() 处理表单对象（递归）
    │
    └──► ProcessTextObject() 收集并排序文本对象
              │
              ▼
         ProcessTextObject() 处理文本对象
              │
              ▼
         ProcessTextObjectItems() 处理字符项目
              │
              ▼
         AddCharInfoByLRDirection() / AddCharInfoByRLDirection()
              │
              ▼
         CloseTempLine() 双向文本处理
              │
              ▼
         char_list_ 最终字符列表
```

### 关键数据结构

| 数据结构 | 用途 |
|----------|------|
| `char_list_` | 最终的字符列表 |
| `temp_char_list_` | 临时字符列表（用于双向处理） |
| `text_buf_` | 文本缓冲区 |
| `temp_text_buf_` | 临时文本缓冲区 |
| `char_indices_` | 文本索引到字符索引的映射 |
| `text_objects_` | 待处理的文本对象列表 |

### 设计模式

- **模板方法**: `GetTextByPredicate()` 提供通用框架
- **迭代器**: 遍历字符和文本对象
- **工厂方法**: `GenerateCharInfo()` 创建字符信息
