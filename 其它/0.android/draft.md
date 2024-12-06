#### 项目结构

andorid 项目中的代码、资源文件等内容均存放在 app 目录下。app 目录的基本结构为：

```
app:
	build		-- 编译文件
	libs		-- 第三方包
	src:
		androidTest	-- andorid test 测试用例
		test		-- unit test 测试用例
		main:
			AndroidManifest.xml
            java		-- java、kotlin 代码
			res:
				drawable	-- 图片目录
				layout		-- 布局目录
				values		-- 字符串、样式、颜色等配置
				mipmap-*	-- 应用图标 
```

---

#### 资源引用

```xml
<!-- res/values/strings.xml -->
<resources>
    <string name="app_name">androidapp</string>
</resources>
```

有两种方式可以引用上面定义的字符串：

* 在代码中使用 `R.string.app_name`。
* 在XML中使用 `@string/app_name`。

---

#### AndroidManifest.xml

`AndroidManifest.xml` 描述应用程序的各种属性，如程序名、图标、版本号等；和组件，如 Activites、Services 等，并指定了其行为、权限。

只有在 Manifest 中注册了的 Activity 才能在项目中使用。

```xml
<manifest>
    <application>
        <!-- name 表示注册的 activity 对应的类名 -->
        <activity android:name=".MainActivity">	
            <intent-filter>
                <!-- 该 activity 是项目的主 activity -->
                <action android:name="android.intent.action.MAIN" />
                <!-- 点击应用图标后，首先驱动该 activity -->
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>
</manifest>
```

```kotlin
class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.main_activity) // 逻辑视图分离
    }
}
```

---

#### 定义ID

在定义 `id` 时，使用 `@+id/id_name` 表示。

---

#### Toast

使用 `Toast` 在程序中弹出一个持续一段时间的提示控件，可以显示一些短消息。

基本使用方式为 `Toast.makeText().show()`。

---

#### findViewById

使用 `findViewById<>(id)` 访问布局文件中的控件，id 可以通过 `R.layout.` 获取。

`findViewById` 内部维护缓存，后续调用直接在缓存中寻找。

---

#### menu

使用布局文件定义菜单时，每个菜单文件中只能存在一个menu，且需要作为根标签。

```xml
<menu>
    <item
        android:id="@+id/add"
        android:title="add" />
    <item
        android:id="@+id/del"
        android:title="del" />
</menu>
```

在 activity 中重载 `onCreateOptionsMenu()`，并在其中调用 `menuInflater.inflate()` 添加菜单。

```kotlin
override fun onCreateOptionsMenu(menu: Menu?): Boolean {
    menuInflater.inflate(R.menu.main_menu, menu)
    return true;
}
```

重载 `onOptionsItemSelected()`，菜单项点击后调用。

