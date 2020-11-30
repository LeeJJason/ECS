# [Dynamic buffer components](https://docs.unity3d.com/Packages/com.unity.entities@0.11/manual/dynamic_buffers.html)

使用动态缓冲区组件将类似数组的数据与实体相关联。动态缓冲区是ECS组件，可以容纳可变数量的元素，并根据需要自动调整大小。

要创建动态缓冲区，首先声明一个实现[IBufferElementData](https://docs.unity3d.com/Packages/com.unity.entities@0.11/api/Unity.Entities.IBufferElementData.html)的结构并定义存储在缓冲区中的元素。例如，可以对存储整数的缓冲区组件使用以下结构：

要将动态缓冲区与实体相关联，请直接向实体添加[IBufferElementData](https://docs.unity3d.com/Packages/com.unity.entities@0.11/api/Unity.Entities.IBufferElementData.html)组件，而不要添加[动态缓冲区容器](https://docs.unity3d.com/Packages/com.unity.entities@0.11/api/Unity.Entities.DynamicBuffer-1.html)本身。

ECS管理容器。对于大多数用途，您可以使用声明的`IBufferElementData`类型将动态缓冲区与其他任何ECS组件相同。例如，您可以`IBufferElementData`在[entity queries](https://docs.unity3d.com/Packages/com.unity.entities@0.11/api/Unity.Entities.EntityQuery.html)中以及在添加或删除缓冲区组件时使用该类型。但是，您必须使用不同的函数来访问缓冲区组件，并且这些函数提供了[DynamicBuffer](https://docs.unity3d.com/Packages/com.unity.entities@0.11/api/Unity.Entities.DynamicBuffer-1.html)实例，该实例为缓冲区数据提供了类似于数组的接口。

若要为动态缓冲区组件指定“内部容量”，请使用[InternalBufferCapacity](https://docs.unity3d.com/Packages/com.unity.entities@0.11/api/Unity.Entities.InternalBufferCapacityAttribute.html)属性。内部容量定义了动态缓冲区与实体的其他组件一起存储在[ArchetypeChunk](https://docs.unity3d.com/Packages/com.unity.entities@0.11/api/Unity.Entities.ArchetypeChunk.html)中的元素数。除了内部容量之外，缓冲区还会在当前块之外分配一个堆内存块并将所有现有元素移走，ECS会自动管理该外部缓冲区内存，并在删除缓冲区组件时释放内存。

注意：如果缓冲区中的数据不是动态的，则可以使用[Blob Asset](https://docs.unity3d.com/Packages/com.unity.entities@0.11/api/Unity.Entities.BlobBuilder.html)代替动态缓冲区。Blob Asset可以存储结构化数据，包括数组。多个实体可以共享Blob Asset。

## Declaring buffer element types
要声明缓冲区，请声明一个结构，该结构定义要放入缓冲区中的元素的类型。该结构必须实现[IBufferElementData](https://docs.unity3d.com/Packages/com.unity.entities@0.11/api/Unity.Entities.IBufferElementData.html)，如下所示：

## Adding buffer types to entities
要将缓冲区添加到实体，请添加`IBufferElementData`定义缓冲区元素数据类型的结构，然后将该类型直接添加到实体或[原型](https://docs.unity3d.com/Packages/com.unity.entities@0.11/api/Unity.Entities.EntityArchetype.html)：

### Using EntityManager.AddBuffer()
有关更多信息，请参见[EntityManager.AddBuffer（）](https://docs.unity3d.com/Packages/com.unity.entities@0.11/api/Unity.Entities.EntityManager.html#Unity_Entities_EntityManager_AddBuffer__1_Unity_Entities_Entity_)上的文档。

### Using an archetype
### Using the `[GenerateAuthoringComponent]` attribute
您可以用来`[GenerateAuthoringComponent]`为仅包含一个字段的简单IBufferElementData实现生成创作组件。设置此属性后，您可以将ECS IBufferElementData组件添加到GameObject，以便可以在编辑器中设置缓冲区元素。

例如，如果您声明以下类型，则可以将其直接添加到编辑器中的GameObject中：
```cs
[GenerateAuthoringComponent]
public struct IntBufferElement: IBufferElementData
{
    public int Value;
}
```

在后台，Unity生成一个名为`IntBufferElementAuthoring`（继承自`MonoBehaviour`）的类，该类公开一个公共`List<int>`类型的字段。将包含此生成的创作组件的GameObject转换为实体时，该列表将转换为`DynamicBuffer<IntBufferElement>`，然后添加到转换后的实体中。

请注意以下限制：
* 单个C＃文件中只有一个组件可以具有生成的创作组件，并且C＃文件中不能包含另一个MonoBehaviour。
* `IBufferElementData` 对于包含多个字段的类型，无法自动生成创作组件。
* `IBufferElementData` 无法为具有显式布局的类型自动生成创作组件。

### Using an [EntityCommandBuffer](https://docs.unity3d.com/Packages/com.unity.entities@0.11/api/Unity.Entities.EntityCommandBuffer.html)
将命令添加到实体命令缓冲区时，可以添加或设置缓冲区组件。

使用[AddBuffer](https://docs.unity3d.com/Packages/com.unity.entities@0.11/api/Unity.Entities.EntityCommandBuffer.html#Unity_Entities_EntityCommandBuffer_AddBuffer__1_Unity_Entities_Entity_)为实体创建一个新的缓冲区，这将更改实体的原型。使用[SetBuffer](https://docs.unity3d.com/Packages/com.unity.entities@0.11/api/Unity.Entities.EntityCommandBuffer.html#Unity_Entities_EntityCommandBuffer_SetBuffer__1_Unity_Entities_Entity_)清除现有缓冲区（必须存在）并在其位置创建一个新的空缓冲区。这两个函数都返回一个[DynamicBuffer](https://docs.unity3d.com/Packages/com.unity.entities@0.11/api/Unity.Entities.DynamicBuffer-1.html)实例，您可以使用该实例来填充新缓冲区。您可以立即将元素添加到缓冲区，但是在执行命令缓冲区时将缓冲区添加到实体之前，否则无法访问它们。

以下作业使用命令缓冲区创建一个新实体，然后使用[EntityCommandBuffer.AddBuffer](https://docs.unity3d.com/Packages/com.unity.entities@0.11/api/Unity.Entities.EntityCommandBuffer.html#Unity_Entities_EntityCommandBuffer_AddBuffer__1_Unity_Entities_Entity_)添加一个动态缓冲区组件。作业还向动态缓冲区添加了许多元素。

注意：不需要立即将数据添加到动态缓冲区。但是，直到执行了您正在使用的实体命令缓冲区后，您才能再次访问该缓冲区。

## Accessing buffers
您可以使用[EntityManager](https://docs.unity3d.com/Packages/com.unity.entities@0.11/api/Unity.Entities.EntityManager.html)，[systems](https://docs.unity3d.com/Packages/com.unity.entities@0.11/manual/ecs_systems.html)和[job](https://docs.unity3d.com/Packages/com.unity.entities@0.11/api/Unity.Entities.DynamicBuffer-1.html)来访问[DynamicBuffer](https://docs.unity3d.com/Packages/com.unity.entities@0.11/api/Unity.Entities.DynamicBuffer-1.html)实例，其方式与访问实体的其他组件类型相同。

### EntityManager
您可以使用[EntityManager](https://docs.unity3d.com/Packages/com.unity.entities@0.11/api/Unity.Entities.EntityManager.html)的实例来访问动态缓冲区：

### Looking up buffers of another entity
当您需要查找作业中另一个实体的缓冲区数据时，可以将[BufferFromEntity](https://docs.unity3d.com/Packages/com.unity.entities@0.11/api/Unity.Entities.BufferFromEntity-1.html)变量传递给作业。

### SystemBase Entities.ForEach
通过将缓冲区作为lambda函数参数之一传递，可以访问与使用Entities.ForEach处理的实体相关联的动态缓冲区。 下面的示例将所有存储在`MyBufferElement`类型的缓冲区中的值相加：

请注意，在此示例中，由于我们使用`Run（）`执行代码，因此可以直接写入捕获的`sum`变量。 如果我们将函数安排为在作业中运行，则即使结果为单个值，我们也只能写入本机容器（例如NativeArray）。

### IJobChunk
要访问`IJobChunk`作业中的单个缓冲区，请将缓冲区数据类型传递给该作业，然后使用该数据类型获取[BufferAccessor](https://docs.unity3d.com/Packages/com.unity.entities@0.11/api/Unity.Entities.BufferAccessor-1.html)。 缓冲区访问器是一种类似于数组的结构，可提供对当前块中所有动态缓冲区的访问。

与前面的示例一样，以下示例将所有动态缓冲区的内容加起来，其中包含类型为`MyBufferElement`的元素。 `IJobChunk`作业也可以在每个块上并行运行，因此在示例中，它首先将每个缓冲区的中间和存储在本机数组中，然后使用第二个作业来计算最终和。 在这种情况下，中间数组为每个块保存一个结果，而不是为每个实体保存一个结果。

## Reinterpreting buffers
缓冲区可以重新解释为相同大小的类型。 目的是允许进行受控的类型合并，并在包装元素类型受到阻碍时摆脱它们。 要重新解释，请调用[`Reinterpret <T>`](https://docs.unity3d.com/Packages/com.unity.entities@0.11/api/Unity.Entities.DynamicBuffer-1.html#Unity_Entities_DynamicBuffer_1_Reinterpret_)：

重新解释的缓冲区实例保留了原始缓冲区的安全性，并且可以安全使用。 重新解释的缓冲区引用原始数据，因此对一个重新解释的缓冲区的修改会立即反映在其他缓冲区中。

注意：重新解释功能仅强制所涉及的类型具有相同的长度。 例如，您可以为uint和float缓冲区加上别名而不会引发错误，因为这两种类型均为32位长。 您必须确保重新解释在逻辑上有意义。

## Buffer reference invalidation
每次[结构更改](https://docs.unity3d.com/Packages/com.unity.entities@0.11/manual/sync_points.html#structural-changes)都会使对动态缓冲区的所有引用无效。 结构变化通常会导致实体从一个块移动到另一个块。 小型动态缓冲区可以引用块中的内存（而不是主内存中的内存），因此，在结构更改后需要重新获取它们。