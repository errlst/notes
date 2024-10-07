#### 创建测试

使用宏 `TEST(测试集, 测试名)` 创建测试。

```cpp
TEST(testset, test_1){
    // 测试代码
}
```

#### 激活测试

`testing::InitGoogleTest([argc, argv])` 初始化所有测试，`argc` 和 `argv` 可选参数。

`RUN_ALL_TESTS()` 执行所有测试，调用之前必须初始化测试。

一个测试单元的标准 main 函数形式如下。

```cpp
auto main() -> int {
  testing::InitGoogleTest();

  return RUN_ALL_TESTS();
}
```

#### 断言

gtest有两种断言：

* `ASSERT_*()`，报错且会导致程序终止。
* `EXPECT_*()`，报错但不会导致程序中止。

###### 基本断言

```cpp
EXPECT_TRUE(cond);
EXPECT_FALSE(cond)。
```

###### 二元比较

```cpp
EXPECT_EQ(val1, val2);	// val1 == val2
EXPECT_NE(val1, val2);	// val1 != val2

EXPECT_LT(val1, val2);	// val1 < val2
EXPECT_LE(val1, val2);	// val1 <= val2

EXPECT_GT(val1, val2);	// val1 > val2
EXPECT_GE(val1, val2);	// val1 >= val2
```

###### 异常

```cpp
EXPECT_THROW(stat, ex);	// stat 表达式抛出 ex 类型的异常
```





