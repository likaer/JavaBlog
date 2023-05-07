# <font color="blue">设计模式</font>

根据设计模式的参考书，目前常用的设计模式总共有23种，这些设计模式可以分为3大类

- 创建型模式
- 结构性模式
- 行为型模式

## 创建型模式

创建型模式提供了创建对象同时隐藏创建逻辑的方式，这使得软件在特定场景下创建特定对象时更加灵活

### 工厂模式

![工厂模式的 UML 图](https://www.runoob.com/wp-content/uploads/2014/08/AB6B814A-0B09-4863-93D6-1E22D6B07FF8.jpg)

使用者只需要关心产品的名称，而不需要关心具体的实现，就能通过ShapeFactory.getShape(name)获取Shape的子类实例



### 抽象工厂模式

![屏幕快照 2021-05-06 下午7.34.04](/Users/likaer/Desktop/屏幕快照 2021-05-06 下午7.34.04.png)

抽象工厂是存在多个不同类型的抽象产品场景下，对产品组合进行组装的场景应用

使用者可以根据不同的业务场景选择合适的工厂实例子类进行产品的组合

### 单例模式

![单例模式的 UML 图](https://www.runoob.com/wp-content/uploads/2014/08/62576915-36E0-4B67-B078-704699CA980A.jpg)

单例类只能有一个实例。

- 2、单例类必须自己创建自己的唯一实例。
- 3、单例类必须给所有其他对象提供这一实例。

[设计方案](https://en.wikipedia.org/wiki/Double-checked_locking#Usage_in_Java)

```
// Works with acquire/release semantics for volatile in Java 1.5 and later
// Broken under Java 1.4 and earlier semantics for volatile
class Foo {
		//由于编译器原因，在某些JDK版本下不使用volatile会导致返回一个未完全初始化的对象
    private volatile Helper helper;
    public Helper getHelper() {
        Helper localRef = helper;
        if (localRef == null) {
            synchronized (this) {
                localRef = helper;
                if (localRef == null) {
                    helper = localRef = new Helper();
                }
            }
        }
        return localRef;
    }

    // other functions and members...
}
```



### 建造者模式

建造者模式（Builder Pattern）使用多个简单的对象一步一步构建成一个复杂的对象

### 原型模式

原型模式用于创建重复的对象

```Java
public class Student {
    private int id;
    private String name;
    private int score;

    public Student copy() {
        Student std = new Student();
        std.id = this.id;
        std.name = this.name;
        std.score = this.score;
        return std;
    }
}
```

## 结构型模式

结构型模式关注类和对象的组合

### 适配器模式

适配器模式（Adapter Pattern）是作为两个不兼容的接口之间的桥梁。

![adapter](https://www.liaoxuefeng.com/files/attachments/1326164190691394/l)

### 桥接模式

桥接（Bridge）是用于把抽象化与实现化解耦，使得二者可以独立变化。

## 行为型模式

行为型模式关注对象之间的通讯



<font color="red">注：类是对象的抽象定义，对象是类的具体实例</font>

