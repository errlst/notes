CAA command 是继承自 `CATCommand` 的类的实例对象，有三种类型的 command：

- One-shot command。不需要用户额外输入即可直接运行，直接继承自 `CATCommand`。

- Dialog boxes。对话框自身就是一个 command，继承自 `CATDlgDialog`。

- State dialog commands。构建为状态机，继承自 `CATStateCommand`。

## Agent

Agent 是用于管理用户输入和后台逻辑的中间件。

### CATDialogAgent

通用用户界面代理。

### CATIndicationAgent

在视图中点击时获取映射后的 2D 点坐标。

### CATPathElementAgent

视图中进行元素选择。
