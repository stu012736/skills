# PDF操作指南

本指南提供PDF文档的创建、编辑、分析和操作的全面指导。

## 目录

1. [概述](#概述)
2. [Python库](#python库)
3. [命令行工具](#命令行工具)
4. [常见任务](#常见任务)
5. [高级功能](#高级功能)
6. [最佳实践](#最佳实践)

---

## 概述

PDF（便携式文档格式）是一种广泛使用的文档格式，具有跨平台兼容性和格式保持能力。本指南涵盖PDF文档的各种操作，包括创建、编辑、内容提取和分析。

### 主要特点
- **格式保持**：文档格式在不同平台上保持一致
- **安全性**：支持加密和权限控制
- **可访问性**：支持文本提取和屏幕阅读器
- **交互性**：支持表单、链接和多媒体

## Python库

### PyPDF2/PyPDF4
```python
import PyPDF2

# 读取PDF文件
with open('document.pdf', 'rb') as file:
    pdf_reader = PyPDF2.PdfReader(file)
    
    # 获取页面数量
    num_pages = len(pdf_reader.pages)
    print(f"文档包含 {num_pages} 页")
    
    # 提取文本内容
    for page_num in range(num_pages):
        page = pdf_reader.pages[page_num]
        text = page.extract_text()
        print(f"第 {page_num + 1} 页内容：")
        print(text[:200])  # 显示前200个字符
```

### pdfplumber
```python
import pdfplumber

# 高级PDF分析
with pdfplumber.open('document.pdf') as pdf:
    # 获取文档信息
    print(f"页面数量：{len(pdf.pages)}")
    
    # 分析每一页
    for page in pdf.pages:
        # 提取文本
        text = page.extract_text()
        print(f"页面文本：{text[:100]}...")
        
        # 提取表格
        tables = page.extract_tables()
        for table in tables:
            print("表格数据：", table)
        
        # 提取图像
        images = page.images
        for img in images:
            print(f"图像位置：{img['x0']}, {img['y0']}, {img['x1']}, {img['y1']}")
```

### reportlab（PDF生成）
```python
from reportlab.lib.pagesizes import letter
from reportlab.pdfgen import canvas
from reportlab.lib.styles import getSampleStyleSheet
from reportlab.platypus import SimpleDocTemplate, Paragraph, Spacer

# 创建PDF文档
def create_pdf(filename, content):
    doc = SimpleDocTemplate(filename, pagesize=letter)
    styles = getSampleStyleSheet()
    
    # 构建文档内容
    story = []
    
    # 添加标题
    title = Paragraph("PDF文档标题", styles['Title'])
    story.append(title)
    story.append(Spacer(1, 12))
    
    # 添加内容
    for paragraph_text in content:
        para = Paragraph(paragraph_text, styles['Normal'])
        story.append(para)
        story.append(Spacer(1, 6))
    
    # 生成PDF
    doc.build(story)
    print(f"PDF文件已创建：{filename}")

# 使用函数
content = [
    "这是第一个段落。",
    "这是第二个段落，包含一些示例文本。",
    "第三个段落展示了PDF生成功能。"
]
create_pdf('generated_document.pdf', content)
```

## 命令行工具

### pdftotext（文本提取）
```bash
# 提取PDF文本到文件
pdftotext document.pdf output.txt

# 提取特定页面
pdftotext -f 1 -l 3 document.pdf pages_1-3.txt

# 保留布局
pdftotext -layout document.pdf formatted_output.txt
```

### pdfinfo（文档信息）
```bash
# 获取PDF文档信息
pdfinfo document.pdf

# 输出格式化的信息
pdfinfo -meta document.pdf
```

### qpdf（PDF操作）
```bash
# 解密PDF
qpdf --decrypt encrypted.pdf decrypted.pdf

# 合并多个PDF
qpdf --empty --pages file1.pdf file2.pdf file3.pdf -- merged.pdf

# 分割PDF
qpdf document.pdf --pages . 1-5 -- part1.pdf
qpdf document.pdf --pages . 6-10 -- part2.pdf
```

### Ghostscript（PDF处理）
```bash
# 压缩PDF
gs -sDEVICE=pdfwrite -dCompatibilityLevel=1.4 -dPDFSETTINGS=/ebook -dNOPAUSE -dQUIET -dBATCH -sOutputFile=compressed.pdf input.pdf

# 转换为图像
gs -dNOPAUSE -sDEVICE=jpeg -r300 -sOutputFile=page-%d.jpg document.pdf

# 调整页面大小
gs -sDEVICE=pdfwrite -dDEVICEWIDTHPOINTS=612 -dDEVICEHEIGHTPOINTS=792 -dFIXEDMEDIA -dPDFFitPage -sOutputFile=resized.pdf input.pdf
```

## 常见任务

### PDF合并
```python
import PyPDF2

def merge_pdfs(output_path, input_paths):
    """合并多个PDF文件"""
    pdf_writer = PyPDF2.PdfWriter()
    
    for path in input_paths:
        pdf_reader = PyPDF2.PdfReader(path)
        for page in pdf_reader.pages:
            pdf_writer.add_page(page)
    
    with open(output_path, 'wb') as out:
        pdf_writer.write(out)
    
    print(f"PDF文件已合并到：{output_path}")

# 使用函数
input_files = ['file1.pdf', 'file2.pdf', 'file3.pdf']
merge_pdfs('merged_document.pdf', input_files)
```

### PDF分割
```python
import PyPDF2

def split_pdf(input_path, output_dir, pages_per_file=10):
    """分割PDF文件"""
    pdf_reader = PyPDF2.PdfReader(input_path)
    total_pages = len(pdf_reader.pages)
    
    for i in range(0, total_pages, pages_per_file):
        pdf_writer = PyPDF2.PdfWriter()
        start_page = i
        end_page = min(i + pages_per_file, total_pages)
        
        for page_num in range(start_page, end_page):
            pdf_writer.add_page(pdf_reader.pages[page_num])
        
        output_path = f"{output_dir}/part_{i//pages_per_file + 1}.pdf"
        with open(output_path, 'wb') as out:
            pdf_writer.write(out)
        
        print(f"创建分割文件：{output_path}")

# 使用函数
split_pdf('large_document.pdf', 'split_parts', pages_per_file=5)
```

### 文本提取和分析
```python
import pdfplumber
import re

def analyze_pdf_content(pdf_path):
    """分析PDF内容"""
    results = {
        'page_count': 0,
        'total_text_length': 0,
        'tables_found': 0,
        'images_found': 0,
        'keywords': {}
    }
    
    with pdfplumber.open(pdf_path) as pdf:
        results['page_count'] = len(pdf.pages)
        
        for page_num, page in enumerate(pdf.pages):
            # 提取文本
            text = page.extract_text()
            if text:
                results['total_text_length'] += len(text)
                
                # 关键词分析
                words = re.findall(r'\b\w{4,}\b', text.lower())
                for word in words:
                    results['keywords'][word] = results['keywords'].get(word, 0) + 1
            
            # 提取表格
            tables = page.extract_tables()
            results['tables_found'] += len(tables)
            
            # 提取图像
            images = page.images
            results['images_found'] += len(images)
    
    # 排序关键词
    results['top_keywords'] = sorted(
        results['keywords'].items(), 
        key=lambda x: x[1], 
        reverse=True
    )[:10]
    
    return results

# 使用函数
analysis = analyze_pdf_content('document.pdf')
print("PDF分析结果：")
for key, value in analysis.items():
    print(f"{key}: {value}")
```

### PDF加密和解密
```python
import PyPDF2

def encrypt_pdf(input_path, output_path, password):
    """加密PDF文件"""
    pdf_reader = PyPDF2.PdfReader(input_path)
    pdf_writer = PyPDF2.PdfWriter()
    
    # 复制所有页面
    for page in pdf_reader.pages:
        pdf_writer.add_page(page)
    
    # 设置密码
    pdf_writer.encrypt(password)
    
    with open(output_path, 'wb') as out:
        pdf_writer.write(out)
    
    print(f"PDF文件已加密：{output_path}")

def decrypt_pdf(input_path, output_path, password):
    """解密PDF文件"""
    try:
        pdf_reader = PyPDF2.PdfReader(input_path)
        if pdf_reader.is_encrypted:
            pdf_reader.decrypt(password)
        
        pdf_writer = PyPDF2.PdfWriter()
        for page in pdf_reader.pages:
            pdf_writer.add_page(page)
        
        with open(output_path, 'wb') as out:
            pdf_writer.write(out)
        
        print(f"PDF文件已解密：{output_path}")
        return True
    except Exception as e:
        print(f"解密失败：{str(e)}")
        return False

# 使用函数
encrypt_pdf('document.pdf', 'encrypted.pdf', 'mypassword')
decrypt_pdf('encrypted.pdf', 'decrypted.pdf', 'mypassword')
```

## 高级功能

### OCR文本识别
```python
import pytesseract
from pdf2image import convert_from_path
import cv2

def pdf_ocr(pdf_path, output_text_path):
    """对PDF进行OCR识别"""
    # 将PDF转换为图像
    images = convert_from_path(pdf_path, dpi=300)
    
    all_text = ""
    
    for i, image in enumerate(images):
        # 转换为OpenCV格式
        open_cv_image = cv2.cvtColor(np.array(image), cv2.COLOR_RGB2BGR)
        
        # 预处理图像
        gray = cv2.cvtColor(open_cv_image, cv2.COLOR_BGR2GRAY)
        
        # OCR识别
        text = pytesseract.image_to_string(gray, lang='chi_sim+eng')
        all_text += f"\n=== 第 {i+1} 页 ===\n{text}\n"
    
    # 保存识别结果
    with open(output_text_path, 'w', encoding='utf-8') as f:
        f.write(all_text)
    
    print(f"OCR识别完成，结果保存到：{output_text_path}")
    return all_text

# 使用函数（需要安装tesseract和pdf2image）
# pdf_ocr('scanned_document.pdf', 'ocr_output.txt')
```

### 表单处理
```python
import pdfrw

def fill_pdf_form(input_path, output_path, form_data):
    """填充PDF表单"""
    template_pdf = pdfrw.PdfReader(input_path)
    
    # 填充表单字段
    for page in template_pdf.pages:
        annotations = page['/Annots']
        if annotations:
            for annotation in annotations:
                if annotation['/Subtype'] == '/Widget':
                    field_name = annotation['/T']
                    if field_name in form_data:
                        annotation.update(
                            pdfrw.PdfDict(V=form_data[field_name])
                        )
    
    # 保存填充后的PDF
    pdfrw.PdfWriter().write(output_path, template_pdf)
    print(f"PDF表单已填充：{output_path}")

# 使用函数
form_data = {
    'name': '张三',
    'email': 'zhangsan@example.com',
    'phone': '13800138000'
}
fill_pdf_form('form_template.pdf', 'filled_form.pdf', form_data)
```

### 水印添加
```python
import PyPDF2
from reportlab.pdfgen import canvas
from reportlab.lib.pagesizes import letter
from io import BytesIO

def add_watermark(input_path, output_path, watermark_text):
    """添加水印到PDF"""
    # 创建水印PDF
    packet = BytesIO()
    can = canvas.Canvas(packet, pagesize=letter)
    
    # 设置水印文本
    can.setFont('Helvetica', 40)
    can.setFillColorRGB(0.5, 0.5, 0.5, alpha=0.3)  # 半透明灰色
    
    # 在页面中心添加水印
    can.drawString(100, 400, watermark_text)
    can.save()
    
    # 移动到开始位置
    packet.seek(0)
    watermark_pdf = PyPDF2.PdfReader(packet)
    
    # 读取原始PDF
    pdf_reader = PyPDF2.PdfReader(input_path)
    pdf_writer = PyPDF2.PdfWriter()
    
    # 合并水印和原始页面
    for page_num in range(len(pdf_reader.pages)):
        page = pdf_reader.pages[page_num]
        page.merge_page(watermark_pdf.pages[0])
        pdf_writer.add_page(page)
    
    # 保存结果
    with open(output_path, 'wb') as out:
        pdf_writer.write(out)
    
    print(f"水印已添加到：{output_path}")

# 使用函数
add_watermark('document.pdf', 'watermarked.pdf', '机密文档')
```

## 最佳实践

### 性能优化
```python
import time

def optimize_pdf_processing(pdf_path):
    """优化PDF处理性能"""
    start_time = time.time()
    
    # 使用适当的库
    with pdfplumber.open(pdf_path) as pdf:
        # 批量处理页面
        pages_data = []
        for page in pdf.pages:
            # 只提取需要的元素
            text = page.extract_text()
            tables = page.extract_tables()
            
            pages_data.append({
                'text': text,
                'table_count': len(tables)
            })
    
    end_time = time.time()
    print(f"处理时间：{end_time - start_time:.2f}秒")
    return pages_data
```

### 错误处理
```python
import os

def safe_pdf_operation(pdf_path, operation_func):
    """安全的PDF操作"""
    try:
        # 检查文件存在性
        if not os.path.exists(pdf_path):
            raise FileNotFoundError(f"PDF文件不存在：{pdf_path}")
        
        # 检查文件格式
        if not pdf_path.lower().endswith('.pdf'):
            raise ValueError("文件不是PDF格式")
        
        # 执行操作
        return operation_func(pdf_path)
        
    except FileNotFoundError as e:
        print(f"文件错误：{str(e)}")
        return None
    except Exception as e:
        print(f"处理错误：{str(e)}")
        return None

# 使用安全操作
def extract_text_safe(pdf_path):
    import PyPDF2
    with open(pdf_path, 'rb') as file:
        pdf_reader = PyPDF2.PdfReader(file)
        return pdf_reader.pages[0].extract_text()

result = safe_pdf_operation('document.pdf', extract_text_safe)
```

### 兼容性考虑
- **版本兼容**：确保使用的库支持目标PDF版本
- **编码处理**：正确处理不同语言的文本编码
- **字体嵌入**：确保生成的PDF包含必要的字体

通过本指南，您可以掌握PDF文档的各种操作技能，从基本的文本提取到高级的OCR识别和表单处理。