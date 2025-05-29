### Java 抽象类（Abstract Class）详解

#### **一、定义与概念**
**抽象类**是一种**不能被实例化**的类，通过 `abstract` 关键字修饰。它用于定义一组相关类的公共接口和部分实现，作为子类的模板。  
**核心作用**：
- 提取子类的公共特征，避免代码重复。  
- 强制子类实现特定方法（通过抽象方法）。  
- 定义框架或接口的默认行为。


#### **二、语法与特性**

1. **声明抽象类**

```java
public abstract class Animal {
    // 抽象类可以包含普通字段
    private String name;
    
    // 构造方法（子类必须调用）
    public Animal(String name) {
        this.name = name;
    }
    
    // 普通方法
    public String getName() {
        return name;
    }
    
    // 抽象方法（无实现，子类必须重写）
    public abstract void makeSound();
}
```

1. **抽象方法**  
   - 无方法体，仅声明签名。  
   - 必须在抽象类中定义。  
   - 子类必须实现所有抽象方法，否则子类也必须声明为抽象类。

2. **继承抽象类**  
   ```java
   public class Dog extends Animal {
       // 实现父类的抽象方法
       @Override
       public void makeSound() {
           System.out.println("汪汪汪！");
       }
   }
   ```


#### **三、抽象类 vs. 接口**
| 特性               | 抽象类                     | 接口                       |
|--------------------|----------------------------|----------------------------|
| 关键字             | `abstract class`           | `interface`                |
| 实例化             | 不可实例化                 | 不可实例化                 |
| 方法实现           | 可包含普通方法和抽象方法   | （Java 8+）可包含默认方法和静态方法 |
| 字段               | 可包含各种访问修饰符的字段 | 只能是 `public static final` 常量 |
| 继承关系           | 单继承（一个类只能继承一个抽象类） | 多实现（一个类可实现多个接口） |
| 设计目的           | 提取共性，定义部分实现     | 定义行为规范，实现多态     |


#### **四、使用场景**
1. **模板方法模式**  
   - 抽象类定义算法骨架，子类实现具体步骤。  
   ```java
   public abstract class Game {
       // 模板方法
       public final void play() {
           initialize();
           start();
           end();
       }
       
       protected abstract void initialize();
       protected abstract void start();
       protected abstract void end();
   }
   ```

2. **框架设计**  
   - Spring 的 `AbstractController`、HttpServlet 的 `doGet()`/`doPost()` 等。

3. **工具类抽象**  
   - 如 `java.util.AbstractList` 定义了 List 接口的部分实现。


#### **五、注意事项**
1. **构造方法**  
   - 抽象类可定义构造方法，但仅用于子类初始化。  
   ```java
   public abstract class Shape {
       protected double area;
       
       // 构造方法（子类必须调用）
       public Shape() {
           calculateArea();
       }
       
       protected abstract void calculateArea();
   }
   ```

2. **与 `final` 冲突**  
   - 抽象类不能声明为 `final`（矛盾：`final` 类不可继承，而抽象类必须被继承）。

3. **抽象方法的限制**  
   - 抽象方法不能是 `private`、`static` 或 `final`（需被子类重写）。


#### **六、经典面试题**
**Q：抽象类是否可以没有抽象方法？**  
**A：可以**。即使没有抽象方法，只要类声明为 `abstract`，就不能被实例化。这种设计通常用于强制子类遵循特定构造逻辑。  
```java
public abstract class BaseEntity {
    private Long id;
    
    // 无抽象方法，但子类必须继承
    public BaseEntity() {
        this.id = generateId();
    }
    
    protected abstract Long generateId();
}
```


#### **七、总结**
- **抽象类**是实现代码复用和多态的重要工具，适合作为一组相关类的基类。  
- 合理使用抽象类可提高代码的可维护性和扩展性，尤其在框架设计和模板方法模式中。  
- 与接口相比，抽象类更侧重于**代码复用**，而接口更侧重于**行为规范**。