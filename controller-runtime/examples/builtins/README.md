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




# Webhook
在 Kubernetes 中，**Mutating Webhook** 是一种通过 HTTP 请求修改（或变更）Kubernetes API 请求的机制。它可以用来在资源创建或修改时，动态地调整请求的数据，比如为某些资源添加默认值或修改字段。

### Mutating Webhook 是什么？
Mutating Webhook 是 Kubernetes 的一种扩展机制，它允许用户在资源创建或修改时，通过外部服务（Webhook）对资源进行变更。Webhook 接收 Kubernetes API 发送的对象，并可以根据逻辑修改对象的内容，然后将修改后的对象返回给 Kubernetes。

Mutating Webhook 与另一种常用的 Webhook — **Validating Webhook** 区别在于，Mutating Webhook 主要用于修改资源，而 Validating Webhook 用于验证资源是否符合某些规则。

### 什么时候使用 Mutating Webhook？
1. **为资源自动注入某些配置**：
   比如，在某些情况下，当创建 Pod 时，你可能希望自动为每个 Pod 注入某些标签或环境变量。Mutating Webhook 可以拦截创建请求并修改它们。

2. **自动化资源的默认值设置**：
   比如，某个资源的某些字段在创建时没有明确设置，你可以通过 Mutating Webhook 自动为它们设置默认值。

3. **Pod 的动态注入（如 Sidecar）**：
   比如，Istio 通过 Mutating Webhook 动态地将 sidecar 容器注入到 Pod 中，以实现代理和服务网格功能。

4. **修改资源状态**：
   当资源创建或修改时，如果需要动态调整某些字段，Mutating Webhook 就可以实现这一功能。

### 解决了什么问题？
1. **动态调整 Kubernetes 资源**：
   Kubernetes 本身提供了一些默认行为和配置，但有些场景下，用户需要更加灵活的动态调整机制，Mutating Webhook 允许你在资源操作的过程中，根据特定逻辑修改资源内容。

2. **减少手动配置错误**：
   当你需要为大量资源添加某些统一的配置或调整时，Mutating Webhook 可以减少人为干预和错误，确保资源的统一性。

3. **无缝集成第三方服务或工具**：
   Mutating Webhook 使得 Kubernetes 能够无缝集成第三方服务，在资源操作的过程中自动做一些修改，或者在创建资源时动态注入相关的配置（比如在容器中自动注入 Sidecar）。

### 示例：
假设你有一个 Pod，它需要一个特定的环境变量或标签来进行跟踪，你可以通过 Mutating Webhook，在创建 Pod 时自动给每个 Pod 加入这些标签或环境变量，而不需要手动干预。

总之，Mutating Webhook 是一种非常强大的机制，用于动态修改 Kubernetes 资源的配置和行为，特别适合需要在集群中进行自动化和一致性管理的场景。


在 Kubernetes 中，**Validating Webhook** 是一种通过 HTTP 请求来验证资源是否合法的机制。它允许你在 Kubernetes 对象创建、更新或删除时，拦截并对这些请求进行验证，确保请求的资源符合预定的规则或要求。

### Validating Webhook 是什么？
Validating Webhook 是一种扩展机制，它通过 HTTP 请求接收 Kubernetes 的资源对象，并根据业务逻辑进行校验。如果资源对象不符合要求，Webhook 可以拒绝这个请求。Validating Webhook 本质上用于**验证**资源，而不对资源进行修改，和 **Mutating Webhook** 的区别在于，Mutating Webhook 会修改资源，Validating Webhook 则是验证资源是否有效。

### 什么时候使用 Validating Webhook？
1. **资源合法性检查**：
   在 Kubernetes 中创建或修改资源时，可以使用 Validating Webhook 来确保这些资源符合某些规范。例如，验证创建的 Pod 是否包含某些必要的标签、环境变量，或者验证请求的资源名称是否符合命名规范。

2. **跨团队的资源约定**：
   在多团队环境中，使用 Validating Webhook 可以强制执行跨团队的一致性要求，例如确保每个应用的 Pod 都有合适的安全配置，或每个服务的名称都符合规定的格式。

3. **资源访问控制**：
   你可以使用 Validating Webhook 来控制资源的创建或修改。例如，验证是否允许某些资源（如特定类型的 Pod 或 Secret）创建，或者拒绝不符合要求的资源。

4. **集群安全性**：
   例如，确保所有 Pod 都启用了某种安全机制（如使用非 root 用户运行容器），或者确保所有应用都启用了合适的资源限制（如 CPU 和内存限制）。

5. **集群操作合规性**：
   可以用来强制执行集群操作合规性，防止不符合规范的资源被创建，从而保证集群的稳定性与一致性。

### 解决了什么问题？
1. **防止无效的资源请求**：
   Validating Webhook 允许集群管理员在资源创建、更新或删除时对这些操作进行实时检查。如果资源不符合某些规则或政策，可以阻止该操作，防止无效或潜在有害的资源进入集群。

2. **增强集群一致性和合规性**：
   当集群需要遵循一些标准或规范时，Validating Webhook 提供了一种确保每个资源请求符合要求的方式，帮助管理员确保集群的资源在创建时就符合合规性要求。

3. **集中管理和治理**：
   Validating Webhook 可以将一些跨应用、跨团队的标准和规范集中化进行管理。通过 webhook 来统一处理验证逻辑，避免每个团队都需要单独检查这些规则。

4. **提升集群安全性**：
   通过验证资源是否符合安全要求（例如是否设置了安全的容器配置、网络策略、角色权限等），Validating Webhook 提供了一个保护集群免受不安全配置侵害的机制。

### 示例：
假设你想确保所有在集群中创建的 Pod 都包含一个名为 `app` 的标签，你可以使用 Validating Webhook 拦截所有 Pod 创建请求并检查其是否包含这个标签。如果没有标签，Webhook 会拒绝这个请求。

另外，如果你希望确保所有的服务都有 `https` 的端口开放，或者每个 Pod 的容器都运行在非 root 用户下，你也可以使用 Validating Webhook 来强制执行这些规则。

### 总结：
Validating Webhook 是 Kubernetes 的一种扩展机制，用于在资源创建或修改时进行验证，确保它们符合指定的规则或标准。它解决了资源操作合规性、集群安全性和一致性管理的问题，尤其适合在多团队或大规模集群中执行复杂的验证逻辑。

