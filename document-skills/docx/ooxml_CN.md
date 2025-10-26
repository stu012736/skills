# Office Open XML (OOXML) 技术参考

本指南提供Office Open XML (OOXML) 格式的技术参考，重点关注DOCX文档的结构和操作。

## 目录

1. [OOXML概述](#ooxml概述)
2. [文档结构](#文档结构)
3. [核心组件](#核心组件)
4. [Python文档库](#python文档库)
5. [修订追踪](#修订追踪)
6. [高级操作](#高级操作)

---

## OOXML概述

Office Open XML (OOXML) 是Microsoft Office文档的国际标准格式（ISO/IEC 29500）。它使用基于XML的开放格式来存储文档内容、格式和元数据。

### 主要特点
- **开放标准**：ISO/IEC 29500标准
- **基于XML**：使用XML格式存储内容
- **模块化结构**：文档由多个XML文件组成
- **向后兼容**：与现有Office文档兼容

### 文件格式
- **.docx**：Word文档
- **.xlsx**：Excel工作簿
- **.pptx**：PowerPoint演示文稿

## 文档结构

### ZIP容器结构
OOXML文档实际上是ZIP压缩文件，包含多个XML文件和资源：

```
my_document.docx
├── [Content_Types].xml
├── _rels/
│   └── .rels
├── docProps/
│   ├── app.xml
│   └── core.xml
├── word/
│   ├── document.xml
│   ├── styles.xml
│   ├── numbering.xml
│   ├── fontTable.xml
│   ├── _rels/
│   │   └── document.xml.rels
│   └── theme/
│       └── theme1.xml
└── customXml/
    └── item1.xml
```

### 核心XML文件

#### [Content_Types].xml
定义文档中所有部分的内容类型：
```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<Types xmlns="http://schemas.openxmlformats.org/package/2006/content-types">
    <Default Extension="rels" ContentType="application/vnd.openxmlformats-package.relationships+xml"/>
    <Default Extension="xml" ContentType="application/xml"/>
    <Override PartName="/word/document.xml" ContentType="application/vnd.openxmlformats-officedocument.wordprocessingml.document.main+xml"/>
    <Override PartName="/word/styles.xml" ContentType="application/vnd.openxmlformats-officedocument.wordprocessingml.styles+xml"/>
</Types>
```

#### word/document.xml
文档主要内容：
```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<w:document xmlns:w="http://schemas.openxmlformats.org/wordprocessingml/2006/main">
    <w:body>
        <w:p>
            <w:r>
                <w:t>文档内容</w:t>
            </w:r>
        </w:p>
    </w:body>
</w:document>
```

#### word/styles.xml
文档样式定义：
```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<w:styles xmlns:w="http://schemas.openxmlformats.org/wordprocessingml/2006/main">
    <w:docDefaults>
        <w:rPrDefault>
            <w:rPr>
                <w:rFonts w:ascii="Calibri" w:hAnsi="Calibri"/>
                <w:sz w:val="24"/>
            </w:rPr>
        </w:rPrDefault>
    </w:docDefaults>
    <w:style w:type="paragraph" w:styleId="Normal">
        <w:name w:val="Normal"/>
        <w:qFormat/>
    </w:style>
</w:styles>
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

## Python文档库

### python-docx库
```python
from docx import Document
from docx.shared import Inches

# 创建新文档
doc = Document()

# 添加标题
doc.add_heading('文档标题', 0)

# 添加段落
p = doc.add_paragraph('这是一个段落。')

# 添加格式化文本
p.add_run('粗体文本').bold = True
p.add_run('和').bold = False
p.add_run('斜体文本').italic = True

# 添加表格
table = doc.add_table(rows=3, cols=3)
for i in range(3):
    for j in range(3):
        table.cell(i, j).text = f'单元格 {i+1},{j+1}'

# 保存文档
doc.save('example.docx')
```

### 读取现有文档
```python
from docx import Document

# 打开现有文档
doc = Document('existing.docx')

# 读取所有段落
for paragraph in doc.paragraphs:
    print(paragraph.text)

# 读取所有表格
for table in doc.tables:
    for row in table.rows:
        for cell in row.cells:
            print(cell.text)
```

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