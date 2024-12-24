# Operator SDK 核心测试用例导读

## 1. Controller Runtime 集成测试
位置: `test/integration/controller-runtime/...`

这些测试用例展示了如何:
- 使用 Controller Runtime 创建基本的控制器
- 处理资源的监视和协调
- 实现自定义资源的验证
- 处理 Finalizer 和资源清理

重点文件:
- `test/integration/controller-runtime/controller_test.go` - 基本控制器实现
- `test/integration/controller-runtime/reconciler_test.go` - 资源协调逻辑

## 2. Helm Operator 测试
位置: `test/e2e/helm/...`

展示了 Helm Operator 的核心功能:
- Helm Chart 的加载和渲染
- 自定义资源与 Helm values 的映射
- Release 管理和升级流程

关键测试:
- `test/e2e/helm/helm_test.go` - Helm Operator 的端到端测试

## 3. Ansible Operator 测试
位置: `test/e2e/ansible/...`

演示了:
- Ansible Playbook 与 Kubernetes 资源的集成
- 自定义资源触发 Ansible 任务
- 状态管理和错误处理

## 4. Scorecard 测试
位置: `test/scorecard/...`

展示了如何:
- 评估 Operator 的最佳实践
- 验证自定义资源定义
- 检查 Operator 捆绑包

## 5. CLI 功能测试
位置: `test/cmd/...`

包含:
- 项目脚手架生成
- 自定义资源生成
- Bundle 和 Catalog 管理

## 学习建议

1. 按照以下顺序阅读测试用例:
   - 先看 Controller Runtime 的基础集成测试
   - 然后根据你的兴趣选择 Helm 或 Ansible Operator 测试
   - 最后看 Scorecard 测试了解最佳实践

2. 每个测试用例都包含了设置（Setup）、执行（Execution）和清理（Cleanup）三个主要部分，建议完整阅读以理解完整流程

3. 特别关注测试中的 helper 函数和 fixture，它们展示了很多实用的模式和最佳实践

4. 结合官方文档一起阅读测试用例，这样可以更好地理解每个功能的设计意图 