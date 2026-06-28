---
name: bau-sop-gen
description: "BAU运维手册生成：基于项目资产自动生成日常运营标准操作程序。"
version: 1.0.0
author: DevFlow AI
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [sdlc, sop, 文档, 自动化, bau]
    related_skills: [api_auto_regression, api_doc_gen]
---

# BAU运维手册生成技能

## 概述

基于项目资产、API文档和运营需求，自动为日常运营活动(Business-As-Usual, BAU)生成标准操作程序(SOP)。创建可用于Wiki的文档，以支持持续运营。

## 使用场景

- API文档完成后
- 系统上线前
- 知识转移准备期间
- 创建值班文档
- 新团队成员入职培训

## 输入要求

### API文档
- 位置: `docs/api/` 或 `wiki/api/`
- 必须包含: 端点规范、错误码

### TDS/FS文档
- 位置: `requirements/`
- 必须包含: 功能需求、验收标准

### 部署/运维文档
- 任何现有的运维文档

### 测试结果
- 位置: `results/` 或 `itqa/`
- 必须包含: 回归测试结果

## 阶段1：SOP内容收集

### 步骤1.1：收集源材料

```python
def collect_sop_sources(project_dir: str) -> dict:
    """收集SOP生成的所有相关文档"""
    sources = {
        "api_docs": [],
        "tds_docs": [],
        "test_results": [],
        "deployment_docs": []
    }
    
    # 收集API文档
    for root, dirs, files in os.walk(f"{project_dir}/docs/api"):
        for f in files:
            if f.endswith(('.md', '.yaml', '.yml')):
                sources["api_docs"].append(os.path.join(root, f))
    
    # 收集TDS
    for root, dirs, files in os.walk(f"{project_dir}/requirements"):
        for f in files:
            if f.startswith('TDS'):
                sources["tds_docs"].append(os.path.join(root, f))
    
    # 收集测试结果
    for root, dirs, files in os.walk(f"{project_dir}/results"):
        for f in files:
            if f.endswith(('.xml', '.html', '.md')):
                sources["test_results"].append(os.path.join(root, f))
    
    return sources
```

### 步骤1.2：提取运营信息

```python
def extract_operational_info(sources: dict) -> dict:
    """提取BAU SOP所需的信息"""
    ops_info = {
        "system_overview": "",
        "api_endpoints": [],
        "error_handling": [],
        "monitoring_points": [],
        "recovery_procedures": []
    }
    
    # 从API文档解析端点
    for doc in sources["api_docs"]:
        content = read_file(doc)
        ops_info["api_endpoints"].extend(extract_endpoints(content))
        ops_info["error_handling"].extend(extract_error_codes(content))
    
    # 从测试结果解析已知问题
    for result in sources["test_results"]:
        content = read_file(result)
        ops_info["monitoring_points"].extend(extract_monitor_points(content))
    
    return ops_info
```

### 步骤1.3：识别BAU活动

```python
def identify_bau_activities(ops_info: dict) -> list:
    """识别常见的BAU活动以生成SOP"""
    activities = [
        {
            "category": "监控",
            "activity": "每日系统健康检查",
            "frequency": "每日",
            "priority": "高"
        },
        {
            "category": "监控", 
            "activity": "API响应时间监控",
            "frequency": "每小时",
            "priority": "中"
        },
        {
            "category": "运维",
            "activity": "部署新版本",
            "frequency": "按需",
            "priority": "关键"
        },
        {
            "category": "运维",
            "activity": "回滚失败部署",
            "frequency": "按需",
            "priority": "关键"
        },
        {
            "category": "维护",
            "activity": "数据库备份",
            "frequency": "每日",
            "priority": "高"
        },
        {
            "category": "维护",
            "activity": "日志轮转和清理",
            "frequency": "每周",
            "priority": "中"
        },
        {
            "category": "故障",
            "activity": "API不可用响应",
            "frequency": "按需",
            "priority": "关键"
        },
        {
            "category": "故障",
            "activity": "高错误率响应",
            "frequency": "按需",
            "priority": "关键"
        },
        {
            "category": "支持",
            "activity": "创建新项目",
            "frequency": "按需",
            "priority": "低"
        },
        {
            "category": "支持",
            "activity": "用户访问管理",
            "frequency": "按需",
            "priority": "中"
        }
    ]
    
    return activities
```

## 阶段2：SOP生成

### 步骤2.1：生成监控SOP

```markdown
# SOP-MON-001: 每日系统健康检查

## 目的
验证系统每天正常运行。

## 频率
每日 - 上午9:00（当地时间）

## 前置条件
- 访问监控仪表板
- 访问API健康端点

## 步骤

### 步骤1: 检查API健康状态
```bash
curl http://localhost:8000/health
```
预期: `{"status":"healthy",...}`

### 步骤2: 检查服务依赖项
```bash
# 检查数据库
curl http://localhost:8000/health | jq '.services.db'

# 检查向量存储  
curl http://localhost:8000/health | jq '.services.chroma'
```

### 步骤3: 验证最近部署
1. 转到部署仪表板
2. 检查过去24小时的部署
3. 验证全部成功

### 步骤4: 检查错误率
```bash
# 检查API错误率
curl http://localhost:8000/metrics | jq '.error_rate'
```
阈值: < 1%

## 升级
如果健康检查失败：
1. 通知值班工程师
2. 检查部署状态
3. 查看最近日志

## 参考资料
- API文档: /wiki/api/overview
- 故障手册: /wiki/incidents/api-down
```

### 步骤2.2：生成运维SOP

```markdown
# SOP-OPS-001: 部署新版本

## 目的
将应用程序的新版本部署到生产环境。

## 频率
按需（通常在发布批准后）

## 前置条件
- 已获得IT质量评审批准
- 已审查回归测试结果
- 已创建备份
- 已准备回滚计划

## 步骤

### 步骤1: 部署前检查清单
- [ ] 已获得IT质量评审签署
- [ ] 已审查回归测试结果
- [ ] 已创建备份
- [ ] 已准备回滚计划

### 步骤2: 停止当前服务
```bash
# 停止前端
pm2 stop devflow-frontend

# 停止后端  
pm2 stop devflow-backend
```

### 步骤3: 备份当前版本
```bash
# 备份数据库
cp data/devflow.db data/backup/devflow_$(date +%Y%m%d).db

# 备份配置
cp -r config config.backup.$(date +%Y%m%d)
```

### 步骤4: 部署新版本
```bash
# 后端
cd backend
source venv/bin/activate
pip install -r requirements.txt
pm2 restart devflow-backend

# 前端
cd frontend
npm run build
pm2 restart devflow-frontend
```

### 步骤5: 验证部署
```bash
# 检查健康状态
curl http://localhost:8000/health

# 运行冒烟测试
pytest tests/smoke_tests.py -v
```

### 步骤6: 监控问题
- 观察错误日志30分钟
- 监控响应时间
- 检查用户报告

## 回滚程序
如果部署失败：
```bash
# 恢复备份
cp data/backup/devflow_YYYYMMDD.db data/devflow.db
cp -r config.backup.YYYYMMDD config

# 重启旧版本
pm2 restart all
```

## 参考资料
- 部署流水线: /wiki/devops/pipeline
- 回滚手册: /wiki/incidents/rollback
```

### 步骤2.3：生成故障响应SOP

```markdown
# SOP-INC-001: API不可用响应

## 目的
API不可用时快速响应。

## 严重程度
关键 - P1故障

## 症状
- API返回503服务不可用
- API持续超时
- 健康检查失败

## 立即操作

### 步骤1: 确认问题
```bash
# 从不同位置测试
curl -w "\n%{http_code}\n" http://localhost:8000/api/v1/projects
# 如果是503或超时，则确认故障
```

### 步骤2: 检查服务状态
```bash
# 检查服务是否运行
pm2 status

# 查看错误日志
pm2 logs devflow-backend --lines 50
```

### 步骤3: 通知利益相关方
- 通知值班团队
- 通知产品负责人
- 更新状态页面

### 步骤4: 尝试恢复
```bash
# 尝试重启服务
pm2 restart devflow-backend
pm2 restart devflow-frontend

# 检查健康是否恢复
sleep 10
curl http://localhost:8000/health
```

### 步骤5: 如需升级
如果15分钟后恢复失败：
- 升级到基础架构团队
- 考虑数据库问题
- 考虑网络问题

## 故障后
- 在故障报告中记录时间线
- 安排故障复盘会议
- 如果发现差距则更新SOP

## 参考资料
- 监控仪表板: /wiki/monitoring/dashboard
- 值班轮换: /wiki/team/oncall
```

### 步骤2.4：生成支持SOP

```markdown
# SOP-SUP-001: 创建新项目

## 目的
指导支持团队为用户创建新项目。

## 频率
按需（常见请求）

## 步骤

### 步骤1: 验证用户身份
- 确认用户具有管理员权限
- 验证账户状态为活跃

### 步骤2: 创建项目
```bash
# 通过API
curl -X POST http://localhost:8000/api/v1/projects \
  -H "Content-Type: application/json" \
  -d '{
    "name": "[项目名称]",
    "description": "[描述]"
  }'
```

### 步骤3: 配置访问权限
- 将用户添加为项目管理员
- 设置初始权限

### 步骤4: 验证访问
- 用户可以登录
- 用户可以访问项目
- 用户可以创建文档

## 常见问题

| 问题 | 解决方案 |
|------|----------|
| 用户无法创建项目 | 检查权限 |
| 项目不可见 | 验证用户已添加到项目 |
| 名称已存在 | 附加唯一标识符 |

## 参考资料
- 用户管理: /wiki/admin/users
- 权限: /wiki/security/permissions
```

## 阶段3：Wiki资产生成

### 步骤3.1：创建SOP索引

```markdown
# BAU标准操作程序 - 索引

## 监控SOP
- [SOP-MON-001: 每日系统健康检查](monitoring/health-check.md)
- [SOP-MON-002: API响应时间监控](monitoring/api-response-time.md)
- [SOP-MON-003: 数据库性能检查](monitoring/db-performance.md)

## 运维SOP
- [SOP-OPS-001: 部署新版本](operations/deploy.md)
- [SOP-OPS-002: 回滚失败部署](operations/rollback.md)
- [SOP-OPS-003: 数据库备份](operations/backup.md)

## 故障响应SOP
- [SOP-INC-001: API不可用响应](incidents/api-down.md)
- [SOP-INC-002: 高错误率响应](incidents/high-errors.md)
- [SOP-INC-003: 数据库连接问题](incidents/db-connection.md)

## 支持SOP
- [SOP-SUP-001: 创建新项目](support/create-project.md)
- [SOP-SUP-002: 用户访问管理](support/user-access.md)
- [SOP-SUP-003: 文档恢复](support/document-recovery.md)

## 快速参考
- [紧急联系人](emergency-contacts.md)
- [故障手册索引](runbooks.md)
- [常见问题](faq.md)
```

### 步骤3.2：生成导航配置

```yaml
# Wiki侧边栏导航
navigation:
  - title: BAU运维手册
    items:
      - title: 监控
        children:
          - SOP-MON-001: 每日健康检查
          - SOP-MON-002: API响应时间
          - SOP-MON-003: 数据库性能
      
      - title: 运维
        children:
          - SOP-OPS-001: 部署版本
          - SOP-OPS-002: 回滚
          - SOP-OPS-003: 备份
      
      - title: 故障
        children:
          - SOP-INC-001: API宕机
          - SOP-INC-002: 高错误率
          - SOP-INC-003: 数据库问题
      
      - title: 支持
        children:
          - SOP-SUP-001: 创建项目
          - SOP-SUP-002: 用户访问
          - SOP-SUP-003: 文档恢复
```

## 阶段4：验证与维护

### 步骤4.1：SOP审查检查清单

```
SOP审查检查清单:
- [ ] 所有步骤清晰明确
- [ ] 所有命令已测试可用
- [ ] 所有错误场景已覆盖
- [ ] 升级联系人是最新
- [ ] 相关文档链接有效
- [ ] 截图是最新的
- [ ] 已由运维团队审查
- [ ] 已分配版本号
```

### 步骤4.2：版本控制

```markdown
# SOP版本历史

| 版本 | 日期 | 作者 | 变更 |
|------|------|------|------|
| 1.0.0 | 2026-06-27 | DevFlow AI | 初始版本 |
| ... | ... | ... | ... |

# 审查计划
- 审查频率: 季度
- 下次审查: 2026-09-27
- 负责人: 运维团队
```

## 输出产物

| 产物 | 位置 | 格式 |
|------|------|------|
| 监控SOP | `wiki/sops/monitoring/` | Markdown |
| 运维SOP | `wiki/sops/operations/` | Markdown |
| 故障SOP | `wiki/sops/incidents/` | Markdown |
| 支持SOP | `wiki/sops/support/` | Markdown |
| SOP索引 | `wiki/sops/index.md` | Markdown |
| 导航配置 | `wiki/_data/nav.yml` | YAML |
| 版本历史 | `wiki/sops/VERSION_HISTORY.md` | Markdown |

## 质量检查清单

- [ ] 所有BAU活动已文档化
- [ ] 每个SOP有清晰步骤
- [ ] 所有命令已测试
- [ ] 错误场景已覆盖
- [ ] 升级联系人是最新
- [ ] 索引和导航已创建
- [ ] 版本控制已建立
- [ ] 已通过IT质量评审

---

*技能版本: 1.0.0*  
*最后更新: 2026-06-27*