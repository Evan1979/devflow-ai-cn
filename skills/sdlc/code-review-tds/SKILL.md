---
name: code-review-tds
description: "基于TDS的代码审查：对照TDS需求自动审查代码实现。"
version: 1.0.0
author: DevFlow AI
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [sdlc, 代码审查, tds, 质量, 自动化]
    related_skills: [tds_driven_dev, api_doc_gen, api_auto_regression]
---

# 基于TDS的代码审查技能

## 概述

通过将实现与TDS(技术设计规格说明书)需求进行比较，执行自动化代码审查。此技能通过系统性验证确保代码质量和TDS合规性。

## 使用场景

- 合并拉取请求之前
- 完成实现步骤后
- IT质量评审阶段
- 自动化回归检查

## 输入要求

### TDS文档
- 位置: `requirements/TDS-{id}.md`
- 必须包含: 实现步骤、验收标准、API规范

### 代码仓库
- 必须已初始化Git
- 应遵循TDS中定义的项目结构
- 代码必须包含REQ-TDS-XXX注释

## 阶段1：TDS解析

### 步骤1.1：提取需求

解析TDS以提取：
```python
def extract_tds_requirements(tds_path: str) -> dict:
    """从TDS中提取所有可审查的需求"""
    requirements = {
        "features": [],      # 来自 ## 5. 验收标准
        "api_endpoints": [], # 来自 ## 3. 接口设计
        "steps": [],         # 来自 ## 4. 实现步骤
        "models": [],        # 来自 ## 2. 数据模型
        "constraints": []    # 来自 ## 6. 边界与约束
    }
    # 解析逻辑
    return requirements
```

### 步骤1.2：映射到代码

创建需求到代码的映射：
```python
requirement_map = {
    "REQ-TDS-001": {
        "description": "创建项目端点",
        "files": ["src/api/projects.py"],
        "acceptance_criteria": "返回带有唯一ID的项目"
    },
    "REQ-TDS-002": {
        "description": "项目数据模型",
        "files": ["src/models/project.py"],
        "acceptance_criteria": "验证必填字段"
    }
}
```

## 阶段2：代码分析

### 步骤2.1：静态分析

运行自动化检查：
```bash
# 语法和风格检查
ruff check src/
eslint src/

# 类型检查
mypy src/        # Python
tsc --noEmit     # TypeScript

# 安全扫描
bandit -r src/
semgrep --config=auto src/
```

### 步骤2.2：TDS合规性检查

验证需求可追溯性：
```bash
# 检查所有TDS需求是否在代码中有对应
grep -r "REQ-TDS-" src/ > requirements_coverage.txt
cat requirements_coverage.txt | wc -l  # 应等于TDS步骤数
```

### 步骤2.3：实现验证

针对TDS中的每个验收标准：
```python
def verify_acceptance_criterion(criterion: str, code_path: str) -> dict:
    """验证是否满足验收标准"""
    result = {
        "criterion": criterion,
        "status": "passed",  # passed/failed/partial
        "evidence": [],
        "suggestions": []
    }
    
    # 检查代码是否实现了需求
    if not has_implementation(criterion, code_path):
        result["status"] = "failed"
        result["suggestions"].append("添加实现: " + criterion)
    
    return result
```

## 阶段3：审查报告生成

### 步骤3.1：生成审查报告

```markdown
# 代码审查报告

## 项目: [名称]
## TDS引用: TDS-XXX
## 审查日期: YYYY-MM-DD

### 摘要
- 需求总数: X
- 已实现: X
- 缺失: X
- 部分完成: X

### 详细发现

| 需求 | 状态 | 证据 | 文件 |
|------|------|------|------|
| REQ-TDS-001 | ✓ 通过 | 代码实现了create_project | projects.py |
| REQ-TDS-002 | ⚠ 部分 | 缺少验证逻辑 | projects.py |

### 代码质量问题

1. **严重**: [问题描述]
2. **警告**: [问题描述]
3. **提示**: [问题描述]

### 建议

- [ ] 完成缺失的实现
- [ ] 为边界情况添加单元测试
- [ ] 修复代码质量问题
```

### 步骤3.2：计算指标

| 指标 | 公式 | 目标 |
|------|------|------|
| TDS覆盖率 | 已实现/总数 | > 95% |
| 测试覆盖率 | 已测试行数/总行数 | > 80% |
| 代码质量 | 问题数/代码行数 | < 0.1% |
| 文档完整性 | 文档字符串覆盖率 | 100% |

### 步骤3.3：生成操作项

```python
def generate_action_items(review_results: dict) -> list:
    """从审查结果生成可操作项"""
    actions = []
    
    for issue in review_results["issues"]:
        if issue["severity"] == "critical":
            actions.append({
                "type": "修复",
                "description": issue["description"],
                "优先级": "高",
                "负责人": issue["file_owner"]
            })
    
    return actions
```

## 阶段4：IT质量评审集成

### 步骤4.1：准备IT质量评审包

生成IT审查包：
```bash
# 打包审查产物
mkdir -p review/TDS-{id}/
cp review_report.md review/TDS-{id}/
cp -r code_coverage/ review/TDS-{id}/
cp test_results/ review/TDS-{id}/
zip -r review_package_TDS-{id}.zip review/TDS-{id}/
```

### 步骤4.2：IT质量评审检查清单

```
IT质量评审检查清单:
- [ ] 代码编译无错误
- [ ] 所有单元测试通过
- [ ] 集成测试通过
- [ ] 安全扫描通过
- [ ] TDS覆盖率 > 95%
- [ ] 代码审查已批准
- [ ] 文档完整
```

### 步骤4.3：结果评审

IT质量评审完成后：
```markdown
## IT质量评审结果

| 项目 | IT质量评审结果 | 备注 |
|------|----------------|------|
| 代码质量 | ✅ 批准 | - |
| TDS合规性 | ✅ 批准 | - |
| 测试覆盖率 | ⚠ 有条件批准 | 添加更多边界情况测试 |
| 安全 | ✅ 批准 | - |

## 下一步
- [ ] 合并到主分支
- [ ] 部署到预发布环境
- [ ] 安排用户验收测试
```

## 输出产物

| 产物 | 位置 | 格式 |
|------|------|------|
| 审查报告 | `review/TDS-{id}/report.md` | Markdown |
| 覆盖率报告 | `review/TDS-{id}/coverage.txt` | 文本 |
| 操作项 | `review/TDS-{id}/actions.md` | Markdown |
| IT质量评审结果 | `review/TDS-{id}/itqa_outcome.md` | Markdown |

## 质量检查清单

- [ ] 所有TDS需求已映射到代码
- [ ] 静态分析通过
- [ ] 代码编译无错误
- [ ] 审查报告已生成
- [ ] IT质量评审已安排
- [ ] 操作项已分配

## GitHub Copilot集成

```python
# 使用Copilot进行代码审查建议
# Copilot可以:
# - 为代码模式建议改进
# - 识别潜在bug
# - 提出测试用例
# - 生成文档

# 代码审查Copilot提示

# 查找未实现的功能
@copilot 查找不符合TDS需求 REQ-TDS-XXX 的代码

# 建议测试用例
@copilot 根据验收标准为此函数建议单元测试

# 识别安全问题
@copilot 检查此代码中的安全漏洞
```

---

*技能版本: 1.0.0*  
*最后更新: 2026-06-27*