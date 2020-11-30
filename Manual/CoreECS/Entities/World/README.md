# World
World 同时拥有[EntityManager](https://docs.unity3d.com/Packages/com.unity.entities@0.11/api/Unity.Entities.EntityManager.html)和一组[ComponentSystem](https://docs.unity3d.com/Packages/com.unity.entities@0.11/manual/component_system.md)。您可以根据需要创建任意多个World对象。通常，您可以创建一个模拟World 以及一个渲染或演示World。

默认情况下，当您进入**Play Mode**时，您将创建一个单一的世界，并使用项目中所有可用的`ComponentSystem`对象填充它。但是，您可以禁用默认世界创建，并通过全局定义将其替换为您自己的代码，如下所示：
* `#UNITY_DISABLE_AUTOMATIC_SYSTEM_BOOTSTRAP_RUNTIME_WORLD` 禁用默认运行时世界的生成。
* `#UNITY_DISABLE_AUTOMATIC_SYSTEM_BOOTSTRAP_EDITOR_WORLD` 禁用默认编辑器世界的生成。
* `#UNITY_DISABLE_AUTOMATIC_SYSTEM_BOOTSTRAP` 禁用两个默认世界的生成。

## Further information
* Default World creation code (see file: Packages/com.unity.entities/Unity.Entities.Hybrid/Injection/DefaultWorldInitialization.cs)
* Automatic bootstrap entry point (see file: Packages/com.unity.entities/Unity.Entities.Hybrid/Injection/AutomaticWorldBootstrap.cs)