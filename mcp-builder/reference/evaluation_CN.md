# MCP服务器评估指南

本文档提供了评估模型上下文协议（MCP）服务器的完整框架和方法。评估是确保MCP服务器质量和可靠性的关键步骤。

## 概述

MCP服务器评估旨在验证服务器是否能够：

- 正确响应代理的请求
- 提供准确和有用的工具功能
- 处理各种边界情况和错误条件
- 在真实使用场景中表现良好

## 评估要求

### 评估环境设置

在开始评估之前，需要准备以下环境：

1. **Anthropic API密钥**: 用于与Claude模型交互
2. **MCP服务器**: 需要评估的目标服务器
3. **评估脚本**: 提供的自动化评估工具
4. **测试数据**: 用于评估的示例问题和场景

### 评估问题设计

设计评估问题时需要考虑以下原则：

#### 问题类型

1. **功能验证问题**: 测试单个工具的基本功能
   - 示例: "使用search_tasks工具查找包含'bug'标签的任务"

2. **工作流问题**: 测试多个工具的组合使用
   - 示例: "创建一个新项目，添加三个任务，然后分配给团队成员"

3. **边界情况问题**: 测试错误处理和边界条件
   - 示例: "使用get_user工具查询不存在的用户ID"

4. **性能问题**: 测试响应时间和资源使用
   - 示例: "检索最近100个项目的详细信息"

#### 问题设计指南

- **明确具体**: 问题应该清晰明确，避免歧义
- **可验证**: 应该有明确的正确答案或验证标准
- **多样性**: 覆盖不同的使用场景和工具组合
- **现实性**: 基于真实的使用场景设计问题

## 评估文件格式

评估文件使用XML格式，包含`<qa_pair>`元素：

```xml
<evaluation>
   <qa_pair>
      <question>查找在2024年第二季度创建且已完成任务数量最多的项目。项目名称是什么？</question>
      <answer>网站重新设计</answer>
   </qa_pair>
   <qa_pair>
      <question>搜索在2024年3月关闭且标记为"bug"的问题。哪位用户关闭的问题最多？提供他们的用户名。</question>
      <answer>sarah_dev</answer>
   </qa_pair>
</evaluation>
```

## 运行评估

评估脚本（`scripts/evaluation.py`）支持三种传输类型：

**重要说明：**
- **stdio传输**: 评估脚本会自动启动和管理MCP服务器进程。不需要手动运行服务器。
- **sse/http传输**: 必须在运行评估之前单独启动MCP服务器。脚本会连接到指定URL上已运行的服务器。

### 1. 本地STDIO服务器

对于本地运行的MCP服务器（脚本自动启动服务器）：

```bash
python scripts/evaluation.py \
  -t stdio \
  -c python \
  -a my_mcp_server.py \
  evaluation.xml
```

使用环境变量：
```bash
python scripts/evaluation.py \
  -t stdio \
  -c python \
  -a my_mcp_server.py \
  -e API_KEY=abc123 \
  -e DEBUG=true \
  evaluation.xml
```

### 2. 服务器发送事件（SSE）

对于基于SSE的MCP服务器（必须先启动服务器）：

```bash
python scripts/evaluation.py \
  -t sse \
  -u https://example.com/mcp \
  -H "Authorization: Bearer token123" \
  -H "X-Custom-Header: value" \
  evaluation.xml
```

### 3. HTTP（可流式HTTP）

对于基于HTTP的MCP服务器（必须先启动服务器）：

```bash
python scripts/evaluation.py \
  -t http \
  -u https://example.com/mcp \
  -H "Authorization: Bearer token123" \
  evaluation.xml
```

## 命令行选项

```
用法: evaluation.py [-h] [-t {stdio,sse,http}] [-m MODEL] [-c COMMAND]
                     [-a ARGS [ARGS ...]] [-e ENV [ENV ...]] [-u URL]
                     [-H HEADERS [HEADERS ...]] [-o OUTPUT]
                     eval_file

位置参数:
  eval_file             评估XML文件路径

可选参数:
  -h, --help           显示帮助信息
  -t, --transport      传输类型: stdio, sse 或 http (默认: stdio)
  -m, --model          使用的Claude模型 (默认: claude-3-7-sonnet-20250219)
  -o, --output         报告输出文件 (默认: 输出到标准输出)

stdio选项:
  -c, --command        运行MCP服务器的命令 (例如: python, node)
  -a, --args           命令的参数 (例如: server.py)
  -e, --env            环境变量，格式为 KEY=VALUE

sse/http选项:
  -u, --url            MCP服务器URL
  -H, --header         HTTP头信息，格式为 'Key: Value'
```

## 输出结果

评估脚本生成详细的报告，包括：

- **摘要统计**:
  - 准确率（正确/总数）
  - 平均任务持续时间
  - 每个任务的平均工具调用次数
  - 总工具调用次数

- **每个任务的结果**:
  - 提示和预期响应
  - 代理的实际响应
  - 答案是否正确（✅/❌）
  - 持续时间和工具调用详情
  - 代理对其方法的总结
  - 代理对工具的反馈

### 保存报告到文件

```bash
python scripts/evaluation.py \
  -t stdio \
  -c python \
  -a my_server.py \
  -o evaluation_report.md \
  evaluation.xml
```

## 完整示例工作流

以下是创建和运行评估的完整示例：

1. **创建评估文件** (`my_evaluation.xml`):

```xml
<evaluation>
   <qa_pair>
      <question>查找在2024年1月创建问题最多的用户。他们的用户名是什么？</question>
      <answer>alice_developer</answer>
   </qa_pair>
   <qa_pair>
      <question>在2024年第一季度合并的所有拉取请求中，哪个仓库的数量最多？提供仓库名称。</question>
      <answer>backend-api</answer>
   </qa_pair>
   <qa_pair>
      <question>查找在2023年12月完成且从开始到结束持续时间最长的项目。花费了多少天？</question>
      <answer>127</answer>
   </qa_pair>
</evaluation>
```

2. **安装依赖**:

```bash
pip install -r scripts/requirements.txt
export ANTHROPIC_API_KEY=your_api_key
```

3. **运行评估**:

```bash
python scripts/evaluation.py \
  -t stdio \
  -c python \
  -a github_mcp_server.py \
  -e GITHUB_TOKEN=ghp_xxx \
  -o github_eval_report.md \
  my_evaluation.xml
```

4. **查看报告** 在 `github_eval_report.md` 中：
   - 查看哪些问题通过/失败
   - 阅读代理对工具的反馈
   - 识别需要改进的领域
   - 迭代改进MCP服务器设计

## 故障排除

### 连接错误

如果出现连接错误：
- **STDIO**: 验证命令和参数是否正确
- **SSE/HTTP**: 检查URL是否可访问且头信息正确
- 确保所有必需的API密钥已在环境变量或头信息中设置

### 准确率低

如果许多评估失败：
- 查看每个任务的代理反馈
- 检查工具描述是否清晰全面
- 验证输入参数是否文档完善
- 考虑工具返回的数据是否过多或过少
- 确保错误消息具有可操作性

### 超时问题

如果任务超时：
- 使用更强大的模型（例如 `claude-3-7-sonnet-20250219`）
- 检查工具是否返回过多数据
- 验证分页是否正常工作
- 考虑简化复杂问题

## 评估最佳实践

### 设计有效的评估问题

1. **覆盖所有工具**: 确保评估涵盖服务器提供的所有工具
2. **测试边界情况**: 包括无效输入、缺失参数等场景
3. **模拟真实使用**: 基于实际用户场景设计问题
4. **渐进式复杂度**: 从简单问题开始，逐步增加复杂度

### 分析评估结果

1. **识别模式**: 查看失败问题的共同特征
2. **工具反馈**: 重点关注代理对工具的改进建议
3. **性能指标**: 分析响应时间和资源使用情况
4. **错误分析**: 分类和统计不同类型的错误

### 迭代改进

基于评估结果进行改进：

1. **工具描述优化**: 根据代理反馈改进工具描述
2. **错误处理增强**: 改进错误消息和边界情况处理
3. **性能优化**: 优化响应时间和资源使用
4. **功能扩展**: 根据评估发现的需求缺口添加新功能

## 高级评估技术

### 自动化评估流水线

设置持续集成流水线自动运行评估：

```yaml
# GitHub Actions 示例
name: MCP评估
on: [push, pull_request]

jobs:
  evaluate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: 设置Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.9'
      - name: 安装依赖
        run: pip install -r scripts/requirements.txt
      - name: 运行评估
        run: |
          python scripts/evaluation.py \
            -t stdio \
            -c python \
            -a my_mcp_server.py \
            -o evaluation_report.md \
            evaluation.xml
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
```

### 自定义评估指标

扩展评估框架支持自定义指标：

```python
class CustomEvaluator:
    def evaluate_tool_quality(self, tool_responses):
        """评估工具响应质量"""
        # 实现自定义质量评估逻辑
        pass
    
    def evaluate_workflow_efficiency(self, workflow_data):
        """评估工作流效率"""
        # 实现工作流效率评估逻辑
        pass
```

### 多模型评估

使用不同的AI模型进行交叉评估：

```bash
# 使用不同模型运行评估
python scripts/evaluation.py -m claude-3-5-sonnet-20241022 evaluation.xml
python scripts/evaluation.py -m claude-3-haiku-20240307 evaluation.xml
```

## 评估报告解读

### 摘要统计解读

- **准确率**: 反映服务器的基本功能正确性
- **平均持续时间**: 指示服务器响应速度
- **工具调用次数**: 反映任务复杂度和工作流效率

### 详细结果分析

- **成功案例**: 分析成功任务的共同特征
- **失败模式**: 识别失败任务的模式和改进方向
- **代理反馈**: 重点关注代理的使用体验和改进建议

### 性能基准

建立性能基准用于后续比较：

- **响应时间基准**: 建立可接受的响应时间范围
- **准确率目标**: 设定准确率改进目标
- **资源使用限制**: 定义资源使用上限

通过系统性的评估和改进，可以显著提升MCP服务器的质量和可靠性。