---
createTime: 2026-04-10
updateTime: 2026-04-10
tags:
  - java
  - oop
  - polymorphism
  - interface
  - abstract-class
belong_canvas:
  - "[[Java基础.canvas]]"
---

# Java面向对象：多态、接口与抽象类

> [!abstract] 一句话摘要
> 多态的核心不是一句“父类引用指向子类对象”，而是“面向统一抽象编程，运行时表现出不同实现”；接口更适合表达能力与行为规范，抽象类更适合表达归属关系与共享实现。

---

## 一、什么是多态？

多态可以直接理解成：

> **同一个方法调用，传入不同对象，会表现出不同的行为。**

很多人只背：

- 父类引用指向子类对象

但这句话只是实现形式，不是多态最重要的价值。

例如：

```java
class Animal {
    public void speak() {
        System.out.println("动物在叫");
    }
}

class Dog extends Animal {
    @Override
    public void speak() {
        System.out.println("汪汪汪");
    }
}

class Cat extends Animal {
    @Override
    public void speak() {
        System.out.println("喵喵喵");
    }
}

Animal a1 = new Dog();
Animal a2 = new Cat();

a1.speak();
a2.speak();
```

这里变量类型都是 `Animal`，但运行结果不同，这就是多态。

---

## 二、多态的本质是什么？

多态的本质是：

> **写代码时面向统一类型，运行时根据对象的实际类型决定具体行为。**

这意味着：

- 调用方不需要分别写 Dog、Cat、Bird 的调用逻辑
- 只需要依赖父类或接口
- 真正执行时由 Java 决定调用哪个实现

---

## 三、多态在真实编码里有什么用？

### 1. 降低耦合

你写代码时不必死绑某个具体类，而是依赖抽象。

```java
public void makeAnimalSpeak(Animal animal) {
    animal.speak();
}
```

这个方法不关心传入的是 Dog、Cat 还是 Bird，只要它是 `Animal` 就行。

这意味着调用方和具体实现解耦了。

### 2. 方便扩展

如果后来新增：

```java
class Bird extends Animal {
    @Override
    public void speak() {
        System.out.println("叽叽叽");
    }
}
```

原来的调用代码通常不用改，直接复用原有接口即可。

### 3. 让代码更通用

例如支付场景：

```java
interface Payment {
    void pay();
}

class Alipay implements Payment {
    public void pay() {
        System.out.println("支付宝支付");
    }
}

class WeChatPay implements Payment {
    public void pay() {
        System.out.println("微信支付");
    }
}

public void checkout(Payment payment) {
    payment.pay();
}
```

这样业务层不需要写大量 if-else 判断支付方式。

---

## 四、多态真正的价值

可以压缩成一句话：

> **多态让我们面向抽象编程，而不是死盯具体实现，从而让代码更灵活、更可扩展、更容易维护。**

你也可以快速记成：

- 同一套调用方式
- 因对象不同而表现不同
- 价值是解耦、扩展、减少分支判断

---

## 五、接口和抽象类怎么区分？

这是多态之后最常见的追问。

### 先给结论

- **接口**：更适合表达“我会做什么”
- **抽象类**：更适合表达“我是什么”

也就是：

- 接口强调**能力 / 行为规范**
- 抽象类强调**归属 / 抽象父类关系**

---

## 六、从三个维度区分接口和抽象类

### 1. 表达“能力”还是表达“归属”

#### 接口：表达能力

比如：

- `Runnable`
- `Comparable`
- `Serializable`

这些不是在说它属于哪一类事物，而是在说：

> 它具备某种能力。

所以接口很适合描述可横跨多个类层次的共同行为。

#### 抽象类：表达归属

比如：

- `Animal`
- `Vehicle`
- `Employee`

这些强调的是一种 `is-a` 关系：

- Dog is an Animal
- Cat is an Animal

这时候用抽象类更自然。

### 2. 是否需要共享实现

如果多个子类之间除了规范一致，还需要复用一部分公共字段、公共方法或默认实现，那么抽象类通常更合适。

如果只是想定义行为契约，不强制共享实现，接口更合适。

### 3. 耦合程度

接口耦合更低，因为它只定义规范，不绑定共同父类实现。

抽象类耦合更强，因为子类进入了同一个继承体系，也就更强调家族归属和结构一致性。

---

## 七、实战选择原则

你可以这样选：

- 如果要表达“这个对象能做什么” -> 优先考虑**接口**
- 如果要表达“这几个类本质上属于同一家族” -> 优先考虑**抽象类**
- 如果既想统一规范，又想复用部分公共实现 -> 抽象类更合适

---

## 八、面试速答版

### 什么是多态？

多态指的是同一个父类或接口引用，在指向不同子类对象时，调用同一个方法会表现出不同的行为。它的核心价值是让代码面向抽象编程，降低耦合，提高扩展性和可维护性。

### 接口和抽象类的区别

接口更适合表达能力和行为规范，强调“我会做什么”；抽象类更适合表达归属关系和共享实现，强调“我是什么”。如果只是定义规范，优先接口；如果需要复用公共实现，优先抽象类。
