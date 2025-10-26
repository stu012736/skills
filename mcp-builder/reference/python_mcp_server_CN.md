# Python MCP服务器实现指南

本文档提供了使用Python实现模型上下文协议（MCP）服务器的全面指南。

## 概述

Python MCP服务器实现基于`mcp`包，该包提供了构建MCP服务器的核心功能。Python实现特别适合快速原型开发和数据科学应用。

### 核心概念

- **MCP服务器**: 实现MCP协议的Python应用程序
- **工具**: 服务器提供的可调用功能
- **资源**: 服务器管理的可读数据源
- **提示**: 服务器提供的预定义模板

## 项目设置

### 环境准备

```bash
# 创建虚拟环境
python -m venv mcp-env

# 激活虚拟环境
# Windows
mcp-env\Scripts\activate
# macOS/Linux
source mcp-env/bin/activate

# 安装MCP包
pip install mcp

# 对于开发，安装额外依赖
pip install pydantic fastmcp
```

### 项目结构

```
my-python-mcp-server/
├── src/
│   ├── __init__.py
│   ├── server.py          # 主服务器文件
│   ├── tools/             # 工具实现
│   │   ├── __init__.py
│   │   ├── calculator.py
│   │   └── file_system.py
│   ├── resources/         # 资源实现
│   │   ├── __init__.py
│   │   └── file_system.py
│   └── prompts/           # 提示实现
│       ├── __init__.py
│       └── templates.py
├── requirements.txt
├── pyproject.toml
└── README.md
```

## 基本服务器实现

### 使用FastMCP框架

FastMCP是一个基于FastAPI的MCP服务器框架，提供了简单易用的API：

```python
# src/server.py
from fastmcp import FastMCP
from pydantic import BaseModel

# 创建FastMCP服务器实例
mcp = FastMCP("Python Calculator Server")

# 定义工具输入模型
class AddNumbersInput(BaseModel):
    a: float
    b: float

# 注册工具
@mcp.tool()
def add_numbers(input: AddNumbersInput) -> str:
    """将两个数字相加"""
    result = input.a + input.b
    return f"计算结果: {input.a} + {input.b} = {result}"

# 另一个工具示例
class MultiplyNumbersInput(BaseModel):
    a: float
    b: float

@mcp.tool()
def multiply_numbers(input: MultiplyNumbersInput) -> str:
    """将两个数字相乘"""
    result = input.a * input.b
    return f"计算结果: {input.a} × {input.b} = {result}"

if __name__ == "__main__":
    # 运行服务器
    mcp.run(transport="stdio")
```

### 使用原生MCP实现

```python
# src/server_native.py
import asyncio
from mcp.server import Server
from mcp.server.stdio import stdio_server
from mcp.types import Tool, TextContent, CallToolRequest

# 创建服务器
server = Server("python-mcp-server")

# 定义工具
@server.list_tools()
async def handle_list_tools() -> list[Tool]:
    return [
        Tool(
            name="calculate_sum",
            description="将两个数字相加",
            inputSchema={
                "type": "object",
                "properties": {
                    "a": {"type": "number", "description": "第一个数字"},
                    "b": {"type": "number", "description": "第二个数字"}
                },
                "required": ["a", "b"]
            }
        )
    ]

# 处理工具调用
@server.call_tool()
async def handle_call_tool(name: str, arguments: dict) -> list[TextContent]:
    if name == "calculate_sum":
        result = arguments["a"] + arguments["b"]
        return [
            TextContent(
                type="text",
                text=f"计算结果: {arguments['a']} + {arguments['b']} = {result}"
            )
        ]
    
    raise ValueError(f"未知工具: {name}")

async def main():
    async with stdio_server() as (read_stream, write_stream):
        await server.run(
            read_stream,
            write_stream,
            server.create_initialization_options()
        )

if __name__ == "__main__":
    asyncio.run(main())
```

## 工具实现

### 简单工具示例

```python
# src/tools/calculator.py
from pydantic import BaseModel
from fastmcp import FastMCP

mcp = FastMCP("Calculator Tools")

class CalculatorInput(BaseModel):
    a: float
    b: float

@mcp.tool()
def add(input: CalculatorInput) -> str:
    """将两个数字相加"""
    result = input.a + input.b
    return f"加法结果: {input.a} + {input.b} = {result}"

@mcp.tool()
def subtract(input: CalculatorInput) -> str:
    """将两个数字相减"""
    result = input.a - input.b
    return f"减法结果: {input.a} - {input.b} = {result}"

@mcp.tool()
def multiply(input: CalculatorInput) -> str:
    """将两个数字相乘"""
    result = input.a * input.b
    return f"乘法结果: {input.a} × {input.b} = {result}"

@mcp.tool()
def divide(input: CalculatorInput) -> str:
    """将两个数字相除"""
    if input.b == 0:
        return "错误: 除数不能为零"
    result = input.a / input.b
    return f"除法结果: {input.a} ÷ {input.b} = {result}"
```

### 文件系统工具

```python
# src/tools/file_system.py
import os
import pathlib
from typing import List, Dict, Any
from pydantic import BaseModel, Field
from fastmcp import FastMCP

mcp = FastMCP("File System Tools")

class ReadFileInput(BaseModel):
    filepath: str = Field(..., description="要读取的文件路径")

class ListDirectoryInput(BaseModel):
    directory: str = Field(..., description="要列出的目录路径")

@mcp.tool()
def read_file(input: ReadFileInput) -> str:
    """读取文件内容"""
    try:
        with open(input.filepath, 'r', encoding='utf-8') as f:
            content = f.read()
        return f"文件内容:\n\n{content}"
    except Exception as e:
        return f"读取文件错误: {e}"

@mcp.tool()
def list_directory(input: ListDirectoryInput) -> str:
    """列出目录内容"""
    try:
        path = pathlib.Path(input.directory)
        if not path.exists():
            return f"错误: 目录 '{input.directory}' 不存在"
        
        if not path.is_dir():
            return f"错误: '{input.directory}' 不是目录"
        
        items = []
        for item in path.iterdir():
            item_type = "目录" if item.is_dir() else "文件"
            size = item.stat().st_size if item.is_file() else 0
            items.append(f"{'📁' if item.is_dir() else '📄'} {item.name} ({item_type}, {size} 字节)")
        
        return "目录内容:\n\n" + "\n".join(items)
    except Exception as e:
        return f"列出目录错误: {e}"
```

### 数据科学工具

```python
# src/tools/data_science.py
import pandas as pd
import numpy as np
from pydantic import BaseModel, Field
from typing import Optional, List
from fastmcp import FastMCP

mcp = FastMCP("Data Science Tools")

class AnalyzeDataInput(BaseModel):
    data: List[List[float]] = Field(..., description="二维数据数组")
    analysis_type: str = Field("summary", description="分析类型: summary, stats, correlation")

@mcp.tool()
def analyze_data(input: AnalyzeDataInput) -> str:
    """分析数据"""
    try:
        df = pd.DataFrame(input.data)
        
        if input.analysis_type == "summary":
            result = f"数据摘要:\n{df.describe()}"
        elif input.analysis_type == "stats":
            result = f"统计信息:\n均值: {df.mean().to_dict()}\n标准差: {df.std().to_dict()}"
        elif input.analysis_type == "correlation":
            result = f"相关性矩阵:\n{df.corr()}"
        else:
            result = "未知分析类型"
        
        return result
    except Exception as e:
        return f"数据分析错误: {e}"
```

## 资源实现

### 文件系统资源

```python
# src/resources/file_system.py
from mcp.server.models import InitializationOptions
from mcp.types import Resource, TextContent
from fastmcp import FastMCP
import pathlib

mcp = FastMCP("File System Resources")

@mcp.resource("file://{path}")
def get_file_resource(path: str) -> Resource:
    """获取文件资源"""
    return Resource(
        uri=f"file://{path}",
        name=pathlib.Path(path).name,
        description=f"文件: {path}",
        mimeType="text/plain"
    )

@mcp.resource("dir://{path}")
def get_directory_resource(path: str) -> Resource:
    """获取目录资源"""
    return Resource(
        uri=f"dir://{path}",
        name=pathlib.Path(path).name,
        description=f"目录: {path}",
        mimeType="application/json"
    )

# 读取资源内容
async def read_file_resource(uri: str) -> list[TextContent]:
    """读取文件资源内容"""
    try:
        path = uri.replace("file://", "")
        with open(path, 'r', encoding='utf-8') as f:
            content = f.read()
        
        return [TextContent(type="text", text=content)]
    except Exception as e:
        return [TextContent(type="text", text=f"读取资源错误: {e}")]
```

### 集成资源到服务器

```python
# 在服务器中集成资源支持
from src.resources.file_system import get_file_resource, get_directory_resource

@mcp.list_resources()
async def handle_list_resources() -> list[Resource]:
    """列出可用资源"""
    # 这里可以动态生成资源列表
    return [
        get_file_resource("/example/path/file.txt"),
        get_directory_resource("/example/directory")
    ]

@mcp.read_resource()
async def handle_read_resource(uri: str) -> list[TextContent]:
    """读取资源内容"""
    if uri.startswith("file://"):
        return await read_file_resource(uri)
    elif uri.startswith("dir://"):
        return await read_directory_resource(uri)
    else:
        return [TextContent(type="text", text=f"不支持的资源类型: {uri}")]
```

## 提示实现

### 模板提示

```python
# src/prompts/templates.py
from pydantic import BaseModel, Field
from typing import Optional
from fastmcp import FastMCP

mcp = FastMCP("Prompt Templates")

class FileAnalysisPromptInput(BaseModel):
    filepath: str = Field(..., description="要分析的文件路径")
    analysis_type: str = Field("summary", description="分析类型")

class CodeReviewPromptInput(BaseModel):
    code_path: str = Field(..., description="代码文件或目录路径")
    language: Optional[str] = Field(None, description="编程语言")

@mcp.prompt("file_analysis")
def file_analysis_prompt(input: FileAnalysisPromptInput) -> str:
    """文件分析提示模板"""
    return f"""
请分析文件 {input.filepath} 的内容。
分析类型: {input.analysis_type}

请提供:
1. 文件内容的简要总结
2. 关键发现和见解
3. 任何建议或改进意见
"""

@mcp.prompt("code_review")
def code_review_prompt(input: CodeReviewPromptInput) -> str:
    """代码审查提示模板"""
    language_suffix = f" ({input.language})" if input.language else ""
    return f"""
请对 {input.code_path} 的代码进行审查{language_suffix}。

审查要点:
1. 代码质量和可读性
2. 潜在的错误和问题
3. 性能优化建议
4. 安全考虑
5. 最佳实践遵循情况
"""

@mcp.prompt("data_analysis")
def data_analysis_prompt() -> str:
    """数据分析提示模板"""
    return """
请分析提供的数据集。

分析要求:
1. 数据质量评估
2. 统计摘要
3. 趋势和模式识别
4. 异常值检测
5. 可视化建议
"""
```

## 结构化输出类型

### 使用Pydantic模型

```python
# src/models/structured_outputs.py
from pydantic import BaseModel, Field
from typing import List, Optional, Dict, Any
from datetime import datetime

class AnalysisResult(BaseModel):
    """分析结果模型"""
    summary: str = Field(..., description="分析摘要")
    key_findings: List[str] = Field(..., description="关键发现")
    recommendations: List[str] = Field(..., description="建议")
    confidence: float = Field(..., ge=0, le=1, description="分析置信度")
    timestamp: datetime = Field(default_factory=datetime.now)

class CodeReviewResult(BaseModel):
    """代码审查结果模型"""
    quality_score: float = Field(..., ge=0, le=10, description="代码质量评分")
    issues: List[str] = Field(..., description="发现的问题")
    suggestions: List[str] = Field(..., description="改进建议")
    security_concerns: List[str] = Field(..., description="安全考虑")

class DataAnalysisResult(BaseModel):
    """数据分析结果模型"""
    dataset_info: Dict[str, Any] = Field(..., description="数据集信息")
    statistical_summary: Dict[str, Any] = Field(..., description="统计摘要")
    insights: List[str] = Field(..., description="洞察发现")
    visualizations: List[str] = Field(..., description="可视化建议")
```

### 集成结构化输出

```python
# 在工具中使用结构化输出
from src.models.structured_outputs import AnalysisResult, CodeReviewResult

@mcp.tool()
def analyze_file_structured(input: FileAnalysisPromptInput) -> AnalysisResult:
    """结构化文件分析"""
    # 实现分析逻辑
    return AnalysisResult(
        summary="文件分析完成",
        key_findings=["发现关键模式", "识别潜在问题"],
        recommendations=["建议优化结构", "改进文档"],
        confidence=0.85
    )

@mcp.tool()
def review_code_structured(input: CodeReviewPromptInput) -> CodeReviewResult:
    """结构化代码审查"""
    return CodeReviewResult(
        quality_score=8.5,
        issues=["缺少错误处理", "代码重复"],
        suggestions=["添加异常处理", "提取公共函数"],
        security_concerns=["输入验证不足"]
    )
```

## 高级功能

### 配置管理

```python
# src/config.py
from pydantic import BaseSettings
from typing import List, Optional

class ServerConfig(BaseSettings):
    """服务器配置"""
    server_name: str = "python-mcp-server"
    version: str = "1.0.0"
    log_level: str = "INFO"
    allowed_paths: List[str] = ["/tmp", "/home"]
    max_file_size: int = 10 * 1024 * 1024  # 10MB
    
    class Config:
        env_file = ".env"
        env_prefix = "MCP_"

# 加载配置
config = ServerConfig()
```

### 错误处理

```python
# src/error_handling.py
from fastmcp import FastMCP
from pydantic import ValidationError

mcp = FastMCP("Error Handling")

@mcp.tool()
def safe_file_operation(filepath: str) -> str:
    """安全的文件操作"""
    try:
        # 验证文件路径
        if ".." in filepath:
            return "错误: 不允许使用相对路径"
        
        # 检查文件大小限制
        import os
        if os.path.getsize(filepath) > config.max_file_size:
            return f"错误: 文件大小超过限制 ({config.max_file_size} 字节)"
        
        # 执行文件操作
        with open(filepath, 'r') as f:
            content = f.read()
        
        return f"成功读取文件，大小: {len(content)} 字节"
        
    except FileNotFoundError:
        return "错误: 文件不存在"
    except PermissionError:
        return "错误: 没有文件访问权限"
    except Exception as e:
        return f"错误: {str(e)}"
```

### 生命周期管理

```python
# src/lifecycle.py
from contextlib import asynccontextmanager
from fastapi import FastAPI
from fastmcp import FastMCP

# 生命周期管理
@asynccontextmanager
async def lifespan(app: FastAPI):
    # 启动时执行
    print("MCP服务器启动中...")
    
    # 初始化资源
    await initialize_resources()
    
    yield
    
    # 关闭时执行
    print("MCP服务器关闭中...")
    await cleanup_resources()

async def initialize_resources():
    """初始化资源"""
    # 初始化数据库连接、缓存等
    pass

async def cleanup_resources():
    """清理资源"""
    # 关闭数据库连接、清理缓存等
    pass

# 创建带生命周期的服务器
mcp = FastMCP("Lifecycle Server", lifespan=lifespan)
```

## 传输选项

### Stdio传输（默认）

```python
# 使用stdio传输
mcp.run(transport="stdio")
```

### HTTP传输

```python
# 使用HTTP传输
mcp.run(transport="http", host="localhost", port=8000)
```

### WebSocket传输

```python
# 使用WebSocket传输
mcp.run(transport="ws", host="localhost", port=8080)
```

## 测试和质量保证

### 单元测试

```python
# tests/test_tools.py
import pytest
from src.tools.calculator import add, subtract

def test_add():
    """测试加法工具"""
    from src.tools.calculator import CalculatorInput
    
    input_data = CalculatorInput(a=5, b=3)
    result = add(input_data)
    
    assert "8" in result
    assert "5 + 3" in result

def test_subtract():
    """测试减法工具"""
    from src.tools.calculator import CalculatorInput
    
    input_data = CalculatorInput(a=10, b=4)
    result = subtract(input_data)
    
    assert "6" in result
    assert "10 - 4" in result
```

### 集成测试

```python
# tests/test_integration.py
import pytest
from fastmcp.testing import TestClient
from src.server import mcp

@pytest.fixture
def client():
    """创建测试客户端"""
    return TestClient(mcp)

def test_tool_listing(client):
    """测试工具列表功能"""
    tools = client.list_tools()
    assert len(tools) > 0
    assert any(tool.name == "add_numbers" for tool in tools)

def test_tool_execution(client):
    """测试工具执行"""
    result = client.call_tool("add_numbers", {"a": 2, "b": 3})
    assert "5" in result
```

### 性能测试

```python
# tests/test_performance.py
import time
import pytest
from src.tools.calculator import add

def test_performance():
    """性能测试"""
    from src.tools.calculator import CalculatorInput
    
    start_time = time.time()
    
    # 执行多次操作
    for i in range(1000):
        input_data = CalculatorInput(a=i, b=i+1)
        add(input_data)
    
    end_time = time.time()
    execution_time = end_time - start_time
    
    # 断言执行时间在合理范围内
    assert execution_time < 1.0  # 1秒内完成1000次操作
```

## 部署和运维

### 依赖管理

```toml
# pyproject.toml
[project]
name = "python-mcp-server"
version = "1.0.0"
description = "Python MCP服务器实现"

[project.dependencies]
fastmcp = "^1.0.0"
pydantic = "^2.0.0"
pandas = "^2.0.0"

[project.optional-dependencies]
dev = ["pytest", "pytest-asyncio", "black", "mypy"]

[build-system]
requires = ["setuptools", "wheel"]
build-backend = "setuptools.build_meta"
```

### Docker部署

```dockerfile
# Dockerfile
FROM python:3.11-slim

WORKDIR /app

# 复制依赖文件
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# 复制应用代码
COPY src/ ./src/

# 设置环境变量
ENV PYTHONPATH=/app/src
ENV MCP_LOG_LEVEL=INFO

# 运行服务器
CMD ["python", "-m", "src.server"]
```

### 监控和日志

```python
# src/monitoring.py
import logging
from datetime import datetime

# 配置日志
logging.basicConfig(
    level=getattr(logging, config.log_level),
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s'
)

logger = logging.getLogger("mcp-server")

def log_tool_usage(tool_name: str, execution_time: float, success: bool):
    """记录工具使用情况"""
    logger.info(
        f"工具调用 - 名称: {tool_name}, "
        f"执行时间: {execution_time:.3f}s, "
        f"成功: {success}"
    )

def log_error(tool_name: str, error: Exception):
    """记录错误"""
    logger.error(f"工具错误 - 名称: {tool_name}, 错误: {error}")
```

## 总结

本文档提供了使用Python实现MCP服务器的完整指南。Python实现具有以下优势：

1. **快速开发**: 使用FastMCP框架可以快速构建功能丰富的服务器
2. **强大的类型系统**: Pydantic提供强大的数据验证和类型提示
3. **丰富的生态系统**: 可以轻松集成数据科学、Web开发等领域的库
4. **易于测试**: 完善的测试框架支持

关键最佳实践包括：

- 使用FastMCP框架简化开发
- 利用Pydantic进行数据验证
- 实现适当的错误处理和日志记录
- 编写全面的测试用例
- 考虑安全性和性能优化

通过遵循这些指南，您可以构建高质量、可靠的Python MCP服务器。