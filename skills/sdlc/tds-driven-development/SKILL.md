---
name: tds-driven-development
description: "技术设计规格驱动开发：基于TDS需求实现代码。"
version: 1.0.0
author: DevFlow AI
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [sdlc, 开发, tds, 实现]
    related_skills: [fs_to_tds, code_review_tds, api_doc_gen]
---

# TDS驱动开发技能

## 概述

基于技术设计规格说明书(TDS)执行开发。此技能确保所有代码实现与预定义的技术设计和验收标准保持一致。

## 使用场景

- TDS批准后开始实现
- 需要根据TDS需求跟踪进度
- 期望实现需求到代码的自动化可追溯性

## 输入要求

### TDS文档（必需）
```markdown
# 技术设计规格说明书 (TDS)

## 1. 项目概述
## 2. 技术方案
## 3. 接口设计
## 4. 实现步骤
### 步骤1: [描述]
### 步骤2: [描述]
## 5. 验收标准
## 6. 边界与约束
```

### 依赖项
- 开发环境已配置
- Git仓库已初始化
- 所需工具已安装

## 阶段1：TDS解析

### 步骤1.1：加载TDS文档

```python
# 解析TDS并提取实现步骤
def parse_tds(tds_path: str) -> dict:
    with open(tds_path, 'r', encoding='utf-8') as f:
        content = f.read()
    
    steps = extract_steps(content)  # 提取 ## 4. 实现步骤
    apis = extract_apis(content)     # 提取 ## 3. 接口设计
    models = extract_models(content) # 提取 ## 2. 技术方案
    
    return {"steps": steps, "apis": apis, "models": models}
```

### 步骤1.2：生成实现计划

创建任务分解：
- 任务1：初始化项目结构
- 任务2：实现数据模型
- 任务3：创建API端点
- 任务4：实现业务逻辑
- 任务5：添加测试
- 任务6：集成测试

## 阶段2：实现

### 步骤2.1：项目初始化

```bash
# REQ-TDS-001: 初始化项目结构
mkdir -p src/{api,models,services,utils,tests}
git init
git checkout -b develop
```

### 步骤2.2：数据模型实现

对于TDS中的每个数据模型：
1. 创建模型类
2. 添加验证规则
3. 创建数据库迁移
4. 编写单元测试

```python
# REQ-TDS-002: 实现数据模型
class Project(BaseModel):
    """REQ-TDS-002: 项目数据模型
    参见: TDS-20260624-001 第2.3节
    """
    id: str = Field(default_factory=generate_id)
    name: str = Field(..., min_length=1, max_length=100)
    description: str = Field(default="", max_length=500)
    created_at: datetime = Field(default_factory=datetime.now)
    status: str = Field(default="active")
```

### 步骤2.3：API实现

对于TDS中的每个API：

```python
# REQ-TDS-003: 实现项目API
# 参见: TDS-20260624-001 第3.1节

@router.post("/projects", response_model=ProjectResponse)
async def create_project(project: ProjectCreate):
    """REQ-TDS-003: 创建新项目
    验收条件: 项目创建成功并返回唯一ID
    """
    # 实现带有需求可追溯性的代码
    return await project_service.create(project)
```

### 步骤2.4：业务逻辑实现

按照TDS中的实现步骤进行：
- 每个步骤作为单独的提交实现
- 添加需求注释以实现可追溯性

## 阶段3：验证

### 步骤3.1：单元测试

针对每个功能：
```python
# REQ-TDS-004: 测试数据模型
def test_project_creation():
    """REQ-TDS-004: 验证项目创建
    验收条件: 有效数据的项目创建成功
    """
    project = Project(name="测试", description="测试项目")
    assert project.id is not None
    assert project.status == "active"
```

### 步骤3.2：集成测试

验证：
- API端点正常工作
- 数据库操作成功
- 错误处理正常

### 步骤3.3：TDS合规性检查

```bash
# 验证所有TDS步骤已实现
grep -r "REQ-TDS-" src/ | wc -l
# 应该等于TDS中的步骤数
```

## 阶段4：文档

### 步骤4.1：代码文档

生成：
- 函数文档字符串
- API文档
- README更新

### 步骤4.2：更新TDS进度

标记完成的步骤：
```markdown
## 4. 实现步骤

- [x] 步骤1: 初始化项目结构
- [x] 步骤2: 实现数据模型  
- [ ] 步骤3: 创建API端点（进行中）
- [ ] 步骤4: 实现业务逻辑
```

## 输出产物

| 产物 | 位置 | 格式 |
|------|------|------|
| 源代码 | `src/` | Python/TypeScript |
| 单元测试 | `tests/` | pytest/Jest |
| TDS进度 | `requirements/` | Markdown |
| 提交历史 | `.git/logs/` | Git |

## 质量检查清单

- [ ] 所有TDS步骤已分配给开发人员
- [ ] 代码包含需求注释(REQ-TDS-XXX)
- [ ] 单元测试覆盖所有验收标准
- [ ] 集成测试通过
- [ ] 代码编译无错误
- [ ] TDS进度已更新

## Git提交规范

```
YYYYMMDD - REQ-TDS-XXX [功能描述]

- 需求: REQ-TDS-XXX - [TDS引用]
- 变更: [变更内容]
- 验证: [验证方式]
```

---

*技能版本: 1.0.0*  
*最后更新: 2026-06-27*