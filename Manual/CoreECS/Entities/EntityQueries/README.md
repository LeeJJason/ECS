# [Using a EntityQuery to query data](https://docs.unity3d.com/Packages/com.unity.entities@0.11/manual/ecs_entity_query.html)
要读取或写入数据，必须首先找到要更改的数据。 ECS中的数据存储在组件中，这些ECS根据它们所属实体的原型在内存中分组在一起。您可以使用[EntityQuery](https://docs.unity3d.com/Packages/com.unity.entities@0.11/api/Unity.Entities.EntityQuery.html)获取ECS数据的视图，该视图仅包含给定算法或流程所需的特定数据。

要读取或写入数据，必须首先找到要更改的数据。ECS中的数据存储在组件中，这些组件根据所属实体的原型在内存中分组。您可以使用[EntityQuery](https://docs.unity3d.com/Packages/com.unity.entities@0.11/api/Unity.Entities.EntityQuery.html)获取ECS数据的视图，该视图只包含给定算法或进程所需的特定数据。

您可以使用EntityQuery执行以下操作：
* 运行作业以处理为视图选择的实体和组件
* 获取一个包含所有选定实体的NativeArray
* 获取选定组件的NativeArrays（按组件类型）

保证EntityQuery返回的实体和组件数组是“并行”的，也就是说，相同的索引值始终应用于任何数组中的相同实体。

**注意**：`SystemBase.Entities.ForEach`构造基于为这些API指定的组件类型和属性创建内部EntityQuery实例。您不能将其他EntityQuery对象与`Entities.ForEach`一起使用（尽管您可以获取`Entities.ForEach`实例构造的查询对象并在其他地方使用它）。

## Defining a query
EntityQuery查询定义了原型必须包含的组件类型集，以便ECS在视图中包含其块和实体。也可以排除包含特定类型组件的原型。

对于简单查询，您可以基于组件类型数组创建EntityQuery。以下示例定义了一个EntityQuery，该查询查找具有RotationQuaternion和RotationSpeed组件的所有实体。
```cs
EntityQuery m_Query = GetEntityQuery(typeof(RotationQuaternion), 
    ComponentType.ReadOnly<RotationSpeed>());
```

该查询使用`ComponentType.ReadOnly <T>`而不是更简单的`typeof`表达式来指定系统不写入RotationSpeed。 尽可能始终指定只读，因为对数据的读取访问限制较少，这可以帮助作业调度程序更有效地执行作业。

### EntityQueryDesc
对于更复杂的查询，可以使用EntityQueryDesc对象创建EntityQuery。 EntityQueryDesc提供了一种灵活的查询机制，可以根据以下几组组件指定要选择的原型：
* `All`：此数组中的所有组件类型必须存在于原型中
* `Any`：原型中必须存在此数组中的至少一种组件类型
* `None`：原型中不能存在此数组中的任何组件类型

例如，以下查询包括包含RotationQuaternion和RotationSpeed组件的原型，但不包括包含Frozen组件的任何原型：
```cs
var query = new EntityQueryDesc
{
   None = new ComponentType[]{ typeof(Frozen) },
   All = new ComponentType[]{ typeof(RotationQuaternion), 
       ComponentType.ReadOnly<RotationSpeed>() }
}
EntityQuery m_Query = GetEntityQuery(query);
```

注意：不要在EntityQueryDesc中包括可选组件。要处理可选组件，请使用`ArchetypeChunk.Has <T>（）`方法确定块中是否包含可选组件。因为同一块中的所有实体都具有相同的组件，所以您只需要检查一个可选组件是否每个块存在一次：而不是每个实体一次。

### Query options
创建EntityQueryDesc时，可以设置其`Options`变量。这些选项允许进行专门的查询（通常不需要设置它们）：
* `Default`：未设置任何选项。查询行为正常。
* `IncludePrefab`：包含包含特殊Prefab标签组件的原型。
* `IncludeDisabled`：包含包含特殊的Disabled标签组件的原型。
* `FilterWriteGroup`：考虑查询中任何组件的WriteGroup。

设置`FilterWriteGroup`选项时，视图中仅包含具有明确包含在查询中的 Write Group 中那些组件的实体。 ECS从同一WriteGroup中排除具有任何其他组件的任何实体。

在以下示例中，C2和C3是基于C1的同一 Write Group 中的组件，并且此查询使用需要C1和C3的FilterWriteGroup选项：
```cs
public struct C1: IComponentData{}

[WriteGroup(C1)]
public struct C2: IComponentData{}

[WriteGroup(C1)]
public struct C3: IComponentData{}

// ... In a system:
var query = new EntityQueryDesc{
    All = new ComponentType{typeof(C1), ComponentType.ReadOnly<C3>()},
    Options = EntityQueryDescOptions.FilterWriteGroup
};
var m_Query = GetEntityQuery(query);
```
此查询排除具有C2和C3的所有实体，因为查询中未明确包含C2。虽然您可以使用`None`将其设计到查询中，但是通过Write Group进行操作有一个重要的好处：您无需更改其他系统使用的查询（只要这些系统也使用Write Groups）。

Write Group是一种可用于扩展现有系统的机制。例如，如果C1和C2是在另一个系统中定义的（也许是您无法控制的库的一部分），则可以将C3与C2放在同一Write Group中以更改C1的更新方式。对于您添加到C3组件的任何实体，系统都会更新C1，而原始系统不会。对于其他没有C3的实体，原始系统将像以前一样更新C1。


有关更多信息，请参见[Write Group](https://docs.unity3d.com/Packages/com.unity.entities@0.11/manual/ecs_write_groups.html)。

## Combining queries
要组合多个查询，可以传递EntityQueryDesc对象的数组而不是单个实例。您必须使用逻辑 `OR` 运算来组合每个查询。以下示例选择包含RotationQuaternion组件或RotationSpeed组件（或两者）的任何原型：
```cs
var query0 = new EntityQueryDesc
{
   All = new ComponentType[] {typeof(RotationQuaternion)}
};

var query1 = new EntityQueryDesc
{
   All = new ComponentType[] {typeof(RotationSpeed)}
};

EntityQuery m_Query 
    = GetEntityQuery(new EntityQueryDesc[] {query0, query1});

```

## Creating a EntityQuery
在系统类之外，可以使用`EntityManager.CreateEntityQuery（）`函数创建EntityQuery，如下所示：
```cs
EntityQuery m_Query = CreateEntityQuery(typeof(RotationQuaternion), 
    ComponentType.ReadOnly<RotationSpeed>());
```

但是，在系统类中，必须将`GetEntityQuery（）`函数与IJobChunk作业一起使用：
```cs
public class RotationSpeedSystem : SystemBase
{
   private EntityQuery m_Query;
   protected override void OnCreate()
   {
       m_Query = GetEntityQuery(typeof(RotationQuaternion), 
           ComponentType.ReadOnly<RotationSpeed>());
   }
   //…
}
```

如果您打算重复使用同一视图，请缓存EntityQuery实例，而不是为每次使用创建一个新视图。例如，在系统中，您可以在系统的`OnCreate（）`函数中创建EntityQuery并将结果存储在实例变量中。上例中的`m_Query`变量用于此目的。

请注意，为系统创建的查询由系统缓存。如果一个查询已经存在，则`GetEntityQuery（）`返回现有查询，而不是创建一个新查询。但是，在评估两个查询是否相同时不考虑过滤器设置。此外，如果您在查询上设置过滤器，则下次使用`GetEntityQuery（）`访问同一查询时，将设置相同的过滤器。使用`ResetFilter（）`清除现有过滤器。

### Defining filters
您可以过滤视图以及定义必须包括在查询中或从查询中排除的组件。您可以指定以下类型的过滤器：
* **Shared component filter**: 根据共享组件的特定值过滤实体集。
* **Change filter**: 根据特定组件类型的值是否已更改来过滤实体集。
  
设置的过滤器将保持有效，直到您在查询对象上调用`ResetFilter（）`为止。

### Shared component filters
若要使用共享组件筛选器，请将共享组件包括在EntityQuery中（以及其他所需组件），然后调用`SetFilter（）`函数。然后传入包含要选择的值的相同ISharedComponent类型的结构。所有值必须匹配。您最多可以向过滤器添加两个不同的共享组件。

您可以随时更改过滤器，但是如果您更改过滤器，则它不会更改从组`ToComponentDataArray（）`或`ToEntityArray（）`函数中接收到的任何现有实体或组件数组。您必须重新创建这些数组。

以下示例定义了一个名为SharedGrouping的共享组件，以及一个仅处理Group字段设置为1的实体的系统。
```cs
struct SharedGrouping : ISharedComponentData
{
    public int Group;
}

class ImpulseSystem : SystemBase
{
    EntityQuery m_Query;

    protected override void OnCreate(int capacity)
    {
        m_Query = GetEntityQuery(typeof(Position), 
            typeof(Displacement), 
            typeof(SharedGrouping));
    }

    protected override void OnUpdate()
    {
        // Only iterate over entities that have the SharedGrouping data set to 1
        m_Query.SetFilter(new SharedGrouping { Group = 1 });

        var positions = m_Query.ToComponentDataArray<Position>(Allocator.Temp);
        var displacememnts = m_Query.ToComponentDataArray<Displacement>(Allocator.Temp);

        for (int i = 0; i != positions.Length; i++)
            positions[i].Value = positions[i].Value + displacememnts[i].Value;
    }
}

```

### Change filters
如果仅在组件值更改后才需要更新实体，则可以使用`SetFilterChanged（）`函数将该组件添加到EntityQuery过滤器中。
例如，以下EntityQuery仅包括来自另一个系统已经写入到Translation组件的块的实体：
```cs
protected override void OnCreate(int capacity)
{
    m_Query = GetEntityQuery(typeof(LocalToWorld), 
        ComponentType.ReadOnly<Translation>());
    m_Query.SetFilterChanged(typeof(Translation));
}
```

注意：为了提高效率，更改过滤器适用于整个块，而不适用于单个实体。更改筛选器还仅检查系统是否已运行了已声明对组件进行写访问的系统，而不检查它是否实际上更改了任何数据。换句话说，如果另一个具有写入该类型组件能力的作业访问该块，则更改过滤器将包括该块中的所有实体。这就是为什么您应该始终声明对不需要修改的组件的只读访问权限的原因。

## Executing the query
当您在作业中使用EntityQuery或调用其中一个返回视图中的实体，组件或块的数组的EntityQuery方法之一时，EntityQuery执行其查询：
* `ToEntityArray()` 返回所选实体的数组。
* `ToComponentDataArray<T>` 返回所选实体的类型为T的组件的数组。
* `CreateArchetypeChunkArray()` 返回包含所选实体的所有块。因为查询操作的原型，共享组件值和更改筛选器对于块中的所有实体都是相同的，所以在返回的块中存储的实体集与实体`ToEntityArray（）`完全相同返回。

### In jobs
在计划IJobChunk作业的系统中，将EntityQuery对象传递给作业的`ScheduleParallel（）`或`ScheduleSingle（）`方法。
在以下示例中，从HelloCube IJobChunk示例中，`m_Query`参数是EntityQuery对象
```cs
// OnUpdate runs on the main thread.
protected override void OnUpdate()
{
    var rotationType 
        = GetArchetypeChunkComponentType<Rotation>(false); 
    var rotationSpeedType 
        = GetArchetypeChunkComponentType<RotationSpeed>(true);

    var job = new RotationSpeedJob()
    {
        RotationType = rotationType,
        RotationSpeedType = rotationSpeedType,
        DeltaTime = Time.deltaTime
    };

    return job.ScheduleParallel(m_Query, this.Dependency);
}
```

EntityQuery在内部使用作业来创建所需的数组。当您将组传递给`Schedule（）`方法之一时，ECS会调度EntityQuery作业以及系统自身的作业，因此您可以利用并行处理。