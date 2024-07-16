## QJsonDocument
`QJsonDocument` 提供完整的 utf8 编码的 json 文档的读写功能。

#### 转换
`::fromJson(json, error)`，json 字符串创建 `QJsonDocument` 对象。

`.toJson(format)`，转换为格式化后的 json 字符串，默认为紧凑锁紧格式。

#### 获取
`.object()`，如果文档是对象，返回对应的 `QJsonObject`，否则返回空的 `QJsonObject`。

`.array()`，同上。

#### 检查
`.isArray()`、`.isEmpty()`、`.isNull()`、`.isObject()`。

## QJsonObject
`QJsonObject` 的实现为键值对列表，键为 `QString`，值为 `QJsonVlaue`。

#### 查
`.contains(key)`，查找是否包含键值对。

`.value(key)`，查找键值对的值，如果不存在，返回 `QJsonValue::Undefined`。

`.take(key)`，移除键值对，并返回值。如果不存在，返回 `QJsonValue::undefined`。

#### 增删改
`.remove(key)`，移除键值对。

`.insert(key, val)`，插入键值对，如果键已经存在，则替换其值。如果值为 `QJsonVlaue::undefined`，改键值对将被移除。

#### operator[]
`.operator(key) const`，等效于 `value()`。

`.operator(key)`，返回值的引用，如果键值对不存在，创建新的键值对，初始化值为 `QJsonValue::null`。