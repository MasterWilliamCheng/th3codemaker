### **spring ioc流程**

###### **BeanFactory对象的创建和增强，完成bean配置信息的加载和解析**
1. 通过xml，注解等方式定义bean信息，由BeanDefinitionReader接口解析生成BeanDefinition，存于BeanDefinitionMap中，这是属于创建BeanFactory的一部分
2. 通过BeanFactoryPostProcessor后置处理器接口对BeanFactory增强处理，其中常用的实现类包括ConfigurationClassPostProcessor（注解），PlaceholderConfigurerSupport（占位符）等

###### **Bean的实例化，初始化过程**
1. 注册BeanPostProcessor后置处理器(registerBeanPostProcessors（beanFactory）)
2. 初始化信息源，国际化处理(initMessageSource())
3. 初始化多播器(initApplicationEventMulticaster())
4. 注册监听器(registerListeners())
5. 实例化并初始化bean对象
>
>这一步才是真正开始创建bean，其中包括通过反射实例化对象，填充属性populateBean()，执行Aware接口方法（invokeAwareMethods()获取容器内置属性），执行实例化前置方法(processor.postProcessBeforeInitialization)，执行自定义init方法(invokeInitMethods())，执行实例化后置方法(processor.postProcessAfterInitialization)等过程

![Spring IOC及bean生命周期](https://user-images.githubusercontent.com/31581862/129466794-54d732a2-e3a8-47ef-a042-09cd1db3cb76.png)
