---
name: gate_checker
description: 'Phase Gate 质量检查技能 - 自动化检查和报告 Phase Gate 状态'
version: '1.0.0'
author: 'Phase Gate System'
---

# Phase Gate Checker Skill

这是一个自动化检查 Phase Gate 质量标准的技能，可以集成到各种工作流中。

## 功能特性

- 🔍 自动读取 Phase Gate 状态文件
- ✅ 执行自动化质量检查
- 📊 生成详细的质量报告
- ⚠️ 识别和报告阻塞项
- 💡 提供改进建议

## 使用方式

### 直接调用
```bash
skill gate_checker
```

### 在工作流中集成
```yaml
# workflow.yaml
check_gates:
  skill: "gate_checker"
  parameters:
    project_path: "docs/user-auth"
    phase: "Phase5-Code"
    auto_fix: false
```

### 作为步骤调用
```bash
Step 5: Check Phase Gates
1. Load gate_checker skill
2. Execute checks
3. Review report
4. Take action based on results
```

## 核心功能

### 1. 自动发现项目
```typescript
// 自动查找所有 Phase Gate 项目
const projects = await discoverProjects('docs/');
// 返回: ["user-auth", "payment-system", "dashboard"]
```

### 2. 读取状态文件
```yaml
# 读取 21_PHASE_GATE_STATUS.yaml
gates:
  phase5_code:
    checks:
      unit_tests:
        status: "pending"
        command: "npm run test:coverage"
```

### 3. 执行检查命令
```bash
# 执行定义的检查命令
npm run test:coverage
npm run lint
npm run build
```

### 4. 生成报告
```
Phase: Phase5-Code
Status: BLOCKED
Passed: 3/5 checks
Failed: 2 checks
Blocker: Test coverage below threshold
```

## 检查规则定义

### 自动化检查
```yaml
automation:
  unit_tests:
    command: "npm run test:coverage"
    threshold: 80
    parser: "coverage"

  linting:
    command: "npm run lint"
    expected: 0
    parser: "eslint"

  type_check:
    command: "npm run type-check"
    expected: "No type errors"
    parser: "typescript"

  build:
    command: "npm run build"
    expected: 0
    parser: "exit_code"
```

### 文件存在检查
```yaml
file_checks:
  design_doc:
    path: "docs/{feature}/10_DESIGN_FINAL.md"
    required: true

  api_doc:
    path: "docs/{feature}/api.md"
    required: false
```

### 内容检查
```yaml
content_checks:
  requirements:
    file: "docs/{feature}/10_DESIGN_FINAL.md"
    pattern: "## 需求分析"
    required: true

  api_design:
    file: "docs/{feature}/10_DESIGN_FINAL.md"
    pattern: "### API 接口"
    required: true
```

## 输出格式

### 标准报告
```json
{
  "project": "user-authentication",
  "phase": "Phase5-Code",
  "timestamp": "2024-12-16T10:30:00Z",
  "status": "passed",
  "summary": {
    "total_checks": 5,
    "passed": 5,
    "failed": 0,
    "blocked": false
  },
  "checks": [
    {
      "name": "unit_tests",
      "status": "passed",
      "value": 85,
      "threshold": 80,
      "message": "Coverage: 85% ✅"
    },
    {
      "name": "linting",
      "status": "passed",
      "message": "No linting errors ✅"
    }
  ],
  "recommendations": [],
  "next_actions": ["Continue to Phase6-Test"]
}
```

### 阻断报告
```json
{
  "status": "blocked",
  "blockers": [
    {
      "check": "unit_tests",
      "reason": "Test coverage 65% below threshold 80%",
      "severity": "high",
      "suggestion": "Add more unit tests to reach 80% coverage"
    }
  ]
}
```

## 集成示例

### 在 GitHub Actions 中
```yaml
name: Check Phase Gates
on: [push, pull_request]

jobs:
  check-gates:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
      - name: Install dependencies
        run: npm install
      - name: Check Phase Gates
        run: |
          curl -X POST https://api.claude.ai/skills/gate_checker \
            -d '{"project_path": "docs"}'
```

### 在 CI/CD 流水线中
```yaml
# .gitlab-ci.yml
phase_gate_check:
  stage: test
  script:
    - npm install
    - skill gate_checker --phase=Phase5-Code
  only:
    - merge_requests
    - main
```

### 作为 Pre-commit Hook
```bash
#!/bin/sh
# .git/hooks/pre-commit

# Run Phase Gate checks
skill gate_checker --phase=current

# Check exit code
if [ $? -ne 0 ]; then
  echo "❌ Phase Gate checks failed"
  echo "Run '/check-gate' for details"
  exit 1
fi

echo "✅ All Phase Gate checks passed"
```

## 高级配置

### 自定义检查规则
```yaml
# .phase-gate-config.yml
custom_checks:
  security_scan:
    command: "npm audit"
    severity: "high"
    auto_fail: true

  performance_test:
    command: "npm run test:performance"
    thresholds:
      lcp: 2.5
      fcp: 1.8

skip_checks:
  - "documentation"  # 跳过文档检查
```

### 环境特定配置
```yaml
environments:
  development:
    strict_mode: false
    skip_optional: true

  production:
    strict_mode: true
    require_all: true
```

### 并行执行
```yaml
parallel:
  enabled: true
  max_workers: 4
  timeout: 300
```

## API 接口

### 检查单个项目
```typescript
interface CheckRequest {
  projectPath: string;
  phase?: string;
  autoFix?: boolean;
  verbose?: boolean;
}

interface CheckResponse {
  project: string;
  phase: string;
  status: 'passed' | 'blocked' | 'failed' | 'partial';
  summary: CheckSummary;
  checks: CheckResult[];
  recommendations: string[];
  blockers?: Blocker[];
}
```

### 批量检查
```typescript
interface BatchCheckRequest {
  projectPaths: string[];
  filter?: {
    phase?: string;
    status?: string;
  };
}

interface BatchCheckResponse {
  total: number;
  passed: number;
  blocked: number;
  results: CheckResponse[];
}
```

## 扩展点

### 自定义解析器
```typescript
// 添加自定义输出解析器
registerParser('custom_test', (output) => {
  const coverage = extractCoverage(output);
  return {
    passed: coverage >= 80,
    value: coverage,
    message: `Coverage: ${coverage}%`
  };
});
```

### 自定义通知
```typescript
// 添加自定义通知渠道
registerNotifier('custom_slack', async (report) => {
  await sendToSlack({
    channel: '#dev-alerts',
    message: formatSlackMessage(report)
  });
});
```

## 故障排除

### 常见错误

**找不到状态文件**
```
Error: Cannot find 21_PHASE_GATE_STATUS.yaml
Solution: Ensure you're in a Phase Gate project directory
```

**检查命令失败**
```
Error: Command 'npm run test' failed with exit code 1
Solution: Check if dependencies are installed
```

**权限错误**
```
Error: Permission denied when writing to files
Solution: Check file and directory permissions
```

### 调试模式
```bash
# 启用详细日志
skill gate_checker --verbose --debug

# 只运行特定检查
skill gate_checker --check unit_tests,linting

# 生成报告但不更新文件
skill gate_checker --dry-run
```

## 最佳实践

1. **定期运行**：在提交代码前运行检查
2. **自动化集成**：集成到 CI/CD 流水线
3. **配置定制**：根据项目需求调整检查规则
4. **团队协作**：共享配置文件确保一致性
5. **持续改进**：根据反馈优化检查标准

## 性能优化

### 缓存策略
- 缓存检查结果（24小时）
- 只检查变更的文件
- 并行执行独立检查

### 优化技巧
- 使用增量检查
- 跳过非必要检查
- 设置合理的超时

## 版本历史

- v1.0.0: 初始版本
  - 基本的 Phase Gate 检查功能
  - 支持自动化和手动检查
  - 生成详细报告

---

**Phase Gate Checker - 让质量检查自动化、标准化！** 🚀