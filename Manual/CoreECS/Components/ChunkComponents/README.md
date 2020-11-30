# Chunk components
使用组块组件将数据与特定[组块](https://docs.unity3d.com/Packages/com.unity.entities@0.11/api/Unity.Entities.ArchetypeChunk.html)关联。

块组件包含适用于特定块中所有实体的数据。 例如，如果您有代表按接近度组织的3D对象的实体块，则可以使用块组件为它们存储集合边界框。 块组件使用接口类型[IComponentData](https://docs.unity3d.com/Packages/com.unity.entities@0.11/api/Unity.Entities.IComponentData.html)。

## Add and set the values of a chunk component
尽管组块组件可以具有单个组块唯一的值，但它们仍然是该组块中实体原型的一部分。 因此，如果您从实体中删除了一个大块组件，ECS会将该实体移动到另一个大块（可能是一个新的大块）。 同样，如果您将块组件添加到实体，则ECS会将该实体移至其他块，因为其原型会更改； 块组件的添加不会影响原始块中的其余实体。

如果您在块中使用实体来更改块组件的值，则它将更改该块中所有实体所共有的块组件的值。 如果更改实体的原型，以便将其移入具有相同类型的组块组件的新组块，那么目标组块中的现有值将不受影响。 注意：如果将实体移至新创建的块，则ECS会为该块创建一个新的块组件并分配其默认值。

使用块组件和通用组件之间的主要区别在于，您使用不同的功能来添加，设置和删除它们。 块组件还具有自己的ComponentType函数，可用于定义实体原型和查询。

### Relevant APIs
|Purpose|Function|
|:------|:-------|
|Declaration|[IComponentData](https://docs.unity3d.com/Packages/com.unity.entities@0.11/api/Unity.Entities.IComponentData.html)|
||
|[ArchetypeChunk methods](https://docs.unity3d.com/Packages/com.unity.entities@0.11/api/Unity.Entities.ArchetypeChunk.html)||
|Read|[GetChunkComponentData(ArchetypeChunkComponentType)](https://docs.unity3d.com/Packages/com.unity.entities@0.11/api/Unity.Entities.ArchetypeChunk.html#Unity_Entities_ArchetypeChunk_GetChunkComponentData_)|
|Check|[HasChunkComponent(ArchetypeChunkComponentType)](https://docs.unity3d.com/Packages/com.unity.entities@0.11/api/Unity.Entities.ArchetypeChunk.html#Unity_Entities_ArchetypeChunk_HasChunkComponent_)|
|Write|[SetChunkComponentData(ArchetypeChunkComponentType, T)](https://docs.unity3d.com/Packages/com.unity.entities@0.11/api/Unity.Entities.ArchetypeChunk.html#Unity_Entities_ArchetypeChunk_SetChunkComponentData_)|
 || |
|[EntityManager methods](https://docs.unity3d.com/Packages/com.unity.entities@0.11/api/Unity.Entities.EntityManager.html)| |
|Create|[AddChunkComponentData(Entity)](https://docs.unity3d.com/Packages/com.unity.entities@0.11/api/Unity.Entities.EntityManager.html#Unity_Entities_EntityManager_AddChunkComponentData__1_Unity_Entities_Entity_)|
|Create|[AddChunkComponentData(EntityQuery, T)](https://docs.unity3d.com/Packages/com.unity.entities@0.11/api/Unity.Entities.EntityManager.html#Unity_Entities_EntityManager_AddChunkComponentData__1_Unity_Entities_EntityQuery___0_)|
|Create|[AddComponents(Entity,ComponentTypes)](https://docs.unity3d.com/Packages/com.unity.entities@0.11/api/Unity.Entities.EntityManager.html#Unity_Entities_EntityManager_AddComponents_Unity_Entities_Entity_Unity_Entities_ComponentTypes_)|
|Get type info|[GetArchetypeChunkComponentType(Boolean)](https://docs.unity3d.com/Packages/com.unity.entities@0.11/api/Unity.Entities.EntityManager.html#Unity_Entities_EntityManager_GetArchetypeChunkComponentType_)|
|Read|[GetChunkComponentData(ArchetypeChunk)](https://docs.unity3d.com/Packages/com.unity.entities@0.11/api/Unity.Entities.EntityManager.html#Unity_Entities_EntityManager_GetChunkComponentData__1_Unity_Entities_ArchetypeChunk_)|
|Read|[GetChunkComponentData(Entity)](https://docs.unity3d.com/Packages/com.unity.entities@0.11/api/Unity.Entities.EntityManager.html#Unity_Entities_EntityManager_GetChunkComponentData__1_Unity_Entities_Entity_)|
|Check|[HasChunkComponent(Entity)](https://docs.unity3d.com/Packages/com.unity.entities@0.11/api/Unity.Entities.EntityManager.html#Unity_Entities_EntityManager_HasChunkComponent_)|
|Delete|[RemoveChunkComponent(Entity)](https://docs.unity3d.com/Packages/com.unity.entities@0.11/api/Unity.Entities.EntityManager.html#Unity_Entities_EntityManager_RemoveChunkComponent__1_Unity_Entities_Entity_)|
|Delete|[RemoveChunkComponentData(EntityQuery)](https://docs.unity3d.com/Packages/com.unity.entities@0.11/api/Unity.Entities.EntityManager.html#Unity_Entities_EntityManager_RemoveChunkComponentData_)|
|Write|[EntityManager.SetChunkComponentData(ArchetypeChunk, T)](https://docs.unity3d.com/Packages/com.unity.entities@0.11/api/Unity.Entities.EntityManager.html#Unity_Entities_EntityManager_SetChunkComponentData_)|

## Declaring a chunk component
块组件使用接口类型[IComponentData](https://docs.unity3d.com/Packages/com.unity.entities@0.11/api/Unity.Entities.IComponentData.html)。

## Creating a chunk component
要直接添加块组件，请在目标块中使用一个实体，或使用选择一组目标块的实体查询。 您不能在作业内添加块组件，也不能与`EntityCommandBuffer`一起添加。

您还可以将块组件作为[EntityArchetype](https://docs.unity3d.com/Packages/com.unity.entities@0.11/api/Unity.Entities.EntityArchetype.html)或ECS用于创建实体的[ComponentType](https://docs.unity3d.com/Packages/com.unity.entities@0.11/api/Unity.Entities.ComponentType.html)对象列表的一部分。 ECS为每个块创建块组件，并存储具有该原型的实体。

通过这些方法使用`ComponentType.ChunkComponent <T>`或`ComponentType.ChunkComponentReadOnly <T>`。否则，ECS将该组件视为通用组件，而不是块组件。

### With an entity in a chunk
给定目标块中的实体，您可以使用`EntityManager.AddChunkComponentData <T>（）`函数将块组件添加到块中：

使用此方法时，不能立即为块组件设置值。

### With an [EntityQuery](https://docs.unity3d.com/Packages/com.unity.entities@0.11/api/Unity.Entities.EntityQuery.html)
给定一个实体查询，该查询选择了要添加块组件的所有块，您可以使用`EntityManager.AddChunkComponentData <T>（）`函数添加和设置组件：

使用此方法时，可以为所有新块组件设置相同的初始值。

### With an [EntityArchetype](https://docs.unity3d.com/Packages/com.unity.entities@0.11/api/Unity.Entities.EntityArchetype.html)
当您创建具有原型或组件类型列表的实体时，请在原型中包含块组件类型：

或组件类型列表：

使用这些方法时，ECS作为实体构造的一部分创建的新块的块组件将接收默认结构值。ECS不会更改现有块中的块组件。请参阅[更新块组件](https://docs.unity3d.com/Packages/com.unity.entities@0.11/manual/ecs_chunk_component.html#update)，以了解如何在给定实体引用的情况下设置块组件值。

## Reading a chunk component
要读取块组件，可以使用代表块的[ArchetypeChunk](https://docs.unity3d.com/Packages/com.unity.entities@0.11/api/Unity.Entities.ArchetypeChunk.html)对象，或在目标块中使用实体。

### With the ArchetypeChunk instance
给定一个块，您可以使用[EntityManager.GetChunkComponentData <T>](https://docs.unity3d.com/Packages/com.unity.entities@0.11/api/Unity.Entities.EntityManager.html#Unity_Entities_EntityManager_GetChunkComponentData__1_Unity_Entities_ArchetypeChunk_)函数读取其块组件。以下代码遍历与查询匹配的所有块，并访问类型的块组件`ChunkComponentA`：

### With an entity in a chunk
给定一个实体，您可以使用[EntityManager.GetChunkComponentData <T>](https://docs.unity3d.com/Packages/com.unity.entities@0.11/api/Unity.Entities.EntityManager.html#Unity_Entities_EntityManager_GetChunkComponentData__1_Unity_Entities_Entity_)访问包含该实体的块中的块组件：

## Updating a chunk component
您可以在引用其所属块的基础上更新块组件。 在`IJobChunk`作业中，可以调用[ArchetypeChunk.SetChunkComponentData](https://docs.unity3d.com/Packages/com.unity.entities@0.11/api/Unity.Entities.ArchetypeChunk.html#Unity_Entities_ArchetypeChunk_SetChunkComponentData_)。 在主线程上，可以使用EntityManager版本：[EntityManager.SetChunkComponentData](https://docs.unity3d.com/Packages/com.unity.entities@0.11/api/Unity.Entities.EntityManager.html#Unity_Entities_EntityManager_SetChunkComponentData_)。 注意：您无法使用SystemBase Entities.ForEach访问块组件，因为您无权访问`ArchetypeChunk`对象或EntityManager。

### With the ArchetypeChunk instance
要更新作业中的块组件，请参阅《[在系统中读写](https://docs.unity3d.com/Packages/com.unity.entities@0.11/manual/ecs_chunk_component.html#read-and-write-jcs)》。

要更新主线程上的块组件，请使用EntityManager：

### With an Entity instance
如果在块中有一个实体，而不是块引用本身，则还可以使用EntityManger来获取包含该实体的块：

注意：如果只想读取块组件而不要写入块组件，则在定义实体查询时应使用[ComponentType.ChunkComponentReadOnly](https://docs.unity3d.com/Packages/com.unity.entities@0.11/api/Unity.Entities.ComponentType.html#Unity_Entities_ComponentType_ChunkComponentReadOnly_)，以避免创建不必要的作业调度约束。

## Deleting a chunk component
使用[EntityManager.RemoveChunkComponent](https://docs.unity3d.com/Packages/com.unity.entities@0.11/api/Unity.Entities.EntityManager.html#Unity_Entities_EntityManager_RemoveChunkComponent_)函数删除块组件。您可以删除目标块中给定实体的块组件，也可以从实体查询选择的所有块中删除给定类型的所有块组件。

如果从单个实体中删除块组件，则该实体将移至其他块，因为该实体的原型会更改。只要该块中还有其他实体，该块就会保留未更改的块组件。

## Using a chunk component in a query
要在实体查询中使用块组件，必须使用`ComponentType.ChunkComponent <T>`或`ComponentType.ChunkComponentReadOnly <T>`函数来指定类型。否则，ECS将该组件视为通用组件，而不是块组件。

在[EntityQueryDesc](https://docs.unity3d.com/Packages/com.unity.entities@0.11/manual/Unity.Entities.EntityQueryDesc)中

您可以使用以下查询描述来创建实体查询，该查询选择所有块以及这些块中具有类型为ChunkComponentA的块组件的实体：

## Iterating over chunks to set chunk components
要遍历要为其设置块组件的所有块，可以创建一个实体查询，该实体查询选择正确的块，然后使用EntityQuery对象获取ArchetypeChunk实例的列表作为本机数组。ArchetypeChunk对象允许您将新值写入组块组件。

请注意，如果需要读取块中的组件以确定块组件的正确值，则应使用[IJobChunk](https://docs.unity3d.com/Packages/com.unity.entities@0.11/manual/ecs_chunk_component.html#read-and-write-jcs)。例如，以下代码为包含具有LocalToWorld组件的实体的所有块计算与轴对齐的边界框：