---
name: api-doc-gen
description: "API文档生成：从代码和TDS自动生成API文档。"
version: 1.0.0
author: DevFlow AI
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [sdlc, 文档, api, openapi, 自动化]
    related_skills: [tds_driven_dev, code_review_tds, api_auto_regression]
---

# API文档生成技能

## 概述

从源代码、注释和TDS规范自动生成全面的API文档。支持OpenAPI 3.0、Markdown和交互式文档格式。

## 使用场景

- API实现完成后
- 代码审查之前
- 用于生成Wiki资产
- 发布准备期间

## 输入要求

### 源代码
- 已实现的API端点
- 存在文档字符串/注释
- 已定义请求/响应模型

### TDS文档
- 来自TDS第3节的API规范

## 阶段1：API提取

### 步骤1.1：扫描API端点

```python
# 从FastAPI/Flask/Express提取API端点
def scan_api_endpoints(code_dir: str) -> list[Endpoint]:
    endpoints = []
    
    for file in glob.glob(f"{code_dir}/**/*.py"):
        with open(file, 'r', encoding='utf-8') as f:
            content = f.read()
        
        # 查找路由装饰器
        routes = re.findall(r'@(router\.(get|post|put|delete|patch))', content)
        paths = re.findall(r'@router\.\w+\(["\']([^["\']+)["\']', content)
        
        for route, method, path in zip(routes, methods, paths):
            endpoints.append({
                "path": path,
                "method": method,
                "file": file,
                "line": find_line_number(content, f"@router.{method}")
            })
    
    return endpoints
```

### 步骤1.2：提取端点详细信息

针对每个端点：
```python
def extract_endpoint_details(endpoint: dict) -> dict:
    """提取完整的端点规范"""
    details = {
        "path": endpoint["path"],
        "method": endpoint["method"],
        "summary": "",
        "description": "",
        "parameters": [],
        "request_body": None,
        "responses": {},
        "tags": []
    }
    
    # 从文档字符串提取
    docstring = get_docstring(endpoint["file"], endpoint["line"])
    details["summary"] = parse_summary(docstring)
    details["description"] = parse_description(docstring)
    
    # 从类型提示和Pydantic模型提取
    details["parameters"] = extract_parameters(endpoint)
    details["request_body"] = extract_request_body(endpoint)
    details["responses"] = extract_responses(endpoint)
    
    return details
```

### 步骤1.3：链接到TDS需求

```python
# 将API端点映射到TDS需求
def link_to_tds(endpoints: list, tds_path: str) -> list:
    """将每个端点链接到其TDS需求"""
    tds_content = read_file(tds_path)
    
    for endpoint in endpoints:
        # 在TDS中查找匹配的需求
        requirement = find_tds_requirement(
            endpoint["path"], 
            tds_content
        )
        endpoint["tds_requirement"] = requirement
    
    return endpoints
```

## 阶段2：文档生成

### 步骤2.1：生成OpenAPI规范

```yaml
# 生成: openapi.yaml
openapi: 3.0.0
info:
  title: DevFlow AI API
  version: 1.0.0
  description: 从源代码自动生成

paths:
  /api/v1/projects:
    post:
      summary: 创建新项目
      description: 根据提供的数据创建新项目
      operationId: create_project
      tags:
        - projects
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/ProjectCreate'
      responses:
        '201':
          description: 项目创建成功
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Project'
```

### 步骤2.2：生成Markdown文档

```markdown
# API文档

## 项目API

### 创建项目
`POST /api/v1/projects`

在系统中创建新项目。

**需求:** REQ-TDS-003

#### 请求
```json
{
  "name": "项目名称",
  "description": "项目描述"
}
```

#### 响应
```json
{
  "id": "uuid",
  "name": "项目名称",
  "created_at": "2026-06-27T10:00:00Z"
}
```

#### 错误响应
| 状态码 | 描述 |
|--------|------|
| 400 | 验证错误 |
| 401 | 未授权 |
| 500 | 服务器内部错误 |
```

### 步骤2.3：生成交互式文档

```bash
# 生成Swagger UI
npx swagger-ui-react -o docs/api/swagger build/openapi.yaml

# 生成Redoc
npx redoc build build/openapi.yaml -o docs/api/redoc.html
```

## 阶段3：Wiki资产生成

### 步骤3.1：创建Wiki结构

```bash
# Wiki目录结构
mkdir -p wiki/
├── api/
│   ├── overview.md
│   ├── authentication.md
│   ├── endpoints/
│   │   ├── projects.md
│   │   ├── documents.md
│   │   └── agent.md
│   └── errors.md
├── models/
│   └── schemas.md
└── guides/
    ├── quickstart.md
    └── integration.md
```

### 步骤3.2：生成Wiki页面

```python
def generate_wiki_pages(endpoints: list) -> None:
    """生成可导入Wiki的markdown文件"""
    
    # 概览
    with open("wiki/api/overview.md", 'w', encoding='utf-8') as f:
        f.write("# API概览\n\n")
        f.write(f"端点总数: {len(endpoints)}\n\n")
        f.write("## 快速链接\n\n")
        for ep in endpoints:
            f.write(f"- [{ep['method'].upper()} {ep['path']}](endpoints/{ep['name']}.md)\n")
    
    # 单独端点文档
    for ep in endpoints:
        filename = f"wiki/api/endpoints/{ep['name']}.md"
        with open(filename, 'w', encoding='utf-8') as f:
            f.write(f"# {ep['summary']}\n\n")
            f.write(f"**端点:** {ep['method'].upper()} {ep['path']}\n\n")
            f.write(f"**TDS需求:** {ep.get('tds_requirement', '不适用')}\n\n")
            f.write(generate_endpoint_content(ep))
```

### 步骤3.3：生成API客户端SDK

```bash
# 生成客户端库
openapi-generator generate \
    -i build/openapi.yaml \
    -g python \
    -o sdk/python/

openapi-generator generate \
    -i build/openapi.yaml \
    -g typescript-fetch \
    -o sdk/typescript/
```

## 阶段4：验证

### 步骤4.1：文档审查检查清单

```
- [ ] 所有端点已文档化
- [ ] 所有参数已文档化
- [ ] 所有响应码已文档化
- [ ] 已提供示例
- [ ] 已链接TDS需求
- [ ] 已包含代码片段
- [ ] 错误码已文档化
```

### 步骤4.2：交叉引用验证

```bash
# 验证所有已文档化的端点存在于代码中
python verify_doc_coverage.py --docs wiki/ --code src/

# 验证所有代码端点已文档化
python verify_code_coverage.py --docs wiki/ --code src/
```

### 步骤4.3：生成文档索引

```markdown
# API文档索引

## 最新版本: 1.0.0
最后更新: 2026-06-27

### 入门指南
- [快速开始指南](guides/quickstart.md)
- [认证](authentication.md)

### API参考
- [项目API](endpoints/projects.md)
- [文档API](endpoints/documents.md)
- [智能体API](endpoints/agent.md)
- [审查API](endpoints/review.md)

### 模型
- [数据Schema](models/schemas.md)

### 变更日志
- [变更日志](changelog.md)
```

## 输出产物

| 产物 | 位置 | 格式 |
|------|------|------|
| OpenAPI规范 | `docs/api/openapi.yaml` | YAML |
| Markdown文档 | `wiki/api/` | Markdown |
| Swagger UI | `docs/api/swagger/` | HTML |
| SDK - Python | `sdk/python/` | Python |
| SDK - TypeScript | `sdk/typescript/` | TypeScript |
| Postman集合 | `docs/api/devflow-api.postman.json` | JSON |

## 质量检查清单

- [ ] 所有API端点已文档化
- [ ] 请求/响应示例已包含
- [ ] 错误码已文档化
- [ ] TDS需求已链接
- [ ] 交互式文档已生成
- [ ] SDK已生成
- [ ] Wiki资产已就绪

---

*技能版本: 1.0.0*  
*最后更新: 2026-06-27*