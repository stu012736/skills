# PPTX文档操作技能指南

本指南提供使用Python和JavaScript创建、编辑和分析PowerPoint演示文稿（PPTX文件）的全面技能。

## 概述

PowerPoint演示文稿（.pptx文件）是Microsoft Office Open XML格式的文档，包含幻灯片、布局、主题和多媒体内容。本指南涵盖从基本操作到高级功能的完整工作流程。

## 目录

1. [Python库选择](#python库选择)
2. [基本PPTX操作](#基本pptx操作)
3. [幻灯片创建和编辑](#幻灯片创建和编辑)
4. [样式和格式](#样式和格式)
5. [图表和图形](#图表和图形)
6. [多媒体内容](#多媒体内容)
7. [模板使用](#模板使用)
8. [分析和提取](#分析和提取)
9. [JavaScript解决方案](#javascript解决方案)
10. [最佳实践](#最佳实践)

---

## Python库选择

### 主要库比较

| 库名称 | 主要功能 | 优点 | 缺点 |
|--------|----------|------|------|
| **python-pptx** | 创建和编辑PPTX | 功能全面，文档完善 | 不支持读取现有文件的所有属性 |
| **Aspose.Slides** | 企业级PPTX处理 | 功能强大，支持复杂操作 | 商业许可，价格较高 |
| **pptx2pdf** | PPTX到PDF转换 | 转换质量高 | 功能单一 |
| **python-pptx-template** | 模板填充 | 简化模板使用 | 依赖python-pptx |

### 推荐工作流程

```python
# 根据需求选择库
def select_pptx_library(requirements):
    """根据需求选择合适的PPTX库"""
    if requirements.get('create_new'):
        return 'python-pptx'
    elif requirements.get('enterprise_features'):
        return 'Aspose.Slides'
    elif requirements.get('template_filling'):
        return 'python-pptx-template'
    elif requirements.get('conversion_only'):
        return 'pptx2pdf'
    else:
        return 'python-pptx'  # 默认选择
```

## 基本PPTX操作

### 安装和设置
```python
# 安装python-pptx
pip install python-pptx

# 验证安装
from pptx import Presentation
print("python-pptx安装成功")
```

### 创建新演示文稿
```python
from pptx import Presentation
from pptx.util import Inches

def create_basic_presentation():
    """创建基本演示文稿"""
    # 创建演示文稿对象
    prs = Presentation()
    
    # 添加标题幻灯片
    slide_layout = prs.slide_layouts[0]  # 标题幻灯片布局
    slide = prs.slides.add_slide(slide_layout)
    
    # 设置标题和副标题
    title = slide.shapes.title
    subtitle = slide.placeholders[1]
    
    title.text = "我的演示文稿"
    subtitle.text = "创建于Python"
    
    # 保存演示文稿
    prs.save('basic_presentation.pptx')
    print("基本演示文稿创建完成")

create_basic_presentation()
```

### 读取现有演示文稿
```python
from pptx import Presentation

def analyze_presentation(file_path):
    """分析现有演示文稿"""
    prs = Presentation(file_path)
    
    print(f"幻灯片数量: {len(prs.slides)}")
    print(f"幻灯片布局数量: {len(prs.slide_layouts)}")
    
    # 分析每个幻灯片
    for i, slide in enumerate(prs.slides):
        print(f"\n幻灯片 {i+1}:")
        print(f"  布局: {slide.slide_layout.name}")
        print(f"  形状数量: {len(slide.shapes)}")
        
        # 分析形状
        for shape in slide.shapes:
            if shape.has_text_frame:
                text = shape.text.strip()
                if text:
                    print(f"  文本形状: '{text[:50]}...'")
    
    return prs

# 使用函数
presentation = analyze_presentation('existing_presentation.pptx')
```

## 幻灯片创建和编辑

### 添加不同类型幻灯片
```python
from pptx import Presentation
from pptx.util import Inches

def create_comprehensive_presentation():
    """创建包含多种幻灯片类型的演示文稿"""
    prs = Presentation()
    
    # 1. 标题幻灯片
    title_slide = prs.slides.add_slide(prs.slide_layouts[0])
    title_slide.shapes.title.text = "综合演示文稿"
    title_slide.placeholders[1].text = "展示各种幻灯片类型"
    
    # 2. 标题和内容幻灯片
    title_content = prs.slides.add_slide(prs.slide_layouts[1])
    title_content.shapes.title.text = "主要内容"
    content = title_content.shapes.placeholders[1]
    content.text = "• 第一点\n• 第二点\n• 第三点"
    
    # 3. 节标题幻灯片
    section_header = prs.slides.add_slide(prs.slide_layouts[2])
    section_header.shapes.title.text = "新章节"
    
    # 4. 两栏内容幻灯片
    two_column = prs.slides.add_slide(prs.slide_layouts[3])
    two_column.shapes.title.text = "两栏布局"
    
    # 5. 空白幻灯片（自定义内容）
    blank_slide = prs.slides.add_slide(prs.slide_layouts[6])
    
    # 在空白幻灯片上添加自定义形状
    left = top = Inches(1)
    width = height = Inches(1.5)
    
    # 添加文本框
    textbox = blank_slide.shapes.add_textbox(left, top, width, height)
    text_frame = textbox.text_frame
    text_frame.text = "自定义内容"
    
    # 添加段落
    p = text_frame.add_paragraph()
    p.text = "这是自定义添加的文本"
    p.level = 1
    
    prs.save('comprehensive_presentation.pptx')
    print("综合演示文稿创建完成")

create_comprehensive_presentation()
```

### 幻灯片操作
```python
from pptx import Presentation

def manipulate_slides(input_path, output_path):
    """幻灯片操作：复制、删除、重新排序"""
    prs = Presentation(input_path)
    
    # 复制第一张幻灯片
    if len(prs.slides) > 0:
        original_slide = prs.slides[0]
        
        # 创建新幻灯片（复制布局）
        slide_layout = prs.slide_layouts[0]
        new_slide = prs.slides.add_slide(slide_layout)
        
        # 复制内容（简化版本）
        new_slide.shapes.title.text = "复制的幻灯片: " + original_slide.shapes.title.text
    
    # 删除特定幻灯片（示例：删除第二张）
    if len(prs.slides) > 1:
        # 注意：python-pptx不支持直接删除，需要重建
        print("注意：删除幻灯片需要重建演示文稿")
    
    # 重新排序（通过XML操作）
    def reorder_slides_xml(prs, new_order):
        """通过XML操作重新排序幻灯片"""
        slides = list(prs.slides._sldIdLst)
        
        # 清除现有顺序
        prs.slides._sldIdLst.clear()
        
        # 按新顺序添加
        for index in new_order:
            if 0 <= index < len(slides):
                prs.slides._sldIdLst.append(slides[index])
    
    # 保存修改
    prs.save(output_path)
    print(f"幻灯片操作完成: {output_path}")

manipulate_slides('input.pptx', 'modified.pptx')
```

## 样式和格式

### 文本格式设置
```python
from pptx import Presentation
from pptx.util import Inches, Pt
from pptx.dml.color import RGBColor
from pptx.enum.text import MSO_ANCHOR, MSO_AUTO_SIZE

def format_text_styles():
    """设置文本样式和格式"""
    prs = Presentation()
    slide = prs.slides.add_slide(prs.slide_layouts[5])  # 仅标题布局
    
    # 设置标题
    title = slide.shapes.title
    title.text = "文本格式示例"
    
    # 添加文本框
    left = Inches(1)
    top = Inches(2)
    width = Inches(8)
    height = Inches(4)
    
    textbox = slide.shapes.add_textbox(left, top, width, height)
    text_frame = textbox.text_frame
    text_frame.word_wrap = True
    
    # 清除默认文本
    text_frame.clear()
    
    # 添加格式化文本
    p = text_frame.paragraphs[0]
    p.text = "这是普通文本"
    
    # 添加带格式的段落
    p = text_frame.add_paragraph()
    run = p.add_run()
    run.text = "这是粗体红色文本"
    font = run.font
    font.bold = True
    font.color.rgb = RGBColor(255, 0, 0)  # 红色
    font.size = Pt(16)
    
    # 添加另一个段落
    p = text_frame.add_paragraph()
    run = p.add_run()
    run.text = "这是斜体蓝色文本"
    font = run.font
    font.italic = True
    font.color.rgb = RGBColor(0, 0, 255)  # 蓝色
    font.size = Pt(14)
    
    # 添加带下划线的文本
    p = text_frame.add_paragraph()
    run = p.add_run()
    run.text = "这是带下划线的文本"
    font = run.font
    font.underline = True
    font.size = Pt(12)
    
    prs.save('text_styles.pptx')
    print("文本样式演示文稿创建完成")

format_text_styles()
```

### 形状和背景格式
```python
from pptx import Presentation
from pptx.util import Inches
from pptx.dml.color import RGBColor
from pptx.enum.shapes import MSO_SHAPE

def format_shapes_and_background():
    """设置形状和背景格式"""
    prs = Presentation()
    slide = prs.slides.add_slide(prs.slide_layouts[6])  # 空白布局
    
    # 设置幻灯片背景
    background = slide.background
    fill = background.fill
    fill.solid()
    fill.fore_color.rgb = RGBColor(240, 248, 255)  # 浅蓝色背景
    
    # 添加各种形状
    shapes_info = [
        (MSO_SHAPE.RECTANGLE, Inches(1), Inches(1), Inches(2), Inches(1), RGBColor(255, 200, 200)),
        (MSO_SHAPE.OVAL, Inches(4), Inches(1), Inches(2), Inches(1), RGBColor(200, 255, 200)),
        (MSO_SHAPE.ROUNDED_RECTANGLE, Inches(1), Inches(3), Inches(2), Inches(1), RGBColor(200, 200, 255)),
        (MSO_SHAPE.CHEVRON, Inches(4), Inches(3), Inches(2), Inches(1), RGBColor(255, 255, 200)),
    ]
    
    for shape_type, left, top, width, height, color in shapes_info:
        shape = slide.shapes.add_shape(shape_type, left, top, width, height)
        
        # 设置形状填充颜色
        fill = shape.fill
        fill.solid()
        fill.fore_color.rgb = color
        
        # 设置形状边框
        line = shape.line
        line.color.rgb = RGBColor(0, 0, 0)  # 黑色边框
        line.width = Pt(2)  # 2磅边框
        
        # 添加文本
        if shape.has_text_frame:
            shape.text = shape_type.name
    
    prs.save('shapes_background.pptx')
    print("形状和背景格式演示文稿创建完成")

format_shapes_and_background()
```

## 图表和图形

### 创建图表
```python
from pptx import Presentation
from pptx.util import Inches
from pptx.chart.data import ChartData
from pptx.enum.chart import XL_CHART_TYPE

def create_charts():
    """创建各种图表"""
    prs = Presentation()
    
    # 柱状图幻灯片
    slide = prs.slides.add_slide(prs.slide_layouts[5])
    slide.shapes.title.text = "销售数据图表"
    
    # 定义图表数据
    chart_data = ChartData()
    chart_data.categories = ['第一季度', '第二季度', '第三季度', '第四季度']
    chart_data.add_series('产品A', (25.4, 30.6, 45.8, 40.2))
    chart_data.add_series('产品B', (15.2, 25.3, 35.1, 30.5))
    
    # 添加柱状图
    x, y, cx, cy = Inches(1), Inches(2), Inches(8), Inches(5)
    chart = slide.shapes.add_chart(
        XL_CHART_TYPE.COLUMN_CLUSTERED, x, y, cx, cy, chart_data
    ).chart
    
    # 饼图幻灯片
    slide2 = prs.slides.add_slide(prs.slide_layouts[5])
    slide2.shapes.title.text = "市场份额分布"
    
    pie_data = ChartData()
    pie_data.categories = ['公司A', '公司B', '公司C', '其他']
    pie_data.add_series('市场份额', (45, 30, 15, 10))
    
    x, y, cx, cy = Inches(1), Inches(2), Inches(8), Inches(5)
    pie_chart = slide2.shapes.add_chart(
        XL_CHART_TYPE.PIE, x, y, cx, cy, pie_data
    ).chart
    
    # 折线图幻灯片
    slide3 = prs.slides.add_slide(prs.slide_layouts[5])
    slide3.shapes.title.text = "趋势分析"
    
    line_data = ChartData()
    line_data.categories = ['1月', '2月', '3月', '4月', '5月', '6月']
    line_data.add_series('网站访问量', (1200, 1500, 1800, 2100, 1900, 2200))
    line_data.add_series('移动端访问量', (800, 1200, 1500, 1800, 2000, 2300))
    
    x, y, cx, cy = Inches(1), Inches(2), Inches(8), Inches(5)
    line_chart = slide3.shapes.add_chart(
        XL_CHART_TYPE.LINE, x, y, cx, cy, line_data
    ).chart
    
    prs.save('charts_presentation.pptx')
    print("图表演示文稿创建完成")

create_charts()
```

### 高级图表定制
```python
from pptx import Presentation
from pptx.util import Inches
from pptx.chart.data import ChartData
from pptx.enum.chart import XL_CHART_TYPE
from pptx.dml.color import RGBColor

def customize_charts():
    """定制图表样式"""
    prs = Presentation()
    slide = prs.slides.add_slide(prs.slide_layouts[5])
    slide.shapes.title.text = "定制图表"
    
    # 基础数据
    chart_data = ChartData()
    chart_data.categories = ['北京', '上海', '广州', '深圳', '杭州']
    chart_data.add_series('2023年', (45, 38, 28, 25, 20))
    chart_data.add_series('2024年', (50, 42, 32, 28, 25))
    
    # 创建图表
    x, y, cx, cy = Inches(1), Inches(2), Inches(8), Inches(5)
    chart_shape = slide.shapes.add_chart(
        XL_CHART_TYPE.COLUMN_CLUSTERED, x, y, cx, cy, chart_data
    )
    chart = chart_shape.chart
    
    # 定制图表样式
    chart.has_title = True
    chart.chart_title.text_frame.text = "城市销售数据"
    
    # 定制系列颜色
    series_1 = chart.series[0]
    series_1.format.fill.solid()
    series_1.format.fill.fore_color.rgb = RGBColor(65, 105, 225)  # 皇家蓝
    
    series_2 = chart.series[1]
    series_2.format.fill.solid()
    series_2.format.fill.fore_color.rgb = RGBColor(220, 20, 60)   # 深红色
    
    # 定制坐标轴
    value_axis = chart.value_axis
    value_axis.has_title = True
    value_axis.axis_title.text_frame.text = "销售额（万元）"
    
    category_axis = chart.category_axis
    category_axis.has_title = True
    category_axis.axis_title.text_frame.text = "城市"
    
    prs.save('customized_charts.pptx')
    print("定制图表演示文稿创建完成")

customize_charts()
```

## 多媒体内容

### 添加图像
```python
from pptx import Presentation
from pptx.util import Inches

def add_images_to_presentation():
    """向演示文稿添加图像"""
    prs = Presentation()
    
    # 图像幻灯片
    slide = prs.slides.add_slide(prs.slide_layouts[5])
    slide.shapes.title.text = "图像展示"
    
    # 添加单张图像
    left = Inches(1)
    top = Inches(2)
    width = Inches(4)
    
    try:
        slide.shapes.add_picture('image1.jpg', left, top, width=width)
    except FileNotFoundError:
        print("图像文件不存在，创建占位符")
        # 添加占位符形状
        from pptx.enum.shapes import MSO_SHAPE
        placeholder = slide.shapes.add_shape(MSO_SHAPE.RECTANGLE, left, top, width, Inches(3))
        placeholder.text = "图像占位符"
    
    # 添加多张图像（网格布局）
    slide2 = prs.slides.add_slide(prs.slide_layouts[5])
    slide2.shapes.title.text = "图像网格"
    
    image_positions = [
        (Inches(1), Inches(2), Inches(2.5)),
        (Inches(4), Inches(2), Inches(2.5)),
        (Inches(1), Inches(5), Inches(2.5)),
        (Inches(4), Inches(5), Inches(2.5)),
    ]
    
    for i, (left, top, width) in enumerate(image_positions):
        try:
            slide2.shapes.add_picture(f'image{i+1}.jpg', left, top, width=width)
        except FileNotFoundError:
            placeholder = slide2.shapes.add_shape(MSO_SHAPE.RECTANGLE, left, top, width, Inches(2))
            placeholder.text = f"图像 {i+1}"
    
    prs.save('images_presentation.pptx')
    print("图像演示文稿创建完成")

add_images_to_presentation()
```

### 处理多媒体文件
```python
from pptx import Presentation
from pptx.util import Inches

def handle_media_files():
    """处理音频和视频文件"""
    prs = Presentation()
    
    # 音频幻灯片
    slide = prs.slides.add_slide(prs.slide_layouts[5])
    slide.shapes.title.text = "音频内容"
    
    # 添加音频占位符（python-pptx不支持直接添加音频）
    left = Inches(1)
    top = Inches(2)
    width = Inches(3)
    height = Inches(1)
    
    audio_placeholder = slide.shapes.add_textbox(left, top, width, height)
    audio_placeholder.text = "音频文件: presentation_audio.mp3"
    
    # 视频幻灯片
    slide2 = prs.slides.add_slide(prs.slide_layouts[5])
    slide2.shapes.title.text = "视频内容"
    
    video_placeholder = slide2.shapes.add_textbox(left, top, width, height)
    video_placeholder.text = "视频文件: presentation_video.mp4"
    
    # 添加说明文本
    left = Inches(1)
    top = Inches(3.5)
    width = Inches(8)
    height = Inches(1)
    
    note_box = slide2.shapes.add_textbox(left, top, width, height)
    note_box.text = "注意：音频和视频文件需要在PowerPoint中手动添加"
    
    prs.save('media_presentation.pptx')
    print("多媒体演示文稿创建完成")

handle_media_files()
```

## 模板使用

### 基于模板创建演示文稿
```python
from pptx import Presentation

def create_from_template():
    """使用模板创建演示文稿"""
    try:
        # 加载模板文件
        prs = Presentation('template.pptx')
        print("模板加载成功")
        
        # 修改模板内容
        for slide in prs.slides:
            for shape in slide.shapes:
                if shape.has_text_frame:
                    text = shape.text.strip()
                    
                    # 替换模板占位符
                    if '{{title}}' in text:
                        shape.text = shape.text.replace('{{title}}', '我的演示文稿')
                    elif '{{date}}' in text:
                        shape.text = shape.text.replace('{{date}}', '2024年1月')
                    elif '{{author}}' in text:
                        shape.text = shape.text.replace('{{author}}', '张三')
        
        prs.save('from_template.pptx')
        print("基于模板的演示文稿创建完成")
        
    except FileNotFoundError:
        print("模板文件不存在，创建默认演示文稿")
        # 创建默认演示文稿
        prs = Presentation()
        slide = prs.slides.add_slide(prs.slide_layouts[0])
        slide.shapes.title.text = "默认演示文稿"
        slide.placeholders[1].text = "模板文件未找到"
        prs.save('from_template.pptx')

create_from_template()
```

### 创建自定义模板
```python
from pptx import Presentation
from pptx.util import Inches
from pptx.dml.color import RGBColor

def create_custom_template():
    """创建自定义模板"""
    prs = Presentation()
    
    # 定义公司主题颜色
    primary_color = RGBColor(0, 84, 159)   # 公司主色
    secondary_color = RGBColor(255, 204, 0) # 公司辅色
    
    # 标题幻灯片
    title_slide = prs.slide_layouts[0]
    title_slide.background.fill.solid()
    title_slide.background.fill.fore_color.rgb = primary_color
    
    # 内容幻灯片布局
    content_layout = prs.slide_layouts[1]
    
    # 添加公司Logo（占位符）
    slide = prs.slides.add_slide(title_slide)
    title = slide.shapes.title
    subtitle = slide.placeholders[1]
    
    title.text = "{{presentation_title}}"
    subtitle.text = "{{presentation_subtitle}}"
    
    # 添加Logo占位符
    left = Inches(7.5)
    top = Inches(0.2)
    width = Inches(1.5)
    height = Inches(0.5)
    
    logo_placeholder = slide.shapes.add_textbox(left, top, width, height)
    logo_placeholder.text = "{{company_logo}}"
    
    # 内容幻灯片示例
    content_slide = prs.slides.add_slide(content_layout)
    content_slide.shapes.title.text = "{{slide_title}}"
    content_slide.placeholders[1].text = "{{slide_content}}"
    
    prs.save('custom_template.pptx')
    print("自定义模板创建完成")

create_custom_template()
```

## 分析和提取

### 提取演示文稿内容
```python
from pptx import Presentation
import json

def extract_presentation_content(file_path):
    """提取演示文稿的完整内容"""
    prs = Presentation(file_path)
    
    presentation_data = {
        'file_name': file_path,
        'slide_count': len(prs.slides),
        'slides': [],
        'metadata': {}
    }
    
    # 提取幻灯片内容
    for slide_num, slide in enumerate(prs.slides):
        slide_data = {
            'slide_number': slide_num + 1,
            'layout_name': slide.slide_layout.name,
            'shapes_count': len(slide.shapes),
            'text_content': [],
            'images_count': 0
        }
        
        # 提取文本内容
        for shape in slide.shapes:
            if shape.has_text_frame:
                text = shape.text.strip()
                if text:
                    slide_data['text_content'].append({
                        'shape_type': shape.shape_type,
                        'text': text,
                        'position': str(shape.left) + "," + str(shape.top)
                    })
            
            # 统计图像
            if hasattr(shape, 'image'):
                slide_data['images_count'] += 1
        
        presentation_data['slides'].append(slide_data)
    
    # 保存提取的数据
    output_file = file_path.replace('.pptx', '_content.json')
    with open(output_file, 'w', encoding='utf-8') as f:
        json.dump(presentation_data, f, ensure_ascii=False, indent=2)
    
    print(f"内容提取完成: {output_file}")
    return presentation_data

# 使用函数
content_data = extract_presentation_content('sample.pptx')
```

### 演示文稿分析报告
```python
from pptx import Presentation
from collections import Counter

def analyze_presentation_structure(file_path):
    """分析演示文稿结构"""
    prs = Presentation(file_path)
    
    analysis = {
        'basic_info': {
            'total_slides': len(prs.slides),
            'layouts_used': [],
            'average_text_per_slide': 0,
            'total_text_characters': 0
        },
        'layout_analysis': {},
        'content_analysis': {}
    }
    
    # 分析布局使用情况
    layout_counter = Counter()
    text_lengths = []
    
    for slide in prs.slides:
        layout_name = slide.slide_layout.name
        layout_counter[layout_name] += 1
        
        # 分析文本内容
        slide_text_length = 0
        for shape in slide.shapes:
            if shape.has_text_frame:
                text = shape.text.strip()
                slide_text_length += len(text)
        
        text_lengths.append(slide_text_length)
        analysis['basic_info']['total_text_characters'] += slide_text_length
    
    # 计算统计数据
    analysis['basic_info']['layouts_used'] = dict(layout_counter)
    analysis['basic_info']['average_text_per_slide'] = \
        sum(text_lengths) / len(text_lengths) if text_lengths else 0
    
    # 布局分析
    analysis['layout_analysis'] = {
        'most_used_layout': layout_counter.most_common(1)[0] if layout_counter else None,
        'layout_diversity': len(layout_counter),
        'layout_distribution': dict(layout_counter)
    }
    
    # 内容分析
    analysis['content_analysis'] = {
        'text_density': analysis['basic_info']['average_text_per_slide'],
        'content_balance': {
            'text_heavy_slides': len([l for l in text_lengths if l > 500]),
            'minimal_text_slides': len([l for l in text_lengths if l < 100])
        }
    }
    
    # 生成报告
    print("=== 演示文稿分析报告 ===")
    print(f"文件: {file_path}")
    print(f"总幻灯片数: {analysis['basic_info']['total_slides']}")
    print(f"使用的布局种类: {analysis['layout_analysis']['layout_diversity']}")
    print(f"最常用布局: {analysis['layout_analysis']['most_used_layout']}")
    print(f"平均每张幻灯片文本长度: {analysis['basic_info']['average_text_per_slide']:.0f} 字符")
    print(f"总文本字符数: {analysis['basic_info']['total_text_characters']}")
    
    return analysis

# 使用函数
analysis_report = analyze_presentation_structure('sample.pptx')
```

## JavaScript解决方案

### 使用PptxGenJS
```javascript
// 在浏览器中生成PPTX
const pptx = new PptxGenJS();

// 创建标题幻灯片
let slide = pptx.addSlide();
slide.addText('JavaScript生成的演示文稿', {
    x: 1, y: 1, w: '80%', h: 1.5,
    fontSize: 24, bold: true, align: 'center'
});

// 添加内容幻灯片
slide = pptx.addSlide();
slide.addText('主要内容', { x: 0.5, y: 0.5, fontSize: 20, bold: true });

// 添加项目符号列表
slide.addText([
    { text: '第一点', options: { x: 1, y: 1.5, fontSize: 16 } },
    { text: '第二点', options: { x: 1, y: 2.0, fontSize: 16 } },
    { text: '第三点', options: { x: 1, y: 2.5, fontSize: 16 } }
]);

// 添加图表
slide = pptx.addSlide();
slide.addChart(pptx.charts.BAR, [
    { name: '产品A', labels: ['Q1', 'Q2', 'Q3', 'Q4'], values: [25, 30, 45, 40] },
    { name: '产品B', labels: ['Q1', 'Q2', 'Q3', 'Q4'], values: [15, 25, 35, 30] }
], { x: 1, y: 1, w: 6, h: 4 });

// 保存PPTX
pptx.writeFile({ fileName: 'javascript_presentation.pptx' });
```

### Node.js中的PPTX处理
```javascript
// 使用node-pptx库（示例）
const pptx = require('node-pptx');

async function createPptxWithNode() {
    // 创建新演示文稿
    const presentation = new pptx.Presentation();
    
    // 添加幻灯片
    const slide = presentation.addSlide();
    
    // 添加标题
    slide.addText('Node.js生成的PPTX', {
        x: 100, y: 100, w: 400, h: 50,
        fontSize: 24, bold: true
    });
    
    // 添加内容
    slide.addText('使用Node.js创建PowerPoint演示文稿', {
        x: 100, y: 200, w: 400, h: 30,
        fontSize: 16
    });
    
    // 保存文件
    await presentation.save('node_presentation.pptx');
    console.log('PPTX文件创建完成');
}

createPptxWithNode().catch(console.error);
```

## 最佳实践

### 代码组织和可维护性
```python
from pptx import Presentation
from pptx.util import Inches
import os

class PowerPointGenerator:
    """PowerPoint生成器类"""
    
    def __init__(self, template_path=None):
        """初始化生成器"""
        if template_path and os.path.exists(template_path):
            self.prs = Presentation(template_path)
        else:
            self.prs = Presentation()
        
        self.slide_count = 0
    
    def add_title_slide(self, title, subtitle=""):
        """添加标题幻灯片"""
        slide = self.prs.slides.add_slide(self.prs.slide_layouts[0])
        slide.shapes.title.text = title
        if subtitle:
            slide.placeholders[1].text = subtitle
        self.slide_count += 1
        return slide
    
    def add_content_slide(self, title, content_items):
        """添加内容幻灯片"""
        slide = self.prs.slides.add_slide(self.prs.slide_layouts[1])
        slide.shapes.title.text = title
        
        content = slide.placeholders[1]
        content.text = "\n".join([f"• {item}" for item in content_items])
        
        self.slide_count += 1
        return slide
    
    def save(self, file_path):
        """保存演示文稿"""
        self.prs.save(file_path)
        print(f"演示文稿已保存: {file_path} (共 {self.slide_count} 张幻灯片)")

# 使用生成器类
generator = PowerPointGenerator()
generator.add_title_slide("项目报告", "2024年第一季度")
generator.add_content_slide("项目进展", ["任务A已完成", "任务B进行中", "任务C待开始"])
generator.add_content_slide("成果展示", ["用户增长20%", "收入提升15%", "客户满意度提高"])
generator.save('project_report.pptx')
```

### 错误处理和验证
```python
import os
from pathlib import Path

def validate_presentation_file(file_path):
    """验证PPTX文件"""
    if not os.path.exists(file_path):
        raise FileNotFoundError(f"文件不存在: {file_path}")
    
    if not file_path.lower().endswith('.pptx'):
        raise ValueError("文件必须是PPTX格式")
    
    file_size = os.path.getsize(file_path)
    if file_size == 0:
        raise ValueError("文件为空")
    
    if file_size > 50 * 1024 * 1024:  # 50MB限制
        raise ValueError("文件过大（超过50MB）")
    
    return True

def safe_presentation_operation(operation_func, *args, **kwargs):
    """安全的演示文稿操作包装器"""
    try:
        # 验证输入
        for arg in args:
            if isinstance(arg, str) and arg.endswith('.pptx'):
                validate_presentation_file(arg)
        
        # 执行操作
        result = operation_func(*args, **kwargs)
        print("操作成功完成")
        return result
        
    except Exception as e:
        print(f"操作失败: {str(e)}")
        
        # 清理临时文件
        temp_files = ['temp.pptx', 'output.pptx']
        for temp_file in temp_files:
            if Path(temp_file).exists():
                Path(temp_file).unlink()
        
        return None

# 使用安全操作
result = safe_presentation_operation(analyze_presentation_structure, 'sample.pptx')
```

### 性能优化
```python
import time
from functools import wraps

def timing_decorator(func):
    """计时装饰器"""
    @wraps(func)
    def wrapper(*args, **kwargs):
        start_time = time.time()
        result = func(*args, **kwargs)
        end_time = time.time()
        print(f"{func.__name__} 执行时间: {end_time - start_time:.2f}秒")
        return result
    return wrapper

@timing_decorator
def optimize_presentation_creation():
    """优化演示文稿创建性能"""
    prs = Presentation()
    
    # 批量添加幻灯片（减少重复操作）
    slide_data = [
        ("幻灯片1", ["内容1", "内容2", "内容3"]),
        ("幻灯片2", ["要点A", "要点B", "要点C"]),
        ("幻灯片3", ["总结1", "总结2", "总结3"]),
    ]
    
    for title, content in slide_data:
        slide = prs.slides.add_slide(prs.slide_layouts[1])
        slide.shapes.title.text = title
        slide.placeholders[1].text = "\n".join([f"• {item}" for item in content])
    
    prs.save('optimized_presentation.pptx')

optimize_presentation_creation()
```

## 总结

本指南提供了使用Python和JavaScript创建、编辑和分析PowerPoint演示文稿的全面技能。通过掌握这些技术，您可以：

1. **高效创建演示文稿**：使用python-pptx库快速生成专业演示文稿
2. **定制样式和格式**：创建符合品牌标准的定制模板
3. **添加丰富内容**：包括图表、图像和多媒体元素
4. **批量处理和分析**：自动化演示文稿的创建和分析流程
5. **跨平台解决方案**：在浏览器和Node.js环境中生成PPTX

遵循最佳实践，包括错误处理、性能优化和代码组织，可以确保您的PPTX处理代码既可靠又高效。

---

*本指南基于python-pptx 0.6.21版本和最新行业最佳实践编写。建议定期检查库的更新和新的功能特性。*
    
    # 提取幻灯片内容
    for slide_num, slide in enumerate(prs.slides):
        slide_data = {
            'slide_number': slide_num + 1,
            'layout_name': slide.slide_layout.name,
            'shapes_count': len(slide.shapes),
            'text_content': [],
            'images_count': 0
        }
        
        # 提取文本内容
        for shape in slide.shapes:
            if shape.has_text_frame:
                text = shape.text.strip()
                if text:
                    slide_data['text_content'].append({
                        'shape_type': shape.shape_type,
                        'text': text,
                        'position': str(shape.left) + "," + str(shape.top)
                    })
            
            # 统计图像
            if hasattr(shape, 'image'):
                slide_data['images_count'] += 1
        
        presentation_data['slides'].append(slide_data)
    
    # 保存提取的数据
    output_file = file_path.replace('.pptx', '_content.json')
    with open(output_file, 'w', encoding='utf-8') as f:
        json.dump(presentation_data, f, ensure_ascii=False, indent=2)
    
    print(f"内容提取完成: {output_file}")
    return presentation_data

# 使用函数
content_data = extract_presentation_content('sample.pptx')
```

### 演示文稿分析报告
```python
from pptx import Presentation
from collections import Counter

def analyze_presentation_structure(file_path):
    """分析演示文稿结构"""
    prs = Presentation(file_path)
    
    analysis = {
        'basic_info': {
            'total_slides': len(prs.slides),
            'layouts_used': [],
            'average_text_per_slide': 0,
            'total_text_characters': 0
        },
        'layout_analysis': {},
        'content_analysis': {}
    }
    
    # 分析布局使用情况
    layout_counter = Counter()
    text_lengths = []
    
    for slide in prs.slides:
        layout_name = slide.slide_layout.name
        layout_counter[layout_name] += 1
        
        # 分析文本内容
        slide_text_length = 0
        for shape in slide.shapes:
            if shape.has_text_frame:
                text = shape.text.strip()
                slide_text_length += len(text)
        
        text_lengths.append(slide_text_length)
        analysis['basic_info']['total_text_characters'] += slide_text_length
    
    # 计算统计数据
    analysis['basic_info']['layouts_used'] = dict(layout_counter)
    analysis['basic_info']['average_text_per_slide'] = \
        sum(text_lengths) / len(text_lengths) if text_lengths else 0
    
    # 布局分析
    analysis['layout_analysis'] = {
        'most_used_layout': layout_counter.most_common(1)[0] if layout_counter else None,
        'layout_diversity': len(layout_counter),
        'layout_distribution': dict(layout_counter)
    }
    
    # 内容分析
    analysis['content_analysis'] = {
        'text_density': analysis['basic_info']['average_text_per_slide'],
        'content_balance': {
            'text_heavy_slides': len([l for l in text_lengths if l > 500]),
            'minimal_text_slides': len([l for l in text_lengths if l < 100])
        }
    }
    
    # 生成报告
    print("=== 演示文稿分析报告 ===")
    print(f"文件: {file_path}")
    print(f"总幻灯片数: {analysis['basic_info']['total_slides']}")
    print(f"使用的布局种类: {analysis['layout_analysis']['layout_diversity']}")
    print(f"最常用布局: {analysis['layout_analysis']['most_used_layout']}")
    print(f"平均每张幻灯片文本长度: {analysis['basic_info']['average_text_per_slide']:.0f} 字符")
    print(f"总文本字符数: {analysis['basic_info']['total_text_characters']}")
    
    return analysis

# 使用函数
analysis_report = analyze_presentation_structure('sample.pptx')
```

## JavaScript解决方案

### 使用PptxGenJS
```javascript
// 在浏览器中生成PPTX
const pptx = new PptxGenJS();

// 创建标题幻灯片
let slide = pptx.addSlide();
slide.addText('JavaScript生成的演示文稿', {
    x: 1, y: 1, w: '80%', h: 1.5,
    fontSize: 24, bold: true, align: 'center'
});

// 添加内容幻灯片
slide = pptx.addSlide();
slide.addText('主要内容', { x: 0.5, y: 0.5, fontSize: 20, bold: true });

// 添加项目符号列表
slide.addText([
    { text: '第一点', options: { x: 1, y: 1.5, fontSize: 16 } },
    { text: '第二点', options: { x: 1, y: 2.0, fontSize: 16 } },
    { text: '第三点', options: { x: 1, y: 2.5, fontSize: 16 } }
]);

// 添加图表
slide = pptx.addSlide();
slide.addChart(pptx.charts.BAR, [
    { name: '产品A', labels: ['Q1', 'Q2', 'Q3', 'Q4'], values: [25, 30, 45, 40] },
    { name: '产品B', labels: ['Q1', 'Q2', 'Q3', 'Q4'], values: [15, 25, 35, 30] }
], { x: 1, y: 1, w: 6, h: 4 });

// 保存PPTX
pptx.writeFile({ fileName: 'javascript_presentation.pptx' });
```

### Node.js中的PPTX处理
```javascript
// 使用node-pptx库（示例）
const pptx = require('node-pptx');

async function createPptxWithNode() {
    // 创建新演示文稿
    const presentation = new pptx.Presentation();
    
    // 添加幻灯片
    const slide = presentation.addSlide();
    
    // 添加标题
    slide.addText('Node.js生成的PPTX', {
        x: 100, y: 100, w: 400, h: 50,
        fontSize: 24, bold: true
    });
    
    // 添加内容
    slide.addText('使用Node.js创建PowerPoint演示文稿', {
        x: 100, y: 200, w: 400, h: 30,
        fontSize: 16
    });
    
    // 保存文件
    await presentation.save('node_presentation.pptx');
    console.log('PPTX文件创建完成');
}

createPptxWithNode().catch(console.error);
```

## 最佳实践

### 代码组织和可维护性
```python
from pptx import Presentation
from pptx.util import Inches
import os

class PowerPointGenerator:
    """PowerPoint生成器类"