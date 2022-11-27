# Spring

## IOC

### 概念

- IOC（Inversion of Control）：控制反转。是一种设计思想而非具体的技术实现，其本质时将手动创建对象的控制权交由Spring框架来管理。
  - 控制：指对象的创建，管理的权力。
  - 反转：控制权从自己手里交给Spring框架。
- 将对象之间相互依赖的关系交给IOC容器管理。可以极大简化应用的开发，把应用从复杂的依赖关系中解放出来。IOC如同一个超级工厂，开发人员只需要按要求配置好配置文件/注解，不需要考虑对象的创建和管理。



### Spring Bean

#### 什么是Spring Bean

- 简单来说，bean指的时那些被IOC容器管理的对象。
- 我们需要告知IOC容器帮助我们管理哪些对象，这个是通过配置元数据来完成的，可以通过xml、注解、Java配置类来完成。注入IOC容器的过程我们成为依赖注入（DI）。

### Bean的生命周期

- 1.实例化bean：

  - 对于BeanFactory容器：当用户向容器请求一个尚未初始化的bean时，调用createBean（）初始化。
  - 对于ApplicationContext容器：容器启动时，创建所有的bean，容器通过BeanDefinition进行实例化。这一步只是简单的实例化，并未进行依赖注入。

  实例化对象被封装在beanWrapper中，BeanWapper提供了设置对象属性的接口。

- 2.设置对象属性（DI，依赖注入）：

  - Spring根据BeanDefinition中的信息进行依赖注入，并且根据BeanWrapper提供的接口对属性完成依赖注入。

- 3.注入Aware接口
  - 检查对象是否实现了xxxAware接口，如果实现，将xxxAware接口实例注入到bean。
    - BeanNameAware接口，调用setBeanName（）方法传入bean的名字。
    - BeanClassLoaderAware接口，调用setBeanClassLoader（）方法，传入ClassLoader的实例。
    - BeanFactoryAware接口，调用setBeanFactory（）方法，传入BeanFactory的实例。
- 4.bean自定义处理（BeanPostProcessor）：经过上面三个步骤后，bean对象已经完成构造，但是如果想要对象在被使用前进行一些自定义处理，可以通过BeanPostProcessor接口实现。
  - postProcessBeforeInitialzation：这一步被称为前置处理，先于InitialzationBean执行。该接口会将bean对象传入，可以对bean作任何处理。
    - InitialzationBean和init-method：
      - 当前置处理完之后就会进入本阶段。这一阶段也是在bean正式构造之后增加自定义逻辑，与前置处理不同的是，该接口不会传入bean本身，因此这一步没办法处理对象本身，只能增加一些额外的逻辑，该接口只有一个函数：afterPropertiesSet()。Spring将在执行完前置处理后执行检查是否实现了这个接口，并执行afterPropertiesSet()。
      - Spring为了降低代码的侵入性，也给bean的配置提供了init-method方法，该属性指定了这一阶段需要执行的函数名，Spring将在初始化阶段执行我们设置的函数。该方法本质上仍然使用了IntialzationBean接口。
  - postProcessAfterIntialzation：这一步被称为后置处理，在IntialzationBean之后执行。该接口会将bean对象传入，可以对bean作任何处理。
- 5.bean的销毁
  - DisposableBean
    - 当bean实现了该接口，在销毁bean时会调用destroy（）方法
  - destroy-method
    - 和init-method一样，如果在配置文件中定义了destory-method属性，会在销毁阶段执行我们指定的方法。

![Spring Bean 生命周期](https://images.xiaozhuanlan.com/photo/2019/24bc2bad3ce28144d60d9e0a2edf6c7f.jpg)

## AOP





