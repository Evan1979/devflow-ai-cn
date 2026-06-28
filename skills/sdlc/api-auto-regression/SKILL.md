---
name: api-auto-regression
description: "API自动化回归测试：基于TDS和API规范的自动化API回归测试。"
version: 1.0.0
author: DevFlow AI
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [sdlc, 测试, api, 回归, 自动化]
    related_skills: [code_review_tds, api_doc_gen, bau_sop_gen]
---

# API自动化回归测试技能

## 概述

通过从TDS规范和API文档生成并执行测试用例，自动化API回归测试。确保代码更改不会破坏现有功能。

## 使用场景

- 任何影响API的代码更改后
- 合并拉取请求之前
- CI/CD流水线执行期间
- 用于夜间构建验证
- 发布部署前

## 输入要求

### OpenAPI规范
- 位置: `docs/api/openapi.yaml`
- 必须包含: 端点、请求/响应Schema

### TDS文档
- 位置: `requirements/TDS-{id}.md`
- 必须包含: 每个API的验收标准

### 测试数据
- 位置: `tests/fixtures/`
- 必须包含: 示例请求/响应数据

## 阶段1：测试用例生成

### 步骤1.1：解析API规范

```python
def parse_openapi_spec(spec_path: str) -> dict:
    """解析OpenAPI规范以提取可测试的端点"""
    with open(spec_path, 'r', encoding='utf-8') as f:
        spec = yaml.safe_load(f)
    
    test_cases = []
    
    for path, methods in spec['paths'].items():
        for method, details in methods.items():
            if method in ['get', 'post', 'put', 'delete', 'patch']:
                test_case = {
                    "endpoint": f"{method.upper()} {path}",
                    "path": path,
                    "method": method,
                    "parameters": details.get('parameters', []),
                    "request_body": details.get('requestBody'),
                    "responses": details.get('responses', {}),
                    "tags": details.get('tags', []),
                    "operation_id": details.get('operationId')
                }
                test_cases.append(test_case)
    
    return test_cases
```

### 步骤1.2：生成测试用例

针对每个API端点生成：
```python
def generate_test_cases(endpoint: dict, tds_acceptance: dict) -> list:
    """从端点和TDS生成测试用例"""
    test_cases = []
    
    # 成功路径测试
    test_cases.append({
        "name": f"{endpoint['operation_id']}_success",
        "type": "positive",
        "endpoint": endpoint["endpoint"],
        "status_code": 200,
        "validate_schema": True
    })
    
    # 必填字段缺失
    for param in endpoint.get('parameters', []):
        if param.get('required'):
            test_cases.append({
                "name": f"{endpoint['operation_id']}_missing_{param['name']}",
                "type": "negative",
                "endpoint": endpoint["endpoint"],
                "status_code": 422,
                "expected_error": "字段必填"
            })
    
    # 无效输入
    test_cases.append({
        "name": f"{endpoint['operation_id']}_invalid_input",
        "type": "negative",
        "endpoint": endpoint["endpoint"],
        "status_code": 400,
        "expected_error": "验证错误"
    })
    
    # TDS验收标准测试
    for criterion in tds_acceptance.get(endpoint['operation_id'], []):
        test_cases.append({
            "name": f"tds_{criterion['id']}",
            "type": "acceptance",
            "endpoint": endpoint["endpoint"],
            "status_code": criterion['expected_status'],
            "validation": criterion['validation']
        })
    
    return test_cases
```

### 步骤1.3：生成测试代码

```python
# 生成: tests/test_api_regression.py
import pytest
import requests

BASE_URL = "http://localhost:8000"

class TestProjectsAPI:
    """项目API自动化回归测试"""
    
    @pytest.mark.positive
    def test_create_project_success(self):
        """REQ-TDS-003: 成功创建项目"""
        response = requests.post(
            f"{BASE_URL}/api/v1/projects",
            json={"name": "测试", "description": "测试项目"}
        )
        assert response.status_code == 201
        data = response.json()
        assert "id" in data
        assert data["name"] == "测试"
    
    @pytest.mark.negative
    def test_create_project_missing_name(self):
        """REQ-TDS-003: 验证 - name为必填"""
        response = requests.post(
            f"{BASE_URL}/api/v1/projects",
            json={"description": "测试"}
        )
        assert response.status_code == 422
    
    @pytest.mark.acceptance
    def test_tds_acceptance_project_creation(self):
        """TDS验收: 项目创建成功并返回唯一ID"""
        response = requests.post(
            f"{BASE_URL}/api/v1/projects",
            json={"name": "验收测试"}
        )
        assert response.status_code == 201
        data = response.json()
        # 验证验收标准
        assert len(data["id"]) > 0  # 必须有唯一ID
```

## 阶段2：测试执行

### 步骤2.1：设置测试环境

```bash
# 启动测试环境
docker-compose up -d test-db test-api

# 等待服务启动
sleep 10

# 以测试模式启动后端
cd backend
source venv/bin/activate
DATABASE_URL="sqlite:///test.db" uvicorn app.main:app --port 8001 &
```

### 步骤2.2：执行测试套件

```bash
# 运行所有回归测试
pytest tests/test_api_regression.py \
    --verbose \
    --junitxml=results/regression_results.xml \
    --html=results/regression_report.html

# 运行特定测试类别
pytest tests/test_api_regression.py -m positive
pytest tests/test_api_regression.py -m negative
pytest tests/test_api_regression.py -m acceptance
```

### 步骤2.3：并行执行（加速）

```bash
# 并行运行测试
pytest tests/test_api_regression.py \
    -n auto \
    --dist loadscope

# 运行特定端点测试
pytest tests/test_api_regression.py -k "projects"
```

## 阶段3：结果分析

### 步骤3.1：生成测试报告

```markdown
# API回归测试报告

## 摘要
- 测试总数: 150
- 通过: 145
- 失败: 3
- 跳过: 2
- 执行时间: 45秒

## 失败的测试

| 测试名称 | 端点 | 状态 | 错误 |
|---------|------|------|------|
| test_create_project_duplicate | POST /api/v1/projects | ❌ 失败 | AssertionError |
| test_update_project_notfound | PUT /api/v1/projects/{id} | ❌ 失败 | 期望404，得到200 |

## TDS验收状态

| TDS需求 | 测试状态 | 备注 |
|---------|----------|------|
| REQ-TDS-003 | ✅ 通过 | 项目创建正常 |
| REQ-TDS-004 | ⚠ 部分 | 更新需要修复 |
| REQ-TDS-005 | ✅ 通过 | 删除正常 |

## 性能指标

| 指标 | 数值 | 目标 | 状态 |
|------|------|------|------|
| 平均响应时间 | 120ms | <200ms | ✅ 通过 |
| P95响应时间 | 280ms | <500ms | ✅ 通过 |
| 最大并发数 | 50 | >30 | ✅ 通过 |
```

### 步骤3.2：生成JUnit XML

```xml
<?xml version="1.0" encoding="UTF-8"?>
<testsuite name="API回归测试" tests="150" failures="3" time="45.123">
  <testcase classname="TestProjectsAPI" name="test_create_project_success" time="0.023"/>
  <testcase classname="TestProjectsAPI" name="test_create_project_duplicate" time="0.045">
    <failure message="AssertionError">期望状态409，得到200</failure>
  </testcase>
</testsuite>
```

### 步骤3.3：集成CI/CD

```yaml
# .github/workflows/api-regression.yml
name: API回归测试

on:
  pull_request:
    branches: [main, develop]
  push:
    branches: [main, develop]

jobs:
  regression:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: 设置Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.11'
      
      - name: 安装依赖
        run: |
          pip install -r requirements.txt
          pip install pytest pytest-xdist pytest-html
      
      - name: 启动API服务
        run: |
          uvicorn app.main:app --port 8001 &
          sleep 5
      
      - name: 运行回归测试
        run: |
          pytest tests/test_api_regression.py \
            --junitxml=results.xml \
            --html=report.html
      
      - name: 上传结果
        uses: actions/upload-artifact@v3
        if: always()
        with:
          name: regression-results
          path: |
            results.xml
            report.html
```

## 阶段4：IT质量评审集成

### 步骤4.1：准备IT质量评审包

```bash
# 打包测试结果供IT质量评审
mkdir -p itqa/regression/
cp results/regression_report.md itqa/regression/
cp results.xml itqa/regression/
cp -r test_logs/ itqa/regression/
zip -r itqa_regression_{date}.zip itqa/regression/
```

### 步骤4.2：IT质量评审验证检查清单

```
IT质量评审回归测试检查清单:
- [ ] 所有正向测试通过
- [ ] 所有负向测试通过  
- [ ] 所有TDS验收测试通过
- [ ] 响应时间在SLA内
- [ ] 未发现安全漏洞
- [ ] 测试覆盖率 > 80%
- [ ] 结果已打包供审查
```

### 步骤4.3：结果文档化

```markdown
## IT质量评审回归测试结果

| 类别 | 结果 | 详情 |
|------|------|------|
| 正向测试 | ✅ 100/100 通过 | 所有成功路径场景 |
| 负向测试 | ✅ 45/45 通过 | 所有错误处理正常 |
| TDS验收 | ✅ 10/10 通过 | 所有验收标准满足 |
| 性能 | ✅ 通过 | 平均120ms < 200ms目标 |
| 安全 | ✅ 通过 | 未发现漏洞 |

### IT质量评审签署
- ✅ 批准部署
- 评审人: [姓名]
- 日期: 2026-06-27
```

## 输出产物

| 产物 | 位置 | 格式 |
|------|------|------|
| 测试套件 | `tests/test_api_regression.py` | Python/pytest |
| 测试结果 | `results/regression_report.html` | HTML |
| JUnit XML | `results.xml` | XML |
| 测试数据 | `tests/fixtures/` | JSON |
| CI/CD配置 | `.github/workflows/api-regression.yml` | YAML |
| IT质量评审包 | `itqa/regression/` | Zip |

## 质量检查清单

- [ ] 所有端点已生成测试用例
- [ ] TDS验收标准已覆盖
- [ ] 正向和负向测试已包含
- [ ] 测试在CI/CD流水线中执行
- [ ] 结果已报告给利益相关方
- [ ] IT质量评审已完成
- [ ] 性能基准已包含

---

*技能版本: 1.0.0*  
*最后更新: 2026-06-27*