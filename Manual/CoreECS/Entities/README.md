# [Entities](https://docs.unity3d.com/Packages/com.unity.entities@0.11/manual/ecs_entities.html)
实体是**Entity Component System**体系结构的三个主要元素之一。它们代表游戏或应用程序中的各个“事物”。实体既没有行为也没有数据；相反，它标识哪些数据属于一起。[Systems](https://docs.unity3d.com/Packages/com.unity.entities@0.11/manual/ecs_systems.html)提供行为，[components](https://docs.unity3d.com/Packages/com.unity.entities@0.11/manual/ecs_components.html)存储数据。

实体本质上是一个ID。想到它的最简单方法是作为一个超轻量级的GameObject，默认情况下甚至没有名称。实体ID稳定；您可以使用它们来存储对另一个组件或实体的引用。例如，层次结构中的子实体可能需要引用其父实体。

[EntityManager](https://docs.unity3d.com/Packages/com.unity.entities@0.11/api/Unity.Entities.EntityManager.html)管理[世界](https://docs.unity3d.com/Packages/com.unity.entities@0.11/api/Unity.Entities.World.html)中的所有实体。 EntityManager维护实体列表，并组织与实体相关联的数据以获得最佳性能。

尽管实体没有类型，但是可以按与实体相关联的数据组件的类型对实体组进行分类。创建实体并向其中添加组件时，EntityManager会跟踪现有实体上组件的唯一组合。这种独特的组合称为原型。将组件添加到实体时，EntityManager会创建[EntityArchetype](https://docs.unity3d.com/Packages/com.unity.entities@0.11/api/Unity.Entities.EntityArchetype.html)结构。您可以使用现有的EntityArchetypes创建符合该原型的新实体。您也可以预先创建EntityArchetype并使用它来创建实体。

## Creating entities
创建实体的最简单方法是在Unity编辑器中。您可以设置ECS在运行时将放置在Scene中的GameObject和Prefabs转换为实体。对于游戏或应用程序中更动态的部分，您可以创建生成系统，以在一个作业中创建多个实体。最后，您可以使用[EntityManager.CreateEntity](https://docs.unity3d.com/Packages/com.unity.entities@0.11/api/Unity.Entities.EntityManager.html#Unity_Entities_EntityManager_CreateEntity)函数之一来一次创建一个实体。

### Creating entities with an EntityManager
使用[EntityManager.CreateEntity](https://docs.unity3d.com/Packages/com.unity.entities@0.11/api/Unity.Entities.EntityManager.html#Unity_Entities_EntityManager_CreateEntity)函数之一创建一个实体。 ECS在与EntityManager相同的世界中创建实体。

您可以通过以下方式一一创建实体：
* 使用[ComponentType](https://docs.unity3d.com/Packages/com.unity.entities@0.11/api/Unity.Entities.ComponentType.html)对象数组创建具有组件的实体。
* 使用[EntityArchetype](https://docs.unity3d.com/Packages/com.unity.entities@0.11/api/Unity.Entities.EntityArchetype.html)创建具有的组件实体。
* 使用[Instantiate](https://docs.unity3d.com/Packages/com.unity.entities@0.11/api/Unity.Entities.EntityManager.html#Unity_Entities_EntityManager_Instantiate_Unity_Entities_Entity_)复制现有实体，包括其当前数据
* 创建一个没有组件的实体，然后向其添加组件。 （您可以立即添加组件，也可以在需要其他组件时添加。）

您也可以一次创建多个实体：
* 使用[CreateEntity](https://docs.unity3d.com/Packages/com.unity.entities@0.11/api/Unity.Entities.EntityManager.html#Unity_Entities_EntityManager_CreateEntity)使用相同原型的新实体填充NativeArray。
* 使用[Instantiate](https://docs.unity3d.com/Packages/com.unity.entities@0.11/api/Unity.Entities.EntityManager.html#Unity_Entities_EntityManager_Instantiate_Unity_Entities_Entity_)，将现有实体的副本（包括其当前数据）填充到NativeArray中。
* 使用CreateChunk显式创建由指定数量的具有给定原型的实体填充的块。

## Adding and removing components
创建实体后，您可以添加或删除组件。当您执行此操作时，受影响实体的原型会更改，并且EntityManager必须将更改后的数据移动到新的内存块中，并将组件数组压缩到原始块中。

对导致结构更改的实体（即添加或删除组件更改`SharedComponentData`的值以及销毁该实体）的更改不能在作业内部完成，因为这些更改可能会使作业正在处理的数据无效。相反，您可以添加命令来对[EntityCommandBuffer](https://docs.unity3d.com/Packages/com.unity.entities@0.11/api/Unity.Entities.EntityCommandBuffer.html)进行这些类型的更改，并在作业完成后执行此命令缓冲区。

EntityManager提供了从单个实体以及NativeArray中的所有实体中删除组件的函数。有关详细信息，请参阅有关[组件](https://docs.unity3d.com/Packages/com.unity.entities@0.11/manual/ecs_components.html)的文档。

## Iterating entities
对具有一组匹配组件的所有实体进行迭代是ECS体系结构的核心。请参阅[访问实体数据](https://docs.unity3d.com/Packages/com.unity.entities@0.11/manual/chunk_iteration.html)。