---
name: docx
description: "DOCX文档创建、编辑和分析。当Claude需要处理文档(.docx文件)时使用：(1)创建新文档，(2)修改或编辑内容，(3)处理布局，(4)添加评论或修订，或任何其他文档任务"
license: 专有。LICENSE.txt包含完整条款
---

# DOCX文档创建、编辑和分析

## 概述

用户可能要求您创建、编辑或分析.docx文件的内容。.docx文件本质上是一个包含XML文件和其他资源的ZIP压缩包，您可以读取或编辑这些内容。针对不同的任务，您可以使用不同的工具和工作流程。

## 读取和分析内容

### 文本提取
如果您只需要读取文档的文本内容，应将文档转换为markdown格式：

```bash
# 将文档转换为markdown
python -m markitdown 文件路径.docx
```

### 原始XML访问
对于以下功能，您需要原始XML访问：评论、修订、样式、复杂格式和设计元素。对于这些功能中的任何一项，您需要解包文档并读取其原始XML内容。

#### 解包文件
`python ooxml/scripts/unpack.py <office_file> <output_dir>`

**注意**：unpack.py脚本位于项目根目录下的`skills/docx/ooxml/scripts/unpack.py`。如果此路径不存在该脚本，请使用`find . -name "unpack.py"`来定位它。

#### 关键文件结构
* `word/document.xml` - 主要文档内容
* `word/comments.xml` - 文档评论
* `word/settings.xml` - 文档设置
* `word/styles.xml` - 样式定义
* `word/fontTable.xml` - 字体信息
* `word/numbering.xml` - 编号和项目符号列表
* `word/footnotes.xml` - 脚注
* `word/endnotes.xml` - 尾注
* `word/header1.xml` - 页眉内容
* `word/footer1.xml` - 页脚内容

#### 修订追踪
**当处理包含修订的文档时**：
1. **检查修订状态**：查看`word/settings.xml`中的`<w:trackRevisions>`设置
2. **读取修订内容**：检查`word/document.xml`中的`<w:ins>`（插入）和`<w:del>`（删除）元素
3. **处理修订作者**：修订包含作者信息和时间戳

## 创建新Word文档

### 工作流程决策树

**选择正确的方法**：
- **简单文档**：使用docx-js库创建基本文档
- **复杂格式**：使用Office Open XML (OOXML)进行精确控制
- **基于模板**：解包模板，修改内容，重新打包

### 使用docx-js库
对于简单的文档创建，使用docx-js库：

1. **阅读完整文件**：完整阅读[`docx-js.md`](docx-js.md)文件，了解详细语法和最佳实践
2. **创建JavaScript文件**：使用docx-js API构建文档结构
3. **生成文档**：运行脚本创建.docx文件

### 使用Office Open XML (OOXML)
对于需要精确控制的复杂文档：

1. **阅读完整文件**：完整阅读[`ooxml.md`](ooxml.md)文件，了解OOXML结构和编辑工作流程
2. **创建基本结构**：构建必要的XML文件和目录结构
3. **添加内容**：在`word/document.xml`中添加段落、表格、图像等
4. **定义样式**：在`word/styles.xml`中创建样式定义
5. **打包文档**：使用打包脚本创建最终的.docx文件

## 编辑现有Word文档

当编辑现有Word文档时，您需要使用原始Office Open XML (OOXML)格式。这涉及解包.docx文件，编辑XML内容，然后重新打包。

### 工作流程
1. **强制阅读完整文件**：完整阅读[`ooxml.md`](ooxml.md)文件，了解OOXML结构和编辑工作流程
2. **解包文档**：`python ooxml/scripts/unpack.py <office_file> <output_dir>`
3. **编辑XML文件**（主要是`word/document.xml`和相关文件）
4. **关键步骤**：每次编辑后立即验证并修复任何验证错误：`python ooxml/scripts/validate.py <dir> --original <file>`
5. **打包最终文档**：`python ooxml/scripts/pack.py <input_directory> <office_file>`

### 修订工作流程
**处理包含修订的文档**：
1. **提取修订信息**：从`word/document.xml`中读取`<w:ins>`和`<w:del>`元素
2. **接受或拒绝修订**：通过删除相应的修订元素来处理修订
3. **保留修订历史**：如果需要保留修订信息，请勿删除修订标记

## 使用模板创建文档

当您需要创建遵循现有模板设计的文档时，需要复制和重新排列模板内容，然后替换占位符内容。

### 工作流程
1. **提取模板文本**：`python -m markitdown 模板.docx > 模板内容.md`
2. **分析模板结构**：阅读模板内容文件了解模板的布局和设计模式
3. **创建内容大纲**：基于模板结构规划文档内容
4. **替换内容**：使用适当的工具替换模板中的占位符文本
5. **验证格式**：确保新内容正确适应模板布局

## 最佳实践

### 文档结构
- **使用标准段落标记**：`<w:p>`用于段落，`<w:r>`用于文本运行
- **保持样式一致性**：重用现有样式而不是创建新样式
- **处理分页**：注意分页符和节分隔符的位置

### 内容处理
- **文本编码**：确保正确处理Unicode字符
- **图像处理**：将图像添加到`word/media/`目录并更新关系文件
- **表格处理**：使用适当的表格结构标记

### 错误预防
- **验证XML**：每次修改后验证XML结构
- **测试兼容性**：在不同版本的Word中测试生成的文档
- **备份原始文件**：在编辑前保留原始文档的备份

## 故障排除

### 常见问题
- **文档无法打开**：检查XML语法错误和文件关系
- **格式丢失**：验证样式定义和引用
- **图像不显示**：检查媒体文件路径和关系

### 调试技巧
- **逐步验证**：每次修改后验证文档
- **比较原始文件**：与工作正常的文档进行比较
- **使用XML验证工具**：利用提供的验证脚本检查错误