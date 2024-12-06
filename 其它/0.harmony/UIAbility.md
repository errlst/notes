`UIAbility` 组件是系统调度的基本单元，为应用提供绘制界面的窗口；一个 `UIAbility` 组件中可以通过多个页面来实现一个功能模块。每一个 `UIAbility` 组件实例，都对应于一个最近任务列表中的任务。

每个 `UIAbility` 具有一个绑定的 `WindowStage` 对象，该对象负责实际的UI管理。

#### 声明配置
需要在 module.json5 中声明 `UIAbility` 的名称、入口、标签等相关信息后，才能正常使用，其必备属性如下：
```json
"abilities": [{   
    "name": "EntryAbility",                             // 组件类名
    "srcEntry": "./ets/entryability/EntryAbility.ts",   // 组件源码路径
    "description": "$string:EntryAbility_desc",         // 组件描述
    "icon": "$media:icon",                              // 组件图标
    "label": "$string:EntryAbility_label",              // 组件标签
    "startWindowIcon": "$media:icon",
    "startWindowBackground": "$color:start_window_background",
    ...
}]
```

#### 生命周期
`UIAbility` 的生命周期包括 Create、Foreground、Background 和 Destroy 四个状态。
```mermaid
graph LR 
start[UIAbility Start]
start --> create(Create)
create --> fore(Foreground)
fore --> back(Background)
back --> fore
back --> dest(Destroy)
dest --> _[UIAbility End]
```

###### create
应用加载过程中，`UIAbility` 实例创建完成后后，触发 `onCreate` 回调。

实例创建完成，进入 Foreground 之前，系统会创建 `WindowStage` 对象，创建完成后进入 `onWindowStageCreate()` 回调，在该回调中设置UI页面、事件订阅等。

###### foreground
`UIAbility` 实例切换至前台，UI界面可见之前触发。

###### background
`UIAbility` 实例切换至后台，UI界面完全不可见后触发。

###### destroy
`UIAbility` 实例销毁时触发，且在触发 `onDestroy` 之前，会先触发 `onWindowStageDestroy`。

#### 启动模式
启动模式是指 `UIAbility` 实例在启动时的不同呈现状态，系统提供了三种模式，通过 module.json5 的 launchType 字段设置。

###### singleton
单例模式，调用 `startAbility()` 时，如果应用进程列表中该类型的 `UIAbility` 实例已经存在，则复用系统中的该实例。即任务列表中只能存在一个该类型的 `UIAbility`。

###### multition
多实例模式，每次调用 `startAbility()` 时，都会新建一个 `UIAbility` 实例。

###### specified
指定实例模式，调用 `startAbility()` 时，指定一个字符串key，如果存在和该字符串key绑定的 `UIAbility` 实例，则复用该实例，否则创建新实例。

#### 设置页面
在 `onWindowStageCreate()` 中，会传入 `WindowStage` 对象，通过该对象的 `loadContent()` 方法可以设置当前 `UIAbility` 的页面。

#### 获取上下文
在 `UIAbilityt` 类内部，可以直接使用 `this.context` 获取当前实例绑定的上下文。

在页面代码中，通过 `getContext(this) as common.UIAbilityContext` 获取实例绑定的上下文。

#### 数据同步
###### EventHub
`EventHub` 提供 Ability 组件级别的事件机制，以 Ability 组件为中心提供了订阅、取消订阅和触发事件的能力，通常直接使用 `Context` 内部维护的 `EventHub` 对象，其基本使用流程为：
1. 使用 `on()` 方法注册事件，需要提供事件名和回调函数。
2. 使用 `emit()` 方法触发事件，提供事件名和传给回调的参数。
3. 使用 `off()` 方法取消事件订阅。

###### globalThis
`globalThis` 是 ArkTS 引擎内部维护的一个实例对象，引擎内部的所有实例均可访问。