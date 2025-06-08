# [flatMap](https://www.geeksforgeeks.org/stream-flatmap-java-examples/)

## Java `flatMap` 详解

`flatMap` 是 Java 8 引入的 Stream API 中的一个**中间操作**，主要用于处理**嵌套流（Stream of Streams）**的扁平化（Flattening），将多层嵌套的流结构展开为单层流，便于后续统一处理。以下从核心作用、原理、使用场景和示例展开说明。

### **一、核心作用：扁平化嵌套流**

`flatMap` 的核心目标是将「流中的每个元素转换为另一个流」，并将这些子流**合并为一个新的流**（即“扁平化”）。  
与 `map` 不同：

- `map` 用于将流中的每个元素**映射为一个新元素**（可能是对象、集合或流），结果是 `Stream<T>` → `Stream<R>`。
- `flatMap` 用于将流中的每个元素**映射为一个子流**，并将所有子流合并为一个流，结果是 `Stream<T>` → `Stream<R>`（其中 `T` 可能包含嵌套结构）。

### **二、方法签名与原理**

#### 方法定义

```java
<R> Stream<R> flatMap(Function<? super T, ? extends Stream<? extends R>> mapper);
```

- **参数**：`mapper` 是一个函数，输入为原始流中的元素 `T`，输出为一个子流 `Stream<? extends R>`。
- **作用**：将每个 `T` 元素通过 `mapper` 转换为子流，然后将所有子流合并为一个新的流 `Stream<R>`。

#### 原理示例

假设有一个嵌套的集合 `List<List<Integer>>`，结构如下：

```java
List<List<Integer>> nestedList = Arrays.asList(
    Arrays.asList(1, 2), 
    Arrays.asList(3, 4), 
    Arrays.asList(5, 6)
);
```

- 使用 `map`：每个子列表会被映射为 `Stream<List<Integer>>`，结果流是 `[ [1,2], [3,4], [5,6] ]`。
- 使用 `flatMap`：每个子列表会被展开为 `Stream<Integer>`，合并后得到 `[1, 2, 3, 4, 5, 6]`。

### **三、典型使用场景**

#### 1. 展开嵌套集合

最常见的场景是将多层嵌套的集合（如 `List<List<String>>`）展开为单层列表。  
**示例**：

```java
List<List<String>> nestedStrings = Arrays.asList(
    Arrays.asList("a", "b"), 
    Arrays.asList("c", "d")
);

// 使用 flatMap 展开为 ["a", "b", "c", "d"]
List<String> flattened = nestedStrings.stream()
    .flatMap(List::stream)  // 将每个子列表转换为流并合并
    .collect(Collectors.toList());
```

#### 2. 处理对象中的集合属性

当对象包含集合类型的属性时（如 `User` 对象有 `List<Order>` 订单列表），可通过 `flatMap` 将所有对象的属性合并为一个流。  
**示例**：

```java
class User {
    private List<Order> orders;
    // 构造方法、getter 省略
}

List<User> users = Arrays.asList(
    new User(Arrays.asList(new Order(100), new Order(200))),
    new User(Arrays.asList(new Order(300), new Order(400)))
);

// 获取所有用户的订单金额总和
int totalAmount = users.stream()
    .flatMap(user -> user.getOrders().stream())  // 展开每个用户的订单流
    .mapToInt(Order::getAmount)
    .sum();  // 结果：100+200+300+400=1000
```

#### 3. 分割字符串为单词流

将包含多个字符串的流按分隔符分割，并合并为单词流。  
**示例**：

```java
List<String> sentences = Arrays.asList(
    "Hello world", 
    "Java is cool"
);

// 分割为 ["Hello", "world", "Java", "is", "cool"]
List<String> words = sentences.stream()
    .flatMap(sentence -> Arrays.stream(sentence.split(" ")))  // 按空格分割为子流
    .collect(Collectors.toList());
```

### **四、注意事项**

1. **空流处理**：如果 `mapper` 返回空流（`Stream.empty()`），`flatMap` 会忽略该流，不会产生任何元素。
2. **避免 `null`**：`mapper` 函数不能返回 `null`，否则会抛出 `NullPointerException`。
3. **顺序性**：合并后的流保持原流中元素的顺序（子流内部顺序也保留）。
4. **中间操作**：`flatMap` 是中间操作，可后续接 `filter`、`map`、`collect` 等操作。

### **五、与 `map` 的对比总结**

| 操作    | 输入类型       | 输出类型                | 典型用途                  |
|---------|----------------|-------------------------|---------------------------|
| `map`   | `Stream<T>`    | `Stream<R>`（元素级映射）| 简单元素转换（如类型、属性提取）|
| `flatMap`| `Stream<T>`   | `Stream<R>`（流合并）   | 嵌套结构扁平化（如集合展开、对象属性展开）|

通过 `flatMap`，可以高效处理嵌套的集合或对象属性，避免手动遍历嵌套结构，提升代码的简洁性和可读性。
