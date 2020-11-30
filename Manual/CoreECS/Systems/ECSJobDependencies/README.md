# ECS job dependencies
Unity根据系统读取和写入的ECS组件分析每个系统的数据依赖性。 如果在帧中较早更新的系统读取了较新系统写入的数据，或者写入了较新系统读取的数据，则第二个系统将依赖于第一个系统。 为了防止出现竞争状况，作业调度程序确保系统所依赖的所有作业在运行该系统的作业之前都已完成。

系统的[Dependency](https://docs.unity3d.com/Packages/com.unity.entities@0.11/api/Unity.Entities.SystemBase.html#Unity_Entities_SystemBase_Dependency)属性是[JobHandle](https://docs.unity3d.com/ScriptReference/Unity.Jobs.JobHandle.html)，代表与系统的ECS相关的依赖关系。 在[OnUpdate（）](https://docs.unity3d.com/Packages/com.unity.entities@0.11/api/Unity.Entities.SystemBase.html#Unity_Entities_SystemBase_OnUpdate_)之前，[Dependency](https://docs.unity3d.com/Packages/com.unity.entities@0.11/api/Unity.Entities.SystemBase.html#Unity_Entities_SystemBase_Dependency)属性反映了系统对先前作业的传入依赖关系。 默认情况下，系统根据您在系统中计划作业时读取和写入的组件来更新[Dependency](https://docs.unity3d.com/Packages/com.unity.entities@0.11/api/Unity.Entities.SystemBase.html#Unity_Entities_SystemBase_Dependency)属性。

要覆盖此默认行为，请使用[Entities.ForEach](https://docs.unity3d.com/Packages/com.unity.entities@0.11/api/Unity.Entities.SystemBase.html#Unity_Entities_SystemBase_Entities)和[Job.WithCode](https://docs.unity3d.com/Packages/com.unity.entities@0.11/api/Unity.Entities.SystemBase.html#Unity_Entities_SystemBase_Job)的重载版本，这些重载版本将作业依赖项作为参数，并将更新后的依赖项作为[JobHandle](https://docs.unity3d.com/ScriptReference/Unity.Jobs.JobHandle.html)返回。 使用这些构造的显式版本时，ECS不会自动将作业句柄与系统的[Dependency](https://docs.unity3d.com/Packages/com.unity.entities@0.11/api/Unity.Entities.SystemBase.html#Unity_Entities_SystemBase_Dependency)属性结合在一起。 您必须在需要时手动组合它们。

请注意，系统的Dependency属性不会跟踪作业对通过NativeArrays或其他类似容器传递的数据可能具有的依赖关系。 如果在一个作业中编写NativeArray并在另一个作业中读取该数组，则必须手动添加第一个作业的JobHandle作为第二个作业的依赖项（通常使用JobHandle.CombineDependencies）。

当您调用Entities.ForEach.Run（）时，作业计划程序会在开始ForEach迭代之前完成系统所依赖的所有计划作业。 如果您还使用WithStructuralChanges（）作为构造的一部分，则作业计划程序将完成所有正在运行和计划的作业。 结构上的更改也会使对组件数据的任何直接引用无效。 有关更多信息，请参见同步点。

有关更多信息，请参见[JobHandle and dependencies](https://docs.unity3d.com/Manual/JobSystemJobDependencies.html)。