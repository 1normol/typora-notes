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

### 概念

- 面向切面编程，AOP能够将与业务无关却为业务模块所共同调用的逻辑或责任封装起来，以切面的方式切入业务，减少系统的重复代码，降低模块间耦合，为代码提供更好的拓展性和维护性。

### AOP相关概念

- 切面：Aspect注解标注在类上，声明当前类是一个组织了切面逻辑的类。
- 切入点：定义进行拦截的连接点。
- 通知：拦截到拦截点之后需要执行的代码，分为前置通知，返回通知，异常通知，后置通知，环绕通知。
- 目标对象：代理的目标对象

#### Advice主要类型

- Before：该注解标注的方法在业务代码执行之前执行，不能阻止业务代码执行，除非抛出异常
- AfterRunning：该注解标注的方法在业务代码之后执行
- AfterThrowing：该注解标注的方法在业务代码发生异常后执行
- After：该注解标注的方法在所有的Advice执行完之后执行。无论业务代码是否抛出异常都会执行。类似于finally
- Around：环绕通知，该注解标注的方法可以在业务前，业务后都可以执行，需要通过反射拿到具体的方法，可以理解为一个动态代理。

### Spring AOP使用的代理

- 如果目标对象实现了接口，默认使用JDK动态代理。
- 如果目标兑现实现了接口，可以强制使用CGLIB进行代理。
- 如果目标对象未实现接口，只能使用CGLIB进行动态代理。

#### JDK动态代理和CGLIB动态代理的区别

- JDK动态代理需要实现InvocationHandler接口，CGLIB是针对指定类生成子类，覆盖其中的方法。
- JDK动态代理通过反射实现，在运行时完成代理对象，CGLIB通过字节码编程实现，在编译期完成对象代理。
- JDK动态代理创建代理对象所花费的时间短，比CGLIB快8倍，而CGLIB所创建的动态代理对象在运行的时候性能要比JDK动态代理对象快10倍左右，因此对于单例对象的代理适合CGLIB，而需要频繁创建代理对象可以考虑选择JDK动态代理。



## 常见面试题

- 1.Spring bean的生命周期
  - 详见IOC
- 2.Spring bean的作用域
  - singleton：单例
  - prototype：原型
  - request：每一次http请求都产生新的bean，只在当前request请求中有效
  - session：每一次http请求都产生新的bean，只在当前session中有效
- 3.Spring Aop和AspectJ AOP有什么区别？
  - AspectJ属于运行时增强，通过字节码编程实现，Spring AOP属于运行时增强，基于代理实现。
  - 当切面较多时，选择使用AspectJ AOP，效率要快很多。
- 4.代理方法可以为private吗
  - CGLIB通过继承代理类，重写子类实现代理，子类不可为私有。
  - JDK动态代理通过实现InvocationHandler接口，接口必须为public的。
- 5.事务的传播机制
  - 以A（）方法调用B（）方法为例。
  - required：如果有事务则加入事务，如果没有则创建事务（默认值）。
  - required_new：不管是否存在是否，都将开启创建新的事务，新的事务完成，执行老的事务，老的事务若回滚不影响新的事务。如果B方法使用此注解，将在B方法执行的时候将原来的事物挂起，创建并执行新的事务。
  - mandatory：使用当前事务，如果事务不存在，就抛出异常。如果A方法设置为not_support，B方法使用mandatory，则会抛出异常。
  - supports：使用当前事务，如果事务不存在，则不使用事务。如果A方法正常执行事务，B方法使用此注解将继续使用A的事务。
  - not_supports：不使用事务，如果当前存在事务，则将事务挂起。如果A方法正常执行事务，执行到B的时候事务将会被挂起。
  - never：不使用事务，如果当前存在事务，则抛出异常。
  - nested：嵌套事务（嵌套事务回滚，主事务不回滚。主事务回滚，嵌套事务跟着回滚），如果当前事务存在，那么在嵌套的事务中执行。如果当前事务不存在，则执行和required一样的操作，创建一个事务。

- 6.事务失效场景
  - @Trancational注解设置事务传播级别propagation设置错误。
  - @Trancational注解设置rollback错误，实际抛出的错误与设置的不一致。
  - @Trancational注解修饰了private方法，也会导致事务失效。
  - 异常被catch处理掉导致注解失效。
- 7.Spring Mvc执行流程
  - 浏览器（客户端）发送请求到达DispatcherServlet。
  - DispatcherServlet根据请求找到对应的handlerMapping，解析到对应的handler。
  - 解析到正确的handler之后，handlerAdaptor会根据handler调用真正的处理器来处理请求，并执行业务逻辑
  - 处理完业务逻辑之后，会返回一个modelAndView对象。Model时数据对象，View是逻辑上的View
  - ViewResovler根据逻辑上的View查找真正的View。
  - 使用View对数据进行渲染并返回。
- 8.Spring Mvc核心组件
  - DispatcherServlet：中央调度器，负责各个组件的调度，可以理解为一个中转站。
  - Handler：处理器，完成具体的业务逻辑，相当于Servlet。
  - HandlerMapping：处理器映射器，负责根据用户请求的URL找到对应的处理器Handler
  - HandlerAdaptor：处理器适配器，根据HandlerMapping解析的调用链，完成对Hadnler的调用
  - ModelAndView：视图模型，用于封装处理器返回的数据以及对应的视图。
  - ViewResovler：视图解析器，负责对ModelAndView进行解析。可以配置多个。
  - View：视图，负责完成数据渲染。
- 9.@RestController和@Controller的区别
  - @RestController等价于@Responsesbody+@Controller。@ResponseBody注解是将Controller返回的对象转换为指定的格式之后写入到Http的响应对象中。
  - @RestController一般用于Restful风格中，用于返回json串，也可以返回xml。@Controller一般用于传统的Spring Mvc，返回一个视图。
- 10.
