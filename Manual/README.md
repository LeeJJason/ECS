# [Overview（0.11.1）](https://docs.unity3d.com/Packages/com.unity.entities@0.11/manual/index.html)
![](https://docs.unity3d.com/Packages/com.unity.entities@0.11/manual/images/WhatIsECSinfographic0000.png)
![](https://docs.unity3d.com/Packages/com.unity.entities@0.11/manual/images/WhatIsECSinfographic0001.png)
![](https://docs.unity3d.com/Packages/com.unity.entities@0.11/manual/images/WhatIsECSinfographic0002.png)
![](https://docs.unity3d.com/Packages/com.unity.entities@0.11/manual/images/WhatIsECSinfographic0003.png)
![](https://docs.unity3d.com/Packages/com.unity.entities@0.11/manual/images/WhatIsECSinfographic0004.png)

# Entity Component System
实体组件系统（ECS）是Unity面向数据的技术堆栈的核心。顾名思义，ECS包含三个主要部分：
* [Entities](https://docs.unity3d.com/Packages/com.unity.entities@0.11/manual/ecs_entities.html) — 填充游戏或程序的实体或事物。
* [Components](https://docs.unity3d.com/Packages/com.unity.entities@0.11/manual/ecs_components.html) — 与您的实体相关联的数据，但由数据本身而非实体进行组织。 （这种组织上的差异是面向对象和面向数据的设计之间的关键差异之一。）
* [Systems](https://docs.unity3d.com/Packages/com.unity.entities@0.11/manual/ecs_systems.html) — 用于将组件数据从其当前状态转换为下一个状态的逻辑-例如，系统可能会通过其速度乘以自上一帧以来的时间间隔来更新所有移动实体的位置。