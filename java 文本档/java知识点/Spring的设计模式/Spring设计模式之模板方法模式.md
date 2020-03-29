# 模板方法模式（Template Method）

## 简单介绍：

在父类中定义一个操作中的**算法的骨架**，而将**部分步骤的实现**在子类中完成。

模板方法模式使得子类可以不改变一个算法的结构即可重定义该算法的某些特定步骤。

该模板模式是所有模式中最为常见的几个模式之一，是**基于继承**的**代码复用**的基本技术。

因此，在模板方法模式的类结构图中，**只有继承关系**。

模板方法需要开发抽象类和具体子类的设计师之间的协作。一个设计师负责给出算法的轮廓和骨架，另一些设计师则负责给出这个算法的各个逻辑步骤。

代表这些逻辑步骤的方法称为**基本方法**（primitive method）；而将这些基本方法汇总起来的方法叫做**模板方法**（template method），这个设计模式的名字就是从此而来。

## 结构：

AbstractClass：抽象类，定义并实现一个模板方法。这个模板方法定义了算法的骨架，而逻辑组成步骤在相应的抽象操作中，推迟到子类去实现。顶级逻辑也有可能调用一些具体方法。

```
abstract class Beverage {
 
    // 模板方法，决定了算法骨架。相当于TemplateMethod()方法
 
    public void prepareBeverage() {
 
        boilWater();
 
        brew();
 
        pourInCup();
 
        if (customWantsCondiments())
 
        {
 
            addCondiments();
 
        }
 
    }
 
    
 
    // 共性操作，直接在抽象类中定义
 
    public void boilWater() {
 
        System.out.println("烧开水");
 
    }
 
    
 
    // 共性操作，直接在抽象类中定义
 
    public void pourInCup() {
 
        System.out.println("倒入杯中");
 
    }
 
    
 
    // 钩子方法，决定某些算法步骤是否挂钩在算法中
 
    public boolean customWantsCondiments() {
 
        return true;
 
    }
 
    
 
    // 特殊操作，在子类中具体实现
 
    public abstract void brew();
 
    
 
    // 特殊操作，在子类中具体实现
 
    public abstract void addCondiments();
 
    
 
}
```

ConcreteClass：实现类实现父类所定义的一个或多个抽象方法。

```
class Tea extends Beverage {
 
    @Override
 
    public void brew() {
 
        System.out.println("冲泡茶叶");
 
    }
 
    @Override
 
    public void addCondiments() {
 
        System.out.println("添加柠檬");
 
    }
 
    
 
}
 
class Coffee extends Beverage {
 
    @Override
 
    public void brew() {
 
        System.out.println("冲泡咖啡豆");
 
    }
 
    @Override
 
    public void addCondiments() {
 
        System.out.println("添加糖和牛奶");
 
    }
 
    
 
}
```

## 要点：

### 模板方法模式中的三类角色：

1.具体方法（Concrete Method）

2.抽象方法（Abstract Method）

3.钩子方法（Hook Method）

### 三类角色的关联：

在模板方法模式中，首先父类会定义一个算法的框架，即实现算法所必须的所有方法。

其中，具有共性的代码放在父类的具体方法中。

各个子类特殊的代码放在子类的具体方法中，但是父类汇总需要有对应抽象方法声明。

钩子方法可以让子类决定是否对算法的不同点进行挂钩。

### 总结：

使用模板方法模式可以将代码的公共行为提取，以达到复用的目的。

而对于特殊化的行为在子类中实现。父类的模板方法就可以控制子类中的具体实现。

子类无需了解整体算法的框架，只需要自己的业务逻辑即可。