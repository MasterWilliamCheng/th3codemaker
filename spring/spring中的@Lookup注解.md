spring在容器中注入bean的时候，scope默认的是单例模式，也就是说在整个应用中只能创建一个实例。当scope为PROTOTYPE类型的时候，在每次注入的时候会自动创建一个新的bean实例。但是当一个单例模式的bean去引用PROTOTYPE类型的bean的时候，PROTOTYPE类型的bean也会变成单例。
@Lookup注解可以用来解决这个问题


当习惯性使用@Resource或者@Autowired注入时：

```
    @SpringBootApplication
    public class DemoApplication implements CommandLineRunner{
        public static void main(String[] args) {
            SpringApplication.run(IOTPluginDemoApplication.class, args);
        }
    
        @Resource
        private  ClassB classB;
    
        @Override
        public void run(String... args) throws Exception {
            for (int i = 0; i < 5; i++) {
                classB.method();
            }
        }
    
        @Component
        public class ClassB{
            @Resource
            public ClassA classA;
            public void method(){
                System.out.println("The hashcode is " + classA.hashCode());
            }
        }
    
        @Component
        @Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)
        public class ClassA{
    
        }
    }
```
    
打印hashcode会发现，此时的ClassA并没有去创建新的实例    

![image](https://user-images.githubusercontent.com/31581862/113102823-1b0fb880-9231-11eb-8780-147e39291024.png)



修改ClassB的代码

```
    @Component
    public abstract class ClassB{
        @Lookup
        public abstract ClassA getClassA();
        public void method(){
            ClassA classA = getClassA();
            System.out.println("The hashcode is " + classA.hashCode());
        }
    }
```
打印hashcode：

![image](https://user-images.githubusercontent.com/31581862/113102843-2236c680-9231-11eb-870a-3aa3b8962639.png)


此时ClassA每次注入都会创建新的实例了

对于@Lookup修饰的方法，有相对应的标准

```
<public|protected> [abstract] <return-type> theMethodName(no-arguments);
```
public|protected要求方法必须是可以被子类重写和调用的
abstract可选，如果是抽象方法，CGLIB的动态代理类就会实现这个方法，如果不是抽象方法，就会覆盖这个方法
return-type是非单例的类型
no-arguments不允许有参数
