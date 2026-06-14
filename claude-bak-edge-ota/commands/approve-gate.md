---
description: '批准 Phase Gate - 手动批准通过某个质量门控'
---

# Phase Gate 批准

**核心价值**：
- 人工质量把关，补充自动化检查的不足
- 明确批准责任，确保质量决策可追溯
- 灵活处理特殊情况和例外

## 执行步骤

### Step 1: 验证权限
<tool_use>
<function_name>AskUserQuestion</function_name>
<parameters>
<parameter name="questions">
[
  {
    "question": "请确认您的身份和权限：",
    "header": "身份验证",
    "options": [
      {"label": "技术负责人", "description": "有权限批准 Code、Test Phase"},
      {"label": "产品负责人", "description": "有权限批准 Spec、Demo Phase"},
      {"label": "项目经理", "description": "有权限批准所有 Phase"},
      {"label": "团队成员", "description": "仅能查看状态"}
    ]
  }
]
</parameter>
</parameters>
</tool_use>

<if condition="user_role == '团队成员'">
<output>
❌ **权限不足**

您当前没有批准 Phase Gate 的权限。

**建议操作**：
- 联系项目负责人进行审批
- 或运行 `/check-gate` 查看需要改进的地方
</output>
<return/>
</if>

### Step 2: 加载 Gate 状态
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

### Step 3: 选择要批准的 Phase
<if condition="phase_not_specified">
<tool_use>
<function_name>AskUserQuestion</function_name>
<parameters>
<parameter name="questions">
[
  {
    "question": "选择要批准的 Phase：",
    "header": "Phase 选择",
    "options": {pending_phases_options}
  }
]
</parameter>
</parameters>
</tool_use>
</if>

### Step 4: 显示当前 Phase 检查项
<output>
📋 **Phase Gate 批准** - {selected_phase.name}

## 🔍 当前状态检查

### 自动检查结果
{automated_checks_status}

### 待批准项
{pending_approval_items}

### ⚠️ 风险提示
{risk_warnings}

## 👥 已有批准记录
{existing_approvals}

## 📝 批准说明
请在批准前确认：
1. [ ] 所有必要的检查已通过
2. [ ] 相关文档已更新
3. [ ] 团队已达成共识
4. [ ] 风险已评估并接受

## 🎯 批准选项
- [a] 批准整个 Phase
- [s] 选择性批准特定检查项
- [c] 带条件批准
- [r] 拒绝批准（说明原因）
</output>

### Step 5: 处理批准选择

#### 批准整个 Phase [a]
<if condition="user_response == 'a'">
<tool_use>
<function_name>AskUserQuestion</function_name>
<parameters>
<parameter name="questions">
[
  {
    "question": "请输入批准理由（将记录在案）：",
    "header": "批准理由",
    "options": [
      {"label": "所有检查项已通过，质量达标", "description": "标准批准理由"},
      {"label": "虽有微小问题，但不影响整体质量", "description": "有条件批准"},
      {"label": "特殊情况下豁免某些检查", "description": "特殊批准"},
      {"label": "自定义理由", "description": "输入详细批准理由"}
    ]
  }
]
</parameter>
</parameters>
</tool_use>

<!-- 验证所有检查项 -->
<!-- 更新 Gate 状态 -->
<!-- 记录批准信息 -->
<output>
✅ **Phase 已批准**

**批准详情**：
- Phase: {approved_phase.name}
- 批准人: {approver_name}
- 批准时间: {approval_timestamp}
- 批准理由: {approval_reason}

**后续操作**：
- 可以使用 `/next-phase` 进入下一阶段
- 所有相关方将收到通知
</output>
</if>

#### 选择性批准 [s]
<if condition="user_response == 's'">
<tool_use>
<function_name>AskUserQuestion</function_name>
<parameters>
<parameter name="questions">
[
  {
    "question": "选择要批准的检查项：",
    "header": "选择性批准",
    "multiSelect": true,
    "options": {check_items_options}
  }
]
</parameter>
</parameters>
</tool_use>

<!-- 更新选中的检查项 -->
<!-- 保持其他项状态 -->
</if>

#### 带条件批准 [c]
<if condition="user_response == 'c'">
<tool_use>
<function_name>AskUserQuestion</function_name>
<parameters>
<parameter name="questions">
[
  {
    "question": "设置批准条件：",
    "header": "批准条件",
    "options": [
      {"label": "需要补充文档", "description": "批准但要求补充文档"},
      {"label": "需要代码审查", "description": "批准但要求代码审查"},
      {"label": "需要额外测试", "description": "批准但要求增加测试"},
      {"label": "自定义条件", "description": "输入具体条件"}
    ]
  }
]
</parameter>
</parameters>
</tool_use>

<!-- 更新 Gate 状态为有条件通过 -->
<!-- 记录条件和截止时间 -->
</if>

#### 拒绝批准 [r]
<if condition="user_response == 'r'">
<tool_use>
<function_name>AskUserQuestion</function_name>
<parameters>
<parameter name="questions">
[
  {
    "question": "请说明拒绝理由：",
    "header": "拒绝理由",
    "options": [
      {"label": "质量未达标", "description": "关键检查项失败"},
      {"label": "文档不完整", "description": "缺少必要文档"},
      {"label": "未完成测试", "description": "测试覆盖率不足"},
      {"label": "其他原因", "description": "详细说明"}
    }
  }
]
</parameter>
</parameters>
</tool_use>

<!-- 保持或设置 Gate 状态为失败 -->
<!-- 记录拒绝理由 -->
<!-- 提供改进建议 -->
</if>

### Step 6: 更新文件
<tool_use>
<function_name>Write</function_name>
<parameters>
<parameter name="file_path">{GATE_STATUS_PATH}</parameter>
<parameter name="new_string">{UPDATED_GATE_STATUS}</parameter>
</parameters>
</tool_use>

<tool_use>
<function_name>Write</function_name>
<parameters>
<parameter name="file_path">{PROGRESS_LOG_PATH}</parameter>
<parameter name="new_string">{UPDATED_PROGRESS_LOG}</parameter>
</parameters>
</tool_use>

### Step 7: 发送通知
<if condition="notifications_enabled">
<tool_use>
<function_name>AskUserQuestion</function_name>
<parameters>
<parameter name="questions">
[
  {
    "question": "是否发送通知给团队成员？",
    "header": "发送通知",
    "multiSelect": true,
    "options": [
      {"label": "Slack 通知", "description": "发送到团队频道"},
      {"label": "邮件通知", "description": "发送给相关成员"},
      {"label": "仅记录，不发送", "description": "仅保存在文档中"}
    ]
  }
]
</parameter>
</parameters>
</tool_use>
</if>

## 批准权限矩阵

| 角色 | Phase 1 | Phase 2 | Phase 3 | Phase 4 | Phase 5 | Phase 6 |
|------|---------|---------|---------|---------|---------|---------|
| 开发者 | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ |
| 技术负责人 | ✅ | ✅ | ❌ | ❌ | ✅ | ✅ |
| 产品负责人 | ✅ | ✅ | ✅ | ✅ | ❌ | ❌ |
| 项目经理 | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| 架构师 | ❌ | ✅ | ❌ | ❌ | ✅ | ❌ |
| 测试负责人 | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ |

## 批准流程规范

### 1. 批准前检查
- [ ] 确认身份和权限
- [ ] 查看所有检查项结果
- [ ] 评估风险和影响
- [ ] 咨询相关干系人

### 2. 批准决策
- **完全批准**：所有检查通过，无风险
- **有条件批准**：基本满足，有改进项
- **拒绝批准**：关键项未达标

### 3. 批准后行动
- 更新状态文件
- 发送通知
- 创建改进任务（如果有条件）
- 记录决策依据

## 记录格式

### Gate 状态更新
```yaml
gates:
  phase5_code:
    status: "passed"
    approval:
      approver: "张三"
      role: "技术负责人"
      timestamp: "2024-12-16T10:30:00Z"
      reason: "所有检查项通过，代码质量达标"
      conditions: []  # 空数组表示无条件批准
```

### 进度日志更新
```yaml
approvals:
  - phase: "Phase5-Code"
    approver: "张三"
    approval_date: "2024-12-16"
    comments: "单元测试覆盖率 85%，代码风格良好"
    approval_type: "full"  # full, conditional, selective
```

## 特殊情况处理

### 1. 紧急批准
```
场景：生产环境紧急修复
处理：允许跳过某些检查
要求：需要 2 个高级人员共同批准
记录：标记为紧急批准
```

### 2. 代理批准
```
场景：负责人不在
处理：指定代理人
要求：有授权书
记录：标注代理关系
```

### 3. 批准撤销
```
场景：发现问题需要撤销
处理：可以撤销批准
要求：说明撤销原因
记录：保留历史记录
```

## 集成配置

### 自动通知
```yaml
notifications:
  slack:
    webhook: "https://hooks.slack.com/..."
    template: |
      Phase Gate 批准通知
      项目: {project_name}
      Phase: {phase_name}
      批准人: {approver}
      时间: {timestamp}

  email:
    recipients: ["team@example.com"]
    subject: "Phase Gate 批准 - {project_name}"
```

### 权限管理
```yaml
permissions:
  approvers:
    "phase_5_code":
      roles: ["tech_lead", "architect"]
      users: ["zhangsan", "lisi"]
      min_approvals: 1

    "phase_6_test":
      roles: ["qa_lead", "tech_lead"]
      min_approvals: 2
```

## 最佳实践

1. **充分了解**：批准前充分了解项目情况
2. **及时决策**：避免拖延影响项目进度
3. **清晰记录**：批准理由要清晰明确
4. **团队沟通**：重大决策要与团队沟通
5. **持续改进**：根据经验优化批准标准

## 审计跟踪

系统自动记录：
- 所有批准操作
- 批准人信息
- 时间戳
- 操作类型
- 变更前后的状态

这些记录用于：
- 项目审计
- 质量改进
- 责任追溯
- 流程优化