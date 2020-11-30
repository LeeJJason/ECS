# [Looking up data](https://docs.unity3d.com/Packages/com.unity.entities@0.11/manual/ecs_lookup_data.html)
访问和修改ECS数据的最有效方法是使用带有实体查询和作业的系统。 这样可以以最少的内存高速缓存未命中来最佳利用CPU资源。 实际上，数据设计的目标之一应该是使用最高效，最快的路径来执行大部分数据转换。 但是，有时您需要在程序的任意位置访问任意实体的任意组件。

给定一个Entity对象，您可以在其IComponentData和动态缓冲区中查找数据。 方法的不同取决于您的代码是在使用Entities.ForEach还是使用IJobChunk作业的系统中执行，还是在主线程上的其他位置执行。

## Looking up entity data in a system
使用[GetComponent <T>（Entity）](https://docs.unity3d.com/Packages/com.unity.entities@0.11/api/Unity.Entities.SystemBase.html#Unity_Entities_SystemBase_GetComponent__1_Unity_Entities_Entity_)从系统的[Entities.ForEach](https://docs.unity3d.com/Packages/com.unity.entities@0.11/api/Unity.Entities.SystemBase.html#Unity_Entities_SystemBase_Entities)或[Job.WithCode]函数内部查找存储在任意实体组件中的数据。

例如，如果您的“目标”组件的“实体”字段标识了要定位的实体，则可以使用以下代码将跟踪实体向其目标旋转：

访问存储在动态缓冲区中的数据需要额外的步骤。 您必须在OnUpdate（）方法中声明BufferFromEntity类型的局部变量。 然后，您可以在lambda函数中“捕获”局部变量。

## Looking up entity data in IJobChunk
要随机访问IJobChunk或其他作业结构中的组件数据，请使用以下类型之一来获取组件的类似数组的接口，并由Entity对象索引：
* [ComponentDataFromEntity](https://docs.unity3d.com/Packages/com.unity.entities@0.11/api/Unity.Entities.ComponentDataFromEntity-1.html)
* [BufferFromEntity](https://docs.unity3d.com/Packages/com.unity.entities@0.11/api/Unity.Entities.BufferFromEntity-1.html)

声明一个类型为[ComponentDataFromEntity](https://docs.unity3d.com/Packages/com.unity.entities@0.11/api/Unity.Entities.ComponentDataFromEntity-1.html)或[BufferFromEntity](https://docs.unity3d.com/Packages/com.unity.entities@0.11/api/Unity.Entities.BufferFromEntity-1.html)的字段，并在计划作业之前设置该字段的值。

例如，如果您的“目标”组件的“实体”字段标识了要定位的实体，则可以将以下字段添加到作业结构中以查找目标的世界位置：

请注意，此声明使用ReadOnly属性。 除非确实写入要访问的组件，否则应始终将ComponentDataFromEntity对象声明为只读。

您可以在安排作业时按以下方式设置此字段：

在作业的Execute（）函数内部，可以使用Entity对象查找组件的值：

以下完整示例显示了一个系统，该系统将具有包含其目标的Entity对象的Target字段的实体移向目标的当前位置：

## Data access errors
如果您要查找的数据与您直接在作业中读取和写入的数据重叠，则随机访问会导致争用情况和细微的错误。 当您确定要在作业中直接读取或写入的特定实体数据与您正在随机读取或写入的特定实体数据之间没有重叠时，则可以使用[NativeDisableParallelForRestriction](https://docs.unity3d.com/ScriptReference/Unity.Collections.NativeDisableParallelForRestrictionAttribute.html)属性标记访问器对象。