# [System State Components](https://docs.unity3d.com/Packages/com.unity.entities@0.11/manual/system_state_components.html)
您可以使用[SystemStateComponentData](https://docs.unity3d.com/Packages/com.unity.entities@0.11/api/Unity.Entities.ISystemStateComponentData.html)跟踪系统内部的资源，并根据需要创建和销毁这些资源，而无需依赖各个回调。

`SystemStateComponentData`和[SystemStateSharedComponentData](https://docs.unity3d.com/Packages/com.unity.entities@0.11/api/Unity.Entities.ISystemStateSharedComponentData.html)与`ComponentData`和`SharedComponentData`相似，但是在销毁实体时ECS不会删除`SystemStateComponentData`。

当实体被销毁时，ECS通常：
1. 查找引用特定实体ID的所有组件。
2. 删除那些组件。
3. 回收实体ID以供重用。

但是，如果`SystemStateComponentData`存在，则ECS不会回收ID。这使系统有机会清理与实体ID相关联的任何资源或状态。ECS仅在`SystemStateComponentData`删除后实体ID后才重新使用。

## When to use system state components
系统可能需要保持基于的内部状态`ComponentData`。例如，可以分配资源。

系统还需要能够将状态作为值进行管理，其他系统可能会进行状态更改。例如，当组件中的值更改时，或添加或删除相关组件时。

“No callbacks”是ECS设计规则的重要组成部分。

`SystemStateComponentData`预期的一般用法 将镜像用户组件，以提供内部状态。

例如，给定：
* FooComponent（`ComponentData`，用户分配）
* FooStateComponent（`SystemComponentData`，系统分配）

### Detecting when a component is added
创建组件时，系统状态组件不存在。系统更新不具有系统状态组件的组件的查询，并可以推断它们已被添加。此时，系统将添加系统状态组件和任何所需的内部状态。

### Detecting when a component is removed
删除组件时，系统状态组件仍然存在。系统更新不带组件的系统状态组件的查询，并可以推断它们已被删除。届时，系统将删除系统状态组件并修复任何需要的内部状态。

### Detecting when an entity is destroyed
`DestroyEntity` 是以下用途的简写实用程序：
1. 查找引用给定实体ID的组件。
2. 删除找到的组件。
3. 回收实体ID。

但是，不会在`DestroyEntity`上删除`SystemStateComponentData`，并且在删除最后一个组件之前不会回收实体ID。 这使系统有机会以与拆卸组件完全相同的方式清理内部状态。

## SystemStateComponent
`SystemStateComponentData`与`ComponentData`相似。
```cs
struct FooStateComponent : ISystemStateComponentData
{
}
```
还可以通过与组件相同的方式来控制`SystemStateComponentData`的可见性（使用私有，公共，内部）。但是，通常，通常希望`SystemStateComponentData`在创建它的系统之外为ReadOnly。

## SystemStateSharedComponent
`SystemStateSharedComponentData`与`SharedComponentData`相似。
```cs
struct FooStateSharedComponent : ISystemStateSharedComponentData
{
  public int Value;
}

```

## Example system using state components
以下示例显示了一个简化的系统，该系统说明了如何使用系统状态组件来管理实体。 该示例定义了通用`IComponentData`实例和系统状态`ISystemStateComponentData`实例。 它还基于这些实体定义了三个查询：
* `m_newEntities`选择具有通用用途但不具有系统状态组件的实体。 该查询查找系统之前未见过的新实体。 系统使用添加了系统状态组件的新实体查询来运行作业。
* `m_activeEntities`选择同时具有通用和系统状态组件的实体。 在实际的应用程序中，其他系统可能是处理或销毁实体的系统。
* `m_destroyedEntities`选择具有系统状态而不是通用组件的实体。 由于系统状态组件永远不会自己添加到实体，因此此系统或其他系统必须删除此查询选择的实体。 系统重用销毁的实体查询以运行作业并从实体中删除系统状态组件，这使ECS代码可以回收实体标识符。

**注意**：此简化示例在系统内不维护任何状态。 系统状态组件的目的之一是跟踪何时需要分配或清除持久性资源。