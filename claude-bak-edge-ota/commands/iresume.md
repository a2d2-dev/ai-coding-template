---
description: '智能断点恢复 - 显示当前项目状态，识别阻断，推荐下一步'
---

# 智能断点恢复

**核心价值**：让你在切换项目时，立即知道：
1. 项目进行到哪一步
2. 上次做了什么
3. 下一步该做什么
4. 是否有阻塞问题

## 执行步骤

### Step 1: 发现当前项目
<tool_use>
<function_name>Bash</function_name>
<parameters>
<parameter name="command">find docs -name "30_PROGRESS_LOG.yaml" -type f | head -5</parameter>
<parameter name="description">查找所有项目的进度日志文件</parameter>
</parameters>
</tool_use>

### Step 2: 如果没有找到进度日志
<if condition="progress_logs_count == 0">
<output>
🔍 **未找到任何 Phase Gate 项目**

建议操作：
1. 使用 `/new-feature` 创建新功能
2. 或者确认项目是否在正确目录

需要我帮你：
- 创建一个新的 Phase Gate 功能？[y/n]
- 查看当前目录结构？[d]
</output>
</if>

### Step 3: 如果找到多个项目
<if condition="progress_logs_count > 1">
<tool_use>
<function_name>AskUserQuestion</function_name>
<parameters>
<parameter name="questions">
[
  {
    "question": "发现多个 Phase Gate 项目，请选择要恢复的项目：",
    "header": "选择项目",
    "options": [
      {
        "label": "{PROJECT_1_NAME}",
        "description": "最近更新: {PROJECT_1_LAST_UPDATE}"
      },
      {
        "label": "{PROJECT_2_NAME}",
        "description": "最近更新: {PROJECT_2_LAST_UPDATE}"
      },
      {
        "label": "{PROJECT_3_NAME}",
        "description": "最近更新: {PROJECT_3_LAST_UPDATE}"
      }
    ],
    "multiSelect": false
  }
]
</parameter>
</parameters>
</tool_use>
</if>

### Step 4: 加载选中项目的数据
<tool_use>
<function_name>Read</function_name>
<parameters>
<parameter name="file_path">{SELECTED_PROGRESS_LOG_PATH}</parameter>
</parameters>
</tool_use>

<tool_use>
<function_name>Read</function_name>
<parameters>
<parameter name="file_path">{SELECTED_GATE_STATUS_PATH}</parameter>
</parameters>
</tool_use>

<tool_use>
<function_name>Read</function_name>
<parameters>
<parameter name="file_path">{SELECTED_CONTEXT_PATH}</parameter>
</parameters>
</tool_use>

### Step 5: 分析和计算状态
1. 计算整体进度
2. 识别当前阶段
3. 查找阻塞项
4. 推荐下一步

### Step 6: 生成状态报告
<output>
🔄 **断点恢复** - {project.display_name}

## 📋 基本信息
- **Session ID**: {project.session_id}
- **当前阶段**: {current_phase.name} ({current_phase.description})
- **整体进度**: {overall_progress.progress_percentage}%
- **负责人**: {project.owner}
- **团队**: {project.team}

## ⏮️ 上次状态
- **最后编辑**: {last_edit_file}
- **最后操作**: {last_operation}
- **上次更新**: {last_update_date}

## ➡️ 下一步建议

### 当前阶段任务
{current_phase.tasks}

<if condition="blockers_count > 0">
### ⚠️ 当前阻塞项
{blockers_list}

**解决建议**：
{resolution_suggestions}
</if>

### 🎯 推荐操作
1. {recommendation_1}
2. {recommendation_2}
3. {recommendation_3}

## 📊 Phase 进度概览
{phase_progress_summary}

## 💡 快速操作
- 继续当前任务: [c]
- 查看阻塞详情: [b]
- 进入下一阶段: [n] (需要所有 Gates 通过)
- 查看详细进度: [d]
- 手动更新任务状态: [u]

选择操作:
</output>

### Step 7: 处理用户响应
<if condition="user_response == 'c'">
<!-- 继续当前任务 -->
<tool_use>
<function_name>Read</function_name>
<parameters>
<parameter name="file_path">{CURRENT_TASK_ARTIFACT_PATH}</parameter>
</parameters>
</tool_use>

<output>
🚀 **继续任务**: {current_task.title}

**任务详情**：
- 描述: {current_task.description}
- 预计时间: {current_task.estimated_hours}h
- 验收标准: {current_task.acceptance_criteria}

**开始执行**...
</output>
</if>

<if condition="user_response == 'b'">
<!-- 查看阻塞详情 -->
<output>
🚫 **阻塞项详情**

{detailed_blocker_info}

**解决方案**：
{blocker_solutions}

需要我帮你：
- 自动解决阻塞? [a]
- 查看相关文档? [d]
- 跳过阻塞（风险）? [s]
</output>
</if>

<if condition="user_response == 'n'">
<!-- 尝试进入下一阶段 -->
<tool_use>
<function_name>Skill</function_name>
<parameters>
<parameter name="skill">check-gate</parameter>
</parameters>
</tool_use>
</if>

<if condition="user_response == 'd'">
<!-- 显示详细进度 -->
<output>
📊 **详细进度报告**

{detailed_progress_report}
</output>
</if>

<if condition="user_response == 'u'">
<!-- 手动更新任务状态 -->
<tool_use>
<function_name>AskUserQuestion</function_name>
<parameters>
<parameter name="questions">
[
  {
    "question": "选择要更新的任务：",
    "header": "任务更新",
    "options": {current_phase_task_options}
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
    "question": "新的任务状态：",
    "header": "状态选择",
    "options": [
      {"label": "完成", "description": "任务已完成"},
      {"label": "进行中", "description": "任务正在执行"},
      {"label": "阻塞", "description": "遇到问题被阻塞"}
    ]
  }
]
</parameter>
</parameters>
</tool_use>

<!-- 更新进度文件 -->
<tool_use>
<function_name>Write</function_name>
<parameters>
<parameter name="file_path">{PROGRESS_LOG_PATH}</parameter>
<parameter name="new_string">{UPDATED_PROGRESS_CONTENT}</parameter>
</parameters>
</tool_use>
</if>

## 模板变量说明

### 变量来源
- `{SELECTED_PROGRESS_LOG_PATH}` - 用户选择或自动发现的进度日志路径
- `{SELECTED_GATE_STATUS_PATH}` - 基于 progress_log 推断的 gate_status 路径
- `{SELECTED_CONTEXT_PATH}` - 基于 progress_log 推断的 context 路径

### 数据结构
```yaml
# project 来自 00_CONTEXT.md
project:
  name: "功能标识符"
  display_name: "功能显示名称"
  session_id: "会话ID"
  owner: "负责人"
  team: "团队名称"

# current_phase 来自 30_PROGRESS_LOG.yaml
current_phase:
  name: "Phase5-Code"
  description: "开发实现"
  progress: 75
  tasks:
    - id: "CODE-008"
      title: "任务标题"
      status: "in_progress"

# blockers 来自 21_PHASE_GATE_STATUS.yaml 和 30_PROGRESS_LOG.yaml
blockers:
  - gate_name: "code_quality"
    reason: "单元测试覆盖率不足"
    status: "blocked"
    resolution: "添加更多单元测试"

# overall_progress 计算得出
overall_progress:
  progress_percentage: 88
  completed_tasks: 15
  total_tasks: 17
```

## 智能推荐逻辑

### 1. 推荐继续任务
条件：
- 有进行中的任务
- 没有紧急阻塞

推荐：
- 继续当前进行中的任务
- 提供任务详情和上下文

### 2. 推荐解决阻塞
条件：
- 存在阻塞项
- 阻塞影响当前阶段推进

推荐：
- 显示具体阻塞原因
- 提供解决方案
- 必要时推荐使用其他命令

### 3. 推荐进入下一阶段
条件：
- 当前阶段所有任务已完成
- 所有 Gates 状态为 passed

推荐：
- 使用 `/next-phase` 命令
- 自动检查 Gates

## 使用场景

### 场景1：早上开工
```
用户: /iresume
输出: 显示昨天的工作进度和今天的建议任务
```

### 场景2：切换项目
```
用户: /iresume
输出: 显示该项目的当前状态，避免遗忘
```

### 场景3：遇到问题
```
用户: /iresume
输出: 自动识别阻塞，提供解决方案
```

## 集成说明

此命令与其他 Phase Gate 命令集成：
- `/new-feature` - 创建新项目时初始化进度日志
- `/check-gate` - 检查 Phase 质量门控
- `/approve-gate` - 批准通过 Gate
- `/next-phase` - 进入下一阶段

## 最佳实践

1. **每天使用**：开始工作前运行 `/iresume`
2. **任务完成**：完成重要任务后更新状态
3. **遇到问题**：被阻塞时运行查看建议
4. **阶段切换**：尝试进入下一阶段前检查