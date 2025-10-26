# Node.js/TypeScript MCP服务器实现指南

本文档提供了使用Node.js和TypeScript实现模型上下文协议（MCP）服务器的全面指南。

## 概述

Node.js MCP服务器实现基于`@modelcontextprotocol/sdk`包，该包提供了构建MCP服务器的核心功能。

### 核心概念

- **MCP服务器**: 实现MCP协议的Node.js应用程序
- **工具**: 服务器提供的可调用功能
- **资源**: 服务器管理的可读数据源
- **提示**: 服务器提供的预定义模板

## 项目设置

### 初始化项目

```bash
# 创建新项目目录
mkdir my-mcp-server
cd my-mcp-server

# 初始化npm项目
npm init -y

# 安装TypeScript和开发依赖
npm install -D typescript @types/node ts-node

# 安装MCP SDK
npm install @modelcontextprotocol/sdk
```

### TypeScript配置

创建`tsconfig.json`文件：

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "CommonJS",
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

### 项目结构

```
my-mcp-server/
├── src/
│   ├── server.ts          # 主服务器文件
│   ├── tools/             # 工具实现
│   │   ├── calculator.ts
│   │   └── fileSystem.ts
│   ├── resources/         # 资源实现
│   │   └── fileSystem.ts
│   └── prompts/           # 提示实现
│       └── templates.ts
├── package.json
├── tsconfig.json
└── README.md
```

## 基本服务器实现

### 创建基础服务器

```typescript
// src/server.ts
import { Server } from '@modelcontextprotocol/sdk/server/index.js';
import { StdioServerTransport } from '@modelcontextprotocol/sdk/server/stdio.js';
import {
  CallToolRequestSchema,
  ListToolsRequestSchema,
} from '@modelcontextprotocol/sdk/types.js';

// 创建服务器实例
const server = new Server(
  {
    name: 'my-mcp-server',
    version: '1.0.0',
  },
  {
    capabilities: {
      tools: {},
    },
  }
);

// 设置工具列表处理器
server.setRequestHandler(ListToolsRequestSchema, async () => {
  return {
    tools: [
      {
        name: 'calculate_sum',
        description: '将两个数字相加',
        inputSchema: {
          type: 'object',
          properties: {
            a: { type: 'number', description: '第一个数字' },
            b: { type: 'number', description: '第二个数字' }
          },
          required: ['a', 'b']
        }
      }
    ]
  };
});

// 设置工具调用处理器
server.setRequestHandler(CallToolRequestSchema, async (request) => {
  if (request.params.name === 'calculate_sum') {
    const { a, b } = request.params.arguments as { a: number; b: number };
    const result = a + b;
    
    return {
      content: [
        {
          type: 'text',
          text: `计算结果: ${a} + ${b} = ${result}`
        }
      ]
    };
  }
  
  throw new Error(`未知工具: ${request.params.name}`);
});

// 启动服务器
async function main() {
  const transport = new StdioServerTransport();
  await server.connect(transport);
  console.error('MCP服务器已启动并运行在stdio上');
}

main().catch((error) => {
  console.error('服务器错误:', error);
  process.exit(1);
});
```

### 运行服务器

在`package.json`中添加启动脚本：

```json
{
  "scripts": {
    "build": "tsc",
    "start": "node dist/server.js",
    "dev": "ts-node src/server.ts"
  }
}
```

## 工具实现

### 简单工具示例

```typescript
// src/tools/calculator.ts
export const calculatorTools = [
  {
    name: 'calculate_sum',
    description: '将两个数字相加',
    inputSchema: {
      type: 'object',
      properties: {
        a: { type: 'number', description: '第一个数字' },
        b: { type: 'number', description: '第二个数字' }
      },
      required: ['a', 'b']
    }
  },
  {
    name: 'calculate_product',
    description: '将两个数字相乘',
    inputSchema: {
      type: 'object',
      properties: {
        a: { type: 'number', description: '第一个数字' },
        b: { type: 'number', description: '第二个数字' }
      },
      required: ['a', 'b']
    }
  }
];

export async function handleCalculatorTool(name: string, args: any) {
  switch (name) {
    case 'calculate_sum':
      return {
        content: [
          {
            type: 'text',
            text: `计算结果: ${args.a} + ${args.b} = ${args.a + args.b}`
          }
        ]
      };
    
    case 'calculate_product':
      return {
        content: [
          {
            type: 'text',
            text: `计算结果: ${args.a} × ${args.b} = ${args.a * args.b}`
          }
        ]
      };
    
    default:
      throw new Error(`未知计算工具: ${name}`);
  }
}
```

### 文件系统工具

```typescript
// src/tools/fileSystem.ts
import * as fs from 'fs/promises';
import * as path from 'path';

export const fileSystemTools = [
  {
    name: 'read_file',
    description: '读取文件内容',
    inputSchema: {
      type: 'object',
      properties: {
        filepath: { type: 'string', description: '文件路径' }
      },
      required: ['filepath']
    }
  },
  {
    name: 'list_directory',
    description: '列出目录内容',
    inputSchema: {
      type: 'object',
      properties: {
        directory: { type: 'string', description: '目录路径' }
      },
      required: ['directory']
    }
  }
];

export async function handleFileSystemTool(name: string, args: any) {
  switch (name) {
    case 'read_file':
      try {
        const content = await fs.readFile(args.filepath, 'utf-8');
        return {
          content: [
            {
              type: 'text',
              text: `文件内容:\n\n${content}`
            }
          ]
        };
      } catch (error) {
        return {
          isError: true,
          content: [
            {
              type: 'text',
              text: `读取文件错误: ${error}`
            }
          ]
        };
      }
    
    case 'list_directory':
      try {
        const items = await fs.readdir(args.directory);
        const itemDetails = await Promise.all(
          items.map(async (item) => {
            const itemPath = path.join(args.directory, item);
            const stats = await fs.stat(itemPath);
            return {
              name: item,
              type: stats.isDirectory() ? 'directory' : 'file',
              size: stats.size,
              modified: stats.mtime.toISOString()
            };
          })
        );
        
        const formattedList = itemDetails.map(item => 
          `${item.type === 'directory' ? '📁' : '📄'} ${item.name} (${item.type}, ${item.size} bytes)`
        ).join('\n');
        
        return {
          content: [
            {
              type: 'text',
              text: `目录内容:\n\n${formattedList}`
            }
          ]
        };
      } catch (error) {
        return {
          isError: true,
          content: [
            {
              type: 'text',
              text: `列出目录错误: ${error}`
            }
          ]
        };
      }
    
    default:
      throw new Error(`未知文件系统工具: ${name}`);
  }
}
```

## 资源实现

### 文件系统资源

```typescript
// src/resources/fileSystem.ts
import { ResourceTemplate } from '@modelcontextprotocol/sdk/types.js';
import * as fs from 'fs/promises';
import * as path from 'path';

export const fileSystemResources: ResourceTemplate[] = [
  {
    uri: 'file://{path}',
    name: 'file',
    description: '文件系统资源',
    mimeType: 'text/plain'
  }
];

export async function readFileSystemResource(uri: string) {
  const url = new URL(uri);
  const filepath = decodeURIComponent(url.pathname);
  
  try {
    const content = await fs.readFile(filepath, 'utf-8');
    return {
      contents: [
        {
          uri: uri,
          mimeType: 'text/plain',
          text: content
        }
      ]
    };
  } catch (error) {
    throw new Error(`读取文件资源错误: ${error}`);
  }
}
```

### 集成资源到服务器

```typescript
// 在服务器中添加资源支持
import {
  ListResourcesRequestSchema,
  ReadResourceRequestSchema
} from '@modelcontextprotocol/sdk/types.js';
import { fileSystemResources, readFileSystemResource } from './resources/fileSystem.js';

// 设置资源列表处理器
server.setRequestHandler(ListResourcesRequestSchema, async () => {
  return {
    resources: fileSystemResources
  };
});

// 设置资源读取处理器
server.setRequestHandler(ReadResourceRequestSchema, async (request) => {
  return await readFileSystemResource(request.params.uri);
});
```

## 提示实现

### 模板提示

```typescript
// src/prompts/templates.ts
import { Prompt } from '@modelcontextprotocol/sdk/types.js';

export const promptTemplates: Prompt[] = [
  {
    name: 'file_analysis',
    description: '分析文件内容的提示模板',
    arguments: [
      {
        name: 'filepath',
        description: '要分析的文件路径',
        required: true
      },
      {
        name: 'analysis_type',
        description: '分析类型',
        required: false,
        default: 'summary'
      }
    ]
  },
  {
    name: 'code_review',
    description: '代码审查提示模板',
    arguments: [
      {
        name: 'code_path',
        description: '代码文件或目录路径',
        required: true
      },
      {
        name: 'language',
        description: '编程语言',
        required: false
      }
    ]
  }
];

export function getPromptTemplate(name: string, args: Record<string, any>) {
  switch (name) {
    case 'file_analysis':
      return {
        messages: [
          {
            role: 'user',
            content: {
              type: 'text',
              text: `请分析文件 ${args.filepath} 的内容。分析类型: ${args.analysis_type || 'summary'}`
            }
          }
        ]
      };
    
    case 'code_review':
      return {
        messages: [
          {
            role: 'user',
            content: {
              type: 'text',
              text: `请对 ${args.code_path} 的代码进行审查。${args.language ? `编程语言: ${args.language}` : ''}`
            }
          }
        ]
      };
    
    default:
      throw new Error(`未知提示模板: ${name}`);
  }
}
```

### 集成提示到服务器

```typescript
// 在服务器中添加提示支持
import {
  GetPromptRequestSchema,
  ListPromptsRequestSchema
} from '@modelcontextprotocol/sdk/types.js';
import { promptTemplates, getPromptTemplate } from './prompts/templates.js';

// 设置提示列表处理器
server.setRequestHandler(ListPromptsRequestSchema, async () => {
  return {
    prompts: promptTemplates
  };
});

// 设置提示获取处理器
server.setRequestHandler(GetPromptRequestSchema, async (request) => {
  return getPromptTemplate(request.params.name, request.params.arguments || {});
});
```

## 高级功能

### 错误处理

```typescript
// 增强的错误处理
server.setRequestHandler(CallToolRequestSchema, async (request) => {
  try {
    // 工具处理逻辑
    return await handleToolRequest(request.params.name, request.params.arguments);
  } catch (error) {
    console.error('工具调用错误:', error);
    
    return {
      isError: true,
      content: [
        {
          type: 'text',
          text: `工具执行错误: ${error.message}`
        }
      ]
    };
  }
});
```

### 配置管理

```typescript
// src/config.ts
export interface ServerConfig {
  port?: number;
  logLevel: 'error' | 'warn' | 'info' | 'debug';
  allowedPaths?: string[];
  maxFileSize?: number;
}

export function loadConfig(): ServerConfig {
  return {
    port: parseInt(process.env.MCP_PORT || '8000'),
    logLevel: (process.env.MCP_LOG_LEVEL || 'info') as ServerConfig['logLevel'],
    allowedPaths: process.env.MCP_ALLOWED_PATHS?.split(','),
    maxFileSize: parseInt(process.env.MCP_MAX_FILE_SIZE || '10485760') // 10MB
  };
}
```

### 日志记录

```typescript
// src/logger.ts
export class Logger {
  constructor(private level: 'error' | 'warn' | 'info' | 'debug') {}
  
  error(message: string, ...args: any[]) {
    if (this.level === 'error' || this.level === 'warn' || this.level === 'info' || this.level === 'debug') {
      console.error(`[ERROR] ${message}`, ...args);
    }
  }
  
  info(message: string, ...args: any[]) {
    if (this.level === 'info' || this.level === 'debug') {
      console.log(`[INFO] ${message}`, ...args);
    }
  }
  
  debug(message: string, ...args: any[]) {
    if (this.level === 'debug') {
      console.log(`[DEBUG] ${message}`, ...args);
    }
  }
}
```

## 传输选项

### Stdio传输（默认）

```typescript
import { StdioServerTransport } from '@modelcontextprotocol/sdk/server/stdio.js';

const transport = new StdioServerTransport();
await server.connect(transport);
```

### WebSocket传输

```typescript
import { WebSocketServerTransport } from '@modelcontextprotocol/sdk/server/ws.js';

const transport = new WebSocketServerTransport({
  port: 8080,
  host: 'localhost'
});
await server.connect(transport);
```

### HTTP传输

```typescript
import { HTTPServerTransport } from '@modelcontextprotocol/sdk/server/http.js';

const transport = new HTTPServerTransport({
  port: 3000,
  host: 'localhost'
});
await server.connect(transport);
```

## 通知支持

### 工具变更通知

```typescript
// 当工具列表变更时发送通知
async function notifyToolsChanged() {
  await server.notify({
    method: 'notifications/tools/list_changed',
    params: {}
  });
}
```

### 资源变更通知

```typescript
// 当资源列表变更时发送通知
async function notifyResourcesChanged() {
  await server.notify({
    method: 'notifications/resources/list_changed',
    params: {}
  });
}
```

## 代码最佳实践

### 模块化设计

1. **分离关注点**: 将工具、资源、提示逻辑分离到不同模块
2. **可重用组件**: 创建可重用的辅助函数和类
3. **配置驱动**: 使用配置文件管理服务器设置
4. **错误处理**: 实现统一的错误处理机制

### 性能优化

1. **异步操作**: 对所有I/O操作使用异步处理
2. **缓存策略**: 对频繁访问的数据实现缓存
3. **连接池**: 对数据库连接使用连接池
4. **内存管理**: 监控内存使用，避免泄漏

### 安全性

1. **输入验证**: 验证所有输入参数
2. **路径清理**: 防止路径遍历攻击
3. **权限控制**: 实现适当的访问控制
4. **错误信息**: 避免暴露敏感错误信息

## 构建和运行

### 开发模式

```bash
# 使用ts-node直接运行
npm run dev

# 或使用nodemon自动重启
npm install -D nodemon
npx nodemon --exec ts-node src/server.ts
```

### 生产构建

```bash
# 编译TypeScript
npm run build

# 运行编译后的代码
npm start
```

### Docker部署

```dockerfile
# Dockerfile
FROM node:18-alpine

WORKDIR /app

COPY package*.json ./
RUN npm ci --only=production

COPY dist/ ./dist/

EXPOSE 3000

CMD ["node", "dist/server.js"]
```

## 测试和质量保证

### 单元测试

```typescript
// src/tests/calculator.test.ts
import { handleCalculatorTool } from '../tools/calculator.js';

describe('Calculator Tools', () => {
  test('calculate_sum should add numbers correctly', async () => {
    const result = await handleCalculatorTool('calculate_sum', { a: 2, b: 3 });
    expect(result.content[0].text).toContain('5');
  });
});
```

### 集成测试

```typescript
// 测试完整的服务器功能
import { Server } from '@modelcontextprotocol/sdk/server/index.js';

describe('MCP Server Integration', () => {
  let server: Server;
  
  beforeEach(() => {
    server = createTestServer();
  });
  
  afterEach(async () => {
    await server.close();
  });
  
  test('should list available tools', async () => {
    // 测试工具列表功能
  });
});
```

## 故障排除

### 常见问题

1. **连接问题**: 检查传输配置和端口可用性
2. **工具调用失败**: 验证工具参数和实现逻辑
3. **资源访问错误**: 检查文件权限和路径正确性
4. **性能问题**: 监控资源使用和优化代码

### 调试技巧

1. **启用详细日志**: 设置日志级别为debug
2. **使用调试器**: 使用Node.js调试工具
3. **监控网络流量**: 检查MCP协议通信
4. **性能分析**: 使用性能分析工具

## 总结

本文档提供了使用Node.js和TypeScript实现MCP服务器的完整指南。通过遵循这些最佳实践，您可以构建高质量、可靠且功能丰富的MCP服务器。关键要点包括：

1. **正确的项目结构**: 组织代码以支持可维护性
2. **全面的功能实现**: 支持工具、资源和提示
3. **强大的错误处理**: 确保服务器稳定性
4. **安全性考虑**: 保护服务器免受攻击
5. **性能优化**: 提供良好的用户体验

通过系统性地应用这些实践，您可以创建在真实世界场景中表现良好的MCP服务器。