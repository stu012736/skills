# 高级PDF处理参考指南

本指南提供高级PDF处理技术的详细参考，涵盖pypdfium2库、JavaScript库和高级命令行操作。

## 目录

1. [pypdfium2库](#pypdfium2库)
2. [JavaScript PDF库](#javascript-pdf库)
3. [高级命令行操作](#高级命令行操作)
4. [PDF操作最佳实践](#pdf操作最佳实践)
5. [故障排除](#故障排除)

---

## pypdfium2库

pypdfium2是Python的PDF处理库，基于Google的PDFium引擎，提供高性能的PDF渲染和操作功能。

### 安装和基本使用
```python
import pypdfium2 as pdfium

# 打开PDF文件
pdf = pdfium.PdfDocument("document.pdf")

# 获取页面数量
page_count = len(pdf)
print(f"文档包含 {page_count} 页")

# 渲染页面为图像
page = pdf[0]  # 第一页
bitmap = page.render(scale=2)  # 2倍缩放
pil_image = bitmap.to_pil()
pil_image.save("page_0.png")

# 关闭文档
pdf.close()
```

### 高级渲染选项
```python
import pypdfium2 as pdfium

def render_pdf_with_options(pdf_path, output_dir, options=None):
    """使用高级选项渲染PDF"""
    default_options = {
        'scale': 2.0,
        'rotation': 0,
        'crop': None,
        'grayscale': False,
        'optimize_mode': pdfium.OptimizeMode.LCD_DISPLAY
    }
    
    if options:
        default_options.update(options)
    
    pdf = pdfium.PdfDocument(pdf_path)
    
    for page_num in range(len(pdf)):
        page = pdf[page_num]
        
        # 应用渲染选项
        bitmap = page.render(
            scale=default_options['scale'],
            rotation=default_options['rotation'],
            crop=default_options['crop'],
            grayscale=default_options['grayscale'],
            optimize_mode=default_options['optimize_mode']
        )
        
        # 转换为PIL图像
        pil_image = bitmap.to_pil()
        
        # 保存图像
        output_path = f"{output_dir}/page_{page_num:03d}.png"
        pil_image.save(output_path)
        print(f"已保存: {output_path}")
    
    pdf.close()

# 使用函数
render_options = {
    'scale': 3.0,
    'grayscale': True,
    'rotation': 90  # 旋转90度
}
render_pdf_with_options("document.pdf", "rendered_pages", render_options)
```

### 文本提取和分析
```python
import pypdfium2 as pdfium

def extract_text_advanced(pdf_path):
    """高级文本提取功能"""
    pdf = pdfium.PdfDocument(pdf_path)
    
    text_data = []
    
    for page_num in range(len(pdf)):
        page = pdf[page_num]
        
        # 获取文本页面
        textpage = page.get_textpage()
        
        # 提取文本
        text = textpage.get_text_bounded()
        
        # 获取文本边界框
        text_objects = textpage.get_text_objects()
        
        page_text_data = {
            'page_number': page_num + 1,
            'text': text,
            'text_objects': []
        }
        
        for obj in text_objects:
            bbox = obj.get_bbox()
            page_text_data['text_objects'].append({
                'text': obj.get_text(),
                'font_size': obj.get_fontsize(),
                'bbox': {
                    'left': bbox.left,
                    'top': bbox.top,
                    'right': bbox.right,
                    'bottom': bbox.bottom
                }
            })
        
        text_data.append(page_text_data)
        textpage.close()
    
    pdf.close()
    return text_data

# 使用函数
text_analysis = extract_text_advanced("document.pdf")
for page_data in text_analysis:
    print(f"第 {page_data['page_number']} 页:")
    print(f"文本长度: {len(page_data['text'])} 字符")
    print(f"文本对象数量: {len(page_data['text_objects'])}")
```

### PDF操作功能
```python
import pypdfium2 as pdfium

def manipulate_pdf(input_path, output_path, operations):
    """PDF文档操作"""
    pdf = pdfium.PdfDocument(input_path)
    
    # 创建新的PDF文档
    new_pdf = pdfium.PdfDocument.new()
    
    for operation in operations:
        if operation['type'] == 'extract_pages':
            # 提取特定页面
            pages = operation['pages']
            for page_num in pages:
                if 0 <= page_num < len(pdf):
                    page = pdf[page_num]
                    new_pdf.insert_pdf(pdf, page_num, page_num + 1)
        
        elif operation['type'] == 'rotate_pages':
            # 旋转页面
            pages = operation['pages']
            rotation = operation['rotation']
            for page_num in pages:
                if 0 <= page_num < len(new_pdf):
                    page = new_pdf[page_num]
                    page.set_rotation(rotation)
        
        elif operation['type'] == 'merge_pdf':
            # 合并其他PDF
            merge_path = operation['path']
            merge_pdf = pdfium.PdfDocument(merge_path)
            new_pdf.insert_pdf(merge_pdf, 0, len(merge_pdf))
            merge_pdf.close()
    
    # 保存新文档
    new_pdf.save(output_path)
    pdf.close()
    new_pdf.close()
    print(f"PDF操作完成: {output_path}")

# 使用函数
operations = [
    {'type': 'extract_pages', 'pages': [0, 2, 4]},
    {'type': 'rotate_pages', 'pages': [0], 'rotation': 90},
    {'type': 'merge_pdf', 'path': 'additional.pdf'}
]

manipulate_pdf("source.pdf", "modified.pdf", operations)
```

## JavaScript PDF库

### PDF.js（Mozilla）
```javascript
// 使用PDF.js加载和显示PDF
const pdfjsLib = window['pdfjs-dist/build/pdf'];

// 设置worker路径
pdfjsLib.GlobalWorkerOptions.workerSrc = 'https://cdnjs.cloudflare.com/ajax/libs/pdf.js/3.4.120/pdf.worker.min.js';

// 加载PDF文档
const loadingTask = pdfjsLib.getDocument('document.pdf');
loadingTask.promise.then(function(pdf) {
    console.log('PDF加载完成');
    
    // 获取页面数量
    console.log('页面数量:', pdf.numPages);
    
    // 渲染第一页
    pdf.getPage(1).then(function(page) {
        const scale = 1.5;
        const viewport = page.getViewport({scale: scale});
        
        // 准备canvas用于渲染
        const canvas = document.getElementById('pdf-canvas');
        const context = canvas.getContext('2d');
        canvas.height = viewport.height;
        canvas.width = viewport.width;
        
        // 渲染页面
        const renderContext = {
            canvasContext: context,
            viewport: viewport
        };
        
        page.render(renderContext).promise.then(function() {
            console.log('页面渲染完成');
        });
    });
});
```

### pdf-lib（Node.js/浏览器）
```javascript
import { PDFDocument, rgb, StandardFonts } from 'pdf-lib';

// 创建新PDF
async function createPDF() {
    const pdfDoc = await PDFDocument.create();
    const page = pdfDoc.addPage([600, 400]);
    
    // 添加字体
    const font = await pdfDoc.embedFont(StandardFonts.Helvetica);
    
    // 添加文本
    page.drawText('Hello PDF!', {
        x: 50,
        y: 350,
        size: 30,
        font: font,
        color: rgb(0, 0, 0),
    });
    
    // 保存PDF
    const pdfBytes = await pdfDoc.save();
    return pdfBytes;
}

// 修改现有PDF
async function modifyPDF(existingPdfBytes) {
    const pdfDoc = await PDFDocument.load(existingPdfBytes);
    const pages = pdfDoc.getPages();
    const firstPage = pages[0];
    
    // 添加水印
    const font = await pdfDoc.embedFont(StandardFonts.Helvetica);
    firstPage.drawText('CONFIDENTIAL', {
        x: 50,
        y: 50,
        size: 12,
        font: font,
        color: rgb(0.5, 0.5, 0.5),
        opacity: 0.5,
    });
    
    return await pdfDoc.save();
}

// 使用函数
createPDF().then(pdfBytes => {
    // 处理生成的PDF字节
    console.log('PDF创建完成，大小:', pdfBytes.length);
});
```

### jsPDF（客户端PDF生成）
```javascript
// 使用jsPDF生成PDF
import jsPDF from 'jspdf';

// 创建简单PDF
function generateSimplePDF() {
    const doc = new jsPDF();
    
    // 添加文本
    doc.text('Hello world!', 10, 10);
    doc.text('这是第二行文本', 10, 20);
    
    // 添加矩形
    doc.rect(5, 5, 200, 50);
    
    // 保存PDF
    doc.save('simple.pdf');
}

// 生成带表格的PDF
function generateTablePDF(data) {
    const doc = new jsPDF();
    
    // 设置表格数据
    const tableData = data.map(item => [
        item.name,
        item.age,
        item.city
    ]);
    
    // 添加表格
    doc.autoTable({
        head: [['姓名', '年龄', '城市']],
        body: tableData,
        startY: 20,
        styles: {
            fontSize: 10,
            cellPadding: 3,
        },
        headStyles: {
            fillColor: [22, 160, 133],
            textColor: 255,
        },
    });
    
    doc.save('table.pdf');
}

// 使用函数
const sampleData = [
    { name: '张三', age: 30, city: '北京' },
    { name: '李四', age: 25, city: '上海' },
    { name: '王五', age: 35, city: '广州' }
];

generateTablePDF(sampleData);
```

## 高级命令行操作

### Ghostscript高级用法
```bash
# 高质量PDF压缩
gs -sDEVICE=pdfwrite -dCompatibilityLevel=1.4 \
   -dPDFSETTINGS=/prepress -dNOPAUSE -dQUIET -dBATCH \
   -sOutputFile=compressed.pdf input.pdf

# 提取特定页面范围
gs -sDEVICE=pdfwrite -dNOPAUSE -dBATCH \
   -dFirstPage=1 -dLastPage=5 \
   -sOutputFile=pages_1-5.pdf input.pdf

# PDF到高质量PNG转换
gs -dNOPAUSE -sDEVICE=png16m -r300 \
   -sOutputFile=page-%d.png input.pdf

# 调整PDF尺寸
gs -sDEVICE=pdfwrite -dDEVICEWIDTHPOINTS=612 \
   -dDEVICEHEIGHTPOINTS=792 -dFIXEDMEDIA \
   -dPDFFitPage -sOutputFile=resized.pdf input.pdf
```

### pdftk高级功能
```bash
# 加密PDF
pdftk input.pdf output encrypted.pdf user_pw mypassword

# 解密PDF
pdftk encrypted.pdf input_pw mypassword output decrypted.pdf

# 旋转特定页面
pdftk input.pdf cat 1E 2-5 6E output rotated.pdf

# 填充PDF表单
pdftk form.pdf fill_form data.fdf output filled.pdf

# 生成PDF报告
pdftk A=document1.pdf B=document2.pdf cat A1-5 B1-3 A6-end output combined.pdf
```

### ImageMagick PDF处理
```bash
# PDF到高质量JPEG转换
convert -density 300 input.pdf -quality 90 output.jpg

# 批量处理PDF页面
for i in {0..9}; do
    convert -density 150 "input.pdf[$i]" "page_$i.png"
done

# 创建PDF缩略图
convert -thumbnail x300 -background white -alpha remove input.pdf[0] thumbnail.jpg

# PDF到多页TIFF
convert -density 200 input.pdf -compress lzw output.tiff
```

### OCRmyPDF（OCR处理）
```bash
# 对扫描的PDF进行OCR
ocrmypdf --output-type pdf --force-ocr scanned.pdf ocr_output.pdf

# 优化和OCR
ocrmypdf --optimize 3 --jpeg-quality 85 --force-ocr input.pdf output.pdf

# 多语言OCR
ocrmypdf -l chi_sim+eng --force-ocr multilingual.pdf ocr_output.pdf

# 批量OCR处理
for pdf in *.pdf; do
    ocrmypdf --force-ocr "$pdf" "ocr_$pdf"
done
```

## PDF操作最佳实践

### 性能优化策略
```python
import time
import psutil
import os

def optimize_pdf_processing(pdf_path, operations):
    """PDF处理性能优化"""
    start_time = time.time()
    start_memory = psutil.Process(os.getpid()).memory_info().rss / 1024 / 1024
    
    # 监控资源使用
    def monitor_resources():
        current_memory = psutil.Process(os.getpid()).memory_info().rss / 1024 / 1024
        elapsed_time = time.time() - start_time
        print(f"内存使用: {current_memory:.2f} MB, 运行时间: {elapsed_time:.2f}秒")
    
    try:
        # 执行PDF操作
        for i, operation in enumerate(operations):
            print(f"执行操作 {i+1}/{len(operations)}")
            
            # 执行具体操作
            result = execute_pdf_operation(pdf_path, operation)
            
            # 定期监控资源
            if i % 5 == 0:
                monitor_resources()
        
        end_time = time.time()
        end_memory = psutil.Process(os.getpid()).memory_info().rss / 1024 / 1024
        
        print(f"处理完成 - 总时间: {end_time - start_time:.2f}秒")
        print(f"内存峰值: {end_memory - start_memory:.2f} MB")
        
        return True
        
    except Exception as e:
        print(f"处理失败: {str(e)}")
        return False

def execute_pdf_operation(pdf_path, operation):
    """执行单个PDF操作"""
    # 实现具体的PDF操作逻辑
    time.sleep(0.1)  # 模拟操作时间
    return f"操作完成: {operation}"

# 使用函数
operations = [f"操作{i}" for i in range(20)]
optimize_pdf_processing("large_document.pdf", operations)
```

### 错误处理和恢复
```python
import logging
from pathlib import Path

def setup_pdf_logging():
    """设置PDF处理日志"""
    logging.basicConfig(
        level=logging.INFO,
        format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
        handlers=[
            logging.FileHandler('pdf_processing.log'),
            logging.StreamHandler()
        ]
    )
    return logging.getLogger(__name__)

logger = setup_pdf_logging()

def safe_pdf_operation(operation_func, *args, **kwargs):
    """安全的PDF操作包装器"""
    try:
        logger.info(f"开始PDF操作: {operation_func.__name__}")
        
        # 验证输入文件
        for arg in args:
            if isinstance(arg, str) and arg.endswith('.pdf'):
                if not Path(arg).exists():
                    raise FileNotFoundError(f"PDF文件不存在: {arg}")
        
        # 执行操作
        result = operation_func(*args, **kwargs)
        
        logger.info(f"PDF操作完成: {operation_func.__name__}")
        return result
        
    except Exception as e:
        logger.error(f"PDF操作失败: {operation_func.__name__} - {str(e)}")
        
        # 尝试恢复或清理
        cleanup_failed_operation()
        
        return None

def cleanup_failed_operation():
    """清理失败的操作"""
    # 删除可能创建的临时文件
    temp_files = ['temp.pdf', 'output.pdf']
    for temp_file in temp_files:
        if Path(temp_file).exists():
            Path(temp_file).unlink()
            logger.info(f"已清理临时文件: {temp_file}")
```

### 批量处理优化
```python
import multiprocessing
from concurrent.futures import ThreadPoolExecutor

def batch_process_pdfs(pdf_files, process_function, max_workers=None):
    """批量处理PDF文件"""
    if max_workers is None:
        max_workers = min(multiprocessing.cpu_count(), 4)
    
    results = []
    
    with ThreadPoolExecutor(max_workers=max_workers) as executor:
        # 提交所有任务
        future_to_pdf = {
            executor.submit(process_function, pdf_file): pdf_file 
            for pdf_file in pdf_files
        }
        
        # 收集结果
        for future in future_to_pdf:
            pdf_file = future_to_pdf[future]
            try:
                result = future.result()
                results.append({
                    'file': pdf_file,
                    'result': result,
                    'status': 'success'
                })
                print(f"处理完成: {pdf_file}")
                
            except Exception as e:
                results.append({
                    'file': pdf_file,
                    'error': str(e),
                    'status': 'failed'
                })
                print(f"处理失败: {pdf_file} - {str(e)}")
    
    return results

# 示例处理函数
def extract_pdf_text(pdf_path):
    """提取PDF文本"""
    import pypdfium2 as pdfium
    
    pdf = pdfium.PdfDocument(pdf_path)
    text = ""
    
    for page in pdf:
        textpage = page.get_textpage()
        text += textpage.get_text_bounded()
        textpage.close()
    
    pdf.close()
    return text

# 使用批量处理
pdf_files = ['doc1.pdf', 'doc2.pdf', 'doc3.pdf', 'doc4.pdf']
results = batch_process_pdfs(pdf_files, extract_pdf_text)

success_count = sum(1 for r in results if r['status'] == 'success')
print(f"批量处理完成: {success_count}/{len(pdf_files)} 成功")
```

## 故障排除

### 常见问题解决

#### 内存不足错误
```python
def handle_memory_issues():
    """处理内存相关问题"""
    import gc
    
    # 强制垃圾回收
    gc.collect()
    
    # 监控内存使用
    import psutil
    memory_info = psutil.virtual_memory()
    
    if memory_info.percent > 85:
        print("警告: 内存使用率过高")
        return False
    
    return True
```

#### 文件损坏处理
```python
def repair_corrupted_pdf(pdf_path):
    """尝试修复损坏的PDF"""
    try:
        # 使用Ghostscript尝试修复
        import subprocess
        
        repaired_path = pdf_path.replace('.pdf', '_repaired.pdf')
        
        cmd = [
            'gs', '-o', repaired_path,
            '-sDEVICE=pdfwrite',
            '-dPDFSETTINGS=/prepress',
            pdf_path
        ]
        
        result = subprocess.run(cmd, capture_output=True, text=True)
        
        if result.returncode == 0:
            print("PDF修复成功")
            return repaired_path
        else:
            print("PDF修复失败")
            return None
            
    except Exception as e:
        print(f"修复过程中出错: {str(e)}")
        return None
```

#### 编码问题处理
```python
def handle_encoding_issues(text):
    """处理文本编码问题"""
    import chardet
    
    # 检测编码
    encoding_info = chardet.detect(text.encode() if isinstance(text, str) else text)
    detected_encoding = encoding_info['encoding']
    confidence = encoding_info['confidence']
    
    print(f"检测到编码: {detected_encoding} (置信度: {confidence:.2f})")
    
    if confidence > 0.7:
        try:
            # 尝试使用检测到的编码
            if isinstance(text, bytes):
                decoded_text = text.decode(detected_encoding)
            else:
                decoded_text = text
            
            return decoded_text
            
        except UnicodeDecodeError:
            # 尝试其他常见编码
            for encoding in ['utf-8', 'gbk', 'latin-1']:
                try:
                    if isinstance(text, bytes):
                        return text.decode(encoding)
                    else:
                        return text
                except UnicodeDecodeError:
                    continue
    
    # 如果所有编码都失败，返回原始文本
    return text if isinstance(text, str) else text.decode('utf-8', errors='ignore')
```

通过本参考指南，您可以掌握高级PDF处理技术，包括高性能库的使用、JavaScript PDF操作、命令行工具的高级功能以及最佳实践和故障排除方法。