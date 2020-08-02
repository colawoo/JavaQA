## 组件注册

### @Configuration 

标注在类上，告诉Spring这是一个配置类。相当于我们之前的xml配置文件。

```java
@Configuration
public class MainConfig {
}
```



### @Bean 

标注在方法上，ioc容器启动时，向容器中注册一个bean。相当于xml配置：`<bean id="" class=""></bean>`

id：@Bean("person") > 方法名

class：方法返回值类型

```java
@Bean
public Person person() {
    return new Person("zhangsan", 23);
}
```



### @ComponentScan

标注在配置类上，包扫描，将`base-package`指定的包中带有`@Controller`、`@Service`、`@Repository`、`@Component`四种注解的组件，注册到容器中。

`@ComponentScan(value="com.xx")`相当于xml配置：`<context:component-scan base-package="com.xx"></context:component-scan>`

- `value`：指定要扫描的包。
- `includeFilters=@Filter[]`：扫描的时候只需要包含哪些组件。需要结合`useDefaultFilters = false`使用。因为默认`useDefaultFilters = true`表示扫描带有`@Controller`、`@Service`、`@Repository`、`@Component`的所有组件，将使includeFilters无效。
- `excludeFilters=@Filter[]`：扫描的时候按照某些规则排除一些组件。

```java
@ComponentScan(value="com.xuxq",
        includeFilters = {
                @ComponentScan.Filter(type = FilterType.ANNOTATION, classes = {Service.class})
        },
        useDefaultFilters = false
)
```

```java
@ComponentScan(value="com.xuxq",
        excludeFilters = {
                @ComponentScan.Filter(type = FilterType.ANNOTATION, classes = {Controller.class})
        }
)
```



#### 添加多个扫描规则

- 由于`@ComponentScan`上有JDK1.8新加的`@Repeatable(ComponentScans.class)`注解，所以可以在类上添加多个`@ComponentScan`注解。
- @ComponentScans：参数`value()=ComponentScan[]`。

```java
@ComponentScans(value = {
    @ComponentScan(value="com.xuxq",
        excludeFilters = {
            @ComponentScan.Filter(type = FilterType.ANNOTATION, classes = {Controller.class})
        }
    )
})
```



#### @Filter

`@ComponentScan.Filter(type = FilterType.ANNOTATION, classes = {Controller.class})`

- `FilterType.ANNOTATION`：按照注解过滤，如`Controller.class`
- `FilterType.ASSIGNABLE_TYPE`：按照指定类型过滤，如`UserService.class`
- `FilterType.ASPECTJ`：使用`ASPECTJ`表达式过滤
- `FilterType.REGEX`：使用正则表达式过滤
- `FilterType.CUSTOM`：使用自定义规则过滤



#### 自定义过滤规则

```java
public class MyTypeFilter implements TypeFilter {
    /**
     * metadataReader 读取到的当前正在扫描的类的信息
     * metadataReaderFactory 可以获取到其他任何类的信息
     */
    @Override
    public boolean match(MetadataReader metadataReader, MetadataReaderFactory metadataReaderFactory) throws IOException {

        // 获取当前扫描到的类的注解元数据
        AnnotationMetadata annotationMetadata = metadataReader.getAnnotationMetadata();

        // 获取当前扫描到的类的元数据
        ClassMetadata classMetadata = metadataReader.getClassMetadata();

        // 获取当前扫描到的类的资源信息
        Resource resource = metadataReader.getResource();

        String className = classMetadata.getClassName();

        System.out.println("--->"+className);

        if (className.contains("My")) {
            return true;
        }

        return false;
    }
}
```



```java
@Configuration
@ComponentScan(value="com.xuxq",
        includeFilters = {
            @ComponentScan.Filter(type = FilterType.CUSTOM, classes = {MyTypeFilter.class})
        },
        useDefaultFilters = false
)
public class MainConfig {
}
```



从下面的输出结果可以看出，禁用了spring默认过滤规则，使用我们自定义的过滤规则，将类名带`My`的组件扫描到容器中了。我们配置的包`com.xuxq`下的每一个类都要经过自定义的过滤器进行匹配。

```shell
org.springframework.context.annotation.internalConfigurationAnnotationProcessor
org.springframework.context.annotation.internalAutowiredAnnotationProcessor
org.springframework.context.annotation.internalCommonAnnotationProcessor
org.springframework.context.event.internalEventListenerProcessor
org.springframework.context.event.internalEventListenerFactory
mainConfig
myTypeFilter
```



测试

```java
public class MainTest {
    
    public static void main(String[] args) {
        // 加载配置文件
        // ApplicationContext ctx = new ClassPathXmlApplicationContext("beans.xml");
        // Person person = ctx.getBean("person", Person.class);        
        
        // 加载配置类
        ApplicationContext ctx = new AnnotationConfigApplicationContext(MainConfig.class);
        System.out.println(ctx.getBeanDefinitionCount());
        for (String name : ctx.getBeanDefinitionNames()) {
            System.out.println(name);
        }
        
    }
}
```







### @Scope

有以下四个值，重点看前两个。

- singleton：单实例（默认）
- prototype：多实例
- request：同一次请求创建一个实例
- session：同一个session创建一个实例

```java
@Configuration
public class MainConfig {

    @Scope("prototype")
    @Bean
    public User user() {
        return new User();
    }
}
```



#### 创建时机

下面我们探索一下，单实例和多实例分别是在什么时候创建实例的？

```java
public class App {

    public static void main(String[] args) {
        ApplicationContext ctx = new AnnotationConfigApplicationContext(MainConfig.class);
        System.out.println("容器启动完成");
        User u1 = ctx.getBean("user", User.class);
        User u2 = ctx.getBean("user", User.class);
        System.out.println(u1 == u2);

    }
}
```



单实例情况`@Scope("singleton")`，输出结果

```shell
给容器中添加user组件
容器启动完成
true
```



多实例情况`@Scope("prototype")`，输出结果

```shell
容器启动完成
给容器中添加user组件
给容器中添加user组件
false
```



结论：

ApplicationContext容器，启动时一并创建所有单实例。而多实例是在调用`getBean()`方法的时候创建。



### @Lazy

懒加载，只对单实例bean有效。在第一次调用`getBean()`方法时创建实例。

```java
@Configuration
public class MainConfig {
  
    @Lazy
    @Bean
    public User user() {
        return new User();
    }
}
```



### @Conditional









### 总结



## 生命周期







## 属性赋值







## 自动装配



