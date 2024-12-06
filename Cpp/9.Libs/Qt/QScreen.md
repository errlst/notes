`QScreen` 用于获取屏幕相关属性。



#### 获取

`QGuiApplication::primaryScreen()`，获取应用程序最初显示的屏幕。

`QGuiApplication::screens()`，获取窗口系统关联的屏幕列表。



#### 尺寸

`.avaliableSize()`，屏幕的可用大小（以像素为单位）。可用大小不包括窗口管理器的保留区域，如任务栏和系统菜单等的大小。

`.devicePixelRatio()`，屏幕物理像素和设备无关像素之间的比率，可以理解为缩放率。

`.physicalSize()`，屏幕的物理尺寸（以毫米为单位）。

#### 其他

`.model()`，屏幕的型号。

`.name()`，屏幕可以呈现给用户的字符串，但不能作为屏幕唯一标识。