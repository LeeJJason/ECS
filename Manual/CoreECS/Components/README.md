# [Components](https://docs.unity3d.com/Packages/com.unity.entities@0.11/manual/ecs_components.html)
组件是实体组件系统体系结构的三个主要元素之一。它们代表您的游戏或应用程序的数据。[实体](https://docs.unity3d.com/Packages/com.unity.entities@0.11/manual/ecs_entities.html)是为您的组件集合建立索引的标识符，而[系统](https://docs.unity3d.com/Packages/com.unity.entities@0.11/manual/ecs_systems.html)则提供了行为。

ECS中的组件是具有以下`marker interfaces`之一的结构：
* [IComponentData](https://docs.unity3d.com/Packages/com.unity.entities@0.11/api/Unity.Entities.IComponentData.html) — 用于[general purpose](https://docs.unity3d.com/Packages/com.unity.entities@0.11/manual/component_data.html)和 [chunk components](https://docs.unity3d.com/Packages/com.unity.entities@0.11/manual/ecs_chunk_component.html)。
* [IBufferElementData](https://docs.unity3d.com/Packages/com.unity.entities@0.11/api/Unity.Entities.IBufferElementData.html) — 将[动态缓冲区](https://docs.unity3d.com/Packages/com.unity.entities@0.11/manual/dynamic_buffers.html)与实体相关联。
* [ISharedComponentData](https://docs.unity3d.com/Packages/com.unity.entities@0.11/api/Unity.Entities.ISharedComponentData.html) — 在原型中按值对实体进行分类或分组。有关更多信息，请参见[共享组件数据](https://docs.unity3d.com/Packages/com.unity.entities@0.11/manual/shared_component_data.html)。
* [ISystemStateComponentData](https://docs.unity3d.com/Packages/com.unity.entities@0.11/api/Unity.Entities.ISystemStateComponentData.html) — 将系统特定的状态与实体相关联，并检测何时创建或销毁各个实体。有关更多信息，请参见[系统状态组件](https://docs.unity3d.com/Packages/com.unity.entities@0.11/manual/system_state_components.html)。
* [ISharedSystemStateComponentData](https://docs.unity3d.com/Packages/com.unity.entities@0.11/api/Unity.Entities.ISystemStateSharedComponentData.html) — 共享和系统状态数据的组合。请参阅[系统状态组件](https://docs.unity3d.com/Packages/com.unity.entities@0.11/manual/system_state_components.html)。
* [Blob assets](https://docs.unity3d.com/Packages/com.unity.entities@0.11/api/Unity.Entities.BlobAssetReference-1.html) – 尽管从技术上讲不是“组件”，但是您可以使用Blob资产来存储数据。 Blob资产可以由一个或多个组件使用[BlobAssetReference](https://docs.unity3d.com/Packages/com.unity.entities@0.11/api/Unity.Entities.BlobAssetReference-1.html)进行引用，并且是不可变的。您可以使用Blob资产在资产之间共享数据并访问C＃作业中的数据。

EntityManager将组件的独特组合组织到**archetypes**中。它将具有相同原型的所有实体的组件一起存储在称为块的内存块中。给定块中的实体均具有相同的组件原型。  
![](ArchetypeChunkDiagram.png)

该图说明了ECS如何通过原型存储组件数据块。共享组件和块组件是例外，因为ECS将它们存储在块外部。这些组件类型的单个实例适用于适用块中的所有实体。此外，您可以选择将动态缓冲区存储在块之外。即使ECS不在块内部存储这些类型的组件，您在查询实体时通常也可以将它们与其他组件类型相同。

此图说明了ECS如何按原型存储组件数据块。共享组件和chunk组件是例外，因为ECS将它们存储在chunk之外。这些组件类型的单个实例应用于适用块中的所有实体。此外，还可以选择在块之外存储动态缓冲区。即使ECS不在chunk内存储这些类型的组件，但在查询实体时，通常可以将它们与其他组件类型一样对待。