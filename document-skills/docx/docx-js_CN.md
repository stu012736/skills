# 使用JavaScript/TypeScript生成DOCX文件指南

本指南介绍如何使用docx库通过JavaScript/TypeScript创建DOCX文档。

## 目录

1. [项目设置](#项目设置)
2. [基本文档创建](#基本文档创建)
3. [文本格式化](#文本格式化)
4. [段落和样式](#段落和样式)
5. [列表](#列表)
6. [表格](#表格)
7. [链接](#链接)
8. [图像](#图像)
9. [页眉和页脚](#页眉和页脚)
10. [高级功能](#高级功能)

---

## 项目设置

### 安装依赖
```bash
npm install docx
```

### 基本导入
```javascript
const { Document, Paragraph, TextRun, Packer } = require("docx");
// 或使用ES6导入
import { Document, Paragraph, TextRun, Packer } from "docx";
```

## 基本文档创建

### 创建简单文档
```javascript
const doc = new Document({
    sections: [{
        properties: {},
        children: [
            new Paragraph({
                children: [
                    new TextRun("Hello World"),
                ],
            }),
        ],
    }],
});

// 保存文档
Packer.toBuffer(doc).then((buffer) => {
    require("fs").writeFileSync("MyDocument.docx", buffer);
});
```

### 文档属性
```javascript
const doc = new Document({
    title: "我的文档",
    subject: "文档主题",
    creator: "作者姓名",
    description: "文档描述",
    keywords: "关键词1, 关键词2",
    // 更多属性...
});
```

## 文本格式化

### 基本文本样式
```javascript
new TextRun({
    text: "格式化文本",
    bold: true,
    italics: true,
    underline: {},
    color: "FF0000", // 红色
    size: 24, // 12pt * 2
    font: "Arial",
});
```

### 高级文本格式化
```javascript
new TextRun({
    text: "高级格式化",
    // 字体属性
    bold: true,
    italics: false,
    underline: {
        type: "single", // single, double, thick, dotted, dash, dotDash
        color: "0000FF",
    },
    // 颜色和大小
    color: "2E74B5",
    size: 36, // 18pt
    // 字体
    font: "Calibri",
    // 字符间距
    characterSpacing: 20, // 1/20pt
    // 文本效果
    highlight: "yellow",
    // 阴影
    shading: {
        fill: "D9E2F3",
    },
});
```

### 内联格式化
```javascript
new Paragraph({
    children: [
        new TextRun({
            text: "这是",
            size: 24,
        }),
        new TextRun({
            text: "粗体",
            bold: true,
            size: 24,
        }),
        new TextRun({
            text: "和",
            size: 24,
        }),
        new TextRun({
            text: "斜体",
            italics: true,
            size: 24,
        }),
        new TextRun({
            text: "文本。",
            size: 24,
        }),
    ],
});
```

## 段落和样式

### 基本段落
```javascript
new Paragraph({
    children: [new TextRun("段落内容")],
    alignment: "center", // left, center, right, both, distribute
    spacing: {
        before: 200, // 1/20pt
        after: 200,
        line: 240, // 单倍行距
    },
    indent: {
        firstLine: 400, // 首行缩进
    },
});
```

### 标题样式
```javascript
// 标题1
new Paragraph({
    children: [
        new TextRun({
            text: "标题1",
            bold: true,
            size: 32,
        }),
    ],
    heading: "Heading1",
    spacing: {
        before: 240,
        after: 120,
    },
});

// 标题2
new Paragraph({
    children: [
        new TextRun({
            text: "标题2",
            bold: true,
            size: 28,
        }),
    ],
    heading: "Heading2",
    spacing: {
        before: 200,
        after: 100,
    },
});
```

### 段落边框和背景
```javascript
new Paragraph({
    children: [new TextRun("带边框的段落")],
    border: {
        top: {
            color: "auto",
            space: 1,
            style: "single",
            size: 6,
        },
        bottom: {
            color: "auto",
            space: 1,
            style: "single",
            size: 6,
        },
        left: {
            color: "auto",
            space: 1,
            style: "single",
            size: 6,
        },
        right: {
            color: "auto",
            space: 1,
            style: "single",
            size: 6,
        },
    },
    shading: {
        fill: "F2F2F2",
    },
});
```

## 列表

### 项目符号列表
```javascript
const bulletPoints = [
    "第一项",
    "第二项",
    "第三项",
].map(
    (text) =>
        new Paragraph({
            children: [new TextRun(text)],
            numbering: {
                reference: "my-bullet-numbering",
                level: 0,
            },
        }),
);

const doc = new Document({
    numbering: {
        config: [
            {
                reference: "my-bullet-numbering",
                levels: [
                    {
                        level: 0,
                        format: "bullet",
                        text: "•",
                        alignment: "left",
                        style: {
                            paragraph: {
                                indent: { left: 720, hanging: 360 },
                            },
                        },
                    },
                ],
            },
        ],
    },
    sections: [{
        children: bulletPoints,
    }],
});
```

### 编号列表
```javascript
const numberedPoints = [
    "第一步",
    "第二步",
    "第三步",
].map(
    (text, index) =>
        new Paragraph({
            children: [new TextRun(text)],
            numbering: {
                reference: "my-numbering",
                level: 0,
            },
        }),
);

const doc = new Document({
    numbering: {
        config: [
            {
                reference: "my-numbering",
                levels: [
                    {
                        level: 0,
                        format: "decimal",
                        text: "%1.",
                        alignment: "left",
                        style: {
                            paragraph: {
                                indent: { left: 720, hanging: 360 },
                            },
                        },
                    },
                ],
            },
        ],
    },
    sections: [{
        children: numberedPoints,
    }],
});
```

### 多级列表
```javascript
const multiLevelPoints = [
    { text: "主要项目1", level: 0 },
    { text: "子项目1.1", level: 1 },
    { text: "子项目1.2", level: 1 },
    { text: "主要项目2", level: 0 },
    { text: "子项目2.1", level: 1 },
];

const listItems = multiLevelPoints.map(
    (item) =>
        new Paragraph({
            children: [new TextRun(item.text)],
            numbering: {
                reference: "multi-level-numbering",
                level: item.level,
            },
        }),
);

const doc = new Document({
    numbering: {
        config: [
            {
                reference: "multi-level-numbering",
                levels: [
                    {
                        level: 0,
                        format: "decimal",
                        text: "%1.",
                        alignment: "left",
                        style: {
                            paragraph: {
                                indent: { left: 720, hanging: 360 },
                            },
                        },
                    },
                    {
                        level: 1,
                        format: "decimal",
                        text: "%1.%2.",
                        alignment: "left",
                        style: {
                            paragraph: {
                                indent: { left: 1440, hanging: 360 },
                            },
                        },
                    },
                ],
            },
        ],
    },
    sections: [{
        children: listItems,
    }],
});
```

## 表格

### 基本表格
```javascript
const { Table, TableRow, TableCell } = require("docx");

const table = new Table({
    width: {
        size: 100,
        type: "pct",
    },
    rows: [
        new TableRow({
            children: [
                new TableCell({
                    children: [new Paragraph("标题1")],
                }),
                new TableCell({
                    children: [new Paragraph("标题2")],
                }),
                new TableCell({
                    children: [new Paragraph("标题3")],
                }),
            ],
        }),
        new TableRow({
            children: [
                new TableCell({
                    children: [new Paragraph("数据1")],
                }),
                new TableCell({
                    children: [new Paragraph("数据2")],
                }),
                new TableCell({
                    children: [new Paragraph("数据3")],
                }),
            ],
        }),
    ],
});
```

### 格式化表格
```javascript
const formattedTable = new Table({
    width: {
        size: 100,
        type: "pct",
    },
    borders: {
        top: { style: "single", size: 4, color: "000000" },
        bottom: { style: "single", size: 4, color: "000000" },
        left: { style: "single", size: 4, color: "000000" },
        right: { style: "single", size: 4, color: "000000" },
        insideHorizontal: { style: "single", size: 2, color: "666666" },
        insideVertical: { style: "single", size: 2, color: "666666" },
    },
    rows: [
        // 表头行
        new TableRow({
            tableHeader: true,
            children: [
                new TableCell({
                    children: [new Paragraph({
                        children: [new TextRun({
                            text: "产品",
                            bold: true,
                            color: "FFFFFF",
                        })],
                        alignment: "center",
                    })],
                    shading: {
                        fill: "2E74B5",
                    },
                }),
                new TableCell({
                    children: [new Paragraph({
                        children: [new TextRun({
                            text: "价格",
                            bold: true,
                            color: "FFFFFF",
                        })],
                        alignment: "center",
                    })],
                    shading: {
                        fill: "2E74B5",
                    },
                }),
            ],
        }),
        // 数据行
        new TableRow({
            children: [
                new TableCell({
                    children: [new Paragraph("产品A")],
                }),
                new TableCell({
                    children: [new Paragraph("$100")],
                    shading: {
                        fill: "F2F2F2",
                    },
                }),
            ],
        }),
    ],
});
```

### 合并单元格
```javascript
const mergedTable = new Table({
    rows: [
        new TableRow({
            children: [
                new TableCell({
                    children: [new Paragraph("合并单元格")],
                    columnSpan: 2, // 跨2列
                }),
            ],
        }),
        new TableRow({
            children: [
                new TableCell({
                    children: [new Paragraph("单元格1")],
                }),
                new TableCell({
                    children: [new Paragraph("单元格2")],
                }),
            ],
        }),
    ],
});
```

## 链接

### 超链接
```javascript
const { ExternalHyperlink } = require("docx");

new Paragraph({
    children: [
        new ExternalHyperlink({
            children: [
                new TextRun({
                    text: "点击访问网站",
                    style: "Hyperlink",
                }),
            ],
            link: "https://example.com",
        }),
    ],
});
```

### 内部链接（书签）
```javascript
const { InternalHyperlink } = require("docx");

// 创建书签
new Paragraph({
    children: [
        new TextRun({
            text: "章节标题",
            bold: true,
        }),
    ],
    bookmark: {
        id: "chapter1",
    },
});

// 创建指向书签的链接
new Paragraph({
    children: [
        new InternalHyperlink({
            children: [
                new TextRun({
                    text: "跳转到章节1",
                    style: "Hyperlink",
                }),
            ],
            anchor: "chapter1",
        }),
    ],
});
```

## 图像

### 添加图像
```javascript
const { ImageRun } = require("docx");

new Paragraph({
    children: [
        new ImageRun({
            data: require("fs").readFileSync("./image.png"),
            transformation: {
                width: 200,
                height: 200,
            },
        }),
    ],
    alignment: "center",
});
```

### 图像格式化
```javascript
new Paragraph({
    children: [
        new ImageRun({
            data: require("fs").readFileSync("./photo.jpg"),
            transformation: {
                width: 400,
                height: 300,
            },
            floating: {
                horizontalPosition: {
                    relative: "page",
                    offset: 1000000, // 1英寸
                },
                verticalPosition: {
                    relative: "page",
                    offset: 1000000,
                },
            },
        }),
    ],
});
```

## 页眉和页脚

### 页眉
```javascript
const { Header } = require("docx");

const doc = new Document({
    sections: [{
        headers: {
            default: new Header({
                children: [
                    new Paragraph({
                        children: [
                            new TextRun({
                                text: "文档标题",
                                bold: true,
                                size: 16,
                            }),
                        ],
                        alignment: "center",
                    }),
                ],
            }),
        },
        children: [/* 文档内容 */],
    }],
});
```

### 页脚
```javascript
const { Footer } = require("docx");

const doc = new Document({
    sections: [{
        footers: {
            default: new Footer({
                children: [
                    new Paragraph({
                        children: [
                            new TextRun({
                                text: "第",
                            }),
                            new TextRun({
                                text: "1",
                                bold: true,
                            }),
                            new TextRun({
                                text: "页",
                            }),
                        ],
                        alignment: "center",
                    }),
                ],
            }),
        },
        children: [/* 文档内容 */],
    }],
});
```

### 页码
```javascript
const { PageNumber } = require("docx");

const doc = new Document({
    sections: [{
        footers: {
            default: new Footer({
                children: [
                    new Paragraph({
                        children: [
                            new TextRun("第"),
                            new PageNumber("current"),
                            new TextRun("页"),
                        ],
                        alignment: "center",
                    }),
                ],
            }),
        },
        children: [/* 文档内容 */],
    }],
});
```

## 高级功能

### 文档样式
```javascript
const doc = new Document({
    styles: {
        paragraphStyles: [
            {
                id: "MyHeading1",
                name: "我的标题1",
                basedOn: "Heading1",
                next: "Normal",
                run: {
                    size: 32,
                    bold: true,
                    color: "2E74B5",
                },
                paragraph: {
                    spacing: { before: 240, after: 120 },
                },
            },
        ],
    },
});
```

### 节分隔
```javascript
const doc = new Document({
    sections: [
        {
            properties: {
                page: {
                    size: {
                        orientation: "portrait",
                        width: 12240, // 8.5英寸
                        height: 15840, // 11英寸
                    },
                },
            },
            children: [/* 第一页内容 */],
        },
        {
            properties: {
                page: {
                    size: {
                        orientation: "landscape",
                        width: 15840,
                        height: 12240,
                    },
                },
                type: "nextPage",
            },
            children: [/* 第二页内容 */],
        },
    ],
});
```

### 完整示例
```javascript
const { Document, Paragraph, TextRun, Table, TableRow, TableCell, Packer } = require("docx");

async function createDocument() {
    const doc = new Document({
        title: "示例文档",
        creator: "AI助手",
        description: "使用docx库创建的示例文档",
        sections: [{
            properties: {},
            children: [
                // 标题
                new Paragraph({
                    children: [
                        new TextRun({
                            text: "示例文档",
                            bold: true,
                            size: 36,
                        }),
                    ],
                    alignment: "center",
                    spacing: { before: 400, after: 200 },
                }),
                
                // 正文
                new Paragraph({
                    children: [
                        new TextRun("这是一个使用docx库创建的示例文档。"),
                    ],
                    spacing: { before: 100, after: 100 },
                }),
                
                // 表格
                new Table({
                    width: { size: 100, type: "pct" },
                    rows: [
                        new TableRow({
                            children: [
                                new TableCell({
                                    children: [new Paragraph("项目")],
                                    shading: { fill: "F2F2F2" },
                                }),
                                new TableCell({
                                    children: [new Paragraph("数量")],
                                    shading: { fill: "F2F2F2" },
                                }),
                            ],
                        }),
                        new TableRow({
                            children: [
                                new TableCell({
                                    children: [new Paragraph("产品A")],
                                }),
                                new TableCell({
                                    children: [new Paragraph("100")],
                                }),
                            ],
                        }),
                    ],
                }),
            ],
        }],
    });

    // 保存文档
    const buffer = await Packer.toBuffer(doc);
    require("fs").writeFileSync("示例文档.docx", buffer);
    console.log("文档创建成功！");
}

createDocument().catch(console.error);
```

## 最佳实践

### 性能优化
- **批量操作**：尽量减少对文件系统的频繁读写
- **内存管理**：处理大型文档时注意内存使用
- **错误处理**：妥善处理异步操作中的错误

### 代码组织
- **模块化**：将文档创建逻辑分解为可重用的函数
- **配置管理**：将样式和格式配置集中管理
- **模板系统**：为常用文档类型创建模板

### 兼容性考虑
- **字体选择**：使用跨平台兼容的字体
- **格式验证**：在不同版本的Word中测试生成的文档
- **Unicode支持**：确保正确处理多语言文本

通过本指南，您可以掌握使用JavaScript/TypeScript创建复杂DOCX文档的技能，满足各种文档生成需求。