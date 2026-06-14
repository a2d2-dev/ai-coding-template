---
description: '进入下一 Phase - 检查当前 Phase 完成度并进入下一阶段'
---

# 进入下一 Phase

**核心价值**：
- 确保阶段性质量达标
- 规范化项目推进流程
- 自动化 Phase 切换

## 执行步骤

### Step 1: 加载当前状态
<tool_use>
<function_name>Read</function_name>
<parameters>
<parameter name="file_path">{GATE_STATUS_PATH}</parameter>
</parameters>
</tool_use>

<tool_use>
<function_name>Read</function_name>
<parameters>
<parameter name="file_path">{PROGRESS_LOG_PATH}</parameter>
</parameters>
</tool_use>

### Step 2: 检查当前 Phase 完成度
<tool_use>
<function_name>Skill</function_name>
<parameters>
<parameter name="skill">check-gate</parameter>
</parameters>
</tool_use>

### Step 3: 判断是否可以进入下一 Phase
<if condition="has_blockers">
<output>
⚠️ **无法进入下一阶段**

**当前阻断**：
{blockers_summary}

**必须完成的任务**：
{required_tasks}

**建议操作**：
1. 运行 `/check-gate` 查看详细问题
2. 解决所有阻断项
3. 重新尝试进��下一阶段

要强制进入下一阶段吗？（不推荐）
[y] 强制进入 / [n] 取消 / [d] 查看详情
</output>
</if>

<if condition="no_blockers">
<output>
✅ **可以进入下一阶段**

**当前状态**：
- 当前 Phase: {current_phase.name}
- 完成度: {completion_percentage}%
- 所有 Gates: 通过

**下一 Phase**: {next_phase.name}
- 描述: {next_phase.description}
- 预计时长: {estimated_duration}

确认进入下一阶段？[y/n]
</output>
</if>

### Step 4: 用户确认
<if condition="user_response == 'y' and no_blockers">
<!-- 执行 Phase 切换 -->
</if>

<if condition="user_response == 'y' and has_blockers">
<!-- 强制进入（警告） -->
<output>
⚠️ **警告：强制进入下一阶段**

这将跳过质量检查，可能导致：
- 技术债务积累
- 质量风险
- 集成问题

确认要强制进入？[y/n]
</output>
</if>

<if condition="user_response == 'n'">
<output>
已取消 Phase 切换。

建议操作：
- 完成当前 Phase 的任务
- 解决阻塞问题
- 使用 `/iresume` 查看下一步
</output>
<return/>
</if>

### Step 5: 执行 Phase 切换
<output>
🚀 **正在进入下一阶段**

正在执行：
1. 更新 Phase 状态
2. 初始化下一 Phase
3. 创建新的任务列表
4. 发送通知...
</output>

#### 5.1 更新当前 Phase 状态
<!-- 更新 21_PHASE_GATE_STATUS.yaml -->
```yaml
current_phase: "{next_phase.name}"
phase_start_date: "{current_timestamp}"

# 标记前一阶段完成
phases:
  {current_phase.key}:
    status: "completed"
    completion_date: "{current_timestamp}"
    final_approver: "{approver_name}"
```

#### 5.2 创建下一 Phase 任务
<if condition="next_phase == 'Phase2-Spec'">
<tool_use>
<function_name>Write</function_name>
<parameters>
<parameter name="file_path">{DESIGN_DOC_PATH}</parameter>
<parameter name="content">{DESIGN_TEMPLATE}</parameter>
</parameters>
</tool_use>
</if>

<if condition="next_phase == 'Phase5-Code'">
<tool_use>
<function_name>Bash</function_name>
<parameters>
<parameter name="command">mkdir -p src/{feature_module}</parameter>
</parameters>
</tool_use>
</if>

#### 5.3 更新进度日志
<!-- 添加新阶段的初始任务 -->
```yaml
phases:
  {next_phase.key}:
    status: "in_progress"
    start_date: "{current_timestamp}"
    tasks:
      - id: "{NEXT_PHASE_TASK_001}"
        title: "初始化 {next_phase.name} 阶段"
        status: "completed"
        completion_date: "{current_timestamp}"

      - id: "{NEXT_PHASE_TASK_002}"
        title: "{first_task_title}"
        status: "pending"
```

### Step 6: 完成切换
<output>
🎉 **已成功进入 {next_phase.name}**

**新阶段信息**：
- Phase: {next_phase.name}
- 开始时间: {start_time}
- 任务数量: {task_count}
- 负责人: {assignee}

**当前任务**：
1. {task_1}
2. {task_2}
3. {task_3}

**下一步操作**：
- 使用 `/iresume` 开始工作
- 查看新 Phase 的任务列表
- 联系相关人员

📨 已通知团队成员
📄 文档已更新
🔗 相关资源已准备
</output>

### Step 7: 发送通知
<tool_use>
<function_name>AskUserQuestion</function_name>
<parameters>
<parameter name="questions">
[
  {
    "question": "是否发送 Phase 切换通知？",
    "header": "发送通知",
    "multiSelect": true,
    "options": [
      {"label": "Slack 通知", "description": "发送到团队频道"},
      {"label": "邮件通知", "description": "发送给所有相关方"},
      {"label": "GitHub Issue", "description": "创建里程碑更新"},
      {"label": "仅记录，不发送", "description": "静默切换"}
    ]
  }
]
</parameter>
</parameters>
</tool_use>
</if>

## Phase 切换规则

### Phase 1 → Phase 2 (Kickoff → Spec)
**进入条件**：
- [x] 目录已创建
- [x] 00_CONTEXT.md 已完成
- [x] 功能范围已确认

**初始化内容**：
- 创建 10_DESIGN_FINAL.md
- 设置设计任务
- 安排设计评审

### Phase 2 → Phase 5 (Spec → Code)
**进入条件**：
- [x] 设计文档已完成
- [x] API 设计已定义
- [x] 设计已通过评审

**初始化内容**：
- 创建代码目录结构
- 初始化开发任务
- 设置开发环境

### Phase 5 → Phase 6 (Code → Test)
**进入条件**：
- [x] 代码实现完成
- [x] 单元测试通过
- [x] 代码审查完成

**初始化内容**：
- 设置测试环境
- 创建测试计划
- 准备部署配置

## 强制进入的风险

### 技术风险
1. **未完成的功能**：可能导致集成问题
2. **质量问题**：技术债务积累
3. **测试不足**：生产环境风险

### 项目风险
1. **进度估算偏差**：后续阶段可能延期
2. **资源分配不当**：关键任务可能被遗漏
3. **团队协作问题**：上下文丢失

### 管理建议
1. **记录决策**：说明强制进入的原因
2. **设置补救计划**：在下一阶段弥补
3. **加强沟通**：确保团队了解风险

## Phase 模板

### Phase 2: Spec 任务模板
```yaml
tasks:
  - id: "SPEC-001"
    title: "详细需求分析"
    description: "分析和细化功能需求"
    estimated_hours: 4
    deliverables: ["需求规格文档"]

  - id: "SPEC-002"
    title: "技术方案设计"
    description: "设计技术实现方案"
    estimated_hours: 6
    deliverables: ["架构设计图", "技术文档"]

  - id: "SPEC-003"
    title: "设计评审"
    description: "组织设计评审会议"
    estimated_hours: 2
    deliverables: ["评审记录"]
```

### Phase 5: Code 任务模板
```yaml
tasks:
  - id: "CODE-001"
    title: "后端 API 实现"
    description: "实现后端 API 接口"
    estimated_hours: 8
    deliverables: ["API 代码", "API 文档"]

  - id: "CODE-002"
    title: "前端组件开发"
    description: "开发前端页面组件"
    estimated_hours: 10
    deliverables: ["组件代码", "单元测试"]

  - id: "CODE-003"
    title: "集成开发"
    description: "前后端集成开发"
    estimated_hours: 6
    deliverables: ["集成代码", "集成测试"]
```

## 通知模板

### Slack 通知
```
🚀 Phase 更新

项目: {project_name}
{current_phase} ✅ → {next_phase} 🏃

完成时间: {completion_time}
负责人: {owner}

下一阶段任务: {task_count} 项
查看详情: /iresume
```

### 邮件通知
```
主题: Phase Gate 推进 - {project_name}

团队好，

{project_name} 已完成 {current_phase}，正式进入 {next_phase}。

📊 完成情况：
- 完成时间: {completion_date}
- 任务完成: {completed_tasks}/{total_tasks}
- 质量检查: ✅ 全部通过

🎯 下一阶段目标：
{next_phase_objectives}

📋 当前任务：
1. {task_1}
2. {task_2}
3. {task_3}

如有疑问，请使用 /iresume 查看详情。

谢谢！
{sender}
```

## 特殊场景处理

### 1. 跳过可选 Phase
```
场景：从 Phase 2 直接跳到 Phase 5
条件：Phase 3 (Demo) 和 Phase 4 (Design) 标记为可选
操作：直接进入 Phase 5，更新进度
```

### 2. 回退到上一 Phase
```
场景：发现设计问题需要回退
处理：将状态重置到上一 Phase
要求：记录回退原因
```

### 3. 并行 Phase
```
场景：Phase 4 (Design) 与 Phase 5 (Code) 并行
处理：在配置中标记并行 Phase
注意：需要有明确的依赖关系
```

## 最佳实践

1. **质量优先**：不要为了进度牺牲质量
2. **完整记录**：每次切换都要有记录
3. **团队同步**：确保团队了解状态变化
4. **持续改进**：根据经验优化 Phase 切换标准

## 配置选项

```yaml
# phase-transition-config.yml
transition_rules:
  strict_mode: true  # 严格模式，必须通过所有检查
  require_approval: true  # 需要批准才能切换
  force_transition_penalty: true  # 强制切换会有标记

notifications:
  auto_send: true
  channels: ["slack", "email"]
  template: "standard"  # standard, detailed, minimal

parallel_phases:
  Phase4-Design:
    can_run_with: ["Phase5-Code"]
    dependency: "Phase2-Spec"

optional_phases:
  - "Phase3-Demo"
  - "Phase4-Design"
```