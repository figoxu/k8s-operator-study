# Controller-Runtime Builtins 示例

此示例展示了 controller-runtime 的核心功能和最佳实践。通过一个完整的控制器实现，演示了以下关键特性：

## 主要功能展示

### 1. 控制器基础设施的搭建

```go
// 设置 Manager（控制器管理器）
mgr, err := manager.New(config.GetConfigOrDie(), manager.Options{})
// 创建具体的控制器
c, err := controller.New("foo-controller", mgr, controller.Options{
Reconciler: &reconcileReplicaSet{client: mgr.GetClient()},
})
```

### 2. 资源监控机制（Watch）

示例展示了两种典型的监控场景：
```go
go:controller-runtime/examples/builtins/README.md
// 场景1：直接监控 ReplicaSet 资源
c.Watch(source.Kind(mgr.GetCache(), &appsv1.ReplicaSet{}), ...)
// 场景2：监控从属资源（Pod）并关联到其所有者（ReplicaSet）
c.Watch(source.Kind(mgr.GetCache(), &corev1.Pod{}),
handler.TypedEnqueueRequestForOwner*corev1.Pod)
```

### 3. Webhook 功能

```go
// 展示了如何为 Pod 资源配置 mutating 和 validating webhook
builder.WebhookManagedBy(mgr).
For(&corev1.Pod{}).
WithDefaulter(&podAnnotator{}). // mutating webhook
WithValidator(&podValidator{}). // validating webhook
Complete()
```

## 核心能力说明

此示例主要展示了 controller-runtime 的以下核心能力：

1. **简化的控制器开发模式**
   - 提供了高级抽象，使得开发 Kubernetes 控制器变得更加简单直观

2. **声明式的 API**
   - 通过 builder 模式提供流畅的 API，使配置更加清晰

3. **资源关系处理**
   - 展示了如何处理 Kubernetes 资源之间的所有者引用关系（owner references）

4. **完整的控制器生态系统**
   - 包含了日志、配置管理、信号处理等完整的基础设施支持

5. **Webhook 集成**
   - 展示了如何轻松地集成准入 webhook，包括 mutating 和 validating webhook

这是一个教育意义的示例，展示了如何使用 controller-runtime 构建一个完整的 Kubernetes 控制器，包含了大多数实际应用中需要的核心功能。
