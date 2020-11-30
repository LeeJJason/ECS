# [Shared component data](https://docs.unity3d.com/Packages/com.unity.entities@0.11/manual/shared_component_data.html)
共享组件是一种特殊的数据组件，您可以使用它根据共享组件中的特定值（除了原型之外）对实体进行细分。 当您将共享组件添加到实体时，EntityManager将具有相同共享数据值的所有实体放入同一块中。

共享组件使您的系统可以像实体一样一起处理。例如，共享组件`Rendering.RenderMesh`是**Hybrid.rendering**包的一部分，它定义了几个字段，包括**mesh**，**material**和**receiveShadows**。当您的应用程序渲染时，最有效的方法是一起处理所有具有相同字段值的3D对象。因为共享组件指定了这些属性，所以EntityManager将匹配的实体放置在内存中，以便渲染系统可以有效地对其进行迭代。

**注意**：如果您过度使用共享组件，则可能导致不佳的块利用率。这是因为当您使用共享组件时，它涉及基于原型和每个共享组件字段的每个唯一值的内存块数量的组合扩展。这样，避免添加将实体分类到共享组件类别中不需要的任何字段。要查看块利用率，请使用[Entity Debugger](https://docs.unity3d.com/Packages/com.unity.entities@0.11/manual/ecs_debugging.html)。

如果您从实体中添加或删除组件，或更改共享组件的值，则EntityManager会将实体移至其他块，并在必要时创建一个新块。

您应该将[IComponentData](https://docs.unity3d.com/Packages/com.unity.entities@0.11/api/Unity.Entities.IComponentData.html)用于不同实体之间变化的数据，例如存储世界位置，特工生命值或粒子生存时间。相反，当许多实体共享某些共同点时，应使用[ISharedComponentData](https://docs.unity3d.com/Packages/com.unity.entities@0.11/api/Unity.Entities.ISharedComponentData.html)。例如，在DOTS软件包的Boids演示中，许多实体都从同一Prefab实例化，结果，`Boid`实体之间的`RenderMesh`许多实体完全相同。
```cs
[System.Serializable]
public struct RenderMesh : ISharedComponentData
{
    public Mesh                 mesh;
    public Material             material;

    public ShadowCastingMode    castShadows;
    public bool                 receiveShadows;
}
```

`ISharedComponentData`每个实体的内存成本为零。您可以使用`ISharedComponentData`将具有相同`InstanceRenderer`数据的所有实体组合在一起，然后有效地提取所有矩阵以进行渲染。生成的代码简单有效，因为在ECS访问数据时对数据进行了布局。

有关此示例，请参阅`RenderMeshSystemV2`文件`Packages/com.unity.entities/Unity.Rendering.Hybrid/RenderMeshSystemV2.cs。`

