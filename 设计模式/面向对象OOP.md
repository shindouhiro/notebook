# 面向对象编程 (OOP) 四大特性

面向对象编程 (Object-Oriented Programming, OOP) 是设计模式的基础。理解这四大特性，才能真正理解为什么要用那些设计模式。

---

## 1. 封装 (Encapsulation)
**核心点：隐藏内部细节，保护数据。**

将数据（属性）和逻辑（方法）封装在一个单元（类或对象）中。通过设置访问权限，防止外部代码直接破坏内部状态。

```javascript
class BankAccount {
  #balance = 0; // 私有属性，外部无法直接访问

  deposit(amount) {
    if (amount > 0) this.#balance += amount;
  }

  getBalance() {
    return this.#balance; // 只暴露读取接口
  }
}
```

## 2. 继承 (Inheritance)
**核心点：代码复用。**

允许创建一个类作为另一个类的扩展。子类自动获取父类的功能，并可以添加自己的特有功能。

```javascript
class Animal {
  eat() { console.log("Eating..."); }
}

class Dog extends Animal {
  bark() { console.log("Woof!"); }
}

const myDog = new Dog();
myDog.eat(); // 继承自 Animal
```

## 3. 多态 (Polymorphism)
**核心点：同一接口，多种实现。**

允许将子类对象视为父类对象。不同的对象可以对同一消息做出不同的响应。这是工厂模式和策略模式的核心。

```javascript
class Shape {
  draw() { console.log("Drawing a shape"); }
}

class Circle extends Shape {
  draw() { console.log("Drawing a circle ○"); }
}

class Square extends Shape {
  draw() { console.log("Drawing a square □"); }
}

// 统一调用 draw 方法，但结果不同
[new Circle(), new Square()].forEach(s => s.draw());
```

## 4. 抽象 (Abstraction)
**核心点：关注“做什么”，而非“怎么做”。**

提取出共同特征而忽略非本质细节。它是对现实世界的简化建模。

```javascript
// 我们只需要知道车可以“开”，不需要知道引擎点火的每一个机械细节
class Car {
  start() {
    this.#ignite();
    this.#checkFuel();
    console.log("Car is ready!");
  }
  #ignite() { /* 复杂逻辑 */ }
  #checkFuel() { /* 复杂逻辑 */ }
}
```

---

## 🎯 总结
- **封装**确保了**安全性**和**自包含性**。
- **继承**实现了**层级关系**和**复用**。
- **多态**提供了**灵活性**。
- **抽象**简化了**系统建模**。
