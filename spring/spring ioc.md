### **spring ioc流程**

###### **BeanFactory对象的创建和增强，bean的创建流程**
1. 创建Beanfactory容器
2. 通过xml，注解等方式定义bean信息，由BeanDefinitionReader接口解析生成BeanDefinition，存于BeanDefinitionMap中，这是属于BeanFactory的一部分
3. 为beanfactory填充各种属性
4. 执行BeanFactoryPostProcessor后置处理器接口对BeanFactory增强处理（invokeBeanFactoryPostProcessors()），其中常用的实现类包括ConfigurationClassPostProcessor（注解），PlaceholderConfigurerSupport（占位符）等
5. 注册BeanPostProcessor后置处理器(registerBeanPostProcessors（beanFactory）)，常用的实现类有ApplicationContextPostProcessor（设置ApplicationContext，Environment等）、AbstractAutoProxyCreator（AOP代理对象的创建）
6. 初始化信息源，国际化处理(initMessageSource())
7. 初始化多播器(initApplicationEventMulticaster())
8. 注册监听器(registerListeners())
9. 实例化并初始化bean对象(finishBeanFactoryInitialization())

###### **Bean的实例化，初始化过程(finishBeanFactoryInitialization())**
>这一步才是真正开始创建bean，其中包括通过反射实例化对象，填充属性populateBean()，执行Aware接口方法（invokeAwareMethods()获取容器内置属性），执行实例化前置方法(processor.postProcessBeforeInitialization)，执行自定义init方法(invokeInitMethods())，执行实例化后置方法(processor.postProcessAfterInitialization)等过程

![Spring IOC及bean生命周期](https://user-images.githubusercontent.com/31581862/129466794-54d732a2-e3a8-47ef-a042-09cd1db3cb76.png)


### **Spring的循环依赖问题**

循环依赖是指多个对象的属性相互依赖了对方，导致创建对象过程中循环嵌套，最终创建失败。spring中引进了三级缓存来解决这个问题。

在创建bean对象之前，要先经过doGetBean这一步先获取是否已经存在了对象，其中核心方法getSingleton引出了三级缓存

//一级缓存 保存成品对象\
private final Map<String, Object> singletonObjects = new ConcurrentHashMap(256);

//三级缓存 保存一个对象工厂，ObjectFactory是一个对象工厂，可以自定义方法\
private final Map<String, ObjectFactory<?>> singletonFactories = new HashMap(16);

//二级缓存 存半成品对象\
private final Map<String, Object> earlySingletonObjects = new ConcurrentHashMap(16);

在ioc的过程中，因为**实例化和初始化是分开进行的**，所以spring可以先保存半成品对象的引用，再进一步填充各自对象，达到解决循环依赖的需求，值得一提的是，三级缓存是用于AOP的时候，通过getEarlyBeanReference方法将动态代理生成的代理对象替换实例化的对象（AnnotationAwareAspectJAutoProxyCreator就是执行aop的后置处理器），如果没有aop，实际上使用一二级缓存就可以解决循环依赖问题。
