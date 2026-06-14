---
description: '创建新功能 - 自动生成 Phase Gate 文件和目录结构'
---

# 创建新功能

**核心价值**：
- 标准化新功能创建流程
- 自动生成 Phase Gate 结构
- 集成到 BMad 工作流

## 执行步骤

### Step 1: 收集基本信息
<tool_use>
<function_name>AskUserQuestion</function_name>
<parameters>
<parameter name="questions">
[
  {
    "question": "请输入功能名称（使用英文标识符，如 user-authentication）：",
    "header": "功能名称"
  }
]
</parameter>
</parameters>
</tool_use>

<tool_use>
<function_name>AskUserQuestion</function_name>
<parameters>
<parameter name="questions">
[
  {
    "question": "请输入功能显示名称（中文，如用户认证系统）：",
    "header": "显示名称"
  }
]
</parameter>
</parameters>
</tool_use>

<tool_use>
<function_name>AskUserQuestion</function_name>
<parameters>
<parameter name="questions">
[
  {
    "question": "选择功能规模：",
    "header": "功能规模",
    "options": [
      {
        "label": "Level 0 - 单个修改",
        "description": "Bug修复、小功能，1个任务，1天完成"
      },
      {
        "label": "Level 1 - 小功能",
        "description": "1-10个任务，1-3天"
      },
      {
        "label": "Level 2 - 中等项目",
        "description": "5-15个任务，1-2周"
      },
      {
        "label": "Level 3 - 复杂系统",
        "description": "12-40个任务，需要架构设计"
      },
      {
        "label": "Level 4 - 企业级",
        "description": "40+个任务，多产品集成"
      }
    ]
  }
]
</parameter>
</parameters>
</tool_use>

<tool_use>
<function_name>AskUserQuestion</function_name>
<parameters>
<parameter name="questions">
[
  {
    "question": "选择负责人：",
    "header": "负责人",
    "options": [
      {"label": "自己", "description": "我将负责这个功能"},
      {"label": "团队成员", "description": "指定其他成员"},
      {"label": "待定", "description": "后续分配"}
    ]
  }
]
</parameter>
</parameters>
</tool_use>

### Step 2: 确认功能信息
<output>
📋 **新功能信息确认**

- **功能标识**: {feature_name}
- **显示名称**: {feature_display_name}
- **规模等级**: {feature_level}
- **负责人**: {assignee}
- **创建时间**: {creation_date}
- **预计工时**: {estimated_hours}

**建议的 Phase 计划**：
{recommended_phases}

确认创建？[y/n/e]
[y] 确认创建
[n] 取消
[e] 编辑信息
</output>

<if condition="user_response == 'n'">
<output>
已取消创建新功能。
</output>
<return/>
</if>

<if condition="user_response == 'e'">
<!-- 返回 Step 1 重新收集信息 -->
</if>

### Step 3: 创建目录结构
<tool_use>
<function_name>Bash</function_name>
<parameters>
<parameter name="command">mkdir -p docs/{feature_name}</parameter>
<parameter name="description">创建功能目录</parameter>
</parameters>
</tool_use>

<output>
📁 创建目录结构...
</output>

### Step 4: 生成文件内容

#### 4.1 生成 00_CONTEXT.md
<tool_use>
<function_name>Read</function_name>
<parameters>
<parameter name="file_path">/Users/neov/src/github.com/a2d2-dev/templates/phase-gate-system/templates/00_CONTEXT.md</parameter>
</parameters>
</tool_use>

<tool_use>
<function_name>Write</function_name>
<parameters>
<parameter name="file_path">docs/{feature_name}/00_CONTEXT.md</parameter>
<parameter name="content">{CONTEXT_CONTENT_WITH_VALUES}</parameter>
</parameters>
</tool_use>

#### 4.2 生成 10_DESIGN_FINAL.md
<tool_use>
<function_name>Write</function_name>
<parameters>
<parameter name="file_path">docs/{feature_name}/10_DESIGN_FINAL.md</parameter>
<parameter name="content">{DESIGN_CONTENT_WITH_VALUES}</parameter>
</parameters>
</tool_use>

#### 4.3 生成 21_PHASE_GATE_STATUS.yaml
<tool_use>
<function_name>Write</function_name>
<parameters>
<parameter name="file_path">docs/{feature_name}/21_PHASE_GATE_STATUS.yaml</parameter>
<parameter name="content">{GATE_STATUS_CONTENT_WITH_VALUES}</parameter>
</parameters>
</tool_use>

#### 4.4 生成 30_PROGRESS_LOG.yaml
<tool_use>
<function_name>Write</function_name>
<parameters>
<parameter name="file_path">docs/{feature_name}/30_PROGRESS_LOG.yaml</parameter>
<parameter name="content">{PROGRESS_LOG_CONTENT_WITH_VALUES}</parameter>
</parameters>
</tool_use>

### Step 5: 初始化进度
<output>
📝 初始化项目进度...

✅ Phase 1 (Kickoff) 已完成：
- [x] 创建功能目录
- [x] 生成标准文件
- [x] 初始化进度日志
</output>

### Step 6: 生成 Session ID
<tool_use>
<function_name>Bash</function_name>
<parameters>
<parameter name="command">date "+cc-%Y-%m-%d-{feature_name}"</parameter>
<parameter name="description">生成会话ID</parameter>
</parameters>
</tool_use>

### Step 7: 安装 Phase Gate 命令
<tool_use>
<function_name>Bash</function_name>
<parameters>
<parameter name="command">cp /Users/neov/src/github.com/a2d2-dev/templates/phase-gate-system/commands/*.md .claude/commands/</parameter>
<parameter name="description">复制 Phase Gate 命令到项目目录</parameter>
</parameters>
</tool_use>

### Step 8: 完成
<output>
🎉 **新功能创建成功！**

## 📁 项目结构
```
docs/{feature_name}/
├── 00_CONTEXT.md              # ✅ 功能上下文
├── 10_DESIGN_FINAL.md         # 📝 待完成
├── 21_PHASE_GATE_STATUS.yaml  # ✅ Phase 1 已通过
└── 30_PROGRESS_LOG.yaml       # ✅ 进度已初始化
```

## 🚀 快速开始

### 选项 1: 立即开始设计
```bash
/iresume
```
查看当前状态和下一步任务

### 选项 2: 使用 BMad 工作流
```bash
# 需求分析
/bmad:bmm:agents:analyst

# 架构设计
/bmad:bmm:agents:architect

# 快速开发
/bmad:bmm:workflows:quick-dev
```

### 选项 3: 标准流程
1. 完善需求设计（10_DESIGN_FINAL.md）
2. 组织设计评审
3. 进入开发阶段

## 📋 下一步任务

根据功能规模 {feature_level}，建议的任务：

{next_tasks_based_on_level}

## 🎯 Phase Gate 命令已安装

你现在可以使用：
- `/iresume` - 查看项目状态
- `/check-gate` - 检查质量门控
- `/approve-gate` - 批准 Phase
- `/next-phase` - 进入下一阶段

## 🔗 相关资源

- [Phase Gate 系统文档](../README.md)
- [BMad 工作流指南](../bmm/)
- [最佳实践](../templates/phase-gate-system/)

---

**功能 {feature_display_name} 已准备就绪！** 🚀
</output>

## 内容模板变量

### 基本变量
```yaml
feature_name: "user-authentication"  # 用户输入的标识符
feature_display_name: "用户认证系统"  # 用户输入的显示名
feature_level: "Level 2"  # 用户选择的规模
assignee: "张三"  # 用户选择的负责人
creation_date: "2024-12-16"  # 当前日期
estimated_hours: 40  # 根据等级估算
session_id: "cc-2024-12-16-user-authentication"  # 生成的会话ID
```

### 根据等级的任务模板

#### Level 0 - 单个修改
```yaml
phases:
  Phase5-Code:
    tasks:
      - id: "CODE-001"
        title: "实现功能修改"
        estimated_hours: 4
```

#### Level 1 - 小功能
```yaml
phases:
  Phase2-Spec:
    tasks:
      - id: "SPEC-001"
        title: "编写功能规格"
        estimated_hours: 2

  Phase5-Code:
    tasks:
      - id: "CODE-001"
        title: "核心功能实现"
        estimated_hours: 8
      - id: "CODE-002"
        title: "单元测试"
        estimated_hours: 2
```

#### Level 2 - 中等项目
```yaml
phases:
  Phase2-Spec:
    tasks:
      - id: "SPEC-001"
        title: "详细需求分析"
        estimated_hours: 4
      - id: "SPEC-002"
        title: "技术方案设计"
        estimated_hours: 6

  Phase5-Code:
    tasks:
      - id: "CODE-001"
        title: "后端实现"
        estimated_hours: 16
      - id: "CODE-002"
        title: "前端实现"
        estimated_hours: 12
      - id: "CODE-003"
        title: "集成测试"
        estimated_hours: 6

  Phase6-Test:
    tasks:
      - id: "TEST-001"
        title: "功能测试"
        estimated_hours: 4
      - id: "TEST-002"
        title: "性能测试"
        estimated_hours: 2
```

#### Level 3 - 复杂系统
```yaml
phases:
  Phase2-Spec:
    tasks:
      - id: "SPEC-001"
        title: "系统架构设计"
        estimated_hours: 12
      - id: "SPEC-002"
        title: "接口设计"
        estimated_hours: 8
      - id: "SPEC-003"
        title: "数据模型设计"
        estimated_hours: 6

  Phase5-Code:
    tasks:
      - id: "CODE-001"
        title: "核心服务开发"
        estimated_hours: 24
      - id: "CODE-002"
        title: "数据层实现"
        estimated_hours: 16
      - id: "CODE-003"
        title: "API层实现"
        estimated_hours: 20
      - id: "CODE-004"
        title: "前端应用开发"
        estimated_hours: 32
```

## BMad 集成

### 自动启动 BMad Agent
<if condition="feature_level >= 'Level 2'">
<output>
🤖 **建议使用 BMad 工作流**

根据功能规模，建议启动以下 BMad Agent：

1. **业务分析师** - 深入理解需求
   ```bash
   /bmad:bmm:agents:analyst
   ```

2. **架构师** - 设计系统架构
   ```bash
   /bmad:bmm:agents:architect
   ```

是否现在启动？[y/n]
</output>
</if>

### 创建 BMad 任务
<tool_use>
<function_name>Write</function_name>
<parameters>
<parameter name="file_path">docs/{feature_name}/bmad-tasks.md</parameter>
<parameter name="content"># BMad 工作流任务

## 推荐的 BMad 流程

### 1. 需求分析
```bash
/bmad:bmm:agents:analyst
```
输入：功能上下文 (00_CONTEXT.md)
输出：详细需求规格

### 2. 架构设计
```bash
/bmad:bmm:agents:architect
```
输入：需求规格
输出：系统架构设计

### 3. 快速开发
```bash
/bmad:bmm:workflows:quick-dev
```
输入：架构设计
输出：代码实现
</parameter>
</parameters>
</tool_use>
</if>

## 特殊处理

### 1. 功能名称验证
```python
# 验证功能名称格式
def validate_feature_name(name):
    if not re.match(r'^[a-z0-9-]+$', name):
        return False, "功能名称只能包含小写字母、数字和连字符"
    if len(name) > 50:
        return False, "功能名称不能超过50个字符"
    return True, "格式正确"
```

### 2. 目录冲突检查
```bash
# 检查目录是否已存在
if [ -d "docs/{feature_name}" ]; then
    echo "⚠️ 功能目录已存在"
    echo "选择操作："
    echo "1. 使用现有目录"
    echo "2. 覆盖现有目录（危险）"
    echo "3. 更改功能名称"
fi
```

### 3. 模板自定义
根据项目类型选择不同的模板：

#### API 项目
```yaml
template_type: "api"
additional_files:
  - "API_SPEC.md"
  - "OPENAPI.yaml"
```

#### UI 组件项目
```yaml
template_type: "ui"
additional_files:
  - "COMPONENT_DESIGN.md"
  - "STYLE_GUIDE.md"
```

#### 数据处理项目
```yaml
template_type: "data"
additional_files:
  - "DATA_FLOW.md"
  - "SCHEMA_DEFINITION.md"
```

## 最佳实践

1. **命名规范**：使用清晰、描述性的功能名称
2. **合理估算**：根据团队经验和历史数据估算工时
3. **完整信息**：尽量填写完整的上下文信息
4. **及时更新**：创建后立即开始填写设计文档
5. **团队沟通**：创建后通知团队成员

## 故障排除

### 常见问题

**Q: 提示权限错误**
```
A: 确保你有写入 docs/ 目录的权限
   运行: chmod u+w docs/
```

**Q: 文件生成失败**
```
A: 检查磁盘空间和文件系统权限
   运行: df -h 和 ls -la docs/
```

**Q: Phase Gate 命令不工作**
```
A: 确保 .claude/commands/ 目录存在
   运行: mkdir -p .claude/commands/
```

## 扩展配置

```yaml
# new-feature-config.yml
project_types:
  api:
    default_level: "Level 2"
    required_phases: ["Phase1", "Phase2", "Phase5", "Phase6"]

  ui:
    default_level: "Level 1"
    required_phases: ["Phase1", "Phase2", "Phase4", "Phase5"]

  infrastructure:
    default_level: "Level 3"
    required_phases: ["Phase1", "Phase2", "Phase5"]

custom_templates:
  level_1:
    task_count: 5
    estimated_days: 2

  level_2:
    task_count: 15
    estimated_days: 7
```