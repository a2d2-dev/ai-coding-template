---
description: '检查 Phase Gate 质量门控 - 验证当前阶段是否满足质量标准'
---

# Phase Gate 质量门控检查

**核心价值**：
- 确保每个阶段都有质量保证
- 防止仓促推进到下一阶段
- 提供具体的改进建议

## 执行步骤

### Step 1: 加载当前项目状态
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

### Step 2: 确定当前 Phase
<if condition="current_phase_not_specified">
<tool_use>
<function_name>AskUserQuestion</function_name>
<parameters>
<parameter name="questions">
[
  {
    "question": "请选择要检查的 Phase：",
    "header": "Phase 选择",
    "options": [
      {"label": "Phase 1 - Kickoff", "description": "项目启动阶段"},
      {"label": "Phase 2 - Spec", "description": "需求设计阶段"},
      {"label": "Phase 5 - Code", "description": "开发实现阶段"},
      {"label": "Phase 6 - Test", "description": "测试验证阶段"}
    ]
  }
]
</parameter>
</parameters>
</tool_use>
</if>

### Step 3: 执行自动化检查
<tool_use>
<function_name>Bash</function_name>
<parameters>
<parameter name="command">{GATE_CHECK_COMMAND_1}</parameter>
<parameter name="description">执行第一个门控检查</parameter>
</parameters>
</tool_use>

<tool_use>
<function_name>Bash</function_name>
<parameters>
<parameter name="command">{GATE_CHECK_COMMAND_2}</parameter>
<parameter name="description">执行第二个门控检查</parameter>
</parameters>
</tool_use>

<!-- 继续其他检查 -->

### Step 4: 汇总检查结果
<output>
🔍 **Phase Gate 检查报告** - {current_phase.name}

## 📊 检查概览
- **检查时间**: {check_timestamp}
- **检查的 Phase**: {current_phase.name}
- **总检查项**: {total_checks}
- **通过项**: {passed_checks}
- **失败项**: {failed_checks}
- **待检查项**: {pending_checks}

## ✅ 通过的检查项
{passed_checks_list}

## ❌ 失败的检查项
{failed_checks_list}

## ⏳ 待检查项
{pending_checks_list}

## 🚫 阻断状态
<if condition="has_blockers">
⚠️ **当前 Phase 被阻断**

**阻断原因**：
{blocker_reasons}

**影响**：
- 无法进入下一阶段
- 需要解决所有阻断项
- 建议优先处理高优先级问题
</if>

<if condition="no_blockers">
✅ **当前 Phase 无阻断**

可以继续：
- 使用 `/next-phase` 进入下一阶段
- 或继续完善当前 Phase
</if>

## 💡 改进建议
{improvement_suggestions}

## 🔧 快速操作
- [r] 重新检查
- [a] 批准通过的 Gate（需要权限）
- [f] 查看失败项的详细信息
- [n] 尝试进入下一阶段
</output>

### Step 5: 处理用户响应

#### 重新检查 [r]
<if condition="user_response == 'r'">
<!-- 重新执行所有检查 -->
<!-- 更新检查时间戳 -->
<!-- 输出新的检查结果 -->
</if>

#### 批准 Gate [a]
<if condition="user_response == 'a'">
<tool_use>
<function_name>AskUserQuestion</function_name>
<parameters>
<parameter name="questions">
[
  {
    "question": "选择要批准的 Gate：",
    "header": "Gate 批准",
    "options": {approvable_gates_options}
  }
]
</parameter>
</parameters>
</tool_use>

<!-- 验证批准权限 -->
<!-- 更新 Gate 状态 -->
<!-- 记录批准信息 -->
</if>

#### 查看失败项 [f]
<if condition="user_response == 'f'">
<output>
📋 **失败项详细信息**

{failed_item_details}

**自动修复建议**：
{auto_fix_suggestions}

**手动修复步骤**：
{manual_fix_steps}

需要我帮你：
- 运行修复命令？[r]
- 打开相关文件？[o]
- 查看修复文档？[d]
</output>
</if>

#### 进入下一阶段 [n]
<if condition="user_response == 'n'">
<tool_use>
<function_name>Skill</function_name>
<parameters>
<parameter name="skill">next-phase</parameter>
</parameters>
</tool_use>
</if>

## Phase Gate 检查规则

### Phase 1: Kickoff
```yaml
checks:
  directory_created:
    command: "ls docs/{feature-name}/"
    expected: "目录存在"

  context_documented:
    command: "test -f docs/{feature-name}/00_CONTEXT.md"
    expected: "文档文件存在"

  basic_planning:
    command: "grep -q 'Phase 计划' docs/{feature-name}/00_CONTEXT.md"
    expected: "包含 Phase 计划"
```

### Phase 2: Spec
```yaml
checks:
  design_completed:
    command: "test -f docs/{feature-name}/10_DESIGN_FINAL.md"
    expected: "设计文档存在"

  requirements_defined:
    command: "grep -q '需求分析' docs/{feature-name}/10_DESIGN_FINAL.md"
    expected: "包含需求分析"

  api_defined:
    command: "grep -q 'API设计' docs/{feature-name}/10_DESIGN_FINAL.md"
    expected: "包含 API 设计"
```

### Phase 5: Code
```yaml
checks:
  unit_tests:
    command: "npm run test:coverage"
    threshold: 80
    expected: "测试覆盖率 >= 80%"

  linting:
    command: "npm run lint"
    expected: "无代码风格错误"

  type_check:
    command: "npm run type-check"
    expected: "无类型错误"

  build_success:
    command: "npm run build"
    expected: "构建成功"
```

### Phase 6: Test
```yaml
checks:
  e2e_tests:
    command: "npm run test:e2e"
    expected: "端到端测试通过"

  performance_tests:
    command: "npm run test:performance"
    threshold:
      response_time: 1000
    expected: "响应时间 < 1s"
```

## 智能检查逻辑

### 1. 自动执行
- 如果有自动化检查命令，自动执行
- 如果命令失败，解析错误信息
- 提供具体的修复建议

### 2. 权限验证
- 批准操作需要验证权限
- 记录批准人和批准时间
- 保存批准意见

### 3. 上下文感知
- 根据当前项目类型调整检查
- 考虑项目规模和复杂度
- 提供定制化的建议

## 错误处理

### 检查命令失败
```
错误：单元测试覆盖率检查失败
覆盖率：65%，要求：80%
建议：添加更多单元测试
运行：npm run test:coverage -- --coverageReporters=text
```

### 文件不存在
```
错误：找不到设计文档
路径：docs/feature-name/10_DESIGN_FINAL.md
建议：先完成需求设计
运行：/new-feature 查看模板
```

### 权限不足
```
错误：您没有批准此 Gate 的权限
当前角色：开发者
需要角色：技术负责人
建议：联系项目负责人审批
```

## 集成点

### 与 CI/CD 集成
- GitHub Actions 自动检查
- 阻止不合格的 PR 合并
- 在部署前验证所有 Gates

### 与项目管理集成
- 更新任务状态
- 标记阻塞项
- 发送通知给相关人员

## 最佳实践

1. **定期检查**：每个 Phase 完成时检查
2. **及时修复**：发现问题立即处理
3. **文档更新**：检查通过后更新文档
4. **团队协作**：通知团队成员检查结果

## 配置自定义

```yaml
# .phase-gate-config.yml
custom_checks:
  phase_5:
    custom_rule_1:
      command: "npm run custom-test"
      threshold: 90
      auto_approve: false

notifications:
  slack:
    webhook: "https://hooks.slack.com/..."
    channels: ["#dev-team"]

approval_workflow:
  required_approvers: 2
  auto_approve_after: "7d"
```