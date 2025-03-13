CAA 提供的控件类型的命名格式为 `CATDlgXxx`。

`CATDialog` 是 CAA 所有 UI 控件的基类。

通过向导定义的 Dialog 有对应的 .CATDlg 文件，用于拖拽式 UI 开发，会在对应的类中定义相关成员和函数。

## Frame

`CATDlgFrame` 是可以组合其他控件的最基本容器。

Frame 可以拥有标题。

## Container

`CATDlgContainer` 拥有可滚动区域。

其只能拥有一个子对象，通常是一个 Frame。

## Label

`CATDlgLabel` 显示文本的标签。

## Editor

`CATDlgEditor` 单行或多行输入框，可以限制输入格式。

## Button

`CATDlgPushButton` 按钮。

## Check

`CATDlgCheckButton` 复选按钮。

## Radio

`CATDlgRadioButton` 单圈按钮。

## Combo

`CATDlgCombo` 下拉框。

## Spin

`CATDlgSpinner`

## Progress

`CATDlgProgress` 进度条。

## Scrollbar

`CATDlgScrollBar` 滚动条。

## Separator

`CATDlgSeparator` 分隔符。

## Selector

`CATDlgSelectorList` 带有滚动条的可选项列表，项通常是字符串。
