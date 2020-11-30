# [System Update Order](https://docs.unity3d.com/Packages/com.unity.entities@0.11/manual/system_update_order.html)
使用`Component System Groups`来指定系统的更新顺序。 您可以使用系统类声明中的`[UpdateInGroup]`属性将系统分组。 然后，您可以使用`[UpdateBefore]`和`[UpdateAfter]`属性在组中指定更新顺序。

ECS框架会创建一组[default system groups](https://docs.unity3d.com/Packages/com.unity.entities@0.11/manual/system_update_order.html#default-system-groups)，可用于在框架的正确阶段更新系统。 您可以将一个组嵌套在另一个组中，以便组中的所有系统在正确的阶段进行更新，然后还根据其组中的顺序进行更新。

## Component System Groups
ComponentSystemGroup类表示应按特定顺序一起更新的相关组件系统的列表。 ComponentSystemGroup是从ComponentSystemBase派生的，因此它在所有重要方面都像组件系统一样工作-可以相对于其他系统进行排序，具有OnUpdate（）方法等。最相关的是，这意味着可以将组件系统组嵌套在其中 其他组件系统组，形成一个层次结构。

默认情况下，当调用ComponentSystemGroup的Update（）方法时，它将在其成员系统的排序列表中的每个系统上调用Update（）。 如果任何成员系统本身就是系统组，则它们将递归更新自己的成员。 生成的系统顺序遵循树的深度优先遍历。

## System Ordering Attributes
现有的系统排序属性维持，略有不同的语义和限制。
* [UpdateInGroup] — 指定此系统应该是其成员的ComponentSystemGroup。 如果忽略此属性，系统将自动添加到默认的World's SimulationSystemGroup（请参见下文）。
* [UpdateBefore] 和 [UpdateAfter] — 排序系统相对于其他系统。 为这些属性指定的系统类型必须是同一组的成员。 跨组边界的排序是在包含两个系统的适当的最深组中进行的：
    * 例如：如果SystemA在GroupA中，而SystemB在GroupB中，并且GroupA和GroupB都是GroupC的成员，则GroupA和GroupB的顺序将隐式确定SystemA和SystemB的相对顺序； 无需对系统进行明确排序。
* [DisableAutoCreation] — 防止在默认世界初始化期间创建系统。 您必须显式创建和更新系统。 但是，您可以将带有此标签的系统添加到ComponentSystemGroup的更新列表中，然后它将像该列表中的其他系统一样自动进行更新。

## Default System Groups
默认的World包含ComponentSystemGroup实例的层次结构。 只有三个根级别的系统组被添加到Unity Player循环（以下列表还显示了每个组中的预定义成员系统）：
* InitializationSystemGroup (updated at the end of the Initialization phase of the player loop)
    * BeginInitializationEntityCommandBufferSystem
    * CopyInitialTransformFromGameObjectSystem
    * SubSceneLiveLinkSystem
    * SubSceneStreamingSystem
    * EndInitializationEntityCommandBufferSystem
* SimulationSystemGroup (updated at the end of the Update phase of the player loop)
  * BeginSimulationEntityCommandBufferSystem
  * TransformSystemGroup
  * EndFrameParentSystem
  * CopyTransformFromGameObjectSystem
  * EndFrameTRSToLocalToWorldSystem
  * EndFrameTRSToLocalToParentSystem
  * EndFrameLocalToParentSystem
  * CopyTransformToGameObjectSystem
  * LateSimulationSystemGroup
  * EndSimulationEntityCommandBufferSystem
* PresentationSystemGroup (updated at the end of the PreLateUpdate phase of the player loop)
  * BeginPresentationEntityCommandBufferSystem
  * CreateMissingRenderBoundsFromMeshRenderer
  * RenderingSystemBootstrap
  * RenderBoundsUpdateSystem
  * RenderMeshSystem
  * LODGroupSystemV1
  * LodRequirementsUpdateSystem
  * EndPresentationEntityCommandBufferSystem

请注意，此列表的特定内容可能会更改。

## Multiple Worlds
除了（或代替）上述默认世界，您可以创建多个世界。 同一组件系统类可以在多个世界中实例化，并且每个实例可以从更新顺序的不同点以不同的速率进行更新。

当前尚无法手动更新给定世界中的每个系统。 相反，您可以控制在哪个世界中创建哪些系统以及应将其添加到哪些现有系统组中。 因此，自定义WorldB可以实例化SystemX和SystemY，将SystemX添加到默认的World's SimulationSystemGroup，并将SystemY添加到默认的World's PresentationSystemGroup。 这些系统可以像往常一样相对于其组同级对其进行排序，并将与相应的组一起进行更新。

为了支持此用例，现在提供了新的ICustomBootstrap接口：
```cs
public interface ICustomBootstrap
{
    // Returns the systems which should be handled by the default bootstrap process.
    // If null is returned the default world will not be created at all.
    // Empty list creates default world and entrypoints
    List<Type> Initialize(List<Type> systems);
}
```

当实现此接口时，组件系统类型的完整列表将在默认世界初始化之前传递给类`Initialize（）`方法。 定制的引导程序可以遍历此列表，并在所需的任何World中创建系统。 您可以从`Initialize（）`方法返回系统列表，这些系统将作为常规默认世界初始化的一部分创建。

例如，以下是自定义`MyCustomBootstrap.Initialize（）`实现的典型过程：
1. 创建任何其他Worlds及其顶级ComponentSystemGroups。
2. 对于系统类型列表中的每个类型：
    1. 向上遍历ComponentSystemGroup层次结构以找到此系统类型的顶级组。
    2. 如果它是在步骤1中创建的组之一，则在该世界中创建系统，然后使用`group.AddSystemToUpdateList（）`将其添加到层次结构中。
    3. 如果不是，请将此类型附加到列表以返回到DefaultWorldInitialization。
3. 在新的顶级组上调用group.SortSystemUpdateList（）。
    1. （可选）将它们添加到默认世界组之一
4. 将未处理系统的列表返回给DefaultWorldInitialization。

注意：ECS框架通过反射找到您的ICustomBootstrap实现。

## Tips and Best Practices
* 使用[UpdateInGroup]为您编写的每个系统指定系统组。 如果未指定，则隐式默认组为SimulationSystemGroup。
* 使用手动选定的ComponentSystemGroups来更新Unity播放器循环中其他位置的系统。 将[DisableAutoCreation]属性添加到组件系统（或系统组）可防止将其创建或添加到默认系统组。 您仍然可以使用World.GetOrCreateSystem（）手动创建系统，并通过从主线程手动调用MySystem.Update（）进行更新。 这是在Unity Player循环中的其他位置插入系统的简便方法（例如，如果您的系统应在框架中的更高或更早版本运行）。
* 如果可能，请使用现有的EntityCommandBufferSystems而不是添加新的。 EntityCommandBufferSystem表示一个同步点，在该同步点，主线程在处理任何未完成的EntityCommandBuffers之前等待工作线程完成。 与创建新的“气泡”相比，在每个根级系统组中重用预定义的Begin / End系统之一不太可能在帧管道中引入新的“气泡”。
* 避免将自定义逻辑放在ComponentSystemGroup.OnUpdate（）中。 由于ComponentSystemGroup在功能上本身就是一个组件系统，因此可能会很想在其OnUpdate（）方法中添加自定义处理，执行一些工作，生成一些工作等。我们通常建议不要这样做，因为从外部尚不清楚 在组的成员更新之前还是之后执行自定义逻辑。 最好将系统组限制为一种分组机制，并在相对于该组显式排序的单独的组件系统中实现所需的逻辑。