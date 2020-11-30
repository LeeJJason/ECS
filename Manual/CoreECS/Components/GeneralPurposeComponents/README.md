# [General purpose components](https://docs.unity3d.com/Packages/com.unity.entities@0.11/manual/component_data.html)
Unity中的`ComponentData`（在标准ECS术语中也称为组件）是仅包含实体[实例](https://docs.unity3d.com/Packages/com.unity.entities@0.11/manual/entities.md)数据的结构。 `ComponentData`不应包含实用程序功能以外的方法来访问结构中的数据。 您应该在系统中实现所有游戏逻辑和行为。 就面向对象的Unity系统而言，这有点类似于Component类，但是只包含变量。

Unity ECS API提供了一个名为[`IComponentData`](https://docs.unity3d.com/Packages/com.unity.entities@0.11/api/Unity.Entities.IComponentData.html)的接口，您可以在代码中实现该接口以声明通用组件类型。

## IComponentData
传统的Unity组件（包括`MonoBehaviour`）是面向对象的类，其中包含行为的数据和方法。 `IComponentData`是纯ECS样式的组件，这意味着它没有定义任何行为，仅定义了数据。 您应该将`IComponentData`实现为struct而不是类，这意味着默认情况下，它是通过值而不是通过引用复制的。 通常，您需要使用以下模式来修改数据：
```cs
var transform = group.transform[index]; // Read

transform.heading = playerInput.move; // Modify
transform.position += deltaTime * playerInput.move * settings.playerMoveSpeed;

group.transform[index] = transform; // Write
```

`IComponentData`结构不得包含对托管对象的引用。 这是因为`ComponentData`驻留在简单的非垃圾收集的跟踪[Chunk内存](https://docs.unity3d.com/Packages/com.unity.entities@0.11/manual/chunk_iteration.html)中，这具有许多性能优势。

## Managed IComponentData
使用托管的`IComponentData`（即使用`class`而不是`struct`声明的`IComponentData`）有助于将现有代码以零碎的方式移植到ECS上，与托管数据进行互操作不适合`ISharedComponentData`，或为数据布局提供原型。

这些组件的使用方式与值类型`IComponentData`相同。 但是，ECS在内部以完全不同（且较慢）的方式来处理它们。 如果不需要托管组件支持，请在应用程序的**Player Settings**（菜单：``Edit > Project Settings > Player > Scripting Define Symbols``）中定义`UNITY_DISABLE_MANAGED_COMPONENTS`，以防止意外使用。

因为托管`IComponentData`是托管类型，所以与值类型`IComponentData`相比，它具有以下性能缺点：
* 不能与Burst编译器一起使用
* 不能在作业结构中使用
* 它不能使用块内存
* 需要垃圾收集

您应该尝试限制托管组件的数量，并尽可能使用blittable类型。

托管`IComponentData`必须实现`IEquatable <T>`接口，并重写`Object.GetHashCode（）`。 此外，出于序列化的目的，托管组件必须是默认可构造的。

您必须在主线程上设置组件的值。 为此，请使用`EntityManager`或`EntityCommandBuffer`。 因为组件是引用类型，所以与[`ISharedComponentData`](https://docs.unity3d.com/Packages/com.unity.entities@0.11/api/Unity.Entities.ISharedComponentData.html)不同，您可以更改组件的值而无需在块之间移动实体。 这不会创建一个同步点。

但是，尽管托管组件在逻辑上与值类型组件分开存储，但是它们仍然有助于实体的`EntityArchetype`定义。 这样，向实体添加新的托管组件仍然会导致ECS创建新的原型（如果尚不存在匹配的原型），并将实体移至新的Chunk。

有关示例，请参见文件：`/Packages/com.unity.entities/Unity.Entities/IComponentData.cs`。