## Spring设计模式

### 简单工厂

​    实现方式：**BeanFactory**。Spring中的BeanFactory就是简单工厂模式的体现，根据传入一个唯一的标识来获得Bean对象，但是否是在传入参数后创建还是传入参数前还是传入参数前创建这个要根据具体情况来定。

​    实质：由一个工厂类根据传入的参数，动态决定该创建哪一个产品类。

```
实现原理 ：
-bean容器的启动阶段：
  ​ 读取bean的xml配置文件，将bean元素分别转换成一个BeanDefinition对象。
  ​ 然后通过BeanDefinitionRegistry将这些bean注册到beanFactory中，保存在它的一个ConcurrentHashMap中。
  ​将BeanDefinition注册到了beanFactory之后，在这里Spring为我们提供了一个扩展的接口，允许我们通过实现接口BeanFactoryPostProcessor在此处来插入我们定义的代码。
  典型的例子就是：PropertyPlaceholderConfigurer，我们一般在配置数据库的DataSource时使用到占位符的值，就是它注入进去的。
-容器中bean的实例化阶段：
  实例化阶段主要是通过反射或者CGLIB对bena进行实例化，在这个阶段Spring又给我们暴露了很多扩展点：
    1.各种的Aware接口，比如BeanFactoryAware，对于实现了这些Aware接口的bean，在实例化bean时Spring会帮我们注入对应的BeanFactory的实例。
    2.BeanPostProcessor接口，实现BeanPostProcessor接口的bean，在实例化bean是Spring会帮我们调用接口中的方法。
    3.InitializingBean接口，实现InitializingBean接口的bean，在实例化bean时Spring会帮我们调用接口中的方法。
    4.DisposableBean接口，实现BeanPostProcessor接口的bean，在bean死亡时，Spring会帮我们调用接口中的方法。
    
补充：
bean的完整的生命周期：
  1.Spring启动，查找并加载需要被Spring管理的bean，进行Bean的实例化
  2.Bean实例化后对将Bean的引入和值注入到Bean的属性中
  3.如果Bean实现了BeanNameAware接口的话，Spring将Bean的Id传递给setBeanName()方法
  4.如果Bean实现了BeanFactoryAware接口的话，Spring将调用setBeanFactory()方法，将BeanFactory容器实例传入
  5.如果Bean实现了ApplicationContextAware接口的话，Spring将调用Bean的setApplicationContext()方法，将bean所在应用上下文引用传入进来。
  6.如果Bean实现了BeanPostProcessor接口，Spring就将调用他们的postProcessBeforeInitialization()方法。
  7.如果Bean 实现了InitializingBean接口，Spring将调用他们的afterPropertiesSet()方法。类似的，如果bean使用init-method声明了初始化方法，该方法也会被调用
  8.如果Bean 实现了BeanPostProcessor接口，Spring就将调用他们的postProcessAfterInitialization()方法。
  9.此时，Bean已经准备就绪，可以被应用程序使用了。他们将一直驻留在应用上下文中，直到应用上下文被销毁。
  10.如果bean实现了DisposableBean接口，Spring将调用它的destory()接口方法，同样，如果bean使用了destory-method 声明销毁方法，该方法也会被调用。
```

设计意义：

   松耦合：可以将原来硬编码的依赖，通过Spring这个beanFactory这个工厂来注入依赖，也就是说原来只有依赖和被依赖方，现在我们引入了第三方——spring这个beanFactory，由他来解决bean之间的依赖问题，达到松耦合的效果。

   bean的额外处理：通过Spring接口的暴露，在实例化bean阶段我们可以进行一些额外的处理，这些额外的处理只需要让bean实现对应的接口即可，那么spring就会在bean的生命周期调用我们实现的接口来处理该bean。

### 工厂方法

​      实现方式：FactoryBean接口。

​      实现原理：

​           实现FactoryBean接口是bean是一类叫做factory的bean，其特点是，Spring会在使用geyBean()调用该bean时，会自动调用该bean的getObject()方法，所以返回的不是factory这个bean，而是这个bean.getObject()方法的返回值。

### 单例模式

**定义**：确保某一个类只有一个实例，而且自行实例化并向整个系统提供整个实例。

```
通用代码：（是线程安全的）
public class Singleton{
    private static  final Singleton = new Singleton;
    
    //限制产生多个对象
    private Singleton(){
    }
    //通过该方法获得实例对象
    public static Singleton getSingleton(){
       return singleton;
    }
    //类中其他方法，尽量使用static
    public static void doSomething(){
    }
    
}
```

**使用场景：**

- 要求生成唯一序列号的环境
- 在整个项目需要一个共享访问点或共享数据，例如一个web页面上的计数器，可以不用每次刷新都记录到数据库中，使用单例模式保持计数器的值，并确保是线程安全的；
- 创建一个对象需要消耗的资源过多，如要访问IO和数据库等资源；
- 需要定义大量的静态常量和静态方法（如工具类）的环境，可以采用单例模式（当然，也可以直接声明为static的方式）。

```
线程不安全的实例：
public Class Singleton{
   private static Singleton singleton = null;
   //限制产生多个对象
   private Singleton(){}
   //通过该方法获取实例对象
   public static Singleton getSingleton(){
   if(singleton==null){
   singleton=new Singleton();
   }
   return singleton;
   }
}
线程安全解决方法：
在getSingleton（）方法前加synchronized关键字，也可以在getSingeleton方法内增加synchronized来实现。最优的方法就是如通用代码那样写。
```

### 工厂模式

**定义：**一个用于创建对象的接口，让子类决定实例化那一个类。工厂方法使一个类的实例化延迟到其子类。

Product为抽象产品类负责定义产品的共性，实现对事物最抽象的定义；

Creator为抽象创建类，也就是抽象工厂，具体如何创建产品类是由具体的实现工厂ConcreteCreator完成的。

```
具体工厂类代码：
public class ConcreteCreator extends Creator{
  public<T extends Product> T createProduct(Class<T> c){
      Product product=null;
      try{
         product=(Product)Class.forName(c.getNmae()).newInstance();
         }catch(Exception e){
             //异常处理
         }
         return (T)product;
      }
}
```

**简单工厂模式：**

一个模块仅需要一个工厂类，没有必要把它产生出来，使用静态的方法

**多个工厂类：**

每个人种（具体的产品类）都对应一个创建者，每个创建者都独立负责创建对应的产品对象，非常符合单一职责原则

**代替单例模式：**

单例模式的核心要求就是在内存中只有一个对象，通过工厂方法模式也可以只在内存中生产一个对象

**延迟初始化：**

ProductFactory负责产品类对象的创建工作，并且通过prMap变量产生一个缓存，对需要再次被重用的对象保留

使用场景： jdbc连接数据库  硬件访问 降低对象的产生和销毁

### 抽象工厂模式

**定义：**为创建一组相关或相互依赖的对象提供一个接口，而且无须指定他们的具体类

```
抽象工厂类代码：
public abstract class AbstractCreator{
   //创建A产品家族
   public abstract AbstractProductA createProductA();
   //创建B产品家族
   public abstract AbstractProductB createProductB();
}
```

**使用场景：**

一个对象族（或是一组没有任何关系的对象）都有相同的约束。涉及不同操作系统的时候，都可以考虑使用抽象工厂模式

### 模板方法模式

**定义：**定义一个操作中的算法的框架，而将一些步骤延迟到子类中。使得子类可以不改变一个算法的结构即可重定义该算法的某些特定步骤。

**AbstractClass**叫做**抽象模板**，它的方法分为两类：

*基本方法

基本方法也叫基本操作，是由子类实现的方法，并且在模板方法被调用。

*模板方法

可以有一个或几个，一般是一个具体方法，也就是一个框架，实现对基本方法的调度，完成固定的逻辑。

注意：为了防止恶意的操作，一般模板方法加上final关键字，不允许被覆写。

**具体模板：**实现父类所定义的一个或多个抽象方法，也就是父类定义的基本方法在子类得以实现。

**使用场景：**

- 多个子类有公有的方法，并且逻辑基本相同时。
- 重要，复杂的算法，可以把核心算法设计成模板方法，周边相关的细节功能则由各个子类实现。
- 重构时，模板方法模式是一个经常使用的模式，把所有的代码抽取到父类中，然后通过钩子函数约束其行为。

### 建造者模式

**定义：**将一个复杂对象的构建与它的表示分离，使得同样的构建过程可以创建不同的表示。

**Product产品类**

通常是实现了模板方法模式，也就是模板方法和基本方法

**Builder抽象建造者**

规范产品的组建，一般是由子类实现。

**具体建造者**

实现抽象类定义的所有方法，并且返回一个组建好的对象。

**导演类**

负责安排已有模块的顺序，然后告诉Builder开始建造

**使用场景：**

- 相同的方法，不同的执行顺序，产生不同的事件结果时，可以采用建造者模式。
- 多个部件或零件，都可以装配到一个对象中，但是产生的运行结果又不相同时，则可以使用该模式。
- 产品类非常复杂，或者产品类中的调用顺序不同产生了不同的效能，这个时候使用建造者模式非常合适。

**建造者模式与工厂模式的不同：**

建造者模式最主要的功能是基本方法的调用顺序安排，这些基本方法已经实现了，顺序不同产生的对象也不同；

工厂方法则重点在创建，创建零件是它的主要职责，组装顺序则不是它所关心的。