在Java中，`ImmutableMap` 是 **Google Guava 库** 提供的一个不可变映射（Map）实现。它的核心特性是：**一旦被创建，其内容（键值对）和结构（大小）就完全无法被修改**。这为程序带来了极强的安全性和稳定性。

### 核心特性与设计哲学

* **真正不可变**：不提供任何会修改其内容的方法（如 `put`、`remove`、`clear`）。任何修改尝试都会抛出 `UnsupportedOperationException`。
* **线程安全**：由于其不可变性，多个线程并发读取绝对安全，无需额外的同步开销。
* **高效存储**：由于无需支持变更操作，其内部实现可以采用更紧凑、更高效的存储结构。
* **防御式编程**：作为方法参数或返回值时，可以明确地向调用者表明“此映射不应被修改”，避免了意外更改。

### 主要创建方式（Guava）

`ImmutableMap` 提供了多种灵活的构建方式，下表清晰地展示了其核心方法及用途：

| 创建方法 | 适用场景与说明 | 示例代码 |
| :--- | :--- | :--- |
| **`of()`** | 创建小型、已知键值对的空映射或小映射（最多5对）。 | `ImmutableMap<String, Integer> map = ImmutableMap.of("a", 1);` |
| **`copyOf()`** | 从现有 `Map`、`Iterable` 等复制并创建不可变副本。 | `ImmutableMap.copyOf(mutableMap);` |
| **`builder()`** | 最强大、最常用的方式。用于创建任意数量、可指定顺序、含去重校验的映射。 | `ImmutableMap.<String, Integer>builder().put("a", 1).putAll(anotherMap).build();` |

### 使用示例详解

以下是一个综合示例，展示了如何使用 `Builder` 模式，这也是最推荐、最灵活的方式：

```java
import com.google.common.collect.ImmutableMap;
import java.util.HashMap;
import java.util.Map;

public class ImmutableMapExample {
    public static void main(String[] args) {
        // 1. 使用 Builder 创建 (最推荐)
        ImmutableMap<String, Integer> scores = ImmutableMap.<String, Integer>builder()
                .put("Alice", 95)
                .put("Bob", 87)
                .put("Charlie", 92)
                // .put("Alice", 100) // 如果重复键，默认会抛出 IllegalArgumentException
                .build();
        System.out.println("使用Builder创建: " + scores);

        // 2. 从现有可变Map复制（“快照”模式）
        Map<String, Integer> mutableMap = new HashMap<>();
        mutableMap.put("Apple", 10);
        mutableMap.put("Banana", 5);
        ImmutableMap<String, Integer> inventory = ImmutableMap.copyOf(mutableMap);
        mutableMap.put("Orange", 8); // 修改原Map不影响不可变副本
        System.out.println("复制创建 (原Map已修改): " + inventory);

        // 3. 尝试修改会抛出异常
        try {
            scores.put("David", 70); // 此行会抛出 UnsupportedOperationException
        } catch (UnsupportedOperationException e) {
            System.out.println("正确捕获修改异常: " + e.getClass().getSimpleName());
        }
    }
}
```

### 核心注意事项

1.  **内容也必须“不可变”**：`ImmutableMap` 本身不可变，但如果其值（Value）对象是可变的（如 `List`、自定义对象），那么值对象内部状态的变化**不会被阻止**。要实现深度不可变，需确保存储的值也是不可变对象。
2.  **拒绝 `null` 值**：Guava 的 `ImmutableMap` **不允许 `null` 作为键或值**。任何插入 `null` 的尝试都会在构建时抛出 `NullPointerException`。这是其设计选择，以消除潜在的歧义。
3.  **构建时去重**：在构建过程中（尤其是 `builder`），如果添加了重复的键（Key），默认情况下**后续值会覆盖前一个**（`build()` 方法的行为）。但更好的实践是在构建时就避免重复，因为 `put` 方法本身遇到重复键时会抛出 `IllegalArgumentException`。
4.  **性能考虑**：`copyOf()` 方法会智能判断，如果传入的本身就是一个 `ImmutableMap`，可能会直接返回原实例而不是复制，从而提升效率。

### 对比与选型建议

* **`ImmutableMap` vs `Collections.unmodifiableMap()`**：
  * `Collections.unmodifiableMap()` 只是原始 Map 的一个**视图包装器**。原始 Map 被修改，这个“不可修改”视图的内容也会跟着变。它提供的是**运行时保护**。
  * `ImmutableMap` 是原始数据的一个**独立、不可变的副本**。与原始数据完全隔绝。它提供的是**编译时和运行时的双重保护**，更安全、意图更明确。
* **何时选择 `ImmutableMap`**：
  * 配置信息、常量映射。
  * 作为方法返回值，确保调用方不会意外修改。
  * 在多线程环境下作为只读数据共享。
  * 需要确保映射在创建后永不改变的任何场景。

### Java 9+ 内置的不可变映射

从 Java 9 开始，标准库也提供了创建不可变集合的工厂方法，可以替代 Guava 的部分基础功能：

```java
// Java 9+ 的 Map.of 和 Map.ofEntries
import java.util.Map;
Map<String, Integer> java9Map1 = Map.of("a", 1, "b", 2); // 最多10对键值
Map<String, Integer> java9Map2 = Map.ofEntries(
    Map.entry("a", 1),
    Map.entry("b", 2)
);
// 同样不可修改
```

### 总结

`ImmutableMap` 的核心优势在于其**绝对的不可变性**，这为代码提供了清晰的数据流契约、线程安全保证和潜在的优化空间。在需要确保映射内容恒定的场景下，它是最佳选择。如果使用 Java 9 及以上版本，并且需求简单（如创建小的常量映射），可以直接使用标准库的 `Map.of`。对于更复杂的构建需求（如按顺序添加、从多种数据源合并），Guava 的 `ImmutableMap.builder()` 依然是最强大、最灵活的工具。
