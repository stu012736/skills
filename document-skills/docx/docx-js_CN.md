# DOCX库教程

使用JavaScript/TypeScript生成.docx文件。

**重要：在开始之前阅读整个文档。** 关键的格式化规则和常见陷阱贯穿全文 - 跳过部分可能导致文件损坏或渲染问题。

## 设置
假设docx已全局安装
如果未安装：`npm install -g docx`

```javascript
const { Document, Packer, Paragraph, TextRun, Table, TableRow, TableCell, ImageRun, Media, 
        Header, Footer, AlignmentType, PageOrientation, LevelFormat, ExternalHyperlink, 
        InternalHyperlink, TableOfContents, HeadingLevel, BorderStyle, WidthType, TabStopType, 
        TabStopPosition, UnderlineType, ShadingType, VerticalAlign, SymbolRun, PageNumber,
        FootnoteReferenceRun, Footnote, PageBreak } = require('docx');

// 创建和保存
const doc = new Document({ sections: [{ children: [/* content */] }] });
Packer.toBuffer(doc).then(buffer => fs.writeFileSync("doc.docx", buffer)); // Node.js
Packer.toBlob(doc).then(blob => { /* download logic */ }); // 浏览器
```

## 文本和格式化
```javascript
// 重要：永远不要使用 \n 进行换行 - 始终使用单独的Paragraph元素
// ❌ 错误：new TextRun("Line 1\nLine 2")
// ✅ 正确：new Paragraph({ children: [new TextRun("Line 1")] }), new Paragraph({ children: [new TextRun("Line 2")] })

// 包含所有格式化选项的基本文本
new Paragraph({
  alignment: AlignmentType.CENTER,
  spacing: { before: 200, after: 200 },
  indent: { left: 720, right: 720 },
  children: [
    new TextRun({ text: "粗体", bold: true }),
    new TextRun({ text: "斜体", italics: true }),
    new TextRun({ text: "下划线", underline: { type: UnderlineType.DOUBLE, color: "FF0000" } }),
    new TextRun({ text: "彩色", color: "FF0000", size: 28, font: "Arial" }), // Arial默认
    new TextRun({ text: "高亮", highlight: "yellow" }),
    new TextRun({ text: "删除线", strike: true }),
    new TextRun({ text: "上标", superScript: true }),
    new TextRun({ text: "下标", subScript: true }),
    new TextRun({ text: "小型大写字母", smallCaps: true }),
    new SymbolRun({ char: "2022", font: "Symbol" }), // 项目符号 •
    new SymbolRun({ char: "00A9", font: "Arial" })   // 版权符号 © - 符号使用Arial字体
  ]
})
```

## 样式和专业格式化

```javascript
const doc = new Document({
  styles: {
    default: { document: { run: { font: "Arial", size: 24 } } }, // 12pt默认
    paragraphStyles: [
      // 文档标题样式 - 覆盖内置Title样式
      { id: "Title", name: "Title", basedOn: "Normal",
        run: { size: 56, bold: true, color: "000000", font: "Arial" },
        paragraph: { spacing: { before: 240, after: 120 }, alignment: AlignmentType.CENTER } },
      // 重要：通过使用精确的ID覆盖内置标题样式
      { id: "Heading1", name: "Heading 1", basedOn: "Normal", next: "Normal", quickFormat: true,
        run: { size: 32, bold: true, color: "000000", font: "Arial" }, // 16pt
        paragraph: { spacing: { before: 240, after: 240 }, outlineLevel: 0 } }, // TOC必需
      { id: "Heading2", name: "Heading 2", basedOn: "Normal", next: "Normal", quickFormat: true,
        run: { size: 28, bold: true, color: "000000", font: "Arial" }, // 14pt
        paragraph: { spacing: { before: 180, after: 180 }, outlineLevel: 1 } },
      // 自定义样式使用您自己的ID
      { id: "myStyle", name: "My Style", basedOn: "Normal",
        run: { size: 28, bold: true, color: "000000" },
        paragraph: { spacing: { after: 120 }, alignment: AlignmentType.CENTER } }
    ],
    characterStyles: [{ id: "myCharStyle", name: "My Char Style",
      run: { color: "FF0000", bold: true, underline: { type: UnderlineType.SINGLE } } }]
  },
  sections: [{
    properties: { page: { margin: { top: 1440, right: 1440, bottom: 1440, left: 1440 } } },
    children: [
      new Paragraph({ heading: HeadingLevel.TITLE, children: [new TextRun("文档标题")] }), // 使用覆盖的Title样式
      new Paragraph({ heading: HeadingLevel.HEADING_1, children: [new TextRun("标题1")] }), // 使用覆盖的Heading1样式
      new Paragraph({ style: "myStyle", children: [new TextRun("自定义段落样式")] }),
      new Paragraph({ children: [
        new TextRun("普通文本 "),
        new TextRun({ text: "自定义字符样式", style: "myCharStyle" })
      ]})
    ]
  }]
});
```

**专业字体组合：**
- **Arial（标题）+ Arial（正文）** - 最通用支持，干净专业
- **Times New Roman（标题）+ Arial（正文）** - 经典衬线标题与现代无衬线正文
- **Georgia（标题）+ Verdana（正文）** - 针对屏幕阅读优化，优雅对比

**关键样式原则：**
- **覆盖内置样式**：使用精确的ID如"Heading1"、"Heading2"、"Heading3"来覆盖Word的内置标题样式
- **HeadingLevel常量**：`HeadingLevel.HEADING_1`使用"Heading1"样式，`HeadingLevel.HEADING_2`使用"Heading2"样式等
- **包含outlineLevel**：为H1设置`outlineLevel: 0`，为H2设置`outlineLevel: 1`等，以确保TOC正常工作
- **使用自定义样式**而不是内联格式化以保持一致性
- **设置默认字体**使用`styles.default.document.run.font` - Arial是通用支持的
- **建立视觉层次**使用不同的字体大小（标题 > 标题 > 正文）
- **添加适当的间距**使用`before`和`after`段落间距
- **谨慎使用颜色**：默认为黑色（000000）和灰色阴影用于标题和标题（标题1，标题2等）
- **设置一致的边距**（1440 = 1英寸是标准）

## 列表（始终使用适当的列表 - 永远不要使用Unicode项目符号）
```javascript
// 项目符号 - 始终使用编号配置，而不是Unicode符号
// 关键：使用LevelFormat.BULLET常量，而不是字符串"bullet"
const doc = new Document({
  numbering: {
    config: [
      { reference: "bullet-list",
        levels: [{ level: 0, format: LevelFormat.BULLET, text: "•", alignment: AlignmentType.LEFT,
          style: { paragraph: { indent: { left: 720, hanging: 360 } } } }] },
      { reference: "first-numbered-list",
        levels: [{ level: 0, format: LevelFormat.DECIMAL, text: "%1.", alignment: AlignmentType.LEFT,
          style: { paragraph: { indent: { left: 720, hanging: 360 } } } }] },
      { reference: "second-numbered-list", // 不同的引用 = 从1重新开始
        levels: [{ level: 0, format: LevelFormat.DECIMAL, text: "%1.", alignment: AlignmentType.LEFT,
          style: { paragraph: { indent: { left: 720, hanging: 360 } } } }] }
    ]
  },
  sections: [{
    children: [
      // 项目符号列表项
      new Paragraph({ numbering: { reference: "bullet-list", level: 0 },
        children: [new TextRun("第一个项目符号点")] }),
      new Paragraph({ numbering: { reference: "bullet-list", level: 0 },
        children: [new TextRun("第二个项目符号点")] }),
      // 编号列表项
      new Paragraph({ numbering: { reference: "first-numbered-list", level: 0 },
        children: [new TextRun("第一个编号项")] }),
      new Paragraph({ numbering: { reference: "first-numbered-list", level: 0 },
        children: [new TextRun("第二个编号项")] }),
      // ⚠️ 关键：不同的引用 = 从1重新开始的独立列表
      // 相同的引用 = 继续先前的编号
      new Paragraph({ numbering: { reference: "second-numbered-list", level: 0 },
        children: [new TextRun("再次从1开始（因为不同的引用）")] })
    ]
  }]
});

// ⚠️ 关键编号规则：每个引用创建一个独立的编号列表
// - 相同的引用 = 继续编号（1, 2, 3... 然后 4, 5, 6...）
// - 不同的引用 = 从1重新开始（1, 2, 3... 然后 1, 2, 3...）
// 为每个单独的编号部分使用唯一的引用名称！

// ⚠️ 关键：永远不要使用Unicode项目符号 - 它们创建无法正常工作的假列表
// new TextRun("• 项目")           // 错误
// new SymbolRun({ char: "2022" }) // 错误
// ✅ 始终使用带有LevelFormat.BULLET常量的编号配置来创建真正的Word列表
```

## 表格
```javascript
// 包含边距、边框、标题和项目符号点的完整表格
const tableBorder = { style: BorderStyle.SINGLE, size: 1, color: "CCCCCC" };
const cellBorders = { top: tableBorder, bottom: tableBorder, left: tableBorder, right: tableBorder };

new Table({
  columnWidths: [4680, 4680], // ⚠️ 关键：在表格级别设置列宽 - 值以DXA为单位（二十分之一磅）
  margins: { top: 100, bottom: 100, left: 180, right: 180 }, // 为所有单元格设置一次
  rows: [
    new TableRow({
      tableHeader: true,
      children: [
        new TableCell({
          borders: cellBorders,
          width: { size: 4680, type: WidthType.DXA }, // 也在每个单元格上设置宽度
          // ⚠️ 关键：始终使用ShadingType.CLEAR以防止Word中出现黑色背景。
          shading: { fill: "D5E8F0", type: ShadingType.CLEAR }, 
          verticalAlign: VerticalAlign.CENTER,
          children: [new Paragraph({ 
            alignment: AlignmentType.CENTER,
            children: [new TextRun({ text: "标题", bold: true, size: 22 })]
          })]
        }),
        new TableCell({
          borders: cellBorders,
          width: { size: 4680, type: WidthType.DXA }, // 也在每个单元格上设置宽度
          shading: { fill: "D5E8F0", type: ShadingType.CLEAR },
          children: [new Paragraph({ 
            alignment: AlignmentType.CENTER,
            children: [new TextRun({ text: "项目符号点", bold: true, size: 22 })]
          })]
        })
      ]
    }),
    new TableRow({
      children: [
        new TableCell({
          borders: cellBorders,
          width: { size: 4680, type: WidthType.DXA }, // 也在每个单元格上设置宽度
          children: [new Paragraph({ children: [new TextRun("常规数据")] })]
        }),
        new TableCell({
          borders: cellBorders,
          width: { size: 4680, type: WidthType.DXA }, // 也在每个单元格上设置宽度
          children: [
            new Paragraph({ 
              numbering: { reference: "bullet-list", level: 0 },
              children: [new TextRun("第一个项目符号点")] 
            }),
            new Paragraph({ 
              numbering: { reference: "bullet-list", level: 0 },
              children: [new TextRun("第二个项目符号点")] 
            })
          ]
        })
      ]
    })
  ]
})
```

**重要：表格宽度和边框**
- 使用BOTH `columnWidths: [width1, width2, ...]`数组AND `width: { size: X, type: WidthType.DXA }`在每个单元格上
- 值以DXA为单位（二十分之一磅）：1440 = 1英寸，Letter可用宽度 = 9360 DXA（带1英寸边距）
- 将边框应用于单个`TableCell`元素，而不是`Table`本身

**预计算列宽（Letter尺寸带1英寸边距 = 9360 DXA总计）：**
- **2列：** `columnWidths: [4680, 4680]`（等宽）
- **3列：** `columnWidths: [3120, 3120, 3120]`（等宽）

## 链接和导航
```javascript
// TOC（需要标题） - 关键：仅使用HeadingLevel，不使用自定义样式
// ❌ 错误：new Paragraph({ heading: HeadingLevel.HEADING_1, style: "customHeader", children: [new TextRun("标题")] })
// ✅ 正确：new Paragraph({ heading: HeadingLevel.HEADING_1, children: [new TextRun("标题")] })
new TableOfContents("目录", { hyperlink: true, headingStyleRange: "1-3" }),

// 外部链接
new Paragraph({
  children: [new ExternalHyperlink({
    children: [new TextRun({ text: "Google", style: "Hyperlink" })],
    link: "https://www.google.com"
  })]
}),

// 内部链接和书签
new Paragraph({
  children: [new InternalHyperlink({
    children: [new TextRun({ text: "转到章节", style: "Hyperlink" })],
    anchor: "section1"
  })]
}),
new Paragraph({
  children: [new TextRun("章节内容")],
  bookmark: { id: "section1", name: "section1" }
}),
```

## 图像和媒体
```javascript
// 包含大小和定位的基本图像
// 关键：始终指定'type'参数 - 对于ImageRun是必需的
new Paragraph({
  alignment: AlignmentType.CENTER,
  children: [new ImageRun({
    type: "png", // 新要求：必须指定图像类型（png, jpg, jpeg, gif, bmp, svg）
    data: fs.readFileSync("image.png"),
    transformation: { width: 200, height: 150, rotation: 0 }, // 旋转角度
    altText: { title: "Logo", description: "公司Logo", name: "名称" } // 重要：所有三个字段都是必需的
  })]
})
```

## 分页符
```javascript
// 手动分页符
new Paragraph({ children: [new PageBreak()] }),

// 段落前分页符
new Paragraph({
  pageBreakBefore: true,
  children: [new TextRun("这在新页面上开始")]
})

// ⚠️ 关键：永远不要单独使用PageBreak - 它会创建Word无法打开的无効XML
// ❌ 错误：new PageBreak() 
// ✅ 正确：new Paragraph({ children: [new PageBreak()] })
```

## 页眉/页脚和页面设置
```javascript
const doc = new Document({
  sections: [{
    properties: {
      page: {
        margin: { top: 1440, right: 1440, bottom: 1440, left: 1440 }, // 1440 = 1英寸
        size: { orientation: PageOrientation.LANDSCAPE },
        pageNumbers: { start: 1, formatType: "decimal" } // "upperRoman", "lowerRoman", "upperLetter", "lowerLetter"
      }
    },
    headers: {
      default: new Header({ children: [new Paragraph({ 
        alignment: AlignmentType.RIGHT,
        children: [new TextRun("页眉文本")]
      })] })
    },
    footers: {
      default: new Footer({ children: [new Paragraph({ 
        alignment: AlignmentType.CENTER,
        children: [new TextRun("第 "), new TextRun({ children: [PageNumber.CURRENT] }), new TextRun(" 页，共 "), new TextRun({ children: [PageNumber.TOTAL_PAGES] })]
      })] })
    },
    children: [/* content */]
  }]
});
```

## 制表符
```javascript
new Paragraph({
  tabStops: [
    { type: TabStopType.LEFT, position: TabStopPosition.MAX / 4 },
    { type: TabStopType.CENTER, position: TabStopPosition.MAX / 2 },
    { type: TabStopType.RIGHT, position: TabStopPosition.MAX * 3 / 4 }
  ],
  children: [new TextRun("左\t中\t右")]
})
```

## 常量和快速参考
- **下划线：** `SINGLE`, `DOUBLE`, `WAVY`, `DASH`
- **边框：** `SINGLE`, `DOUBLE`, `DASHED`, `DOTTED`  
- **编号：** `DECIMAL` (1,2,3), `UPPER_ROMAN` (I,II,III), `LOWER_LETTER` (a,b,c)
- **制表符：** `LEFT`, `CENTER`, `RIGHT`, `DECIMAL`
- **符号：** `"2022"` (•), `"00A9"` (©), `"00AE"` (®), `"2122"` (™), `"00B0"` (°), `"F070"` (✓), `"F0FC"` (✗)

## 关键问题和常见错误
- **关键：PageBreak必须始终放在Paragraph内部** - 独立的PageBreak会创建Word无法打开的无効XML
- **始终使用ShadingType.CLEAR进行表格单元格着色** - 切勿使用ShadingType.SOLID（会导致黑色背景）。
- 测量单位DXA（1440 = 1英寸） | 每个表格单元格需要≥1个Paragraph | TOC仅需要HeadingLevel样式
- **始终使用自定义样式**和Arial字体以获得专业外观和适当的视觉层次
- **始终设置默认字体**使用`styles.default.document.run.font` - 推荐Arial
- **始终使用columnWidths数组用于表格** + 单个单元格宽度以获得兼容性
- **永远不要使用Unicode符号作为项目符号** - 始终使用带有`LevelFormat.BULLET`常量的适当编号配置（而不是字符串"bullet"）
- **永远不要在任何地方使用\n进行换行** - 始终为每行使用单独的Paragraph元素
- **始终在Paragraph子元素中使用TextRun对象** - 永远不要直接在Paragraph上使用text属性
- **图像的关键：** ImageRun需要`type`参数 - 始终指定"png"、"jpg"、"jpeg"、"gif"、"bmp"或"svg"
- **项目符号的关键：** 必须使用`LevelFormat.BULLET`常量，而不是字符串"bullet"，并包含`text: "•"`作为项目符号字符
- **编号的关键：** 每个编号引用创建一个独立的列表。相同的引用 = 继续编号（1,2,3然后4,5,6）。不同的引用 = 从1重新开始（1,2,3然后1,2,3）。为每个单独的编号部分使用唯一的引用名称！
- **TOC的关键：** 使用TableOfContents时，标题必须仅使用HeadingLevel - 不要向标题段落添加自定义样式，否则TOC会中断
- **表格：** 设置`columnWidths`数组 + 单个单元格宽度，将边框应用于单元格而不是表格
- **在TABLE级别设置表格边距**以获得一致的单元格填充（避免每个单元格重复）