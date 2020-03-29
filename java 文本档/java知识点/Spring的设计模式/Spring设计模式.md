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



