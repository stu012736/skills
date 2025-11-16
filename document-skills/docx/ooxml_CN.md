# Office Open XML 技术参考

**重要：在开始之前请完整阅读本文档。** 本文档涵盖：
- [技术指南](#技术指南) - 架构合规规则和验证要求
- [文档内容模式](#文档内容模式) - 标题、列表、表格、格式化等的XML模式
- [文档库（Python）](#文档库python) - 推荐使用自动基础设施设置的OOXML操作方法
- [修订追踪（红线标记）](#修订追踪红线标记) - 实现修订追踪的XML模式

## 技术指南

### 架构合规性
- **`<w:pPr>`中的元素顺序**：`<w:pStyle>`、`<w:numPr>`、`<w:spacing>`、`<w:ind>`、`<w:jc>`
- **空白字符**：在包含前导/尾随空格的`<w:t>`元素中添加`xml:space='preserve'`
- **Unicode**：在ASCII内容中转义字符：`"`变为`&#8220;`
  - **字符编码参考**：花引号`""`变为`&#8220;&#8221;`，撇号`'`变为`&#8217;`，破折号`—`变为`&#8212;`
- **修订追踪**：在`<w:r>`元素外部使用带有`w:author="Claude"`的`<w:del>`和`<w:ins>`标签
  - **关键**：`<w:ins>`以`</w:ins>`结束，`<w:del>`以`</w:del>`结束 - 切勿混合使用
  - **RSID必须是8位十六进制**：使用类似`00AB1234`的值（仅包含0-9、A-F字符）
  - **trackRevisions位置**：在settings.xml中的`<w:proofState>`之后添加`<w:trackRevisions/>`
- **图像**：添加到`word/media/`，在`document.xml`中引用，设置尺寸以防止溢出

## 文档内容模式

### 基本结构
```xml
<w:p>
  <w:r><w:t>文本内容</w:t></w:r>
</w:p>
```

### 标题和样式
```xml
<w:p>
  <w:pPr>
    <w:pStyle w:val="Title"/>
    <w:jc w:val="center"/>
  </w:pPr>
  <w:r><w:t>文档标题</w:t></w:r>
</w:p>

<w:p>
  <w:pPr><w:pStyle w:val="Heading2"/></w:pPr>
  <w:r><w:t>章节标题</w:t></w:r>
</w:p>
```

### 文本格式化
```xml
<!-- 粗体 -->
<w:r><w:rPr><w:b/><w:bCs/></w:rPr><w:t>粗体</w:t></w:r>
<!-- 斜体 -->
<w:r><w:rPr><w:i/><w:iCs/></w:rPr><w:t>斜体</w:t></w:r>
<!-- 下划线 -->
<w:r><w:rPr><w:u w:val="single"/></w:rPr><w:t>下划线</w:t></w:r>
<!-- 高亮 -->
<w:r><w:rPr><w:highlight w:val="yellow"/></w:rPr><w:t>高亮</w:t></w:r>
```

### 列表
```xml
<!-- 编号列表 -->
<w:p>
  <w:pPr>
    <w:pStyle w:val="ListParagraph"/>
    <w:numPr><w:ilvl w:val="0"/><w:numId w:val="1"/></w:numPr>
    <w:spacing w:before="240"/>
  </w:pPr>
  <w:r><w:t>第一项</w:t></w:r>
</w:p>

<!-- 从1重新开始编号列表 - 使用不同的numId -->
<w:p>
  <w:pPr>
    <w:pStyle w:val="ListParagraph"/>
    <w:numPr><w:ilvl w:val="0"/><w:numId w:val="2"/></w:numPr>
    <w:spacing w:before="240"/>
  </w:pPr>
  <w:r><w:t>新列表项1</w:t></w:r>
</w:p>

<!-- 项目符号列表（第2级） -->
<w:p>
  <w:pPr>
    <w:pStyle w:val="ListParagraph"/>
    <w:numPr><w:ilvl w:val="1"/><w:numId w:val="1"/></w:numPr>
    <w:spacing w:before="240"/>
    <w:ind w:left="900"/>
  </w:pPr>
  <w:r><w:t>项目符号项</w:t></w:r>
</w:p>
```

### 表格
```xml
<w:tbl>
  <w:tblPr>
    <w:tblStyle w:val="TableGrid"/>
    <w:tblW w:w="0" w:type="auto"/>
  </w:tblPr>
  <w:tblGrid>
    <w:gridCol w:w="4675"/><w:gridCol w:w="4675"/>
  </w:tblGrid>
  <w:tr>
    <w:tc>
      <w:tcPr><w:tcW w:w="4675" w:type="dxa"/></w:tcPr>
      <w:p><w:r><w:t>单元格1</w:t></w:r></w:p>
    </w:tc>
    <w:tc>
      <w:tcPr><w:tcW w:w="4675" w:type="dxa"/></w:tcPr>
      <w:p><w:r><w:t>单元格2</w:t></w:r></w:p>
    </w:tc>
  </w:tr>
</w:tbl>
```

### 布局
```xml
<!-- 新章节前的分页符（常见模式） -->
<w:p>
  <w:r>
    <w:br w:type="page"/>
  </w:r>
</w:p>
<w:p>
  <w:pPr>
    <w:pStyle w:val="Heading1"/>
  </w:pPr>
  <w:r>
    <w:t>新章节标题</w:t>
  </w:r>
</w:p>

<!-- 居中对齐段落 -->
<w:p>
  <w:pPr>
    <w:spacing w:before="240" w:after="0"/>
    <w:jc w:val="center"/>
  </w:pPr>
  <w:r><w:t>居中对齐文本</w:t></w:r>
</w:p>

<!-- 字体更改 - 段落级别（应用于所有运行） -->
<w:p>
  <w:pPr>
    <w:rPr><w:rFonts w:ascii="Courier New" w:hAnsi="Courier New"/></w:rPr>
  </w:pPr>
  <w:r><w:t>等宽文本</w:t></w:r>
</w:p>

<!-- 字体更改 - 运行级别（特定于此文本） -->
<w:p>
  <w:r>
    <w:rPr><w:rFonts w:ascii="Courier New" w:hAnsi="Courier New"/></w:rPr>
    <w:t>此文本为Courier New字体</w:t>
  </w:r>
  <w:r><w:t> 此文本使用默认字体</w:t></w:r>
</w:p>
```

## 文件更新

添加内容时，更新这些文件：

**`word/_rels/document.xml.rels`:**
```xml
<Relationship Id="rId1" Type="http://schemas.openxmlformats.org/officeDocument/2006/relationships/numbering" Target="numbering.xml"/>
<Relationship Id="rId5" Type="http://schemas.openxmlformats.org/officeDocument/2006/relationships/image" Target="media/image1.png"/>
```

**`[Content_Types].xml`:**
```xml
<Default Extension="png" ContentType="image/png"/>
<Override PartName="/word/numbering.xml" ContentType="application/vnd.openxmlformats-officedocument.wordprocessingml.numbering+xml"/>
```

### 图像
**关键**：计算尺寸以防止页面溢出并保持宽高比。

```xml
<!-- 最小必需结构 -->
<w:p>
  <w:r>
    <w:drawing>
      <wp:inline>
        <wp:extent cx="2743200" cy="1828800"/>
        <wp:docPr id="1" name="图片1"/>
        <a:graphic xmlns:a="http://schemas.openxmlformats.org/drawingml/2006/main">
          <a:graphicData uri="http://schemas.openxmlformats.org/drawingml/2006/picture">
            <pic:pic xmlns:pic="http://schemas.openxmlformats.org/drawingml/2006/picture">
              <pic:nvPicPr>
                <pic:cNvPr id="0" name="image1.png"/>
                <pic:cNvPicPr/>
              </pic:nvPicPr>
              <pic:blipFill>
                <a:blip r:embed="rId5"/>
                <!-- 添加拉伸填充以保持宽高比 -->
                <a:stretch>
                  <a:fillRect/>
                </a:stretch>
              </pic:blipFill>
              <pic:spPr>
                <a:xfrm>
                  <a:ext cx="2743200" cy="1828800"/>
                </a:xfrm>
                <a:prstGeom prst="rect"/>
              </pic:spPr>
            </pic:pic>
          </a:graphicData>
        </a:graphic>
      </wp:inline>
    </w:drawing>
  </w:r>
</w:p>
```

### 链接（超链接）

**重要**：所有超链接（包括内部和外部）都需要在styles.xml中定义超链接样式。没有此样式，链接将看起来像常规文本而不是蓝色下划线的可点击链接。

**外部链接：**
```xml
<!-- 在document.xml中 -->
<w:hyperlink r:id="rId5">
  <w:r>
    <w:rPr><w:rStyle w:val="Hyperlink"/></w:rPr>
    <w:t>链接文本</w:t>
  </w:r>
</w:hyperlink>

<!-- 在word/_rels/document.xml.rels中 -->
<Relationship Id="rId5" Type="http://schemas.openxmlformats.org/officeDocument/2006/relationships/hyperlink" 
              Target="https://www.example.com/" TargetMode="External"/>
```

**内部链接：**

```xml
<!-- 链接到书签 -->
<w:hyperlink w:anchor="myBookmark">
  <w:r>
    <w:rPr><w:rStyle w:val="Hyperlink"/></w:rPr>
    <w:t>链接文本</w:t>
  </w:r>
</w:hyperlink>

<!-- 书签目标 -->
<w:bookmarkStart w:id="0" w:name="myBookmark"/>
<w:r><w:t>目标内容</w:t></w:r>
<w:bookmarkEnd w:id="0"/>
```

**超链接样式（必需在styles.xml中）：**
```xml
<w:style w:type="character" w:styleId="Hyperlink">
  <w:name w:val="Hyperlink"/>
  <w:basedOn w:val="DefaultParagraphFont"/>
  <w:uiPriority w:val="99"/>
  <w:unhideWhenUsed/>
  <w:rPr>
    <w:color w:val="467886" w:themeColor="hyperlink"/>
    <w:u w:val="single"/>
  </w:rPr>
</w:style>
```

## 核心组件

### 段落 (Paragraph)
```xml
<w:p>
    <w:pPr>
        <w:spacing w:before="200" w:after="200"/>
        <w:ind w:firstLine="400"/>
        <w:jc w:val="center"/>
    </w:pPr>
    <w:r>
        <w:t>段落内容</w:t>
    </w:r>
</w:p>
```

### 文本运行 (Text Run)
```xml
<w:r>
    <w:rPr>
        <w:b/>
        <w:i/>
        <w:color w:val="FF0000"/>
        <w:sz w:val="28"/>
    </w:rPr>
    <w:t>格式化文本</w:t>
</w:r>
```

### 表格 (Table)
```xml
<w:tbl>
    <w:tblPr>
        <w:tblW w:w="5000" w:type="pct"/>
        <w:tblBorders>
            <w:top w:val="single" w:sz="4" w:space="0" w:color="000000"/>
            <w:left w:val="single" w:sz="4" w:space="0" w:color="000000"/>
            <w:bottom w:val="single" w:sz="4" w:space="0" w:color="000000"/>
            <w:right w:val="single" w:sz="4" w:space="0" w:color="000000"/>
        </w:tblBorders>
    </w:tblPr>
    <w:tr>
        <w:tc>
            <w:p><w:r><w:t>单元格1</w:t></w:r></w:p>
        </w:tc>
        <w:tc>
            <w:p><w:r><w:t>单元格2</w:t></w:r></w:p>
        </w:tc>
    </w:tr>
</w:tbl>
```

## 文档库（Python）

使用`scripts/document.py`中的Document类处理所有修订追踪和注释。它会自动处理基础设施设置（people.xml、RSIDs、settings.xml、注释文件、关系、内容类型）。仅在库不支持的复杂场景中使用直接XML操作。

**处理Unicode和实体：**
- **搜索**：实体表示法和Unicode字符都有效 - `contains="&#8220;Company"`和`contains="\u201cCompany"`找到相同的文本
- **替换**：使用实体（`&#8220;`）或Unicode（`\u201c`） - 两者都有效，将根据文件的编码进行适当转换（ascii → 实体，utf-8 → Unicode）

### 初始化

**查找docx技能根目录**（包含`scripts/`和`ooxml/`的目录）：
```bash
# 搜索document.py以定位技能根目录
# 注意：/mnt/skills在此处用作示例；检查您的上下文以获取实际位置
find /mnt/skills -name "document.py" -path "*/docx/scripts/*" 2>/dev/null | head -1
# 示例输出：/mnt/skills/docx/scripts/document.py
# 技能根目录是：/mnt/skills/docx
```

**使用PYTHONPATH设置为docx技能根目录运行脚本：**
```bash
PYTHONPATH=/mnt/skills/docx python your_script.py
```

**在脚本中**，从技能根目录导入：
```python
from scripts.document import Document, DocxXMLEditor

# 基本初始化（自动创建临时副本并设置基础设施）
doc = Document('unpacked')

# 自定义作者和缩写
doc = Document('unpacked', author="张三", initials="ZS")

# 启用修订追踪模式
doc = Document('unpacked', track_revisions=True)

# 指定自定义RSID（如果未提供则自动生成）
doc = Document('unpacked', rsid="07DC5ECB")
```

### 创建修订追踪

**关键**：仅标记实际更改的文本。将所有未更改的文本保留在`<w:del>`/`<w:ins>`标签外部。标记未更改的文本会使编辑不专业且更难审查。

**属性处理**：Document类自动将属性（w:id、w:date、w:rsidR、w:rsidDel、w16du:dateUtc、xml:space）注入新元素。当保留原始文档中的未更改文本时，复制原始的`<w:r>`元素及其现有属性以维护文档完整性。

**方法选择指南**：
- **将您自己的更改添加到常规文本**：使用带有`<w:del>`/`<w:ins>`标签的`replace_node()`，或使用`suggest_deletion()`删除整个`<w:r>`或`<w:p>`元素
- **部分修改其他作者的修订追踪**：使用`replace_node()`将您的更改嵌套在他们的`<w:ins>`/`<w:del>`内部
- **完全拒绝其他作者的插入**：在`<w:ins>`元素上使用`revert_insertion()`（不是`suggest_deletion()`）
- **完全拒绝其他作者的删除**：在`<w:del>`元素上使用`revert_deletion()`以使用修订追踪恢复已删除的内容

```python
# 最小编辑 - 更改一个词："报告是月度的" → "报告是季度的"
# 原始：<w:r w:rsidR="00AB12CD"><w:rPr><w:rFonts w:ascii="Calibri"/></w:rPr><w:t>报告是月度的</w:t></w:r>
node = doc["word/document.xml"].get_node(tag="w:r", contains="报告是月度的")
rpr = tags[0].toxml() if (tags := node.getElementsByTagName("w:rPr")) else ""
replacement = f'<w:r w:rsidR="00AB12CD">{rpr}<w:t>报告是 </w:t></w:r><w:del><w:r>{rpr}<w:delText>月度的</w:delText></w:r></w:del><w:ins><w:r>{rpr}<w:t>季度的</w:t></w:r></w:ins>'
doc["word/document.xml"].replace_node(node, replacement)

# 最小编辑 - 更改数字："在30天内" → "在45天内"
# 原始：<w:r w:rsidR="00XYZ789"><w:rPr><w:rFonts w:ascii="Calibri"/></w:rPr><w:t>在30天内</w:t></w:r>
node = doc["word/document.xml"].get_node(tag="w:r", contains="在30天内")
rpr = tags[0].toxml() if (tags := node.getElementsByTagName("w:rPr")) else ""
replacement = f'<w:r w:rsidR="00XYZ789">{rpr}<w:t>在 </w:t></w:r><w:del><w:r>{rpr}<w:delText>30</w:delText></w:r></w:del><w:ins><w:r>{rpr}<w:t>45</w:t></w:r></w:ins><w:r w:rsidR="00XYZ789">{rpr}<w:t> 天内</w:t></w:r>'
doc["word/document.xml"].replace_node(node, replacement)

# 完全替换 - 即使替换所有文本也保留格式
node = doc["word/document.xml"].get_node(tag="w:r", contains="苹果")
rpr = tags[0].toxml() if (tags := node.getElementsByTagName("w:rPr")) else ""
replacement = f'<w:del><w:r>{rpr}<w:delText>苹果</w:delText></w:r></w:del><w:ins><w:r>{rpr}<w:t>香蕉 橙子</w:t></w:r></w:ins>'
doc["word/document.xml"].replace_node(node, replacement)

# 插入新内容（不需要属性 - 自动注入）
node = doc["word/document.xml"].get_node(tag="w:r", contains="现有文本")
doc["word/document.xml"].insert_after(node, '<w:ins><w:r><w:t>新文本</w:t></w:r></w:ins>')

# 部分删除其他作者的插入
# 原始：<w:ins w:author="李四" w:date="..."><w:r><w:t>季度财务报告</w:t></w:r></w:ins>
# 目标：仅删除"财务"以使其成为"季度报告"
node = doc["word/document.xml"].get_node(tag="w:ins", attrs={"w:id": "5"})
# 重要：保留外部<w:ins>上的w:author="李四"以维护作者身份
replacement = '''<w:ins w:author="李四" w:date="2025-01-15T10:00:00Z">
  <w:r><w:t>季度 </w:t></w:r>
  <w:del><w:r><w:delText>财务 </w:delText></w:r></w:del>
  <w:r><w:t>报告</w:t></w:r>
</w:ins>'''
doc["word/document.xml"].replace_node(node, replacement)

# 更改其他作者插入的部分内容
# 原始：<w:ins w:author="李四"><w:r><w:t>在沉默中，安全无恙</w:t></w:r></w:ins>
# 目标：将"安全无恙"更改为"柔软无界"
node = doc["word/document.xml"].get_node(tag="w:ins", attrs={"w:id": "8"})
replacement = f'''<w:ins w:author="李四" w:date="2025-01-15T10:00:00Z">
  <w:r><w:t>在沉默中， </w:t></w:r>
</w:ins>
<w:ins>
  <w:r><w:t>柔软无界</w:t></w:r>
</w:ins>
<w:ins w:author="李四" w:date="2025-01-15T10:00:00Z">
  <w:del><w:r><w:delText>安全无恙</w:delText></w:r></w:del>
</w:ins>'''
doc["word/document.xml"].replace_node(node, replacement)

# 删除整个运行（仅在删除所有内容时使用；部分删除使用replace_node）
node = doc["word/document.xml"].get_node(tag="w:r", contains="要删除的文本")
doc["word/document.xml"].suggest_deletion(node)

# 删除整个段落（原地处理，处理常规和编号列表段落）
para = doc["word/document.xml"].get_node(tag="w:p", contains="要删除的段落")
doc["word/document.xml"].suggest_deletion(para)

# 添加新编号列表项
target_para = doc["word/document.xml"].get_node(tag="w:p", contains="现有列表项")
pPr = tags[0].toxml() if (tags := target_para.getElementsByTagName("w:pPr")) else ""
new_item = f'<w:p>{pPr}<w:r><w:t>新项</w:t></w:r></w:p>'
tracked_para = DocxXMLEditor.suggest_paragraph(new_item)
doc["word/document.xml"].insert_after(target_para, tracked_para)
# 可选：在内容前添加间距段落以获得更好的视觉分离
# spacing = DocxXMLEditor.suggest_paragraph('<w:p><w:pPr><w:pStyle w:val="ListParagraph"/></w:pPr></w:p>')
# doc["word/document.xml"].insert_after(target_para, spacing + tracked_para)
```

### 添加注释

```python
# 添加跨越两个现有修订追踪的注释
# 注意：w:id是自动生成的。仅在通过XML检查知道w:id时按w:id搜索
start_node = doc["word/document.xml"].get_node(tag="w:del", attrs={"w:id": "1"})
end_node = doc["word/document.xml"].get_node(tag="w:ins", attrs={"w:id": "2"})
doc.add_comment(start=start_node, end=end_node, text="对此更改的解释")

# 在段落上添加注释
para = doc["word/document.xml"].get_node(tag="w:p", contains="段落文本")
doc.add_comment(start=para, end=para, text="对此段落的注释")

# 在新创建的修订追踪上添加注释
# 首先创建修订追踪
node = doc["word/document.xml"].get_node(tag="w:r", contains="旧")
new_nodes = doc["word/document.xml"].replace_node(
    node,
    '<w:del><w:r><w:delText>旧</w:delText></w:r></w:del><w:ins><w:r><w:t>新</w:t></w:r></w:ins>'
)
# 然后在新建的元素上添加注释
# new_nodes[0]是<w:del>，new_nodes[1]是<w:ins>
doc.add_comment(start=new_nodes[0], end=new_nodes[1], text="根据要求将旧更改为新")

# 回复现有注释
doc.reply_to_comment(parent_comment_id=0, text="我同意此更改")
```

### 拒绝修订追踪

**重要**：使用`revert_insertion()`拒绝插入，使用`revert_deletion()`使用修订追踪恢复删除。仅在常规未标记内容上使用`suggest_deletion()`。

```python
# 拒绝插入（将其包装在删除中）
# 当其他作者插入了您想要删除的文本时使用此方法
ins = doc["word/document.xml"].get_node(tag="w:ins", attrs={"w:id": "5"})
nodes = doc["word/document.xml"].revert_insertion(ins)  # 返回[ins]

# 拒绝删除（创建插入以恢复已删除的内容）
# 当其他作者删除了您想要恢复的文本时使用此方法
del_elem = doc["word/document.xml"].get_node(tag="w:del", attrs={"w:id": "3"})
nodes = doc["word/document.xml"].revert_deletion(del_elem)  # 返回[del_elem, new_ins]

# 拒绝段落中的所有插入
para = doc["word/document.xml"].get_node(tag="w:p", contains="段落文本")
nodes = doc["word/document.xml"].revert_insertion(para)  # 返回[para]

# 拒绝段落中的所有删除
para = doc["word/document.xml"].get_node(tag="w:p", contains="段落文本")
nodes = doc["word/document.xml"].revert_deletion(para)  # 返回[para]
```

### 插入图像

**关键**：Document类使用`doc.unpacked_path`处的临时副本。始终将图像复制到此临时目录，而不是原始解压缩文件夹。

```python
from PIL import Image
import shutil, os

# 首先初始化文档
doc = Document('unpacked')

# 复制图像并计算全宽尺寸，保持宽高比
media_dir = os.path.join(doc.unpacked_path, 'word/media')
os.makedirs(media_dir, exist_ok=True)
shutil.copy('image.png', os.path.join(media_dir, 'image1.png'))
img = Image.open(os.path.join(media_dir, 'image1.png'))
width_emus = int(6.5 * 914400)  # 6.5"可用宽度，914400 EMUs/英寸
height_emus = int(width_emus * img.size[1] / img.size[0])

# 添加关系和内容类型
rels_editor = doc['word/_rels/document.xml.rels']
next_rid = rels_editor.get_next_rid()
rels_editor.append_to(rels_editor.dom.documentElement,
    f'<Relationship Id="{next_rid}" Type="http://schemas.openxmlformats.org/officeDocument/2006/relationships/image" Target="media/image1.png"/>')
doc['[Content_Types].xml'].append_to(doc['[Content_Types].xml'].dom.documentElement,
    '<Default Extension="png" ContentType="image/png"/>')

# 插入图像
node = doc["word/document.xml"].get_node(tag="w:p", line_number=100)
doc["word/document.xml"].insert_after(node, f'''<w:p>
  <w:r>
    <w:drawing>
      <wp:inline distT="0" distB="0" distL="0" distR="0">
        <wp:extent cx="{width_emus}" cy="{height_emus}"/>
        <wp:docPr id="1" name="图片1"/>
        <a:graphic xmlns:a="http://schemas.openxmlformats.org/drawingml/2006/main">
          <a:graphicData uri="http://schemas.openxmlformats.org/drawingml/2006/picture">
            <pic:pic xmlns:pic="http://schemas.openxmlformats.org/drawingml/2006/picture">
              <pic:nvPicPr><pic:cNvPr id="1" name="image1.png"/><pic:cNvPicPr/></pic:nvPicPr>
              <pic:blipFill><a:blip r:embed="{next_rid}"/><a:stretch><a:fillRect/></a:stretch></pic:blipFill>
              <pic:spPr><a:xfrm><a:ext cx="{width_emus}" cy="{height_emus}"/></a:xfrm><a:prstGeom prst="rect"><a:avLst/></a:prstGeom></pic:spPr>
            </pic:pic>
          </a:graphicData>
        </a:graphic>
      </wp:inline>
    </w:drawing>
  </w:r>
</w:p>''')
```

### 获取节点

```python
# 按文本内容
node = doc["word/document.xml"].get_node(tag="w:p", contains="特定文本")

# 按行范围
para = doc["word/document.xml"].get_node(tag="w:p", line_number=range(100, 150))

# 按属性
node = doc["word/document.xml"].get_node(tag="w:del", attrs={"w:id": "1"})

# 按精确行号（必须是标签打开的行号）
para = doc["word/document.xml"].get_node(tag="w:p", line_number=42)

# 组合过滤器
node = doc["word/document.xml"].get_node(tag="w:r", line_number=range(40, 60), contains="文本")

# 当文本多次出现时消除歧义 - 添加line_number范围
node = doc["word/document.xml"].get_node(tag="w:r", contains="章节", line_number=range(2400, 2500))
```

### 保存

```python
# 使用自动验证保存（复制回原始目录）
doc.save()  # 默认验证，如果验证失败则引发错误

# 保存到不同位置
doc.save('modified-unpacked')

# 跳过验证（仅用于调试 - 在生产中需要此功能表明存在XML问题）
doc.save(validate=False)
```

### 直接DOM操作

对于库未涵盖的复杂场景：

```python
# 访问任何XML文件
editor = doc["word/document.xml"]
editor = doc["word/comments.xml"]

# 直接DOM访问（defusedxml.minidom.Document）
node = doc["word/document.xml"].get_node(tag="w:p", line_number=5)
parent = node.parentNode
parent.removeChild(node)
parent.appendChild(node)  # 移动到末尾

# 常规文档操作（无修订追踪）
old_node = doc["word/document.xml"].get_node(tag="w:p", contains="原始文本")
doc["word/document.xml"].replace_node(old_node, "<w:p><w:r><w:t>替换文本</w:t></w:r></w:p>")

# 多次插入 - 使用返回值维护顺序
node = doc["word/document.xml"].get_node(tag="w:r", line_number=100)
nodes = doc["word/document.xml"].insert_after(node, "<w:r><w:t>A</w:t></w:r>")
nodes = doc["word/document.xml"].insert_after(nodes[-1], "<w:r><w:t>B</w:t></w:r>")
nodes = doc["word/document.xml"].insert_after(nodes[-1], "<w:r><w:t>C</w:t></w:r>")
# 结果：original_node, A, B, C

### 样式操作
```python
from docx import Document
from docx.shared import Pt
from docx.enum.text import WD_ALIGN_PARAGRAPH

# 创建文档
doc = Document()

# 添加带样式的段落
p = doc.add_paragraph()
p.alignment = WD_ALIGN_PARAGRAPH.CENTER

run = p.add_run('居中文本')
run.bold = True
run.font.size = Pt(16)

# 添加编号列表
doc.add_paragraph('第一项', style='List Number')
doc.add_paragraph('第二项', style='List Number')
doc.add_paragraph('第三项', style='List Number')

# 添加项目符号列表
doc.add_paragraph('项目A', style='List Bullet')
doc.add_paragraph('项目B', style='List Bullet')

doc.save('styled_document.docx')
```

## 修订追踪

**使用上述Document类处理所有修订追踪。** 下面的模式用于构建替换XML字符串时的参考。

### 验证规则
验证器检查在还原Claude的更改后文档文本是否与原始文本匹配。这意味着：
- **永远不要修改其他作者的`<w:ins>`或`<w:del>`标签内的文本**
- **始终使用嵌套删除**来移除其他作者的插入
- **每个编辑都必须使用`<w:ins>`或`<w:del>`标签进行适当追踪**

### 修订追踪模式

**关键规则**：
1. 永远不要修改其他作者修订追踪内的内容。始终使用嵌套删除。
2. **XML结构**：始终将`<w:del>`和`<w:ins>`放置在段落级别，包含完整的`<w:r>`元素。永远不要嵌套在`<w:r>`元素内部 - 这会创建无效的XML，破坏文档处理。

**文本插入：**
```xml
<w:ins w:id="1" w:author="Claude" w:date="2025-07-30T23:05:00Z" w16du:dateUtc="2025-07-31T06:05:00Z">
  <w:r w:rsidR="00792858">
    <w:t>插入的文本</w:t>
  </w:r>
</w:ins>
```

**文本删除：**
```xml
<w:del w:id="2" w:author="Claude" w:date="2025-07-30T23:05:00Z" w16du:dateUtc="2025-07-31T06:05:00Z">
  <w:r w:rsidDel="00792858">
    <w:delText>删除的文本</w:delText>
  </w:r>
</w:del>
```

**删除其他作者的插入（必须使用嵌套结构）：**
```xml
<!-- 在原始插入内部嵌套删除 -->
<w:ins w:author="Jane Smith" w:id="16">
  <w:del w:author="Claude" w:id="40">
    <w:r><w:delText>每月</w:delText></w:r>
  </w:del>
</w:ins>
<w:ins w:author="Claude" w:id="41">
  <w:r><w:t>每周</w:t></w:r>
</w:ins>
```

**恢复其他作者的删除：**
```xml
<!-- 保持他们的删除不变，在其后添加新的插入 -->
<w:del w:author="Jane Smith" w:id="50">
  <w:r><w:delText>30天内</w:delText></w:r>
</w:del>
<w:ins w:author="Claude" w:id="51">
  <w:r><w:t>30天内</w:t></w:r>
</w:ins>
```

### 修订标记
```xml
<w:p>
    <w:pPr>
        <w:rPr>
            <w:ins w:id="1" w:author="User1" w:date="2023-01-01T00:00:00Z"/>
        </w:rPr>
    </w:pPr>
    <w:r>
        <w:t>插入的文本</w:t>
    </w:r>
</w:p>

<w:p>
    <w:pPr>
        <w:rPr>
            <w:del w:id="2" w:author="User2" w:date="2023-01-02T00:00:00Z"/>
        </w:rPr>
    </w:pPr>
    <w:delText>删除的文本</w:delText>
</w:p>
```

### Python中的修订处理
```python
from docx import Document
from docx.opc.constants import RELATIONSHIP_TYPE as RT

# 检查文档是否包含修订
def has_revisions(docx_path):
    doc = Document(docx_path)
    
    # 检查修订关系
    for rel in doc.part.rels.values():
        if rel.reltype == RT.TRACKED_REVISION:
            return True
    
    return False

# 提取修订信息
def extract_revisions(docx_path):
    doc = Document(docx_path)
    revisions = []
    
    # 遍历文档内容查找修订
    for paragraph in doc.paragraphs:
        for run in paragraph.runs:
            # 检查插入修订
            if hasattr(run, '_r') and run._r.ins:
                revision = {
                    'type': 'insertion',
                    'author': run._r.ins.author,
                    'date': run._r.ins.date,
                    'text': run.text
                }
                revisions.append(revision)
            
            # 检查删除修订
            if hasattr(run, '_r') and run._r.del_:
                revision = {
                    'type': 'deletion',
                    'author': run._r.del_.author,
                    'date': run._r.del_.date,
                    'text': run.text
                }
                revisions.append(revision)
    
    return revisions
```

## 高级操作

### 自定义XML部件
```python
from docx import Document
from docx.oxml import parse_xml

# 创建文档
doc = Document()

# 添加自定义XML数据
custom_xml = '''
<customData>
    <metadata>
        <source>AI生成</source>
        <version>1.0</version>
    </metadata>
    <content>
        <section id="1">主要内容</section>
    </content>
</customData>
'''

# 将自定义XML添加到文档
part = doc.part
custom_part = part.add_part('application/xml', 'customXml/item1.xml')
custom_part._blob = custom_xml.encode('utf-8')

# 添加关系
part.rels.add_relationship(
    'http://schemas.openxmlformats.org/officeDocument/2006/relationships/customXml',
    custom_part,
    'customXml/item1.xml'
)

doc.save('document_with_custom_xml.docx')
```

### 处理复杂表格
```python
from docx import Document
from docx.shared import Inches

# 创建复杂表格
def create_complex_table(doc):
    # 创建5x5表格
    table = doc.add_table(rows=5, cols=5)
    
    # 设置表格样式
    table.style = 'Table Grid'
    
    # 合并单元格
    # 合并第一行的前两个单元格
    table.cell(0, 0).merge(table.cell(0, 1))
    table.cell(0, 0).text = "合并的标题"
    
    # 设置单元格格式
    for i in range(5):
        for j in range(5):
            cell = table.cell(i, j)
            
            # 设置交替行颜色
            if i % 2 == 0:
                for paragraph in cell.paragraphs:
                    for run in paragraph.runs:
                        run.font.color.rgb = (0, 0, 128)  # 深蓝色
            
            # 设置表头格式
            if i == 0:
                for paragraph in cell.paragraphs:
                    for run in paragraph.runs:
                        run.bold = True

# 使用函数
doc = Document()
create_complex_table(doc)
doc.save('complex_table.docx')
```

### 图像处理
```python
from docx import Document
from docx.shared import Inches

# 添加图像
def add_images_to_document(doc, image_paths):
    for i, image_path in enumerate(image_paths):
        # 添加图像标题
        doc.add_paragraph(f'图像 {i+1}')
        
        # 添加图像
        doc.add_picture(image_path, width=Inches(4))
        
        # 添加图像描述
        doc.add_paragraph(f'这是图像 {i+1} 的描述。')
        
        # 添加分页符（除了最后一个图像）
        if i < len(image_paths) - 1:
            doc.add_page_break()

# 使用函数
doc = Document()
image_paths = ['image1.jpg', 'image2.png', 'image3.gif']
add_images_to_document(doc, image_paths)
doc.save('document_with_images.docx')
```

### 文档元数据操作
```python
from docx import Document
from datetime import datetime

# 设置文档属性
def set_document_properties(doc, properties):
    core_props = doc.core_properties
    
    core_props.title = properties.get('title', '')
    core_props.subject = properties.get('subject', '')
    core_props.creator = properties.get('creator', '')
    core_props.description = properties.get('description', '')
    core_props.keywords = properties.get('keywords', '')
    core_props.last_modified_by = properties.get('last_modified_by', '')
    core_props.revision = properties.get('revision', 1)
    core_props.created = properties.get('created', datetime.now())
    core_props.modified = datetime.now()

# 使用函数
doc = Document()

properties = {
    'title': '技术文档',
    'subject': 'OOXML技术参考',
    'creator': 'AI助手',
    'description': '关于Office Open XML格式的技术文档',
    'keywords': 'OOXML, DOCX, XML, Office',
    'last_modified_by': '系统管理员'
}

set_document_properties(doc, properties)
doc.add_paragraph('文档内容...')
doc.save('document_with_properties.docx')
```

## 最佳实践

### 性能优化
- **批量操作**：尽量减少对文档的频繁读写
- **内存管理**：处理大型文档时注意内存使用
- **缓存策略**：对重复使用的样式和格式进行缓存

### 错误处理
```python
import zipfile
from docx import Document
from docx.exceptions import PackageNotFoundError

def safe_open_document(file_path):
    try:
        # 检查是否为有效的ZIP文件
        with zipfile.ZipFile(file_path, 'r') as zip_file:
            pass
        
        # 打开文档
        doc = Document(file_path)
        return doc
        
    except zipfile.BadZipFile:
        print("错误：文件不是有效的ZIP文件")
        return None
    except PackageNotFoundError:
        print("错误：文件不是有效的DOCX文档")
        return None
    except Exception as e:
        print(f"错误：{str(e)}")
        return None
```

### 兼容性考虑
- **版本兼容**：在不同版本的Word中测试生成的文档
- **字体可用性**：使用跨平台兼容的字体
- **Unicode支持**：确保正确处理多语言文本

通过本指南，您可以深入理解OOXML格式的结构，并使用Python库高效地创建和处理DOCX文档。