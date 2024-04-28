arkts 提供了 `@Builder` 修饰器，实现更轻量的ui复用机制。使用 `@Builder` 修饰的函数遵守 `build()` 函数一致的规则，且可在 `build()` 函数内调用。

构建函数可以定义在组件内，作为私有成员；也可以定义为全局函数，整个应用内共享。

#### 参数传递
构建函数的参数传递可以按值传递或按引用传递，但需要遵守以下规则：
* 传递参数的类型必须和参数声明的类型一致。
* 构建函数内部不能修改参数值。
* 只有传入一个参数时，且传入对象字面量时，才会按引用传递该参数，否则均按值传递。

###### 引用传递
arkts 建议使用 `$$` 作为引用变量的名字，作为引用传递的范式。

如果传递状态变量，只有按引用传递时，状态变量的变化才会引起构建函数内的ui刷新。
```ts
@Builder
function numberText($$: { n: number }) {
    Text($$.n.toString())
        .fontSize(50)
}

@Entry
@Component
struct Index {
    @State n: number = 0

    build() {
        Column() {
            numberText({
                n: this.n
            })
            Button('inc')
                .onClick(() => {
                    this.n += 1
                })
        }
    }
}
```