# 1. Spring Framework

## 1.1 为什么需要框架

一个应用程序，真正满足用户需求的是业务逻辑的代码。但要保证代码能够正确、正常地运行，实际上也需要有其它的、与业务逻辑无关的代码的支持：

* 日志记录
* 事务
* 安全防护
* 数据持久化
* ...

![image-20251030103628629](asset/image-20251030103628629.png)

这些东西，其实每一个应用程序都需要用到。如果没有框架，那么每一个应用程序的开发者都需要自己写一套Logging的逻辑、Transaction的逻辑...实际上并不高效。

框架就是帮你把这所有的与业务逻辑无关的、通用的功能都写好，做成一个现成的工具，需要的时候直接拿来就用即可。

举一个类比的例子，例如我们现在需要做一个自己的衣柜，IKEA（框架）给我们提供了所有制作家具的工具，木板、螺丝、油漆...，我们只需要根据我们的需要，去选择合适的工具，来构造自己的衣柜就可以了，而不再需要自己从0到1地砍树，构造模板。

这里的Spring框架就类似于IKEA，他为我们提供好我们可能需要的所有工具，那我们真正用到的时候，可以直接拿来就用，不用自己去重新开发。

总结一下，使用框架的好处：

1. 节约时间成本
2. bug更少，因为已经被充分测试过
3. 有便于讨论交流的community

# 2. 创建Bean

## 2.1 创建Maven项目

* 创建maven项目
  * A group ID, which we use to group multiple related projects
  * An artifact ID, which is the name of the current application
  * A version, which is an identifier of the current implementation state

![image-20251030111418223](asset/image-20251030111418223.png)

* maven文件结构

  ![image-20251030112033733](asset/image-20251030112033733.png)

* 添加依赖

  这是原本的pom文件

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <project xmlns="http://maven.apache.org/POM/4.0.0"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
      <modelVersion>4.0.0</modelVersion>
  
      <groupId>com.tudou</groupId>
      <artifactId>sq-ch2-ex6.</artifactId>
      <version>1.0-SNAPSHOT</version>
  
      <properties>
          <maven.compiler.source>11</maven.compiler.source>
          <maven.compiler.target>11</maven.compiler.target>
          <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
      </properties>
  
  </project>
  ```

  原本External Libraries部分只有JDK

  ![image-20251030112411987](asset/image-20251030112411987.png)

  往pom文件中增加依赖

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <project xmlns="http://maven.apache.org/POM/4.0.0"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
  http://maven.apache.org/xsd/maven-4.0.0.xsd">
      <modelVersion>4.0.0</modelVersion>
      <groupId>org.example</groupId>
      <artifactId>sq_ch2_ex1</artifactId>
      <version>1.0-SNAPSHOT</version>
      <dependencies>
          <dependency>
              <groupId>org.springframework</groupId>
              <artifactId>spring-jdbc</artifactId>
              <version>5.2.6.RELEASE</version>
          </dependency>
      </dependencies>
  </project>
  ```

  IDE会下载这些依赖，在External Libraries可以找到这些下载的依赖

  ![image-20251030112620346](asset/image-20251030112620346.png)

## 2.2 如何知道需要添加的maven依赖

Whenever you work with a new Spring project, you can search for the dependencies you need to add directly in the Spring reference (https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html). Generally, Spring dependencies are part of the org.springframework group ID.

## 2.3 创建Bean并添加入Spring容器中

### 2.3.1 使用@Bean注解

![image-20251030115136846](asset/image-20251030115136846.png)

* step1：定义一个配置类

  ```java
  @Configuration
  public class ProjectConfig {
  }
  ```

* step2：在配置类中添加一个方法，并在方法上添加@Bean注解，这个方法返回一个对象实例。该实例对象会被加入到spring-context中

  ```Java
  @Configuration
  public class ProjectConfig {
      // By adding the @Bean annotation
      // we instruct Spring to call this method when at context initialization 
      // and add the returned value to the context.
      @Bean
      Parrot parrot() {  // the method's name also becomes the bean's name
          Parrot p = new Parrot();
          p.setName("Koko");
          return p;
      }
  
      @Bean
      String hello() {
          return "Hello";
      }
  
      @Bean
      Integer ten() {
          return 10;
      }
  }
  ```

* step3：基于配置类，创建spring-context

  可以从context中获取到bean

  ```java
  public class Main {
      public static void main(String[] args) {
          // When creating the Spring context instance,
          // send the configuration class as a parameter to instruct Spring to use it.
          AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(ProjectConfig.class);
          Parrot p = context.getBean(Parrot.class);
          System.out.println(p.getName());  // Koko
  
          String s = context.getBean(String.class);
          System.out.println(s);  // Hello
  
          Integer n = context.getBean(Integer.class);
          System.out.println(n);  // 10
      }
  }
  ```

​		![image-20251030115541214](asset/image-20251030115541214.png)

==注意：==

* 默认情况下，bean的名称为方法名

  ```java
  @Configuration
  public class ProjectConfig {
      @Bean
      Parrot parrot() {  // 该bean的名称为parrot
          Parrot p = new Parrot();
          p.setName("Koko");
          return p;
      }
  }
  ```

* 也可以通过以下注解属性来为bean重命名

  ```java
  @Bean(name = "miki")
  @Bean(value = "miki")
  @Bean("miki")
  ```

* 如果往spring容器中添加多个类型相同的Bean，则不能再通过类型来获取bean，而需要通过bean的标识符来获取bean。用`context.getBean("parrot2", Parrot.class);`方法，第一个参数为bean的名称，第二个参数为bean的类型

  ```java
  @Configuration
  public class ProjectConfig {
      @Bean
      Parrot parrot1() {  // the method's name also becomes the bean's name
          Parrot p = new Parrot();
          p.setName("Koko");
          return p;
      }
  
      @Bean
      Parrot parrot2() {  // the method's name also becomes the bean's name
          Parrot p = new Parrot();
          p.setName("Miki");
          return p;
      }
  
      @Bean
      @Primary
      Parrot parrot3() {  // the method's name also becomes the bean's name
          Parrot p = new Parrot();
          p.setName("Riki");
          return p;
      }
  }
  
  public class Main {
      public static void main(String[] args) {
          AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(ProjectConfig.class);
          Parrot p = context.getBean("parrot2", Parrot.class);
          System.out.println(p.getName());  // Miki
      }
  }
  ```

* 如果有多个类型相同的bean，可以通过@Primary注解来设置默认选择

  这样有多个类型相同的bean时，会选择默认的那个bean

  ```java
  @Configuration
  public class ProjectConfig {
      @Bean
      Parrot parrot1() {  // the method's name also becomes the bean's name
          Parrot p = new Parrot();
          p.setName("Koko");
          return p;
      }
  
      @Bean
      Parrot parrot2() {  // the method's name also becomes the bean's name
          Parrot p = new Parrot();
          p.setName("Miki");
          return p;
      }
  
      @Bean
      @Primary
      Parrot parrot3() {  // the method's name also becomes the bean's name
          Parrot p = new Parrot();
          p.setName("Riki");
          return p;
      }
  }
  
  public class Main {
      public static void main(String[] args) {
          AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(ProjectConfig.class);
          Parrot p = context.getBean(Parrot.class);
          System.out.println(p.getName());  // Riki
      }
  }
  ```



### 2.3.2 使用stereotype annotations

![image-20251030170145905](asset/image-20251030170145905.png)

以@Component注解为例

* step1：在需要创建Bean的类增加@Component注解

  【tell Spring ==Which== classes to add an instance to its context (Parrot)】

  ```java
  package com.tudou.main;
  
  @Component
  public class Parrot {
      private String name;
  
      public String getName() {
          return name;
      }
  
      public void setName(String name) {
          this.name = name;
      }
  }
  ```

* step2：在配置类中增加 @ComponentScan 注解，该注解标识要扫描的位置

  【tell Spring ==Where== to find these classes (using @ComponentScan)】

  ```java
  @Configuration
  @ComponentScan(basePackages = "com.tudou")
  public class ProjectConfig {
  
  }
  ```

* step3：基于配置类，创建spring-context

  可以从context中获取到bean

  ```java
  public class Main {
      public static void main(String[] args) {
          // When creating the Spring context instance, send the configuration class as a parameter to instruct Spring to use it.
          AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(ProjectConfig.class);
          Parrot p = context.getBean(Parrot.class);
          System.out.println(p);  // com.tudou.pojo.Parrot@59662a0b
          System.out.println(p.getName());  // null 因为没有给name属性赋值，则默认为null
      }
  }
  ```

* step4：如果想要创建完bean之后，对bean进行一些处理，可以使用@PostConstruct注解。spring会在执行完bean的构造函数之后，马上执行被@PostConstruct注解的方法

  * 要使用该注解，需要先增加该依赖

    ```xml
    <dependency>
        <groupId>javax.annotation</groupId>
        	<artifactId>javax.annotation-api</artifactId>
        <version>1.3.2</version>
    </dependency>
    ```

  * 在类中增加@PostConstruct方法

    ```java
    @Component
    public class Parrot {
        private String name;
    
        public String getName() {
            return name;
        }
    
        public void setName(String name) {
            this.name = name;
        }
    
        @PostConstruct
        public void init() {
            this.name = "Kiki";
        }
    }
    ```

  * 测试效果

    ```java
    public class Main {
        public static void main(String[] args) {
            // When creating the Spring context instance, send the configuration class as a parameter to instruct Spring to use it.
            AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(ProjectConfig.class);
            Parrot p = context.getBean(Parrot.class);
            System.out.println(p);  // com.tudou.pojo.Parrot@59662a0b
            System.out.println(p.getName());  // Kiki
        }
    }
    ```



注意：

* 和@PostConstruct注解类似的，还有@PreDestroy注解

  With this annotation, you define a method that Spring calls immediately before closing and clearing the context. But generally I recommend developers avoid using it and find a different approach to executing something before Spring clears the context, *mainly because you can expect Spring to fail to clear the context*. Say you defined something sensitive (like closing a database connection) in the @PreDestroy method; if Spring doesn’t call the method, you may get into big problems.

* 使用类似@Component这样的stereotype annotation（模式注解）来创建bean，默认 bean 名 = 类名首字母小写。例如Parrot类的bean默认bean名为parrot。当然也可以用@Qualifier注解显式自定义指定bean名称。

### 2.3.3 编程式

使用 ApplicationContext实例的registerBean()方法来将bean注册到Spring容器中

* 配置类

  ```java
  @Configuration
  @ComponentScan(basePackages = "com.tudou")
  public class ProjectConfig {
  
  }
  ```

* Bean类【普通的pojo类，没有被任何注解修饰】

  ```java
  public class Parrot {
      private String name;
  
      public String getName() {
          return name;
      }
  
      public void setName(String name) {
          this.name = name;
      }
  }
  ```

* main方法

  ```java
  public class Main {
      public static void main(String[] args) {
          // When creating the Spring context instance, send the configuration class as a parameter to instruct Spring to use it.
          AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(ProjectConfig.class);
          // We create the instance we want to add to the Spring context.
          Parrot p = new Parrot();
          p.setName("Kik1");
          // We define a Supplier to return this instance.
          Supplier<Parrot> parrotSupplier = () -> p;
          // We call the registerBean() method to add the instance to the Spring context.
          context.registerBean("parrot1", Parrot.class, parrotSupplier);
          // To verify the bean is now in the context, we refer to the parrot bean and print its name in the console.
          Parrot parrot = context.getBean(Parrot.class);
          System.out.println(p.getName());  // Kik1
      }
  }
  ```

  

==注意：==

* registerBean方法详解

  ```java
  <T> void registerBean(
  	String beanName,  // 定义bean名称 可以为null
  	Class<T> beanClass,  // bean所属类型
      // Supplier是一个@FunctionalInterface，
  	// The implementation of this Supplier needs to 
  	// return the value of the instance you add to the context.
      Supplier<T> supplier,
      // the BeanDefinitionCustomizer is just an interface 
      // you implement to configure different characteristics of the bean; 
      // e.g.,making it primary.
      // customizers是一个可变参数
  	BeanDefinitionCustomizer... customizers);
  ```

  customizers的使用示例：

  ```java
  context.registerBean("parrot1",
  	Parrot.class,
  	parrotSupplier,
  	bc -> bc.setPrimary(true));
  ```

* 只有Spring5及其之后可以使用编程式方法给bean容器添加bean

# 3. Wiring Bean【管理Bean之间的关系】

## 3.1 方法调用

直接通过方法调用的方式创建Bean之间的关系。

例如现在有一个Person类，有一个Parrot类，需要实现让person实例own parrot实例。

![image-20251031101904837](asset/image-20251031101904837.png)

![image-20251031101941748](asset/image-20251031101941748.png)

代码如下：

```java
// Parrot类
public class Parrot {
    private String name;

    public Parrot() {
        System.out.println("Parrot created");
    }

    // Omitted getters and setters

    @Override
    public String toString() {
        return "Parrot : " + name;
    }
}
```



```java
// Person类
public class Person {
    private String name;
    private Parrot parrot;

    // Omitted getters and setters
}
```



```java
// 配置类
@Configuration
public class ProjectConfig {
    @Bean
    public Parrot parrot() {
        Parrot p = new Parrot();
        p.setName("Koko");
        return p;
    }
    @Bean
    public Person person() {
        Person p = new Person();
        p.setName("Ella");
        // ***构建person实例和parrot实例的关系***
        p.setParrot(parrot());
        return p;
    }
}
```



```java
// main函数
public class Main {
    public static void main(String[] args) {
        var context = new AnnotationConfigApplicationContext(ProjectConfig.class);
        Person person = context.getBean(Person.class);
        Parrot parrot = context.getBean(Parrot.class);
        System.out.println("Person's name: " + person.getName());
        System.out.println("Parrot's name: " + parrot.getName());
        System.out.println("Person's parrot: " + person.getParrot());
    }
}

// 执行结果
// Parrot created
// Person's name: Ella
// Parrot's name: Koko
// Person's parrot: Parrot : Koko
```

![image-20251031102719828](asset/image-20251031102719828.png)



==注意：==

* 在创建Person bean的时候，调用了parrot()，是否意味着又再创建了一个新的Parrot对象？

  ![image-20251031103621003](asset/image-20251031103621003.png)

​		答案：

![image-20251031103906228](asset/image-20251031103906228.png)

​	从上面示例代码中，也可以看到`Parrot created`只打印了一次，说明Parrot的构造函数只被调用过一次。

## 3.2 参数注入

和3.1方法的区别，仅在于配置类

```java
@Configuration
public class ProjectConfig {
    @Bean
    public Parrot parrot() {
        Parrot p = new Parrot();
        p.setName("Koko");
        return p;
    }
    // 参数注入
    // Spring injects the parrot bean into this parameter
    @Bean
    public Person person(Parrot parrot) {
        Person p = new Person();
        p.setName("Ella");
        p.setParrot(parrot);
        return p;
    }
}
```

![image-20251031111233260](asset/image-20251031111233260.png)

==注意：==

* 这里使用的是依赖注入的思想【I is a technique involving the framework setting a value into a specific field or parameter.】
* DI is an application of the IoC principle, and IoC implies that the framework controls the application at execution. 

## 3.3 @Autowire注入

### 3.3.1 属性注入

```java
// Parrot类
package com.tudou.pojo;
@Component
public class Parrot {
    private String name = "Koko";

    // Omitted getters and setters

    @Override
    public String toString() {
        return "Parrot : " + name;
    }
}
```



```java
// Person类
package com.tudou.pojo;
@Component
public class Person {
    private String name = "Ella";
    
    // Annotating the field with @Autowired, 
    // we instruct Spring to inject 
    // an appropriate value from its context.
    @Autowired
    private Parrot parrot;

    // Omitted getters and setters
}
```



```java
// 配置类
@Configuration
@ComponentScan(basePackages = "com.tudou.pojo")
public class ProjectConfig {
}
```



```java
// main函数
public class Main {
    public static void main(String[] args) {
        var context = new AnnotationConfigApplicationContext(ProjectConfig.class);
        Person p = context.getBean(Person.class);
        System.out.println("Person's name: " + p.getName());  // Ella
        System.out.println("Person's parrot: " + p.getParrot());  // Parrot : Koko
    }
}
```



注意：

* 这种方式在生产环境并不常使用：

  * 如果我们要使用@Autowire属性注入，就不能将该属性用final修饰

    ```java
    @Component
    public class Person {
        private String name = "Ella";
        
        @Autowired
        private final Parrot parrot;
    }
    ```

    例如上面的代码片段，是无法编译通过的。因为`private final Parrot parrot;`被final修饰的变量必须要进行初始化，而如果我们进行了初始化，则由于parrot是被final修饰的，则不能被修改，则@Autowire后续会注入失败，导致报错。

  * it’s more difficult to manage the value yourself at initialization.

### 3.3.2 构造注入

和属性注入的区别只在Person类中。

```java
// Parrot类
package com.tudou.pojo;
@Component
public class Parrot {
    private String name = "Koko";

    // Omitted getters and setters

    @Override
    public String toString() {
        return "Parrot : " + name;
    }
}
```



```java
// Person类
@Component
public class Person {
    private String name = "Ella";
    private Parrot parrot;
	
    @Autowired
    public Person(Parrot parrot) {
        this.parrot = parrot;
    }

    // Omitted getters and setters
}

```

![image-20251101113301088](asset/image-20251101113301088.png)

```java
// 配置类
@Configuration
@ComponentScan(basePackages = "com.tudou.pojo")
public class ProjectConfig {
}
```



```java
// main函数
public class Main {
    public static void main(String[] args) {
        var context = new AnnotationConfigApplicationContext(ProjectConfig.class);
        Person p = context.getBean(Person.class);
        System.out.println("Person's name: " + p.getName());  // Ella
        System.out.println("Person's parrot: " + p.getParrot());  // Parrot : Koko
    }
}
```

==注意：==

* **这种方式是实际生产环境使用最多的方式。**通过这种方式可以将Person类的parrot字段用final修饰。
* Spring 4.3起，如果只有一个构造函数，可以将@Autowired注解省略

### 3.3.3 setter注入

和属性注入的区别只在Person类中。

```java
@Component
public class Person {
    private String name = "Ella";
    private Parrot parrot;

    // Omitted getters and setters

    @Autowired
    public void setParrot(Parrot parrot) {
        this.parrot = parrot;
    }
}
```

==注意：==

* 这种方式非常少用
  * 可读性差
  * 不能用final修饰Person类的parrot字段

## 3.4 循环依赖

循环依赖问题

![image-20251102092241127](asset/image-20251102092241127.png)

```java
// Person类
@Component
public class Person {
    private final Parrot parrot;
    @Autowired
    public Person(Parrot parrot) {
        this.parrot = parrot;
    }
    // Omitted code
}
```



```java
// Parrot类
public class Parrot {
    private String name = "Koko";
    private final Person person;
    @Autowired
    public Parrot(Person person) {
        this.person = person;
    }
    // Omitted code
}
```

这里使用的是构造注入，则Spring无法解决循环依赖问题，会报错：

```java
Caused by:
org.springframework.beans.factory.BeanCurrentlyInCreationException: Error
creating bean with name 'parrot': Requested bean is currently in creation:
Is there an unresolvable circular reference?
at
org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.before
SingletonCreation(DefaultSingletonBeanRegistry.java:347)
```

如果将任意一个类改用字段注入，则可以解决：



```java
// Parrot类
@Component
public class Parrot {
    private String name = "Koko";
    
    @Autowired  // 字段注入
    private Person person;

    // Omitted code
}
```



```java
// Person类
@Component
public class Person {
    private String name = "Ella";

    @Autowired
    private final Parrot parrot;

    @Autowired
    public Person(Parrot parrot) {
        this.parrot = parrot;
    }
    
    // Omitted code
}
```

运行结果

```java
public class Main {
    public static void main(String[] args) {
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(ProjectConfig.class);
        Person person = context.getBean(Person.class);
        Parrot parrot = context.getBean(Parrot.class);
        // Ella has Koko
        System.out.println(person.getName()+" has "+person.getParrot().getName());
        // Koko
        System.out.println(parrot.getName());
    }
}
```

## 3.5 有多个bean 注入时如何选择

### 3.5.1 选择bean名和参数名相同的bean注入

情况1：使用@Bean创建Person bean，并注入依赖parrot bean

```java
@Configuration
public class ProjectConfig {
    @Bean
    public Parrot parrot1() {
        Parrot p = new Parrot();
        p.setName("Koko");
        return p;
    }

    @Bean
    public Parrot parrot2() {
        Parrot p = new Parrot();
        p.setName("Miki");
        return p;
    }

    @Bean
    // The name of the parameter matches the name of the bean representing parrot Miki.
    public Person person(Parrot parrot2) {
                Person p = new Person();
        p.setName("Ella");
        p.setParrot(parrot2);
        return p;
    }
}
```

![image-20251102102039668](asset/image-20251102102039668.png)



情况2：使用@Autowire注解注入parrot bean

```java
public class Parrot {
    private String name;

    // Omitted code
}
```



```java
@Component
public class Person {
    private String name = "Ella";

    private final Parrot parrot;
	
    @Autowire  // 构造注入时，@Autowire注解可以省略
    public Person(Parrot parrot2) {
        this.parrot = parrot2;
    }

    // Omitted code
}
```



```java
@Configuration
@ComponentScan(basePackages = "com.tudou.pojo")
public class ProjectConfig {
    @Bean
    public Parrot parrot1() {
        Parrot p = new Parrot();
        p.setName("Koko");
        return p;
    }

    @Bean
    public Parrot parrot2() {  // 会将这个bean注入到Person bean
        Parrot p = new Parrot();
        p.setName("Miki");
        return p;
    }
}
```

![image-20251102103942023](asset/image-20251102103942023.png)



### 3.5.2 使用@Qualifier注解

@Qualifier注解可以显式选择要使用的bean的名称【推荐使用，清晰，可维护性好】

```java
@Configuration
public class ProjectConfig {
    @Bean
    public Parrot parrot1() {
        Parrot p = new Parrot();
        p.setName("Koko");
        return p;
    }

    @Bean
    public Parrot parrot2() {
        Parrot p = new Parrot();
        p.setName("Miki");
        return p;
    }

    @Bean
    public Person person(@Qualifier("parrot1") Parrot parrot) {
                Person p = new Person();
        p.setName("Ella");
        p.setParrot(parrot);
        return p;
    }
}
```

注意：

* @Qualifier除了能够显式指定要使用的bean的名称，还能够用于给bean命名

### 3.5.3 使用@Primary

使用@Primary指定优先级最高的bean

```java
@Configuration
public class ProjectConfig {
    @Bean
    public Parrot parrot1() {
        Parrot p = new Parrot();
        p.setName("Koko");
        return p;
    }

    @Bean
    @Primary
    public Parrot parrot2() {
        Parrot p = new Parrot();
        p.setName("Miki");
        return p;
    }

    @Bean
    public Person person(Parrot parrot) {
        Person p = new Person();
        p.setName("Ella");
        p.setParrot(parrot);  // 注入名为Miki的parrot
        return p;
    }
}
```



# 4. Using abstractions

> Interface specifies the “what needs to happen,” while every object implementing the interface specifies the “how it should happen.”

## 4.1 为什么要面向抽象编程

用一个例子说明，为什么要面向抽象编程。

例如现在要实现一个功能，有一个Printer要实现打印运送订单信息，并且要按照地址信息升序打印，一般来说这个Printer会将排序的任务委托给其他的类实现，而自己专注于实现这个打印任务，那么会是像下面这样实现的：
![image-20251105104525813](asset/image-20251105104525813.png)

这样会存在的问题是：如果要更改排序的方式，比如现在不按照地址信息排序了，而是按照sender姓名信息排序，那DeliveryDetailsPrinter这个类也需要修改

![image-20251105104814259](asset/image-20251105104814259.png)

DeliveryDetailsPrinter和下层调用的排序类耦合在一起，就会导致如果要调用的排序类发生了改变，则DeliveryDetailsPrinter类的代码也需要更改。

这是面向实现编程的情况，如果我们面向接口编程，会是怎么样的？

如果DeliveryDetailsPrinter类仅是依赖一个Sorter接口，而不依赖具体的实现，那么无论下层的实现怎么改，都不会影响DeliveryDetailsPrinter的代码。

![image-20251105105132122](asset/image-20251105105132122.png)

![image-20251105105228140](asset/image-20251105105228140.png)

## 4.2 不使用Spring如何面向抽象编程

背景：实现一个发布评论的功能，当有人发布评论：1. 将评论存储到DB中 2. 将评论发送到一个邮箱中

项目结构：

![image-20251105110441702](asset/image-20251105110441702.png)

实现代码：

```java
// Comment类
package com.tudou.pojo;

public class Comment {
    private String author;
    private String text;

    public String getAuthor() {
        return author;
    }

    public void setAuthor(String author) {
        this.author = author;
    }

    public String getText() {
        return text;
    }

    public void setText(String text) {
        this.text = text;
    }
}
```



```java
// CommentRepository接口
// ps一般如果一个类/接口作用是和DB交互，就会命名为Repository
package com.tudou.repositories;

public interface CommentRepository {
    void storeComment(Comment comment);
}

```



```java
// CommentRepository接口实现类
package com.tudou.repositories;

public class DBCommentRepository implements CommentRepository{
    @Override
    public void storeComment(Comment comment) {
        System.out.println("Storing comment: " + comment.getText());
    }
}
```



```java
// CommentNotificationProxy接口
// ps  when implementing objects whose responsibility is 
// to establish communication with something outside the app, 
// we name these objects proxies
package com.tudou.proxies;

public interface CommentNotificationProxy {
    void sendComment(Comment comment);
}
```



```java
// CommentNotificationProxy接口实现类
package com.tudou.proxies;

public class EmailCommentNotificationProxy implements CommentNotificationProxy{
    @Override
    public void sendComment(Comment comment) {
        System.out.println("Sending email notification for comment: " + comment.getText());
    }
}

```



```java
// 发布评论 Service
public class CommentService {
    private final CommentRepository commentRepository;
    private final CommentNotificationProxy commentNotificationProxy;

    public CommentService(CommentRepository commentRepository, CommentNotificationProxy commentNotificationProxy) {
        this.commentRepository = commentRepository;
        this.commentNotificationProxy = commentNotificationProxy;
    }

    public void publishComment(Comment comment) {
        commentRepository.storeComment(comment);  // 存储评论
        commentNotificationProxy.sendComment(comment);  // 发送至邮箱
    }
}

```



```java
// main
// 创建接口的实例对象，并且构建CommentService和这些实例对象的关系
public class Main {
    public static void main(String[] args) {
        // Creates the instance for the dependencies
        DBCommentRepository commentRepository = new DBCommentRepository();
        EmailCommentNotificationProxy commentNotificationProxy = new EmailCommentNotificationProxy();
        // Creates the instance of the service class and providing the dependencies
        CommentService commentService = new CommentService(commentRepository, commentNotificationProxy);

        Comment comment = new Comment();
        comment.setAuthor("tudou");
        comment.setText("demo comment");

        commentService.publishComment(comment);
    }
}
```



## 4.2 Spring如何面向抽象编程



```java
// Comment类
public class Comment {
    private String author;
    private String text;

    public String getAuthor() {
        return author;
    }

    public void setAuthor(String author) {
        this.author = author;
    }

    public String getText() {
        return text;
    }

    public void setText(String text) {
        this.text = text;
    }
}
```



```java
// CommentRepository接口
package com.tudou.repositories;

public interface CommentRepository {
    void storeComment(Comment comment);
}
```



```java
// CommentRepository接口实现类
package com.tudou.repositories;

@Component
public class DBCommentRepository implements CommentRepository{
    @Override
    public void storeComment(Comment comment) {
        System.out.println("Storing comment: " + comment.getText());
    }
}
```



```java
// CommentNotificationProxy接口
// ps  when implementing objects whose responsibility is 
// to establish communication with something outside the app, 
// we name these objects proxies
package com.tudou.proxies;

public interface CommentNotificationProxy {
    void sendComment(Comment comment);
}
```



```java
// CommentNotificationProxy接口实现类
package com.tudou.proxies;

@Component
public class EmailCommentNotificationProxy implements CommentNotificationProxy{
    @Override
    public void sendComment(Comment comment) {
        System.out.println("Sending notification for comment: " + comment.getText());
    }
}
```



```java
// 发布评论 Service
@Component
public class CommentService {
    private final CommentRepository commentRepository;
    private final CommentNotificationProxy commentNotificationProxy;

    // 当只有一个构造函数的时候，可以省略@Autowire注解
    //  Spring会自动从Spring容器搜索实现了这些接口的bean并进行注入
    public CommentService(
            CommentRepository commentRepository,
            CommentNotificationProxy commentNotificationProxy
    ) {
        this.commentRepository = commentRepository;
        this.commentNotificationProxy = commentNotificationProxy;
    }

    public void publishComment(Comment comment) {
        commentRepository.storeComment(comment);
        commentNotificationProxy.sendComment(comment);
    }
}
```



```java
// 配置类
@Configuration
@ComponentScan(basePackages = "com.tudou")
public class ProjectConfiguration {
}
```



```java
public class Main {
    public static void main(String[] args) {
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(ProjectConfiguration.class);
        
        Comment comment = new Comment();
        comment.setAuthor("tudou");
        comment.setText("demo comment");
        
        CommentService commentService = context.getBean(CommentService.class);
        commentService.publishComment(comment);
    }
}
//执行结果：
Storing comment: demo comment
Sending notification for comment: demo comment
```



```xml
// 涉及的依赖
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>5.2.7.RELEASE</version>
    </dependency>
</dependencies>
```





==注意：==

* 为什么CommentService、DBCommentRepository、EmailCommentNotificationProxy要用@Component注解注释以创建spring bean，而Comment类不用呢？

  * 我们为什么要把bean放入Spring容器，是因为我们需要Spring容器帮我们创建它，并且管理bean之间的关系，在当前的场景，CommentService依赖DBCommentRepository和EmailCommentNotificationProxy，因此这三个bean放入Spring容器，能够让Spring容器方便地管理它们之间的关系，而Comment类不和任何其他类有依赖关系，因此没有必要放入Spring容器管理。如果硬要放入，只是增加了程序的复杂度。

* 如果一个接口有多个实现类bean，Spring怎么确定应该注入哪一个？

  例如CommentNotificationProxy接口有两个实现类，CommentPushNotificationProxy和EmailCommentNotificationProxy，注入的时候应该注入哪个类对应的bean？

  ![image-20251105162752208](asset/image-20251105162752208.png)

  * 使用@Primary注解标识默认使用的bean

    ```java
    @Component
    @Primary
    public class CommentPushNotificationProxy implements CommentNotificationProxy {
        @Override
        public void sendComment(Comment comment) {
            System.out.println("Sending push notification for comment: " + comment.getText());
        }
    }
    ```

    可能有人会好奇，什么情况下会用到这种方式，既然用@Primary标识默认使用的bean，那只保留那一个实现类就好了，其他的实现类是不是就用不到了，为什么还要保留？

    实际上，我们开发过程中应用程序会依赖很多依赖，这些依赖的接口可能存在已经提供的默认实现类，但并不match你的应用程序，所以这个时候你要写一个自己的实现类来满足你自己的应用程序的需求，就可以用@Primary来指定默认使用你自定义的bean。

    ![image-20251105163818695](asset/image-20251105163818695.png)

  * 使用@Qualifier注解定义bean名称，并指定要使用的bean

    ```java
    // CommentPushNotificationProxy实现类
    @Component
    @Qualifier("PUSH")  // 设置bean的标识符
    public class CommentPushNotificationProxy implements CommentNotificationProxy {
        @Override
        public void sendComment(Comment comment) {
            System.out.println("Sending push notification for comment: " + comment.getText());
        }
    }
    
    ```

    ```java
    // EmailCommentNotificationProxy实现类
    @Component
    @Qualifier("EMAIL")  // 设置bean的标识符
    public class EmailCommentNotificationProxy implements CommentNotificationProxy{
        @Override
        public void sendComment(Comment comment) {
            System.out.println("Sending notification for comment: " + comment.getText());
        }
    }
    
    ```

    ```java
    // 依赖CommentNotificationProxy接口的Service
    @Component
    public class CommentService {
        private final CommentRepository commentRepository;
        private final CommentNotificationProxy commentNotificationProxy;
    
        // 当只有一个构造函数的时候，可以省略@Autowire注解
        public CommentService(
                CommentRepository commentRepository,
            	// 指定要注入的bean
                @Qualifier("PUSH") CommentNotificationProxy commentNotificationProxy
        ) {
            this.commentRepository = commentRepository;
            this.commentNotificationProxy = commentNotificationProxy;
        }
    
        public void publishComment(Comment comment) {
            commentRepository.storeComment(comment);
            commentNotificationProxy.sendComment(comment);
        }
    }
    ```

    

    ![image-20251105163334956](asset/image-20251105163334956.png)

* 上面介绍了构造注入场景下，面向抽象编程怎么写；那其他注入场景下，对应的抽象编程代码怎么写？

  * 字段注入

    ![image-20251105162231673](asset/image-20251105162231673.png)

  * 使用@Bean创建bean

    ![image-20251105162548687](asset/image-20251105162548687.png)

## 4.3 注解是加在Class上还是Interface上

class上，因为注解是用于让Spring扫描到这个类，然后实例化这个类的bean加入Spring容器进行管理。而接口本身是不能被实例化的，只有类才能被实例化，所以注解是要加在Class上。

## 4.4 @Service @Repository注解

@Service、@Repository、@Component注解都是一样的，都是让Spring扫描到对应的类，并创建这个类的bean，加入Spring容器进行管理。

如果全部都使用@Component注解，实际上代码的层次并不明显，可读性不够好。

使用@Service注解来标识专门负责提供服务的Bean，使用@Repository注解来标识专门负责进行DB操作的Bean，无特殊含义的Bean用@Component注解标识，这样代码的可能性更佳，层次更加明显。



# 5. Bean scopes and life cycle

## 5.1 Bean scopes

### 5.1.1 Singleton

* 是什么：Spring中的Singleton作用域，不是代表一个类只能有一个Bean。相反，一个类可以有多个Bean，但*每个Bean都有自己唯一的 bean 名称，或者说 Bean id* 。

![image-20251106094636244](asset/image-20251106094636244.png)

* 是否要使用单例模式：

  * 使用单例模式需要保证这个bean是不可变的。因为如果使用单例模式，则在多线程场景下，这多个线程共享这一个单例bean，如果这个bean是可变的，在多线程并发场景下，会出现竞态条件。则需要额外加锁来保证线程安全，影响性能。

    ![image-20251106112819353](asset/image-20251106112819353.png)

  * 如果这个bean需要可变，则可以使用多例模式。

* eager instantiation or lazy instantiation：

  * eager instantiation（默认）：Spring会在初始化context的时候就创建所有单例bean。

    示例：

    ```java
    // Service
    @Service
    public class CommentService {
        public CommentService() {
            System.out.println("CommentService instance created.");
        }
    }
    ```

    ```java
    // 配置类
    @Configuration
    @ComponentScan(basePackages = "com.tudou")
    public class ProjectConfig {
    }
    ```

    ```java
    public class Main {
        public static void main(String[] args) {
            AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(ProjectConfig.class);
        }
    }
    ```

    ```java
    // 控制台打印
    CommentService instance created.
    ```

    可以看到，代码中并没有真正用到commentService这个bean，但是却被初始化了。

    

  * lazy instantiation：Spring只有在第一次用到这个单例bean的时候，才会创建这个bean。

    示例：

    ```java
    // Service
    @Service
    @Lazy  // 开启懒加载
    public class CommentService {
        public CommentService() {
            System.out.println("CommentService instance created.");
        }
    }
    ```

    ```java
    public class Main {
        public static void main(String[] args) {
            AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(ProjectConfig.class);
            System.out.println("Before retrieving the CommentService");
            CommentService service = context.getBean(CommentService.class);
            System.out.println("After retrieving the CommentService");
        }
    }
    ```

    ```java
    // 控制台打印
    Before retrieving the CommentService
    CommentService instance created.
    After retrieving the CommentService
    ```

    可以看到，只有在真正使用commentService这个bean的时候，才会创建该bean。

  * 对比
    * eager instantiation：要用的时候就有；如果bean的创建存在异常，可以在启动Spring应用程序的时候就马上发现；【推荐使用】
    * lazy instantiation：用之前需要检查是否存在，如果不存在还要创建，影响性能；如果bean的创建存在异常，无法马上发现，需要后续用到该bean的时候才能被发现；

### 5.1.2 Prototype

* 是什么：每一次尝试从Spring容器获取作用域为prototype的bean，Spring都会创建一个新bean返回。

  例如下图，CommentService这个类的作用域为prototype，每一次调用context.getBean(CommentService.class)的时候，都会创建一个新bean，因此`cs1==cs2`结果永远为false。

  ![image-20251107105131491](asset/image-20251107105131491.png)

* 应用场景：如果一个bean是Prototype的作用域，那么在多线程场景下，每个线程拿到的bean都是不同的，因此是线程安全的，可以随意修改bean。

  ![image-20251107105640890](asset/image-20251107105640890.png)

* 如何将bean定义为Prototype的作用域？

  在所有定义Bean的地方，增加上`@Scope(BeanDefinition.SCOPE_PROTOTYPE)`注解。

  * 例1：使用@Bean定义Bean

    ```java
    // Service类
    public class CommentService {
    }
    ```

    ```java
    // 配置类
    @Configuration
    public class ProjectConfig {
    
        @Bean
        @Scope(BeanDefinition.SCOPE_PROTOTYPE)  // 确定作用域
        public CommentService commentService() {
            return new CommentService();
        }
    }
    ```

    ```java
    public class Main {
        public static void main(String[] args) {
            AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(ProjectConfig.class);
            CommentService cs1 = context.getBean(CommentService.class);
            CommentService cs2 = context.getBean(CommentService.class);
            System.out.println(cs1 == cs2);  // false
        }
    }
    ```

    

  * 例2：使用模板注解定义Bean

    ```java
    // Repository类
    @Repository
    @Scope(BeanDefinition.SCOPE_PROTOTYPE)
    public class CommentRepository {
    }
    ```

    ```java
    // Service类
    @Service
    public class CommentService {
        // 字段注入
        @Autowired
        private CommentRepository commentRepository;
    
        public CommentRepository getCommentRepository() {
            return commentRepository;
        }
    }
    ```

    ```java
    // Service类
    @Service
    public class UserService {
        // 字段注入
        @Autowired
        private CommentRepository commentRepository;
    
        public CommentRepository getCommentRepository() {
            return commentRepository;
        }
    }
    ```

    ```java
    public class Main {
        public static void main(String[] args) {
            AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(ProjectConfig.class);
            CommentService commentService = context.getBean(CommentService.class);
            UserService userService = context.getBean(UserService.class);
            // 结果为false
            // 说明每次注入的都是一个新bean
            System.out.println(userService.getCommentRepository() == commentService.getCommentRepository());
        }
    }
    ```

* 什么时候要将Bean的作用域定义为Prototype？

  示例：

  假设当前有一个CommentProcessor类，负责处理Comment和验证Comment。CommentService类需要调用CommentProcessor实例来对Comment进行处理。

  ```java
  // CommentProcessor类
  public class CommentProcessor {
      private Comment comment;
  
      public Comment getComment() {
          return comment;
      }
  
      public void setComment(Comment comment) {
          this.comment = comment;
      }
  
      public void processComment() {
          // changing the comment attribute
      }
  
      public void validateComment() {
          // validating and changing the comment attribute
      }
  }
  ```

  ```java
  @Service
  public class CommentService {
      public void sendComment(Comment c) {
          CommentProcessor processor = new CommentProcessor();
          processor.setComment(c);
          // Uses the CommentProcessor instance to 
          // alter the Comment instance
          processor.processComment();
          processor.validateComment();
      }
  }
  ```

  这里并没有将CommentProcessor类设置为bean，因为CommentProcessor这个类本身*并不需要使用 Spring 提供的任何功能*，因此CommentProcessor不需要作为bean加入Spring容器。

  假如说，现在场景变成这样：

  CommentRepository是一个bean，而CommentProcessor需要调用这个bean来实现对Comment的持久化。

  ![image-20251107174001791](asset/image-20251107174001791.png)

  而在CommentProcessor中获取CommentRepository bean最简单的方式，就是通过Spring的依赖注入。这里CommentProcessor要用到Spring的功能，因此也将其作为bean，加入到Spring容器中。

  这里的CommentProcessor作用域为prototype，因为CommentService需要调用CommentProcessor实例的方法来对comment对象进行处理，如果CommentProcessor是单例的话，则CommentService每次调用sendComment方法的时候，对comment进行处理的都是同一个CommentProcessor实例。这样在多线程的场景下，会存在竞态条件。

  所以，需要保证CommentService每一次调用sendComment方法的时候，处理comment的CommentProcessor实例都是不同的，以避免出现竞态条件。

  ```java
  @Component
  @Scope(BeanDefinition.SCOPE_PROTOTYPE)
  public class CommentProcessor {
      private Comment comment;
  
      @Autowired
      private CommentRepository commentRepository;
  
      public Comment getComment() {
          return comment;
      }
  
      public void setComment(Comment comment) {
          this.comment = comment;
      }
  
      public void processComment() {
          // changing the comment attribute
      }
  
      public void validateComment() {
          // validating and changing the comment attribute
      }
  }
  ```

  那我们的CommentService应该怎么改来注入CommentProcessor呢？

  下面这样吗？

  CommentService是一个单例对象，如果按照下面这样写法，则只会在创建CommentService单例对象的时候，去创建CommentProcessor对象，然后进行注入，后续使用的都是同一个。这和将CommentProcessor设为singleton作用域没有任何区别。

  ```java
  @Service
  public class CommentService {
      // Spring injects this bean when creating
      // the CommentService bean. But because
      // CommentService is singleton, Spring
      // will also create and inject the
      // CommentProcessor just once.
      @Autowired
      private CommentProcessor processor;
      public void sendComment(Comment c) {
          processor.setComment(c);
          // Uses the CommentProcessor instance to alter the Comment instance
          processor.processComment();
          processor.validateComment();
      }
  }
  ```

  正确的做法是注入context对象，然后每一次执行sendComment方法的时候，都context.getBean来获取一个新的CommentProcessor bean，这样就能保证每次执行sendComment方法的时候的bean都是独一份，线程安全的。

  ```java
  @Service
  public class CommentService {
      @Autowired
      private ApplicationContext context;
      public void sendComment(Comment c) {
          CommentProcessor processor = context.getBean(CommentProcessor.class);
          processor.setComment(c);
          // Uses the CommentProcessor instance to alter the Comment instance
          processor.processComment();
          processor.validateComment();
      }
  }
  ```



### 5.1.3 总结

什么时候要用Singleton什么时候用Prototype？

一般来说，如果这个bean是不可变的，则用Singleton；如果这个bean是可变的，则使用prototype来保证线程安全，避免出现竞态条件。

一般会把bean设计为不可变的，如果设计为可变的，需要额外考虑线程安全的问题。



# 6. AOP

## 6.1 AOP是什么

AOP面向切面编程指的是，拦截调用的方法，并且改变方法的执行。

通过AOP，我们可以将一些和业务逻辑无关的代码抽取出来，将其单独配置，使其能够和业务代码逻辑解耦，提高代码的可读性和可维护性。

例如，我们每一个方法中都需要写日志，但是日志和业务逻辑并没有关系，通过切面，我们可以将日志的部分单独提取出来，在其它的地方编写配置，而业务方法中，只保留业务逻辑的代码，以提高代码的可读性和可维护性。

![image-20251108102730334](asset/image-20251108102730334.png)

## 6.2 AOP概念

* aspect: *what* code you want Spring to execute when you call specific methods. This is named an aspect.

* advice: *When* the app should execute this logic of the aspect (e.g., before or after the method call, instead of the method call). This is named the advice.

* pointcut: *Which* methods the framework needs to intercept and execute the aspect for them. This is named a pointcut.

* join point: which defines the event that triggers the execution of an aspect. But with Spring, this event is always a method call.

* target object: The bean that declares the method intercepted by an aspect is named the target object.

* proxy project: if you made the object an aspect target object, Spring won’t directly give you an instance reference for the bean when you request it from the context. Instead, Spring gives you an object that calls the aspect logic instead of the actual method. We say that Spring gives you a proxy object instead of the real bean. You will now receive the proxy instead of the bean anytime you get the bean from the context, either if you directly use the getBean() method of the context or if you use DI. This approach is named weaving.

* joint point和point cut的区别：

  * Joinpoints 指的是所有可以进行拦截的点，而point cut是你根据业务需求具体指定的要进行拦截的点。

  * 例如你写了很多方法，这里的所有方法调用都是连接点

    ```java
    public class UserService {
        public void addUser() {}
        public void deleteUser() {}
        public void updateUser() {}
    }
    ```

    如果你这样定义切点表达式，则只有addUser这一个方法调用是切点，表示只对以add开头的方法进行拦截。

    ```java
    @Pointcut("execution(* com.example.UserService.add*(..))")
    public void userAddOperation() {}
    ```



ps：如果我们要对一个对象进行AOP编程，则说明这个对象需要用到Spring提供的功能，则这个对象要被作为bean加入到Spring容器中，Spring才能够管理这个bean，这个bean从而才能使用Spring提供的AOP功能。

![image-20251108110559733](asset/image-20251108110559733.png)

## 6.3 用 vs 不用AOP

不用AOP，我们调用一个对象的方法，是直接调用这个方法本身。

使用AOP，调用一个对象的方法，实际上Spring会调用这个目标对象的proxy对象增强的方法，这个proxy对象的增强方法中，一方面包含增强/切面逻辑，另一方面会再去调用目标对象的方法来完成具体的业务逻辑。

![image-20251108110659017](asset/image-20251108110659017.png)

## 6.4 AOP实践

当前的项目是，这个Spring app提供一个发布评论的功能。

先搭建一个项目的架子，然后在此基础上，增加AOP功能。

* 项目代码

  项目结构：

  ![image-20251109165926997](asset/image-20251109165926997.png)

  

  ```xml
  <!--pom依赖-->
  <dependencies>
      <dependency>
          <groupId>org.springframework</groupId>
          <artifactId>spring-context</artifactId>
          <version>5.2.8.RELEASE</version>
      </dependency>
  </dependencies>
  ```

  ```java
  // Comment类
  package com.tudou.pojo;
  
  public class Comment {
      private String text;
      private String author;
  
      // Omitted getters and setters
  }
  ```

  ```java
  // Service类
  package com.tudou.service;
  
  @Service
  public class CommentService {
      //  When declaring a logger object, you need to give it a name as a parameter.
      //  This name then appears in the logs and makes it easy for you to
      //  observe the log message source.
      private Logger logger = Logger.getLogger(CommentService.class.getName());
      public void publishComment(Comment comment) {
          logger.info("Publishing comment:" + comment.getText());
      }
  }
  ```

  ```java
  // 配置类
  package com.tudou.config;
  
  @Configuration
  @ComponentScan(basePackages = "com.tudou")
  public class ProjectConfig {
  }
  ```

  ```java
  // main函数
  public class Main {
      public static void main(String[] args) {
          var c = new AnnotationConfigApplicationContext(ProjectConfig.class);
          var service = c.getBean(CommentService.class);
          Comment comment = new Comment();
          comment.setText("Demo comment");
          comment.setAuthor("Natasha");
          service.publishComment(comment);
      }
  }
  
  // 执行结果
  11月 09, 2025 5:24:49 下午 com.tudou.service.CommentService publishComment
  信息: Publishing comment:Demo comment
  ```



现在有一个新增的需求，需要统计所有方法的执行时长，下面介绍如何用AOP实现。

增加AOP功能，有以下4步：

项目结构

![image-20251109165509448](asset/image-20251109165509448.png)

1. 增加AOP依赖

   ```xml
   <dependency>
       <groupId>org.springframework</groupId>
       <artifactId>spring-aspects</artifactId>
       <version>5.2.8.RELEASE</version>
   </dependency>
   ```

2. 在配置类增加 @EnableAspectJAutoProxy注解来开启AOP功能

   ```java
   @Configuration
   @ComponentScan(basePackages = "com.tudou")
   // Enables the aspects mechanism in our Spring app
   @EnableAspectJAutoProxy
   public class ProjectConfig {
   }
   ```

3. 创建一个切面类，增加@Aspect注解，会在该类中定义AOP相关配置

   ```java
   package com.tudou.aspect;
   
   @Aspect
   public class LoggingAspect {
   }
   ```

   ```java
   package com.tudou.aspect;
   
   @Aspect
   public class LoggingAspect {
       private Logger logger = Logger.getLogger(LoggingAspect.class.getName());
   	
       
       // 环绕通知 Defines which are the intercepted methods
       @Around("execution(* com.tudou.service.*.*(..))")
       // the ProceedingJoinPoint parameter, 
       // which represents the intercepted method. 
       // The main thing you do with this parameter is tell the aspect 
       // when it should delegate further to the actual method.
       public void log(ProceedingJoinPoint joinPoint) throws Throwable {
           logger.info("Method will execute");
           // 调用实际被拦截的方法
           // proceed() method throws a Throwable. 
           // The method proceed() is designed to 
           // throw any exception coming from the intercepted method. 
           joinPoint.proceed();
           logger.info("Method executed");
       }
   }
   ```

4. 创建一个LoggingAspect类的Bean

   ```java
   package com.tudou.config;
   
   @Configuration
   @ComponentScan(basePackages = "com.tudou")
   // Enables the aspects mechanism in our Spring app
   @EnableAspectJAutoProxy
   public class ProjectConfig {
       @Bean
       public LoggingAspect aspect() {
           return new LoggingAspect();
       }
   }
   ```

​	

最终执行结果：

```java
// 增强逻辑
11月 09, 2025 5:25:39 下午 com.tudou.aspect.LoggingAspect log
信息: Method will execute
// 调用目标对象方法
11月 09, 2025 5:25:39 下午 com.tudou.service.CommentService publishComment
信息: Publishing comment:Demo comment
// 增强逻辑
11月 09, 2025 5:25:39 下午 com.tudou.aspect.LoggingAspect log
信息: Method executed
```



==注意：==

* 一定要手动创建一个切面类的Bean。

  因为@Aspect这个注解并不会创建对应Bean，这个注解只是用来告诉Spring，这个类中定义了AOP相关配置。

* *execution(\* services.*.*(..))* 这是用于定义切点的表达式

  * 含义

    ![image-20251109171836822](asset/image-20251109171836822.png)

  * 不需要死记硬背，如果要写的时候，直接查询文档即可http://mng.bz/4K9g


## 6.5 获取/修改 拦截的方法的参数和返回值

**获取被拦截的方法的参数和返回值**

```java
// main方法
public class Main {
    private static Logger logger = Logger.getLogger(Main.class.getName());
    public static void main(String[] args) {
        var c = new AnnotationConfigApplicationContext(ProjectConfig.class);
        var service = c.getBean(CommentService.class);
        Comment comment = new Comment();
        comment.setText("Demo comment");
        comment.setAuthor("Natasha");
        String value = service.publishComment(comment);
        logger.info(value);
    }
}
```

```java
// 被拦截的方法
@Service
public class CommentService {
    private Logger logger = Logger.getLogger(CommentService.class.getName());
    public String publishComment(Comment comment) {
        logger.info("Publishing comment:" + comment.getText());
        return "SUCCESS";
    }
}
```

```java
// 切面类
@Aspect
public class LoggingAspect {
    private Logger logger = Logger.getLogger(LoggingAspect.class.getName());

    @Around("execution(* com.tudou.service.*.*(..))")
    public Object log(ProceedingJoinPoint joinPoint) throws Throwable {
        // Obtains the name and parameters of the intercepted method
        String methodName = joinPoint.getSignature().getName();
        Object[] args = joinPoint.getArgs();
        logger.info("Method " + methodName + "with parameters " + Arrays.asList(args) + " will execute");

        // Calls the intercepted method
        Object returnedByMethod = joinPoint.proceed();
        logger.info("Method executed and returned " + returnedByMethod);

        // Returns the value returned by the intercepted method
        return returnedByMethod;
    }
}
```

```java
// 执行结果
11月 10, 2025 11:10:31 上午 com.tudou.aspect.LoggingAspect log
信息: Method publishCommentwith parameters [Comment{text='Demo comment', author='Natasha'}] will execute
11月 10, 2025 11:10:31 上午 com.tudou.service.CommentService publishComment
信息: Publishing comment:Demo comment
11月 10, 2025 11:10:31 上午 com.tudou.aspect.LoggingAspect log
信息: Method executed and returned SUCCESS
11月 10, 2025 11:10:31 上午 com.tudou.main.Main main
信息: SUCCESS
```



**获取被拦截的方法的参数和返回值**

和上面的例子只有切面类有区别

```java
// 切面类
@Aspect
public class LoggingAspect {
    private Logger logger = Logger.getLogger(LoggingAspect.class.getName());

    // Defines which are the intercepted methods
    @Around("execution(* com.tudou.service.*.*(..))")
    public Object log(ProceedingJoinPoint joinPoint) throws Throwable {
        // Obtains the name and parameters of the intercepted method
        String methodName = joinPoint.getSignature().getName();
        Object[] args = joinPoint.getArgs();
        logger.info("Method " + methodName + "with parameters " + Arrays.asList(args) + " will execute");

        // Send a different comment instance as a value to the method’s parameter.
        Comment comment = new Comment();
        comment.setText("Some other text!");
        Object[] newArgs = {comment};

        // Calls the intercepted method
        Object returnedByMethod = joinPoint.proceed(newArgs);
        logger.info("Method executed and returned " + returnedByMethod);

        // Return a different value to the caller.
        return "FAILED";
    }
}
```

```java
// 执行结果
11月 10, 2025 10:53:59 上午 com.tudou.aspect.LoggingAspect log
信息: Method publishCommentwith parameters [Comment{text='Demo comment', author='Natasha'}] will execute
11月 10, 2025 10:53:59 上午 com.tudou.service.CommentService publishComment
信息: Publishing comment:Some other text!
11月 10, 2025 10:53:59 上午 com.tudou.aspect.LoggingAspect log
信息: Method executed and returned SUCCESS
11月 10, 2025 10:53:59 上午 com.tudou.main.Main main
信息: FAILED
```

## 6.6 通过注解指定需要拦截的方法

1. 定义一个自定义注解

   ```java
   // Enables the annotation to be intercepted at runtime
   @Retention(RetentionPolicy.RUNTIME)
   // Restricts this annotation to only be used with methods
   @Target(ElementType.METHOD)
   public @interface ToLog {
   }
   ```

2. 在需要拦截的方法上增加这个注解

   ```java
   package com.tudou.service;
   
   @Service
   public class CommentService {
       private Logger logger = Logger.getLogger(CommentService.class.getName());
       public void publishComment(Comment comment) {
           logger.info("Publishing comment:" + comment.getText());
       }
   	
       // 需要拦截的方法
       @ToLog
       public void deleteComment(Comment comment) {
           logger.info("Deleting comment:" + comment.getText());
       }
   
       public void editComment(Comment comment) {
           logger.info("Editing comment:" + comment.getText());
       }
   }
   ```

3. 定义Aspect表达式

   ```java
   package com.tudou.aspect;
   
   @Aspect
   public class LoggingAspect {
       private Logger logger = Logger.getLogger(LoggingAspect.class.getName());
   
       // 这里的Aspect表达式代表包含ToLog注解的方法
       @Around("@annotation(com.tudou.annotation.ToLog))")
       public Object log(ProceedingJoinPoint joinPoint) throws Throwable {
           // Obtains the name and parameters of the intercepted method
           String methodName = joinPoint.getSignature().getName();
           Object[] args = joinPoint.getArgs();
           logger.info("Method " + methodName + "with parameters " + Arrays.asList(args) + " will execute");
   
           // Calls the intercepted method
           Object returnedByMethod = joinPoint.proceed();
           logger.info("Method executed and returned " + returnedByMethod);
   
           return returnedByMethod;
       }
   }
   
   ```

4. main方法

   ```java
   public class Main {
       private static Logger logger = Logger.getLogger(Main.class.getName());
       public static void main(String[] args) {
           var c = new AnnotationConfigApplicationContext(ProjectConfig.class);
           var service = c.getBean(CommentService.class);
           Comment comment = new Comment();
           comment.setText("Demo comment");
           comment.setAuthor("Natasha");
           service.publishComment(comment);
           service.deleteComment(comment);
           service.editComment(comment);
       }
   }
   ```

   ```java
   // 执行结果
   // publishComment方法执行
   11月 10, 2025 12:12:15 下午 com.tudou.service.CommentService publishComment
   信息: Publishing comment:Demo comment
   // deleteComment方法执行 被增强
   11月 10, 2025 12:12:15 下午 com.tudou.aspect.LoggingAspect log
   信息: Method deleteCommentwith parameters [Comment{text='Demo comment', author='Natasha'}] will execute
   11月 10, 2025 12:12:15 下午 com.tudou.service.CommentService deleteComment
   信息: Deleting comment:Demo comment
   11月 10, 2025 12:12:15 下午 com.tudou.aspect.LoggingAspect log
   信息: Method executed and returned null
   // editComment方法执行
   11月 10, 2025 12:12:15 下午 com.tudou.service.CommentService editComment
   信息: Editing comment:Demo comment
   ```

   

==注意：==

* `@Retention(RetentionPolicy.RUNTIME)`这是什么意思？

  * @Retention是用于修饰注解的注解，也就是说，通过@Retention，我们可以对@ToLog注解进行配置。

    `RetentionPolicy.RUNTIME`就是对@ToLog注解的保留策略进行配置。

  * RetentionPolicy代表注解的保留策略

    | RetentionPolicy 值 | 生命周期阶段                             | 能否在运行时通过反射读取？ | 常见用途                                                     |
    | ------------------ | ---------------------------------------- | -------------------------- | ------------------------------------------------------------ |
    | **SOURCE**         | 只存在于源码中，编译后就丢弃             | ❌ 否                       | 编译检查（例如 `@Override`）                                 |
    | **CLASS**          | 编译进 `.class` 文件，但运行时被丢弃     | ❌ 否                       | 框架很少直接用（默认值）                                     |
    | **RUNTIME**        | 编译进 `.class` 文件，并在运行时依然存在 | ✅ 是                       | 需要在运行时通过反射/AOP 处理的注解，如 `@Autowired`、`@Transactional`、`@RequestMapping` |

    简单来说，`RetentionPolicy.RUNTIME`表示运行时注解依然保留，而其他两种保留策略运行时不会保留注解。

    而AOP通过注解实现，是Spring在运行时通过反射获取方法的注解，然后对持有@ToLog注解的方法进行拦截。

    只有`RetentionPolicy.RUNTIME`这种保留策略才能保证Spring运行时还能够看到方法的@ToLog注解，以对对应的方法进行增强。

    总结一句话：

    `@Retention(RetentionPolicy.RUNTIME)`：告诉 JVM“请在运行时也保留这个注解的信息，让反射或 AOP 能看见它”。

* `@Around("@annotation(com.tudou.annotation.ToLog))")`这里的注解名称一定要写全限定名，如果写成`@Around("@annotation(ToLog))")`，Spring会报下面的异常

  11月 10, 2025 12:14:17 下午 org.springframework.context.support.AbstractApplicationContext refresh
  警告: Exception encountered during context initialization - cancelling refresh attempt: org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'projectConfig': Initialization of bean failed; nested exception is java.lang.IllegalArgumentException: **error Type referred to is not an annotation type: ToLog**

## 6.7 通知注解

* @Around：在目标方法执行前/后都可以进行增强
* @Before：在目标方法执行前进行增强
* @AfterReturning：calls the method defining the aspect logic after the method successfully returns, and provides the returned value as a parameter to the aspect method. *The aspect method isn’t called if the intercepted method throws an exception.*
* @AfterThrowing：Calls the method defining the aspect logic if the intercepted
  method throws an exception, and provides the exception instance as a parame-
  ter to the aspect method.
* @After：Calls the method defining the aspect logic only after the intercepted method execution, whether the method successfully returned or threw an exception.



注意：

* 只有@Around注解的增强方法拥有ProceedingJoinPoint参数，用于决定何时调用目标方法。拥有其他通知注解的方法没有该参数，因为根据它们的语义，它们无法控制目标方法什么时候被调用。



示例：

```java
// @AfterReturning通知注解的使用示例
@Aspect
public class LoggingAspect {
    private Logger logger = Logger.getLogger(LoggingAspect.class.getName());

    // returning 属性指定log方法声明中用于接收目标方法执行结果的参数的参数名
    @AfterReturning(value = "@annotation(com.tudou.annotation.ToLog)", returning = "returnedValue")
    public void log(Object returnedValue) {
        logger.info("Method executed and returned " + returnedValue);
    }
}
```

```java
// 执行结果
// publishComment方法执行
11月 10, 2025 4:43:59 下午 com.tudou.service.CommentService publishComment
信息: Publishing comment:Demo comment
// deleteComment方法执行
11月 10, 2025 4:43:59 下午 com.tudou.service.CommentService deleteComment
信息: Deleting comment:Demo comment
// deleteComment方法被增强
11月 10, 2025 4:43:59 下午 com.tudou.aspect.LoggingAspect log
信息: Method executed and returned null
// editComment方法执行
11月 10, 2025 4:43:59 下午 com.tudou.service.CommentService editComment
信息: Editing comment:Demo comment
```



## 6.8 切面执行链

### 6.8.1 切面执行顺序

如果有多个切面，那这多个切面的执行顺序是怎样的？

假设当前app需要实现两个切面：

* SecurityAspect：对所有的请求先进行权限校验，当通过校验才放行执行业务逻辑
* LoggingAspect：对所有的请求进行拦截，对请求的开始和结束都需要打日志

如果我们指定先执行SecurityAspect切面，则对于那些权限校验没有通过的请求，不会再把请求委托给LoggingAspect切面，则这部分请求不会被打日志，这并不符合我们的要求。我们对LoggingAspect的要求是，对于所有请求都要进行打日志。

所以，我们必须要指定先执行LoggingAspect切面，这会对所有请求都打日志，之后再将请求委托给SecurityAspect切面继续执行，如果权限校验没有通过，则不会调用业务逻辑代码。

![image-20251110200403608](asset/image-20251110200403608.png)

### 6.8.2 如何指定切面执行的先后顺序

* 默认场景下，如果不进行显式指定，切面的执行顺序是随机的，Spring doesn’t guarantee the order in which two aspects in the same execution chain are called. 如果业务中对切面的顺序没有要求，则无需显式指定顺序
* 如果要显式指定顺序，则可以在切面类上使用@Order注解，接受一个序号数为参数，值越小，优先级越高。

示例：

```java
// LoggingAspect切面类
package com.tudou.aspect;

@Aspect
@Order(1)  // 指定顺序
public class LoggingAspect {
    private Logger logger = Logger.getLogger(LoggingAspect.class.getName());

    @Around(value = "@annotation(com.tudou.annotation.ToLog)")
    public Object log(ProceedingJoinPoint joinPoint) throws Throwable {
        logger.info("Logging Aspect: Calling the intercepted method");
        // The proceed() method here delegates further in the aspect execution chain. \
        // It can either call the next aspect or the intercepted method.
        Object returnedValue = joinPoint.proceed();
        logger.info("Logging Aspect: Method executed and returned " + returnedValue);
        return returnedValue;
    }
}
```

```java
// SecurityAspect切面类
package com.tudou.aspect;

@Aspect
@Order(2)  // 指定顺序
public class SecurityAspect {
    private Logger logger = Logger.getLogger(SecurityAspect.class.getName());

    @Around(value = "@annotation(com.tudou.annotation.ToLog)")
    public Object secure(ProceedingJoinPoint joinPoint) throws Throwable {
        logger.info("Security Aspect: Calling the intercepted method");
        Object returnedValue = joinPoint.proceed();
        logger.info("Security Aspect: Method executed and returned " + returnedValue);
        return returnedValue;
    }
}

```

```java
// 被增强的Service类
@Service
public class CommentService {
    private Logger logger = Logger.getLogger(CommentService.class.getName());
    @ToLog
    public String publishComment(Comment comment) {
        logger.info("Publishing comment:" + comment.getText());
        return "SUCCESS";
    }
}
```

```java
// main方法
public class Main {
    private static Logger logger = Logger.getLogger(Main.class.getName());
    public static void main(String[] args) {
        var c = new AnnotationConfigApplicationContext(ProjectConfig.class);
        var service = c.getBean(CommentService.class);
        Comment comment = new Comment();
        comment.setText("Demo comment");
        comment.setAuthor("Natasha");
        service.publishComment(comment);
    }
}
```

```java
// LoggingAspect切面逻辑优先执行
11月 10, 2025 8:12:52 下午 com.tudou.aspect.LoggingAspect log
信息: Logging Aspect: Calling the intercepted method
// SecurityAspect切面逻辑第二执行
11月 10, 2025 8:12:52 下午 com.tudou.aspect.SecurityAspect secure
信息: Security Aspect: Calling the intercepted method
// 调用目标方法
11月 10, 2025 8:12:52 下午 com.tudou.service.CommentService publishComment
信息: Publishing comment:Demo comment
// 目标方法结果返回给SecurityAspect
11月 10, 2025 8:12:52 下午 com.tudou.aspect.SecurityAspect secure
信息: Security Aspect: Method executed and returned SUCCESS
// SecurityAspect结果返回给LoggingAspect
11月 10, 2025 8:12:52 下午 com.tudou.aspect.LoggingAspect log
信息: Logging Aspect: Method executed and returned SUCCESS
```

![image-20251110201229238](asset/image-20251110201229238.png)



==注意：==

* 如果使用`@Around`环绕通知，方法可以接受一个`ProceedingJoinPoint`类型的参数（假设参数名为`joinPoint`），这里`joinPoint.proceed();`并不完全代表调用目标方法，而是代表调用执行链的下一个方法，可以是下一个切面，可以是目标方法。

  假设有两个切面`LoggingAspect`（order=1）、`SecurityAspect`（order=2），且都是用环绕通知。我们首先调用`LoggingAspect`，第二调用`SecurityAspect`切面，则在`LoggingAspect`切面的增强方法中调用`joinPoint.proceed();`是表示调用`SecurityAspect`切面的增强方法。在`SecurityAspect`切面中的增强方法中调用`joinPoint.proceed();`是表示调用目标方法。简而言之，就是表示调用执行链的下一个方法。

# 7. SpringBoot & Spring MVC

## 7.1 web app是什么

**是什么：**

通过浏览器访问的应用就是web app。

以前，人们使用各种应用都需要下载对应的软件，现在，通过浏览器就可以看youtube、听音乐、玩游戏，不需要再下载软件，非常方便，这些通过浏览器就可以访问的应用就是web app。

![image-20251123171602698](asset/image-20251123171602698.png)



**big picture of a web app:**

![image-20251123173712578](asset/image-20251123173712578.png)

注意：the backend serves multiple clients concurrently，因此后端需要考虑竞态条件问题。

![image-20251123173846576](asset/image-20251123173846576.png)

## 7.2 前后端不分离 vs 前后端分离

实现一个web app有多种方式：

* 前后端不分离：浏览器获取到后端传来的数据直接展示给用户。

  后端会返回浏览器能够展示的数据类型，例如HTML、CSS、JS...

  ![image-20251123174734793](asset/image-20251123174734793.png)

* 前后端分离：后端只提供要展示的raw data，前端会对raw data进行处理、渲染，浏览器会将前端处理后的数据展示给用户。

  ![image-20251123175110370](asset/image-20251123175110370.png)

## 7.3 Servlet

**servlet container是什么：**

一句话解释：servlet container(e.g., Tomcat): software with the capability to translate HTTP requests and responses to the Java app. 

在web-app中，有client和server两个角色，client和server通过HTTP协议进行通信。

那我们在开发web-app应用的时候，需要考虑如何将HTTP协议请求、响应转化为Java应用能够理解的对象。

servlet container可以理解为一个翻译器，用以将HTTP messages翻译为Java应用能够理解的对象。这样的话，我们的web-app本身不用再关心如何进行HTTP messages和Java对象的转换、翻译，而只需要关注我们业务逻辑本身。

Tomcat是One of the most appreciated servlet container，除了Tomcat之外，还有其他替代品例如Jetty、JBoss、Payara等。

![image-20251124204610715](asset/image-20251124204610715.png)

自己的话：tomcat是servlet container的一种，它的作用就是将http请求翻译成Java对象，让web app开发者不用关注如何处理http格式的请求。将响应内容翻译成http响应，开发者也无需关注如何将请求封装成http格式的响应。简单来说，他就是做了一层抽象，让web app开发者无需关注网络传输协议是什么，而只需要关注实现自己的业务逻辑，其中具体的转化，交给tomcat来实现。除此之外，servlet container还能够管理servlet，一个servlet container中包含多个servlet，servlet container管理这些servlet，并在请求到达的时候，确定调用哪一个servlet的方法。

**servlet是什么：**

servlet是一个Java对象，能够直接和servlet container交互。当servlet container接收到一个HTTP请求，它会调用某一个servlet对象的方法，并将请求作为方法的参数request传入。该方法也有一个响应对象参数，开发者通过这个response对象可以设置返回给client的内容。

浏览器 → HTTP 请求 → **Tomcat（Servlet 容器）**
 Tomcat: 解析请求 → 调用 Servlet 方法 → 传入 req & resp
 Servlet：读取 req → 处理逻辑 → 写入 resp
 Tomcat：发送 HTTP 响应 → 浏览器



**当servlet container接受到一个请求，它如何决定调用哪一个servlet对象的方法？**

当我们创建一个servlet的时候，需要将这个servlet和一个路径绑定在一起。当servlet接受到一个请求的时候，会根据请求的路径，找到对应的servlet，调用对应的servlet对象的方法。The servlet contained the logic associated with the user’s request and the ability to prepare a response. servlet is the entry point to your app’s logic. 

![image-20251124211210599](asset/image-20251124211210599.png)

## 7.4 SpringBoot

Spring Boot 是 Spring 的快速开发脚手架，目的是让你无需繁琐配置就能快速构建 Spring 应用。

SpringBoot提供了很多功能，使得我们创建web app的过程能够大大简化，让开发者能够做到最大程度只需关注业务逻辑开发。

Spring Boot提供的主要功能：

* Simplified project creation—You can use a project initialization service to get an empty but configured skeleton app.
* Dependency starters—Spring Boot groups certain dependencies used for a specific purpose with dependency starters. You don’t need to figure out all the must-have dependencies you need to add to your project for one particular purpose nor which versions you should use for compatibility.
* Autoconfiguration based on dependencies—Based on the dependencies you added to your project, Spring Boot defines some default configurations. Instead of writing all the configurations yourself, you only need to change the ones provided by Spring Boot that don’t match what you need. Changing the configs likely requires less code (if any).

### 7.4.1 如何创建一个SpringBoot项目

有两种方式：

1. 有一些ide中，集成了SpringBoot project initializer，通过这个initializer就可以直接创建SpringBoot项目。

   ![image-20251125102707825](asset/image-20251125102707825.png)

2. 如果ide中没有集成SpringBoot project initializer，可以通过`http://start.spring.io`网站来创建SpringBoot初始项目。

   ![image-20251125103110706](asset/image-20251125103110706.png)

   点击generate之后，会得到Spring Boot project的压缩文件，解压之后就可以将其导入ide中。

### 7.4.2 新SpringBoot项目的构成

![image-20251125104908841](asset/image-20251125104908841.png)

* The Spring app main class

  SpringBoot项目的main class是拥有@SpringBootApplication注解的class

  ```java
  @SpringBootApplication  // This annotation defines the Main class of a Spring Boot app.
  public class Main {
  
      public static void main(String[] args) {
          SpringApplication.run(Main.class, args);
      }
  
  }
  ```

* The Spring Boot POM.xml

  下面的POM文件是新建的SpringBoot项目默认的pom文件，可以看到SpringBoot已经根据我们在Spring initializer勾选的信息帮我们做了很多配置工作，让开发者可以专注于业务逻辑开发。

  * 通过BOM (Bill Of Materials)，SpringBoot提供的版本管理清单。即SpringBoot会提供所有依赖的版本，开发者无需再手动指定版本号，以保证依赖之间版本的可兼容性。【书中的图是pom parent，而新版本的Spring中已经使用BOM替代原方案，因此这里的介绍是基于BOM】
  * 插件
  * provided dependency.这是因为在创建SpringBoot项目的时候，选中了Spring-web依赖，则SpringBoot会自动帮你将web应用需要使用到的依赖导入到SpringBoot中。

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
      <!--基本项目信息-->
      <modelVersion>4.0.0</modelVersion>
      <groupId>com.tudou</groupId>  <!--公司/组织的反向域名-->
      <artifactId>sq-ch7-ex1</artifactId>  <!--项目名称-->
      <version>0.0.1-SNAPSHOT</version>  <!--版本号-->
      <name>sq-ch7-ex1</name>
      <description>sq-ch7-ex1</description>
      <!--项目配置 & 全局变量-->
      <properties>
          <java.version>11</java.version>
          <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
          <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
          <spring-boot.version>2.6.13</spring-boot.version>
      </properties>
      <!--项目真正用到的依赖-->
      <dependencies>
          <dependency>
              <groupId>org.springframework.boot</groupId>
              <artifactId>spring-boot-starter-web</artifactId>
          </dependency>
  
          <dependency>
              <groupId>org.springframework.boot</groupId>
              <artifactId>spring-boot-starter-test</artifactId>
              <scope>test</scope> <!--表示仅用于测试，不会打包进入最终的jar-->
          </dependency>
      </dependencies>
      <!--依赖版本统一管理-->
      <!--dependencyManagement不会引入任何依赖，只负责告诉Maven:如果你的依赖列表有某个库，你又没写版本号，就按照我提供的这个版本来。-->
      <dependencyManagement>
          <dependencies>
              <!--BOM-->
              <!--org.springframework.boot:spring-boot-dependencies内部为SpringBoot生态中的所有依赖统一指定了版本号。则开发者无需自己指定版本号，保证所有依赖之间的版本能够兼容。-->
              <dependency>
                  <groupId>org.springframework.boot</groupId>
                  <artifactId>spring-boot-dependencies</artifactId>
                  <version>${spring-boot.version}</version>
                  <type>pom</type>
                  <scope>import</scope>
              </dependency>
          </dependencies>
      </dependencyManagement>
      <!--插件-->
      <build>
          <plugins>
              <plugin>
                  <groupId>org.apache.maven.plugins</groupId>
                  <artifactId>maven-compiler-plugin</artifactId>  <!--用于编译Java-->
                  <version>3.8.1</version>
                  <configuration>
                      <source>11</source>
                      <target>11</target>
                      <encoding>UTF-8</encoding>
                  </configuration>
              </plugin>
              <plugin>
                  <groupId>org.springframework.boot</groupId>
                  <artifactId>spring-boot-maven-plugin</artifactId>  <!--用于将SpringBoot项目打包成可执行jar-->
                  <version>${spring-boot.version}</version>
                  <configuration>
                      <mainClass>com.tudou.SqCh7Ex1Application</mainClass>
                      <skip>true</skip>
                  </configuration>
                  <executions>
                      <execution>
                          <id>repackage</id>
                          <goals>
                              <goal>repackage</goal>
                          </goals>
                      </execution>
                  </executions>
              </plugin>
          </plugins>
      </build>
  
  </project>
  ```

* The properties file: application.properties文件在resources目录下，该文件为配置文件。

### 7.4.3 dependency starter

* dependency starter是一个依赖，这个依赖中包含了你要实现某个目的所涉及的一组依赖。例如如果你的app中需要实现web功能，以前，你需要自己手动添加web功能所涉及的所有依赖，并且保证这些所有依赖版本的兼容性。如果我们导入spring-boot-starter-web依赖，这个依赖内部包含了所有要实现web功能所涉及的依赖，并且保证了这些所有依赖的版本的可兼容性。大大减少开发者的工作量。

  ![image-20251125113933247](asset/image-20251125113933247.png)

* starter通常被命名为spring-boot-starter-xxx，例如spring-boot-starter-web

### 7.4.4 convention-over-configuration principle

一些约定俗成的配置，会被SpringBoot设为默认配置，开发者无需再手动配置。只有当自定义的和默认配置不同的，才需要显式配置。



例如，如果我们增加了`spring-boot-starter-web`依赖，启动项目，可以看到这一行在console中。

我们没有进行任何配置，就启动了tomcat容器，并且是在8080端口启动。

![image-20251125115741134](asset/image-20251125115741134.png)

这是因为，这是web开发中最常用的配置，SpringBoot会帮我们按照最常用的配置设置默认配置。这样，我们只需要针对那些需要个性化配置的配置进行配置，其他的使用默认配置即可。这样大大减少了我们需要编写的配置代码。

## 7.5 SpringMVC

### 7.5.1 SpringMVC开发示例

1. 在resources/static增加一个home.html页面【 This folder is the default place where the Spring Boot app expects to find the pages to render.】

   ```html
   <!DOCTYPE html>
   <html lang="en">
   <head>
       <meta charset="UTF-8">
       <title>Home Page</title>
   </head>
   <body>
   <h1>Welcome!</h1>
   </body>
   </html>
   ```

   

2. 编写一个Controller【The controller is a component of the web app that contains methods (often named actions) executed for a specific HTTP request. In the end, the controller’s action returns a reference to the web page the app returns in response. 】

   * 如果希望用户访问/home路径的时候，执行某个方法，则需要在方法上增加@RequestMapping("/home")注解
   * 方法需要返回一个字符串，该字符串对应你希望用户访问的页面的名称

   ```java
   @Controller
   public class MainController {
       // We use the @RequestMapping annotation to
       // associate the action with an HTTP request path.
       @RequestMapping("/home")
       public String home() {
           // We return the HTML document name that contains
           // the details we want the browser to display.
           return "home.html";
       }
   }
   ```

3. 结果验证

   ![image-20251125121457352](asset/image-20251125121457352.png)

### 7.5.2 SpringMVC architecture



![image-20251125163848128](asset/image-20251125163848128.png)



==注意：==

* 在前面介绍servlet的时候，是这样说的【当我们创建一个servlet的时候，需要将这个servlet和一个路径绑定在一起。当servlet接受到一个请求的时候，会根据请求的路径，找到对应的servlet，调用对应的servlet对象的方法。】![image-20251124211210599](asset/image-20251124211210599.png)

  为什么在SpringMVC中，只有一个servlet（Dispatcher Servlet）来处理所有请求，然后再分发给对应的Controller呢？

  答：在原始的Java web模式下，确实每一个路径和一个Servlet绑定。请求到达web server（tomcat）之后，tomcat会根据URL找到对应的servlet，并调用对应的方法。

  在SpringMVC模式中，只有一个servlet，就是Dispatcher Servlet，来处理所有请求。实际上，他是拦截一个大范围的URL，在SpringBoot自动配置中，默认为`<url-pattern>/</url-pattern>`，表示默认匹配所有路径。

  则访问所有路径的请求都会被Dispatcher Servlet拦截，然后Dispatcher Servlet会根据根据后续路径（e.g. /home）找到匹配的Controller，来调用对应的Controller方法。

  这里的Dispatcher Servlet就是一个前端控制器。

  它的好处在于：

  * 实现统一入口
  * 支持拦截器、过滤器、AOP
  * 支持统一异常管理
  * ...

  自己的理解：

  基于Java web的时候，相当于每一个路径的请求都有一个处理员。（每个 URL 对应一个处理员（Servlet）。 请求来 → Tomcat 根据路径找到对应的 Servlet → 调用它处理。）

  而SpringMVC的场景下，所有的请求都被一个前置处理员处理，再由前置处理员根据注册表来分发给后续的哪一个处理员处理，这里的注册表就是记录路径和对应处理员的关系（例如/user交给处理员Bob处理，/home交给处理员Alice处理）。这样的好处在于，有一个前置处理员，我们可以对所有的请求执行统一的入口处理，可以实现拦截器、AOP、统一异常管理、同一JSON转换等等。

* 可以看到，整个SpringMVC流程图，只有Controller（图中颜色加深部分）是需要开发者自己写的，其他都是Spring底层已经实现好的。
* Spring & SpringMVC & SpringBoot的关系是什么
  * Spring是一个提供IOC、AOP能力的框架
  * SpringMVC是基于Spring实现的Web应用框架，即SpringMVC是专门用来做Web应用的
  * SpringBoot是对Spring/SpringMVC的封装，使它们能够更容易使用，目的是提升开发体验

# 8. Implementing web apps with Spring Boot and Spring MVC

## 8.1 如何实现动态页面

在ch7中，我们学会了如何创建一个静态页面：1. 在resources/static目录下增加一个静态html页面 2. 新增一个Controller，在Controller中增添一个方法，用@RequestMapping注解标识方法和路径的映射关系，在方法返回html页面的名称例如home.html。



SpringMVC的底层执行逻辑如图：

1. The client sends an HTTP request to the web server.
2. The dispatcher servlet uses the handler mapping to find out what controller action to call.
3. The dispatcher servlet calls the controller’s action.
4. After executing the action associated with the HTTP request, the controller returns the view name the dispatcher servlet needs to render into the HTTP response.
5. The response is sent back to the client.

![image-20251126112919605](asset/image-20251126112919605.png)

上面介绍的是如何实现静态页面，那如果我要实现动态页面呢？

例如manning.com/cart这个页面，用于显式用户的购物车。那这个页面显式的内容必须要根据每个用户的购物车展示不同的内容，那如何实现呢？

![image-20251126113219651](asset/image-20251126113219651.png)

下面介绍如何用Thymeleaf模板引擎实现：

*模板引擎是什么： The template engine is a dependency that allows us to easily send data from the controller to the view and display this data in a specific way. Thymeleaf只是其中一种，比较简单易学的一种。*

1. 增加Thymeleaf依赖

   ```xml
   <!--The dependency starter that needs to be added to use Thymeleaf as a template engine-->
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-thymeleaf</artifactId>
   </dependency>
   ```

2. 在Controller方法中，增加Model入参。

   Model用于存储要发送给view的数据。通过key-value方式存储。

   通过Model.addAttribute()方法能够往model中添加数据。

   ```java
   @Controller
   public class MainController {
       @RequestMapping("/home")
       // The action method defines a parameter of type Model that
       // stores the data the controller sends to the view.
       public String home(Model page) {
           // We add the data we want the controller to send to the view.
           page.addAttribute("username", "Katy");
           page.addAttribute("color", "red");
   
           // The controller’s action returns the view to be rendered into the HTTP response
           return "home.html";
       }
   }
   ```

3. 往`resources/templates`文件夹中增加html页面

   ```html
   <!DOCTYPE html>
   <html lang="en" xmlns:th="http://www.thymeleaf.org">
   <head>
       <meta charset="UTF-8">
       <title>Home Page</title>
   </head>
   <body>
   <h1>Welcome
       <span th:style="'color:' + ${color}"
             th:text="${username}"></span>!</h1>
   </body>
   </html>
   ```

   * 在ch7中，我们是在`resources/static`目录下增加html页面，我们增加的是静态页面。

     这里我们需要模板引擎帮我们渲染动态页面，因此要把html页面加入到`resources/templates`目录下。

   * `xmlns:th="http://www.thymeleaf.org"` This definition is equivalent to an import in Java. It allows us further to use the prefix “th” to refer to specific features provided by Thymeleaf in the view.

   * 通过`${attribute_key}`可以获取Controller往Model中添加的key-value的v值。例如`${username}`获取到的就是`Katy`

4. 执行结果

   ![image-20251126120345561](asset/image-20251126120345561.png)

   

==注意：==

* Thymeleaf和react和vue的区别？

  答：thymeleaf用于前后端不分离的场景，由后端使用模板引擎渲染好html页面返回给浏览器展示。react和vue时用于前后端分离的场景，后端只返回raw data（例如json格式），由react和vue这种前端应用单独负责渲染到html页面。

* 如果用户访问的路径没有对应的Controller Action（Controller中路径匹配的方法），则SpringBoot会返回一个默认的404页面。

  ![image-20251126115607617](asset/image-20251126115607617.png)

## 8.2 client如何通过HTTP发数据给server

我们在8.1介绍实现动态页面，其实数据完全是固定写好在server端的。

```java
page.addAttribute("username", "Katy");
page.addAttribute("color", "red");
```

如果我们想要从client通过HTTP协议发数据给server应该怎么做？

* HTTP request parameter

  示例：`http://example.com/products?brand=honda`

* path variable

  示例：`http://localhost:8080/home/blue`

* HTTP request body

  *will discuss in chapter 10*

* HTTP request header

### 8.2.1 HTTP request parameter

**是什么：**

示例：

`http://example.com/products?brand=honda`

`http://example.com/products?brand=honda&price=7000`

**代码：**

```java
@Controller
public class MainController {
    @RequestMapping("/home")
    // The action method defines a parameter of type Model that
    // stores the data the controller sends to the view.
    public String home(@RequestParam(required = false) String name,
                       @RequestParam(required = false) String color,
                       Model page) {
        // We add the data we want the controller to send to the view.
        page.addAttribute("username", name);
        page.addAttribute("color", color);

        // The controller’s action returns the view to be rendered into the HTTP response
        return "home.html";
    }
}
```

* @RequestParam注解告诉Spring要从request parameter获取值的字段名。这里的参数名要和请求路径的key名完全一致，例如`http://localhost:8080/home?color=blue&name=Jane`

* A request parameter is mandatory by default. If the client doesn’t provide a value for it, the server sends back a response with the status HTTP “400 Bad Request.” If you wish the value to be optional, you need to explicitly specify this on the annotation using the optional attribute: @RequestParam(optional=true).

* 执行效果：

  ![image-20251126164345928](asset/image-20251126164345928.png)

* 下图展示了client发送的数据，controller如何获取到，然后再交给View resolver，渲染到html页面中。

  ![image-20251126162911082](asset/image-20251126162911082.png)



### 8.2.2 path variables

**是什么：**

Using request parameters:
`http://localhost:8080/home?color=blue`
Using path variables:
`http://localhost:8080/home/blue`

简单来说，就是不以key-value的方式来写参数，而是直接在路径的一部分写入参数。 On the server side, you extract that value from the path from the specific position.

![image-20251126163753937](asset/image-20251126163753937.png)

**代码：**

```java
@Controller
public class MainController {
    // To define a path variable, you assign it a name 
    // and put it in the path between curly braces.
    @RequestMapping("/home/{color}")
    // You mark the parameter where you
	// want to get the path variable value
	// with the @PathVariable annotation.
	// The name of the parameter must be
	// the same as the name of the variable
	// in the path.
    public String home(@PathVariable String color, Model page) {
        
        page.addAttribute("username", "Katy");
        page.addAttribute("color", color);

        return "home.html";
    }
}
```

* 要使用path variable，在RequestMapping匹配的路径用花括号阔起路径变量的位置。用@PathVariable注解标注接收路径变量的变量。两个变量名必须完全一致。

* 执行效果：

  ![image-20251126164415484](asset/image-20251126164415484.png)

## 8.3 如何实现GET、POST请求

假设现在有一个这样的场景，思考如何实现？

产品product有两个属性：1. 产品名称 2. 产品价格。

* 用户每次可以新增一个产品（POST）
* 用户可以查看完整的产品列表信息（GET）



实现思路：

1. 新创建一个SpringBoot项目
2. 增加web依赖；增加Thymeleaf依赖；
3. 新增一个Model类Product
4. 新增一个Service类ProductService，专门处理Product相关的业务逻辑
5. 新增一个Controller类ProductController，专门处理Product相关的请求
6. 在static/templates目录下新增product.html页面，用户在该页面能够1.新增产品信息 2.查看所有产品信息

代码实现：

1. 新创建一个SpringBoot项目

   略

2. 增加web依赖；增加Thymeleaf依赖；

   ```xml
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-thymeleaf</artifactId>
   </dependency>
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-web</artifactId>
   </dependency>
   ```

3. 新增一个Model类Product

   ```java
   public class Product {
       private String name;
       private double price;
   
       // omitted getter setter code
   }
   ```

4. 新增一个Service类ProductService，专门处理Product相关的业务逻辑

   ```java
   @Service
   public class ProductService {
       public List<Product> products = new ArrayList<>();
   
       public List<Product> getAllProducts() {
           return products;
       }
   
       public void addProduct(Product product) {
           products.add(product);
       }
   }
   ```

   

5. 新增一个Controller类ProductController，专门处理Product相关的请求

   ```java
   @Controller
   public class ProductController {
       private ProductService productService;
   
       ProductController(ProductService productService) {
           this.productService = productService;
       }
   
       @RequestMapping("/products")
       public String getAllProducts(Model model) {
           // 获取所有product
           List<Product> products = productService.getAllProducts();
           // 展示
           model.addAttribute("products", products);
           return "product.html";
       }
   
       @RequestMapping(value = "/products", method = RequestMethod.POST)
       public String addProduct(@RequestParam String name,
                                @RequestParam double price,
                                Model model) {
           // 创建product对象
           Product product = new Product();
           product.setName(name);
           product.setPrice(price);
   
           // 增加到product列表
           productService.addProduct(product);
   
           // 展示最新product列表
           List<Product> products = productService.getAllProducts();
           model.addAttribute("products", products);
           return "product.html";
       }
   }
   ```

   

6. 在static/templates目录下新增products.html页面，用户在该页面能够1.新增产品信息 2.查看所有产品信息

   ```html
   <!DOCTYPE html>
   <html lang="en" xmlns:th="http://www.thymeleaf.org">
   <head>
       <meta charset="UTF-8">
       <title>Home Page</title>
   </head>
   <body>
   <h1>Products</h1>
   <h2>View products</h2>
   <table>
       <tr>
           <th>PRODUCT NAME</th>
           <th>PRODUCT PRICE</th>
       </tr>
       <tr th:each="p: ${products}" >
           <td th:text="${p.name}"></td>
           <td th:text="${p.price}"></td>
       </tr>
   </table>
   
   <h2>Add a product</h2>
   <form action="/products" method="post">
       Name: <input
           type="text"
           name="name"><br />
       Price: <input
           type="number"
           step="any"
           name="price"><br />
       <button type="submit">Add product</button>
   </form>
   </body>
   </html>
   ```

7. 效果展示

   ![image-20251127120926675](asset/image-20251127120926675.png)



注意：

* 当访问/products的时候，获取product列表到展示的SpringMVC底层执行流程

  1. The client sends an HTTP request for the /products path.
  2. The dispatcher servlet uses the handler mapping to find the controller’s action to call for the /products path.
  3. The dispatcher servlet calls the controller’s action.
  4. The controller requests the product list from the service and sends it to be rendered with the view.
  5. The view is rendered into an HTTP response.
  6. The HTTP response is sent back to the client.

  ![image-20251127121400735](asset/image-20251127121400735.png)

* 如果@RequestMapping("/products")注解不显式指定Method，则默认是使用GET

* @GetMapping("/products")等于@RequestMapping(value = "/products", method = RequestMethod.GET)

  @PostMapping("/products")等于@RequestMapping(value = "/products", method = RequestMethod.POST)

* 下面两种写法完全一样

  ```java
  @RequestMapping(value = "/products", method = RequestMethod.POST)
  public String addProduct(@RequestParam String name,
                           @RequestParam double price,
                           Model model) {
      // 创建product对象
      Product product = new Product();
      product.setName(name);
      product.setPrice(price);
  
      // 增加到product列表
      productService.addProduct(product);
  
      // 展示最新product列表
      List<Product> products = productService.getAllProducts();
      model.addAttribute("products", products);
      return "product.html";
  }
  ```

  ```java
  @RequestMapping(value = "/products", method = RequestMethod.POST)
  public String addProduct(Product product, Model model) {
      // 增加到product列表
      productService.addProduct(product);
  
      // 展示最新product列表
      List<Product> products = productService.getAllProducts();
      model.addAttribute("products", products);
      return "product.html";
  }
  ```

  因为由于Product类的属性名和（在product.html指定的）request parameter的名称完全一致，都是name和price，Spring能够识别它们相互匹配，并且自动帮你创建Product对象。这是Spring帮助开发者减少代码量的一种机制。

  注意这里Product类一定要有构造函数，以便于Spring能够调用构造函数来创建Product实例对象。

  *be aware that Spring tends to have plenty of syntaxes to hide as much code as possible. Whenever you find a syntax you don’t clearly understand in an example or article, try finding the framework specification details.*

# 9. Using the Spring web scopes

本章学习三个注解的使用`@RequestScope`、`@SessionScope`、`@ApplicationScope`

web scopes包含三个：

* Request scope: 对于每一个HTTP请求，都会创建一个新的bean

  ![image-20251203160309899](asset/image-20251203160309899.png)

  Key aspects of request-scoped beans:

  ![image-20251203160858810](asset/image-20251203160858810.png)

* Session scope: 开启一个会话的时候，会创建一个bean。后续同一个会话，始终用同一个bean。

  A session-scoped bean allows us to store data shared by multiple requests of the same client.

  ![image-20251204150333409](asset/image-20251204150333409.png)

  Key aspects of session-scoped beans:

  ![image-20251204152016928](asset/image-20251204152016928.png)

* Application scope: 所有请求共用同一个bean

  ![image-20251204171207821](asset/image-20251204171207821.png)





通过一个示例来介绍这三个scope的应用。

需求：实现一个登陆功能。

1. 用户通过输入username和password来实现登陆 - request scope
2. 如果登陆成功，要保证会话范围内，能够持续显示用户username - session scope
3. 需要记录app范围总的登陆次数 - application scope



**实现需求1：用户通过输入username和password来实现登陆 - request scope**

* LoginController

  * `/` - login() - GET: 访问`login.html`

  * `/` - login(String username, String password) - POST: 登陆

    通过调用Request Scope的LoginProcessor来实现登陆逻辑

    * 登陆成功 - 跳转`login.html`，显示`You are now logged in.`
    * 登陆失败 - 跳转`login.html`，显示`Login failed!`

  ```java
  @Controller
  public class LoginController {
      private final LoginProcessor loginProcessor;
  
      public LoginController(LoginProcessor loginProcessor) {
          this.loginProcessor = loginProcessor;
      }
  
      @GetMapping("/")
      public String login() {
          return "login.html";
      }
  
      @PostMapping("/")
      public String login(@RequestParam String username, @RequestParam String password, Model model) {
          loginProcessor.setUsername(username);
          loginProcessor.setPassword(password);
          boolean loggedIn = loginProcessor.login();
          if (loggedIn) {
              model.addAttribute("message", "You are now logged in.");
          } else {
              model.addAttribute("message", "Login failed!");
          }
          return "login.html";
      }
  }
  ```



* LoginProcessor

  ```java
  @Component
  @RequestScope
  // 每个HTTP请求都会创建一个新的LoginProcessor对象
  // why?因为 LoginProcessor 保存了 username/password，所以如果是单例，会产生线程安全问题。
  public class LoginProcessor {
      private String username;
      private String password;
  
      public boolean login() {
          boolean loggedIn = false;
          if ("alice".equals(this.username) && "password".equals(this.password)) {
              loggedIn = true;
          }
          return loggedIn;
      }
  
      // omitted getters & setters
  }
  ```

  

* `static/templates/login.html`

  ```html
  <!DOCTYPE html>
  <html lang="en" xmlns:th="http://www.thymeleaf.org">
  <head>
      <meta charset="UTF-8">
      <title>Login</title>
  </head>
  <body>
  <form action="/" method="post">
      Username: <input type="text" name="username" /><br />
      Password: <input type="password" name="password" /><br />
      <button type="submit">Log in</button>
  </form>
  <p th:text="${message}"></p>
  </body>
  </html>
  ```



* 实现效果：

  输入错误的username和password

  ![image-20251203172801362](asset/image-20251203172801362.png)

  输入正确的username和password

  ![image-20251203172850211](asset/image-20251203172850211.png)



==注意：==

* 这里注入的是LoginProcessor的bean，而LoginController只会注入一次LoginProcessor，那怎么实现每次http请求都创建新的LoginProcessor实例？

  ```java
  @Controller
  public class LoginController {
      private final LoginProcessor loginProcessor;
  
      public LoginController(LoginProcessor loginProcessor) {
          // 这里注入的是一个代理对象proxy
          // 每次请求进来的时候，都会创建一个新的LoginProcessor实例，proxy对象会指向该实例
          // request A -> proxy指向LoginProcessor A
          // request B -> proxy指向LoginProcessor B
          // ...
          this.loginProcessor = loginProcessor;
      }
      // omitted code
  }
  ```

  LoginController注入的实际上是LoginProcessor的一个proxy对象，每次有新的http请求到达的时候，都会创建一个新的LoginProcessor对象。则proxy对象会指向这个新的LoginProcessor对象。

* RequestScope和多例的区别是什么？

  * prototype scope: 每次注入/获取Bean时创建新实例
  * request scope: 每次http请求时创建新实例



**实现需求2：如果登陆成功，要保证会话范围内，能够持续显示用户username - session scope**

目标：

* 如果登陆成功，从login.html跳转到home.html，并在home.html持续显示用户username - session scope
* 实现logout功能



具体实现：

* LoggedUserManagementService

  * 负责在session维度存储username信息

  ```java
  @Service
  @SessionScope  // 在同一session维度所有请求都会用同一个bean，保证所有请求都能获得username信息
  public class LoggedUserManagementService {
      private String username;
  
      public String getUsername() {
          return username;
      }
  
      public void setUsername(String username) {
          this.username = username;
      }
  }
  ```

  

* HomeController

  * `/home` - home() - GET: 访问主页面home.html
    * 首先检查是否需要登出，如果需要登出跳转至login.html，否则继续
    * 在访问home.html之前，需要检查一下是否已登录（LoggedUserManagementService.username是否不为空）
      * 是 - 则跳转到home.html
      * 否 - 则跳转到login.html页面

  ```java
  @Controller
  public class HomeController {
      private LoggedUserManagementService loggedUserManagementService;
  
      public HomeController(LoggedUserManagementService loggedUserManagementService) {
          this.loggedUserManagementService = loggedUserManagementService;
      }
  
      @GetMapping("/home")
      public String home(@RequestParam(required = false) String logout, Model model) {
          // 检查是否需要登出
          if (logout != null) {
              loggedUserManagementService.setUsername(null);
              return "redirect:/";
          }
          // 检查用户是否已登录
          boolean loggedIn = false;
          String username = loggedUserManagementService.getUsername();
          if (username != null) {
              loggedIn = true;
          }
          // 是则跳转到home页面
          if (loggedIn) {
              model.addAttribute("username", username);
              return "home.html";
              // 否则跳转到登陆页面
          } else {
              return "redirect:/";
          }
      }
  }
  ```

  

* LoginController更改

  * `/` - login(String username, String password) - POST: 登陆

    通过调用Request Scope的LoginProcessor来实现登陆逻辑

    * 登陆成功 - 将username信息通过LoggedUserManagementService存储到session维度->跳转到home.html
    * 登陆失败 - 跳转`login.html`，显示`Login failed!`

  ```java
  @Controller
  public class LoginController {
      private final LoginProcessor loginProcessor;
      private final LoggedUserManagementService loggedUserManagementService;
  
      public LoginController(LoginProcessor loginProcessor,
                             LoggedUserManagementService loggedUserManagementService) {
          // 这里注入的是一个代理对象proxy
          // 每次请求进来的时候，都会创建一个新的LoginProcessor实例，proxy对象会指向该实例
          // request A -> proxy指向LoginProcessor A
          // request B -> proxy指向LoginProcessor B
          // ...
          this.loginProcessor = loginProcessor;
          this.loggedUserManagementService = loggedUserManagementService;
      }
  
      @GetMapping("/")
      public String login() {
          return "login.html";
      }
  
      @PostMapping("/")
      public String login(@RequestParam String username, @RequestParam String password, Model model) {
          loginProcessor.setUsername(username);
          loginProcessor.setPassword(password);
          boolean loggedIn = loginProcessor.login();
          if (loggedIn) {
              // 在session维度存储username信息
              loggedUserManagementService.setUsername(username);
              return "redirect:/home";
          } else {
              model.addAttribute("message", "Login failed!");
          }
          return "login.html";
      }
  }
  ```



* `static\templates\home.html`

  ```html
  <!DOCTYPE html>
  <html lang="en" xmlns:th="http://www.thymeleaf.org">
  <head>
      <meta charset="UTF-8">
      <title>Login</title>
  </head>
  <body>
  <h1>Welcome, <span th:text="${username}"></span></h1>
  <a href="/home?logout">Log out</a>
  </body>
  </html>
  ```



* 实现效果

  成功登陆的效果

  ![image-20251204205127072](asset/image-20251204205127072.png)

  点击log out的效果

  ![image-20251204205137388](asset/image-20251204205137388.png)



**实现需求3：需要统计app范围总的登陆次数 - application scope**

* LoginCountService用于统计总的登陆次数

  ```java
  @Service
  @ApplicationScope  // 保证在整个application维度存储loginCount
  public class LoginCountService {
      private int loginCount;
  
      public void increment() {
          this.loginCount++;
      }
  
      public int getLoginCount() {
          return loginCount;
      }
  }
  ```

* LoginController 每次尝试登陆都更新登录次数

  ```java
  @Controller
  public class LoginController {
      // omitted code
      
      @PostMapping("/")
      public String login(@RequestParam String username, @RequestParam String password, Model model) {
          // 增加尝试登陆的次数
          loginCountService.increment();
  
          loginProcessor.setUsername(username);
          loginProcessor.setPassword(password);
          boolean loggedIn = loginProcessor.login();
          if (loggedIn) {
              loggedUserManagementService.setUsername(username);
              return "redirect:/home";
          } else {
              model.addAttribute("message", "Login failed!");
          }
          return "login.html";
      }
  }
  
  ```

  

* HomeController 用户成功登陆可以展示登陆次数

  ```java
  @Controller
  public class HomeController {
      // omitted code
  
      @GetMapping("/home")
      public String home(@RequestParam(required = false) String logout, Model model) {
          // 检查是否需要登出
          if (logout != null) {
              loggedUserManagementService.setUsername(null);
              return "redirect:/";
          }
          // 检查用户是否已登录
          boolean loggedIn = false;
          String username = loggedUserManagementService.getUsername();
          if (username != null) {
              loggedIn = true;
          }
          // 是则跳转到home页面
          if (loggedIn) {
              model.addAttribute("username", username);
              model.addAttribute("loginCount", loginCountService.getLoginCount());
              return "home.html";
              // 否则跳转到登陆页面
          } else {
              return "redirect:/";
          }
      }
  }
  ```

* `static\templates\home.html`增加展示登陆次数信息

  ```html
  <!DOCTYPE html>
  <html lang="en" xmlns:th="http://www.thymeleaf.org">
  <head>
      <meta charset="UTF-8">
      <title>Login</title>
  </head>
  <body>
  <h1>Welcome, <span th:text="${username}"></span>!</h1>
  <h2>
      Your login number is
      <span th:text="${loginCount}"></span>
  </h2>
  <a href="/home?logout">Log out</a>
  </body>
  </html>
  ```

  

==注意：==

* 现实场景中，一般不会自定义实现login或授权功能，一般使用Spring Security来实现相关功能。该示例仅用于教学场景。why使用Spring Security？
  1. 已经封装好，直接用更简单
  2. 避免出现bug
  3. 作者推荐Spring Security学习书籍：*Spring Security in Action (Manning, 2020)*
* 如果要从一个页面重定向要另一个页面，则Controller action方法需要返回这样形式的字符串`redirect:/xxx` 注意，`/xxx`是把请求重定向到另一个Controller action，因此不是`/home.html`，而是`/home`形式。
* Request scope、Session scope、Application scope only make sense in web apps, and that’s why we call them web scopes.
* 尽量避免使用Application scope，因为这意味着所有的 web request 共享这些作用范围为`application-scope`的beans，对这些 beans 的写操作会涉及竞态条件，需要加锁等操作保证线程安全，会严重影响性能。且Application scope的bean不会被GC，会一直占用内存。A better approach is to directly store the data in a database.

# 10. 实现REST服务

## 10.1 REST是什么

REST是一种设计Web API的风格，用URL标识资源，用HTTP方法标识操作。

REST不是协议，也不是框架，而是一种思想。

* 资源：

  REST的核心思想：把所有东西看作一种资源。例如：

  | 资源       | URL               |
  | ---------- | ----------------- |
  | 用户列表   | `/users`          |
  | 单个用户   | `/users/1`        |
  | 用户的订单 | `/users/1/orders` |

  这些都是“资源”。

* 操作

  用HTTP的方法`GET/POST/PUT/DELETE`标识对资源要执行什么操作。

  REST强调不要在URL写动作，而是用HTTP标识。

  例如

  不推荐写：

  ```
  GET /getUsers
  POST /createUser
  DELETE /removeUser
  ```

  推荐写：

  ```
  GET /users
  POST /users
  DELETE /users/1
  ```



REST的好处：

* 简洁、规范、清晰、易于维护。

## 10.2 REST如何返回数据给client（概念）

**1. Controller action和REST endpoint是什么？**

* Controller action: controller类中的某个方法

  例如：`getUsers()` 就是一个 **controller action**（也叫 handler method）。

  ```java
  @GetMapping("/users")
  public List<User> getUsers() { ... }
  ```

* REST endpoint: 一个遵循REST风格的URL接口

  例如：`GET /users` 就是一个REST endpoint

* 两者的关系：一个 REST endpoint 通常由一个 controller action 实现，如果项目是100% RESTful，则基本可以视为是一 一对应关系。

**2. RESTful 接口和普通Controller action的写法有什么区别？**

* Controller action return返回的是表示要显示的页面的名称，例如`home.html`。通过Model来向View resolver传递数据，由View resolver对页面进行渲染然后返回给client。

* RESTful接口返回的内容直接作为HTTP Response响应体返回给client，不再需要View resolver。例如如果返回了一个对象，就以下面Json形式在响应体传递给client。

  ```json
  {
      "amount": 10001
  }
  ```

  ![image-20251206165428303](asset/image-20251206165428303.png)

## 10.3 实现REST 服务（实现）

RESTful controller和普通web controller的区别：

普通web controller

```java
@Controller
public class HelloController {
    @GetMapping("/hello")
    public String hello() {
        return "hello.html";  // return a view name
    }
}
```



RESTful controller

```java
@Controller
public class HelloController {
    @GetMapping("/hello")
    @ResponseBody
    public String hello() {
        return "Hello!";
    }
}

// The @ResponseBody annotation tells the dispatcher servlet that 
// the controller’s action doesn’t return a view name 
// but the data sent directly in the HTTP response.
```



总结：

所以对于一个普通web controller，controller action会返回一个字符串，这个字符串代表的是view name。对于RESTful controller action，会在方法上增加@ResponseBody注解，表示返回的内容直接作为HTTP Reponse响应体返回，而不是表示view name。



REST这种写法不足在于，如果controller中的方法很多，那么每一个方法上都需要增加@ResponseBody注解，例如：

```java
@Controller
public class HelloController {
    @GetMapping("/hello")
    @ResponseBody
    public String hello() {
        return "Hello!";
    }

    @GetMapping("/ciao")
    @ResponseBody
    public String ciao() {
        return "Ciao!";
    }
}
```



为了解决避免出现重复代码，可以在controller类上使用`@Controller`和`@ResponseBody`两个注解，或者直接用`@RestController`（两个注解的结合）。这样表示该controller类的所有action都属于RESTful endpoints。如下：

```java
@RestController
public class HelloController {
    @GetMapping("/hello")
    public String hello() {
        return "Hello!";
    }

    @GetMapping("/ciao")
    public String ciao() {
        return "Ciao!";
    }
}
```



现在我们写好了代码，是时候测试一下效果了，如何发起http请求呢，我们之前都是直接在浏览器输入URL，对于开发者，这种方式是远远不够的，我们需要能够便捷地 1. 指定完整的http请求内容（包括请求体、请求头、URL...） 2. 查看完整的相应内容（包括响应体、响应头...）。因此Postman和Curl是更加专业的两个工具。

postman是图形化工具，更加便于使用。Curl是命令行工具，适合用于脚本。下面展示分别用两种工具测试的效果：

postman：

![image-20251206203656718](asset/image-20251206203656718.png)



curl：

![image-20251206203740657](asset/image-20251206203740657.png)

（当然curl也可以展示详细的相应内容，需要增加对应的参数，这里不进行详细介绍）



总结：

* 如何实现REST服务？

  答：其写法和普通的Controller是类似的。区别只在于普通的Controller action返回view name字符串，需要View Resolver来渲染页面返回给client。而RESTful Controller action返回的内容，会直接作为HTTP Response的响应内容。

  如何写编写代码呢？

  1. 在RESTful Controller action上增加@ResponseBody注解 或
  2. 用@RestController注解替代@Controller注解即可。
  3. return xxx `xxx`为想要在response body返回的内容，而不是view name



## 10.4 自定义HTTP Response

HTTP Response包含三部分：

1. Response headers
2. Response body
3. Response status

### 10.4.1 如何response body中返回对象

直接在RESTful Controller action中以该对象作为返回值即可。

例一：

Country类，作为返回的对象类型

```java
public class Country {
    private String name;
    private int population;

    public static Country of(String name, int population) {
        Country country = new Country();
        country.setName(name);
        country.setPopulation(population);
        return country;
    }
    // omitted getters and setters
}
```

RESTful Controller

```java
@RestController
public class CountryController {
    @GetMapping("/france")
    public Country france() {  // 直接以Country作为返回类型
        Country c = Country.of("France", 67);
        return c;  // 直接返回要返回的对象
    }
}
```

测试效果：

可以看到，Spring是默认将对象转化为Json格式进行返回。

![image-20251206205522776](asset/image-20251206205522776.png)

例二：

返回Country列表

RESTful Controller

```java
@RestController
public class CountryController {
    @GetMapping("/all")
    public List<Country> countries() {
        Country c1 = Country.of("France", 67);
        Country c2 = Country.of("Spain", 45);
        return List.of(c1, c2);
    }
}

```

测试效果：

Spring依然会将列表转化为JSON格式返回，只不过这是数组的形式。

![image-20251206205858891](asset/image-20251206205858891.png)

==注意：==

* When we use an object (such as Country) to model the data transferred between two apps, we name this object a data transfer object (DTO). We can say that Country is our DTO.

### 10.4.2 如何自定义response status和headers

通过ResponseEntity这个类，可以自定义reponse body、status、headers...

示例：

RESTful Controller

```java
@RestController
public class CountryController {

    @GetMapping("/france")
    public ResponseEntity<Country> france() {
        Country c = Country.of("France", 67);
        return ResponseEntity
                .status(HttpStatus.ACCEPTED)
                .header("continent", "Europe")
                .header("capital", "Paris")
                .header("favorite_food", "cheese and wine")
                .body(c);
    }
}
```

测试效果：

![image-20251206211153517](asset/image-20251206211153517.png)

![image-20251206211124344](asset/image-20251206211124344.png)

### 10.4.3 异常控制

#### 方法一：每个controller action单独处理

情景介绍：

假设现在有一个支付接口makePayment()，该接口调用service层的processPayment()方法进行业务处理。

- 用户余额充足，支付成功，则Controller action需要返回一个PaymentDetail对象，包含订单支付详情（例如支付的金额等）。
- 用户余额不足，service层会抛出一个余额不足异常，Controller action需要返回一个ErrorDetail对象，包含异常详情。

代码设计：

* pojo
  * PaymentDetail
  * ErrorDetail
* exception
  * NotEnoughMoneyException
* Service
  * PaymentService
    * processPayment方法 - 支付成功，返回PaymentDetail；支付失败，抛出NotEnoughMoneyException异常。
* Controller
  * PaymentController
    * makePayment方法 - 调用#PaymentService.processPayment方法，如果执行成功，则返回ResponseEntity with 1.PaymentDetail 2.status 202ACCEPTED；如果获取到异常，则返回ResponseEntity with 1.ErrorDetail 2.status 400 badrequest；

代码实现：

* pojo

  * PaymentDetail

    ```java
    public class PaymentDetail {
        private double fee;
    
        public double getFee() {
            return fee;
        }
    
        public void setFee(double fee) {
            this.fee = fee;
        }
    }
    ```

  * ErrorDetail

    ```java
    public class ErrorDetail {
        private String message;
    
        public String getMessage() {
            return message;
        }
    
        public void setMessage(String message) {
            this.message = message;
        }
    }
    ```

* exception

  * NotEnoughMoneyException

    ```java
    public class NotEnoughMoneyException extends RuntimeException{
    }
    ```

* Service

  * PaymentService

    * processPayment方法 - 支付成功，返回PaymentDetail；支付失败，抛出NotEnoughMoneyException异常。

      ```java
      @Service
      public class PaymentService {
          public PaymentDetail processPayment(double money) {
              if (money < 100) {
                  // 如果余额不足100，则抛出异常
                  throw new NotEnoughMoneyException();
              }
              PaymentDetail paymentDetail = new PaymentDetail();
              paymentDetail.setFee(money);
              return paymentDetail;
          }
      }
      ```

      

* Controller

  * PaymentController

    * makePayment方法 - 调用#PaymentService.processPayment方法，如果执行成功，则返回ResponseEntity with 1.PaymentDetail 2.status 202ACCEPTED；如果获取到异常，则返回ResponseEntity with 1.ErrorDetail 2.status 400 badrequest；

      ```java
      @RestController
      public class PaymentController {
      
          private final PaymentService paymentService;
      
          public PaymentController(PaymentService paymentService) {
              this.paymentService = paymentService;
          }
      
          @PostMapping("/payment")
          public ResponseEntity<?> makePayment(@RequestParam double money) {
              try {
                  PaymentDetail paymentDetail = paymentService.processPayment(money);
      
                  return ResponseEntity
                          .status(HttpStatus.ACCEPTED)
                          .body(paymentDetail);
              } catch (NotEnoughMoneyException e) {
                  ErrorDetail errorDetail = new ErrorDetail();
                  errorDetail.setMessage("Not enough money for this payment.");
                  return ResponseEntity
                          .badRequest()  // 将status设为400
                          .body(errorDetail);
              }
          }
      }
      ```



实现效果：

余额不足

![image-20251206214831461](asset/image-20251206214831461.png)



支付成功

![image-20251206214845647](asset/image-20251206214845647.png)

#### 方法二：异常处理切面（推荐）

上述方法的不足：

1. 如果多个controller action需要处理同一个异常，则存在代码冗余
2. 异常处理逻辑分散在各个action中，不方便阅读

解决方案：使用切面进行异常处理，这个切面会拦截所有异常，开发者可以在这个切面类中自定义所有异常的处理方式，则一方面，不会再出现代码冗余，另一方面，异常处理逻辑可以集中在这个切面类中，controller action中不需要再对异常进行专门的处理，职能分工，提升代码可读性以及可维护性。

![image-20251206215508115](asset/image-20251206215508115.png)



将方法一的代码改写为方法二的做法。

Controller只需要关注happy flow（正常/没有异常的case）

```java
@RestController
public class PaymentController {

    private final PaymentService paymentService;

    public PaymentController(PaymentService paymentService) {
        this.paymentService = paymentService;
    }

    @PostMapping("/payment")
    public ResponseEntity<PaymentDetail> makePayment(@RequestParam double money) {
        PaymentDetail paymentDetail = paymentService.processPayment(money);

        return ResponseEntity
                .status(HttpStatus.ACCEPTED)
                .body(paymentDetail);
    }
}
```

由advice专门对异常进行统一处理

```java
@RestControllerAdvice  // 标识这是一个RESTController的增强(advice)
public class ExceptionControllerAdvice {
    @ExceptionHandler(NotEnoughMoneyException.class)  // 标识要处理的异常类
    public ResponseEntity<ErrorDetail> exceptionNotEnoughMoneyHandler() {
        ErrorDetail errorDetails = new ErrorDetail();
        errorDetails.setMessage("Not enough money to make the payment.");
        return ResponseEntity
                .badRequest()
                .body(errorDetails);
    }
}
```

和方法一的效果完全一致。

==注意：==

* 如果我们希望在ExceptionHandler的方法获取到完整的Exception信息，可以在方法参数增加这个异常类型的参数，Spring会帮我们将Controller抛出的异常作为参数传入，从而ExceptionHandler能够获取到完整的异常信息。

  例如：

  ```java
  @RestControllerAdvice
  public class ExceptionControllerAdvice {
      @ExceptionHandler(NotEnoughMoneyException.class)
      public ResponseEntity<ErrorDetail> exceptionNotEnoughMoneyHandler
          (NotEnoughMoneyException ex) {  // controller抛出的异常会作为参数传入
          System.out.println(ex.getMessage());
          // omitted code
      }
  }



## 10.5 接收client在Request Body携带的数据

如果server要接收client在request body携带的数据，需要：

1. To use the request body, you just need to annotate a parameter of the controller’s
   action with `@RequestBody`.
2. By default, Spring assumes you used `JSON` to represent the parameter you annotated and will try to `decode the JSON string into an instance of your parameter type`.
3. In the case Spring cannot decode the JSON-formatted string into that type, the app sends back a response with the status “`400 Bad Request.`” 



示例：

client发送一个PaymentDetail实例给server，server打印日志，然后将同一个实例返回给client。

Controller

```java
@RestController
public class PaymentController {
    private static Logger logger = Logger.getLogger(PaymentController.class.getName());
   
    private final PaymentService paymentService;

    public PaymentController(PaymentService paymentService) {
        this.paymentService = paymentService;
    }

    @PostMapping("/payment")
    public ResponseEntity<PaymentDetails> makePayment(@RequestBody PaymentDetails paymentDetails) {
        logger.info("Received payment " + paymentDetails.getAmount());
        return ResponseEntity
                .status(HttpStatus.ACCEPTED)
                .body(paymentDetails);
    }
}
```

日志

![image-20251206221918129](asset/image-20251206221918129.png)

响应

![image-20251206221931165](asset/image-20251206221931165.png)

==注意：==

* 从2014年起，GET 请求也可以携带request body



# 11. 跨服务调用RESTful接口

RESTful接口不仅是web client会调用，有时候服务之间也会相互调用。

本章介绍一下，不同服务直接如何互相调用RESTful接口。

服务A调用服务B的RESTful接口，有三种方式：

1. OpenFeign：最新、最推荐的方式。
2. RestTemplate：以前最被广泛使用的方式，虽然不是当前的首选，但是有很多服务在以前使用这种方式实现的。
3. WebClient：reactive app首选的方式。



接下来通过示例介绍一下这三种方式如何实现。

假设现在我们有一个app A，其中包含的一项功能为支付功能，而支付功能是由另一个服务service Payment来实现的，也就是说，当client尝试进行支付的时候，请求会到达app A，app A会调用service Payment的RESTful接口来实现支付。

app A如何调用service Payment的支付接口，就是我们要讨论的问题。

![image-20251209202840275](asset/image-20251209202840275.png)



1. 实现一个service Payment

   该支付接口的逻辑为：

   1. RequestHeader接收requestId，RequestBody接收Payment信息
   2. log打印日志，输出requestId、payment的amount信息
   3. 为payment对象设置id
   4. 返回 ResponseHeader中增加requestId，ResponseBody中增加Payment信息

2. 通过上面介绍的三种方式来调用service Payment的RESTful接口



1. 实现一个service Payment

   该支付接口的逻辑为：

   1. RequestHeader接收requestId，RequestBody接收Payment信息
   2. log打印日志，输出requestId、payment的amount信息
   3. 为payment对象设置id
   4. 返回 ResponseHeader中增加requestId，ResponseBody中增加Payment信息

   ```java
   public class Payment {
       private String id;
       private double amount;
   	// omitted getter setter
   }
   ```

   ```java
   @RestController
   public class PaymentController {
       private static Logger logger = Logger.getLogger(PaymentController.class.getName());
   
       @PostMapping("/payment")
       public ResponseEntity<Payment> createPayment(
               @RequestHeader String requestId,
               @RequestBody Payment payment
       ) {
          logger.info("request id: " + requestId +
                  "; payment's amount: " + payment.getAmount() + ".");
           payment.setId(UUID.randomUUID().toString());
           return ResponseEntity
                   .status(HttpStatus.OK)
                   .header("requestId", requestId)
                   .body(payment);
       }
   }

2. 通过上面介绍的三种方式来调用service Payment的RESTful接口



## 11.1 OpenFeign

1. 注入OpenFeign相关依赖

   ```xml
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-openfeign</artifactId>
   </dependency>
   ```

   注意OpenFeign是spring-cloud内部的依赖，因此BOM中要引入spring-cloud-dependencies

   ```xml
   <dependencyManagement>
       <dependencies>
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-dependencies</artifactId>
               <version>${spring-boot.version}</version>
               <type>pom</type>
               <scope>import</scope>
           </dependency>
           <dependency>
               <groupId>org.springframework.cloud</groupId>
               <artifactId>spring-cloud-dependencies</artifactId>
               <version>${spring-cloud.version}</version>
               <type>pom</type>
               <scope>import</scope>
           </dependency>
       </dependencies>
   </dependencyManagement>
   ```

2. 编写FeignClient接口及内部方法

   ```java
   @FeignClient(name = "payment", url = "${payment.service.url}")
   public interface PaymentProxy {
       @PostMapping("/payment")
       Payment createPayment(@RequestHeader("requestId") String requestId,
                             @RequestBody Payment payment);
   }
   ```

   * `@FeignClient`注解标识这是REST client，在接口上增加`@FeignClient`注解，OpenFeign会自动生成该接口的实现类，并生成对应的bean，注入到Spring容器。
   * 使用`@FeignClient`注解，至少要指定两个属性 `name` 和 `url` 。
     * `name`是OpenFeign内部使用到的REST client唯一标识。
     * `URL`指定要访问的REST的url地址，例如`http://localhost:8080`。注意，类似这样的URL一定要在配置文件进行配置，而不是硬编码在代码中。

3. 配置中启用OpenFeign以及指定OpenFeign client所在的包

   ```java
   @Configuration
   // We enable the OpenFeign clients and tell the OpenFeign dependency
   // where to search for the proxy contracts.
   @EnableFeignClients(basePackages = "com.tudou.sqch11ex1.proxy")
   public class ProjectConfig {
   }
   ```

4. 编写controller，并注入Feign Client bean，调用方法

   ```java
   @RestController
   public class PaymentController {
       private final PaymentProxy paymentProxy;
   
       public PaymentController(PaymentProxy paymentProxy) {
           this.paymentProxy = paymentProxy;
       }
   
       @PostMapping("/payment")
       public Payment createPayment(@RequestBody Payment payment) {
           String requestId = UUID.randomUUID().toString();
           return paymentProxy.createPayment(requestId, payment);
       }
   }
   ```

5. application.properties

   ```properties
   # app A在9090端口启动
   server.port = 9090
   # service payemnt在8080启动，访问该url以访问payment service
   payment.service.url = http://localhost:8080
   ```

6. 测试效果

   ps：在response body中的id是在payment Service中生成的payment id。在控制台打印的这个request Id，是app A生成，然后作为request header参数传给payment service的request Id。

![image-20251209214422733](asset/image-20251209214422733.png)

![image-20251209214449329](asset/image-20251209214449329.png)



解析：

* 我在看到这里的时候会有一个问题。我访问app A的createPayment接口的时候，只传入了payment对象，requestId是此时生成然后传入paymentProxy.createPayment中的。我很好奇为什么paymentProxy拿到了这个requestId之后，就能够在后续调用payment Service的createPayment方法时作为request header传入呢？

  答：这里只是定义了接口，并没有写真正的HTTP请求逻辑。

  ```java
  @FeignClient(name = "payment", url = "${payment.service.url}")
  public interface PaymentProxy {
      @PostMapping("/payment")
      Payment createPayment(@RequestHeader("requestId") String requestId,
                            @RequestBody Payment payment);
  }
  ```

  但 Feign 会在运行时为这个接口生成一个代理类（动态代理），这个代理：

  * 看到你标了 @PostMapping("/payment")
  * 看到你标了 @RequestHeader("requestId") String requestId
  * 看到 @RequestBody Payment payment

  知道它应该构造一条 HTTP POST 请求：

  ```css
  POST /payment
  Header: requestId=<你传入的值>
  Body: { "amount": ..., ... }
  ```

  也就是说：**你调用 Java 方法 = Feign 帮你构造并发送 HTTP 请求**

  所以OpenFeign的优点在于，把整个发送HTTP请求的复杂过程，简化成接口+注解的形式。

  通过各个注解，我们可以声明我们需要以*什么HTTP方法*、向*哪里*发送*什么请求*。

  而OpenFeign会根据我们声明的接口和注解，自动帮我们去生成实现类，这个实现类内部的方法，就是运行过程中Spring会真正调用的方法，内部实现了真正的构造HTTP请求，发送HTTP请求，接收HTTP响应的行为。

## 11.2 RestTemplate

为什么OpenFeign逐渐替代RestTemplate：并不是因为RestTemplate有什么不足，而是因为OpenFeign更加易用，简单。很多现有的服务的运行代码依然是使用RestTemplate，因为开发这些服务的时候RestTemplate在当时是最好的方案，因此掌握RestTemplate也是非常必要的。

1. 引入spring-boot-starter-web依赖

   因为`RestTemplate`是`spring-boot-starter-web`依赖下的类

   ```xml
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-web</artifactId>
   </dependency>
   ```

2. 配置中创建RestTemplate Bean

   ```java
   @Configuration
   public class ProjectConfig {
       @Bean
       public RestTemplate restTemplate() {
           return new RestTemplate();
       }
   }
   ```

3. 编写proxy，在proxy方法中实现构造HTTP请求，使用restTemplate实现发送HTTTP请求，接收HTTP响应

   ```java
   @Component
   public class PaymentProxy {
       private final RestTemplate restTemplate;
   
       @Value("${payment.service.url}")
       private String url;
   
       public PaymentProxy(RestTemplate restTemplate) {
           this.restTemplate = restTemplate;
       }
   
       public Payment createPayment(Payment payment) {
           // 1. 构造url
           String url = this.url + "/payment";
           // 2. 构造HTTP request headers
           String requestId = UUID.randomUUID().toString();
           HttpHeaders headers = new HttpHeaders();
           headers.add("requestId", requestId);
           // 3. 构造HTTP Entity（HTTP请求体，包含request body、request header）
           HttpEntity<Payment> httpEntity = new HttpEntity<>(payment, headers);
           // 4. 发送http请求
           ResponseEntity<Payment> responseEntity = restTemplate.exchange(url, HttpMethod.POST, httpEntity, Payment.class);
           // 5. 返回response body
           return responseEntity.getBody();
       }
   }
   ```

4. 编写Controller，注入proxy bean，调用方法

   ```java
   @RestController
   public class PaymentController {
       private final PaymentProxy paymentProxy;
   
       public PaymentController(PaymentProxy paymentProxy) {
           this.paymentProxy = paymentProxy;
       }
   
       @PostMapping("/payment")
       public Payment createPayment(@RequestBody Payment payment) {
           return paymentProxy.createPayment(payment);
       }
   }
   ```

5. application.properties

   ```properties
   # app A在9090端口启动
   server.port = 9090
   # service payemnt在8080启动，访问该url以访问payment service
   payment.service.url = http://localhost:8080
   ```

6. 测试效果

   ![image-20251210170112913](asset/image-20251210170112913.png)

   ![image-20251210170146909](asset/image-20251210170146909.png)

解析：

* 可以非常明显地看到，RestTemplate的编码复杂度比OpenFeign高得多，RestTemplate需要手动编写生成HTTP header的代码、生成HTTP Entity（请求体）的代码，再调用RestTemplate的exchange方法才能真正调用payment service。而OpenFeign只需要定义一个接口，通过注解的方式进行配置，大大减少了代码量，也很大程度提高了可读性

## 11.3 WebClient

结论：如果是一个reactive app，则使用WebClient；如果不是reactive app，则使用OpenFeign。

* 非reactive app是什么

  通过举例的方式来说明。

  假设A银行业务需要开发一个新的系统，该系统用于统计一个顾客的总负债情况。考虑到一个顾客可能在多家不同银行都设立了账户，因此这个系统的实现思路如下：

  1. 接收到统计某顾客总负债情况请求，userId为请求参数
  2. 根据userId从service x 获取用户信息
  3. 根据userId从service y 获取用户在本银行的负债情况
  4. 根据1. 得到的用户信息从service z 获取用户在外部银行的负债信息
  5. 将内部负债和外部负债值进行加和，得到该名用户的总负债值

  对于非reactive app，每一个请求都由一个线程来负责。

  例如现在有一个查询顾客Alice的负债情况的请求，此时系统会分配一个线程，这个线程负责完整的1-5的工作。

  ![image-20251211165144437](asset/image-20251211165144437.png)

  由于系统调用service x、y、z都是IO操作，该线程在每一个IO操作的时候都需要闲置等待，因此，本质上是一个线性操作。

  ![image-20251211165316678](asset/image-20251211165316678.png)

  这种方式存在的问题是什么？

  * 当线程遇到IO操作的时候，会被阻塞，此时该线程不能做其它的事情，只能闲置等待IO操作完成。线程即占用着宝贵的内存，还不在工作。
  * 部分task之间并没有相互依赖的关系。例如Figure 11.8图中的2、3步，他们是相互独立的，第3步并不依赖第2步的执行，因此它们实际上是可以并行地计算的。

* reactive app是什么

  Reactive apps change the idea of having one atomic flow in which one thread executes all its tasks from the beginning to the end. With reactive apps, we think of tasks as independent, and multiple threads can collaborate to complete a flow composed of multiple tasks.

  也就是说，reactive app会将所有的task都看作相互独立的，所有线程都可以去执行的。

  假设现在有两个线程在共同执行一些任务，线程A执行执行任务1，线程B正在执行任务2，此时任务1正在执行IO操作，线程1被阻塞，则线程1继续去执行其它任务，然后任务1 IO完成之后继续执行任务1。或者线程B空闲之后如果发现任务1可以继续往下执行了，线程B可以去执行。

  

  ![image-20251211170546768](asset/image-20251211170546768.png)

  类比：

  非reactive app：厨房的厨师A负责做水煮鱼这道菜，他每接受到一个水煮鱼订单，就烧水->切菜->炒菜，只有水烧开了他才会切菜，烧水过程中他只会站在那里等待水烧开。他不会边切菜边等水烧开，等水烧开的过程中，就算有其他厨师非常忙，他也不会去帮忙，因此效率是比较低的。

  reactive app：同样的一个厨师负责做水煮鱼，他会先烧水，等水开的过程会去切菜，提高效率。当他没事干的时候还会去帮其它的厨师。大家相互帮忙，提升效率。

* reactive app如何表达task之间的依赖关系？

  producer + subscriber

  subscriber 订阅 producer，表示当producer task执行完毕，subscriber task会根据producer task的执行结果开始执行。

  银行示例task之间的依赖关系如图：通过这个依赖关系，可以知道TASK C、D没有相互的依赖关系，可以同时执行。

  ![image-20251211173445778](asset/image-20251211173445778.png)

* WebClient是如何体现reactive app的思想的

  通过Mono。

  WebClient是HTTP请求的执行器。Mono表示一个task的异步结果。

  如果我们给一个方法的入参传入的是一个Mono，表示这个方法 subscribe to Mono对应的task，只有当这个task执行完毕，这个方法才能够执行。

  如果一个方法的返回值类型是Mono，则调用这个方法的地方 就是subscribe to这个方法，这个方法执行返回返回成功，调用的地方才能够继续往下执行。

  也就是说，通过Mono我们能够给task之间建立producer和consumer这样的依赖关系。

  因此得以builds the flow, not by chaining them on a thread, but by linking the dependencies between tasks through producers and consumers 

  

  注意：Mono只是帮助我们搭建任务之间的producer-consumer这样的依赖关系。只有Spring WebFlux订阅整个WebClient链的时候，订阅关系才会生效，单独的Mono的使用并没有真正的订阅关系。

  当收到一个 HTTP 请求时，Spring WebFlux 调用你的 Controller 方法 -> Controller 返回一个Mono对象，例如`Mono<Payment>` -> 然后 Spring WebFlux 会订阅这个 Mono，从而触发整个 WebClient 链条的执行。

  

* WebClient的开发流程是什么

  1. 引入WebFlux依赖

     ```xml
     <dependency>
         <groupId>org.springframework.boot</groupId>
         <artifactId>spring-boot-starter-webflux</artifactId>
     </dependency>
     ```

  2. 配置创建WebClient Bean

     ```java
     @Configuration
     public class ProjectConfig {
         @Bean
         public WebClient webClient() {
             return WebClient
                     .builder()
                     .build();
         }
     }
     ```

  3. 创建proxy类，注入WebClient bean，该bean负责真正的构造http请求、发送http请求、接收http响应

     ```java
     @Component
     public class PaymentProxy {
         private final WebClient webClient;
     
         @Value("${payment.service.url}")
         private String url;
     
         public PaymentProxy(WebClient webClient) {
             this.webClient = webClient;
         }
     
         public Mono<Payment> createPayment(String requestId, Payment payment) {
             // 1. 构造url
             String url = this.url + "/payment";
             // 2. 发起http请求
             return webClient
                     .post()
                     .uri(url)
                     .header("requestId", requestId)
                     .body(Mono.just(payment), Payment.class)
                     .retrieve()
                     .bodyToMono(Payment.class);
         }
     }
     ```

  4. 创建controller，注入proxy bean，调用方法

     ```java
     @RestController
     public class PaymentController {
         private final PaymentProxy paymentProxy;
     
         public PaymentController(PaymentProxy paymentProxy) {
             this.paymentProxy = paymentProxy;
         }
     
         @PostMapping("/payment")
         public Mono<Payment> createPayment(@RequestBody Payment payment) {
             String requestId = UUID.randomUUID().toString();
             return paymentProxy.createPayment(requestId, payment);
         }
     }

  5. application.properties

     ```properties
     server.port=9090
     payment.service.url=http://localhost:8080
     ```

  6. 测试效果

     ![image-20251211201741300](asset/image-20251211201741300.png)

     ![image-20251211201804918](asset/image-20251211201804918.png)

  proxy这段代码的task之间的依赖关系：

  ![image-20251211194920404](asset/image-20251211194920404.png)

## 11.4 总结

非reactive app - 使用OpenFeign

​    reactive app - 使用WebClient



# 12. 使用data source

## 12.1 data source是什么

问题1：Java如何建立DB连接？

Java其实不会直接和DB进行连接，而是和DBMS进行连接，DBMS（the database management system） 用于管理DB中的数据，通过DBMS我们可以方便地进行增删改查。

ok 问题转化为Java如何建立和DBMS的连接？在后续的讨论钟，我们不严格对DBMS和DB的表述进行区分。

使用JDBC。JDBC是什么？

JDBC: 是Java所提供的用于和关系型数据库进行连接的功能（Java Database Connectivity）。

JDBC仅提供抽象（接口），不提供连接各个类型的DBMS（MySQL、Postgres、Oracle）的功能实现，具体的实现称为JDBC driver，由具体的DBMS厂商根据JDBC所提供的接口进行实现。

不同的DBMS会提供不同的JDBC driver实现，例如MySQL会提供MySQL JDBC driver，Oracle会提供Oracle JDBC driver。在进行Java开发的过程中，如果我们是要连接MySQL数据库，则我们引入MySQL JDBC driver的依赖，就可以与之建立连接了。

那具体怎么建立连接呢？

JDBC提供了一个DriverManager类，这个类提供了一个getConnetion(...)方法用于建立连接

```java
Connection con = DriverManager.getConnection(url, username, password);
```

问题2：

直接使用JDBC建立连接，存在什么问题吗？

我们在程序中，直接使用DriverManager类来建立连接，则每一次进行DB操作，都需要调用getConnection(...)方法，那每一次都需要创建新的连接，进行验证，需要多次进行重复的操作。就类似于你去酒吧点酒，酒保需要查看你的身份证看是否成年，然后你每点一次酒都要检查一次身份证，是不合理的。

![image-20251212161743469](asset/image-20251212161743469.png)

问题3：如何解决这个问题？

引入data source。

data source的作用就是帮助我们管理数据库连接，它能够保证能够复用连接的时候尽量复用连接，只有在必须的情况下，才会创建新的连接，以提高性能。

![image-20251212161856086](asset/image-20251212161856086.png)

所以我们现在知道了data source是什么，data source就是一个负责管理数据库连接的组件，它实现了复用数据库连接，避免重复创建新连接，只有在必要的时候才会创建新连接，提升我们的服务性能。

## 12.2 JdbcTemplate的使用

这一节我们来介绍一下如何使用JdbcTemplate来建立DB连接，并实现基本的数据库操作。

JdbcTemplate是什么？

JdbcTemplate 是 Spring 提供的 JDBC 工具类，用来简化数据库访问。它本身不管理连接，也不直接和数据库通信，而是通过 DataSource 获取 Connection，再调用 JDBC API（由 JDBC Driver 实现）执行 SQL。

为什么要用JdbcTemplate？

如果我们使用JDBC接口提供的方法来执行SQL，其实并不方便，下面这段代码只是向purchase表中插入一条数据，需要写这么多行代码，更不必说已经省略了catch块中的代码。

```java
String sql = "INSERT INTO purchase VALUES (?,?)";
try (PreparedStatement stmt = con.prepareStatement(sql)) {
stmt.setString(1, name);
stmt.setDouble(2, price);
stmt.executeUpdate();
} catch (SQLException e) {
// do something when an exception occurs
}
```

JdbcTemplate就是Spring提供的帮我们进一步减少代码量的工具类。

怎么使用JdbcTemplate？

以下面这个例子来介绍：

有一张Purchase表，这个表中包含三列：id、product、price

* id—An auto-incrementing unique value that also takes the responsibility of the primary key of the table
* product—The name of the purchased product
* price—The purchase price

![image-20251213122256813](asset/image-20251213122256813.png)

**业务需求：**

* 访问GET-/purchase可以获得purchase表中的所有数据
* 访问POST-/purchase可以向purchase插入一条新的数据

**代码设计：**

* Repository层：
  * PurchaseRepository
    * void storePurchase(Payment) 插入一条新数据
    * List\<Purchase> findAllPurchase() 获取所有数据
* Controller层（调用Repository层的方法）
  * GET-/purchase-storePurchase(Payment)
  * POST-/purchase-findAllPurchase()

![image-20251213122211139](asset/image-20251213122211139.png)

**代码实现：**

* schema.sql

  示例使用了H2数据库，在resources目录下新增schema.sql文件，可以在里面编写DDL，Spring启动时会执行。【该方法仅适用于学习】

  ```sql
  CREATE TABLE IF NOT EXISTS purchase (
      id INT AUTO_INCREMENT PRIMARY KEY,
      product varchar(50) NOT NULL,
      price double NOT NULL
  );
  ```

* model层

  ```java
  public class Purchase {
      private Integer id;
      private String product;
      private BigDecimal price;
  	// omitted getter/setter
  }
  ```

* pom.xml

  ```xml
  <!--引入该依赖才能够实现web功能-->
  <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
  </dependency>
  <!--引入该依赖：
  	1. 触发 Spring Boot 自动创建 DataSource Bean
  	2. 引入JdbcTemplate
  	3. 自动创建JdbcTemplate Bean
  -->
  <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-jdbc</artifactId>
  </dependency>
  <!--We add the H2 dependency to get both
          an in-memory database for this example and
          a JDBC driver to work with it.-->
  <dependency>
      <groupId>com.h2database</groupId>
      <artifactId>h2</artifactId>
      <!--The app only needs the database and the JDBC driver at runtime.
              The app doesn't need them for compilation-->
      <scope>runtime</scope>
  </dependency>
  ```

* Repository层

  ```java
  @Repository
  public class PurchaseRepository {
      private final JdbcTemplate jdbcTemplate;
  
      public PurchaseRepository(JdbcTemplate jdbcTemplate) {
          this.jdbcTemplate = jdbcTemplate;
      }
  
      public void storePurchase(Purchase purchase) {
          String sql = "INSERT INTO PURCHASE VALUES (NULL, ?, ?)";
          jdbcTemplate.update(sql, purchase.getProduct(), purchase.getPrice());
      }
  
      public List<Purchase> findAllPurchase() {
          String sql = "SELECT * FROM PURCHASE";
          // We implement a RowMapper object
          // that tells JdbcTemplate how to map a 
          // row in the result set into a Purchase
          // object. In the lambda expression,
          // parameter “r” is the ResultSet (the
          // data you get from the database),
          // while parameter “i” is an int
          // representing the row number.
          // JdbcTemplate will use
          // this logic for each row in the
          // result set.
          RowMapper rowMapper = (r, i) -> {
              Purchase purchase = new Purchase();
              purchase.setId(r.getInt("id"));
              purchase.setProduct(r.getString("product"));
              purchase.setPrice(r.getBigDecimal("price"));
              return purchase;
          };
          return jdbcTemplate.query(sql, rowMapper);
      }
  }
  ```

* Controller层

  ```java
  @RestController
  @RequestMapping("/purchase")
  public class PurchaseController {
      private final PurchaseRepository purchaseRepository;
  
      public PurchaseController(PurchaseRepository purchaseRepository) {
          this.purchaseRepository = purchaseRepository;
      }
  
      @PostMapping
      public void storePurchase(@RequestBody Purchase purchase) {
          purchaseRepository.storePurchase(purchase);
      }
  
      @GetMapping
      public List<Purchase> findAllPurchase() {
          return purchaseRepository.findAllPurchase();
      }
  }
  ```

* 测试效果

  ![image-20251213132722562](asset/image-20251213132722562.png)

  ![image-20251213132855939](asset/image-20251213132855939.png)

​	

==注意：==

* repository是什么意思：

  A repository is a class responsible with working with a database.

* 为什么Purchase的price属性类型为BigDecimal，而不是double/float
  * 使用double/float在计算过程中可能会丧失精度，对精度敏感的属性如价格，都应该用BigDecimal类型

* JdbcTemplate实例是谁创建的 为什么可以直接注入

  * spring-boot-starter-jdbc依赖内部包含HikariCP data source依赖和spring-jdbc依赖（其中包含JdbcTemplate类）。

  * 当我们引入spring-boot-starter-jdbc依赖，实际上也引入了HikariCP data source依赖。

  * 如果我们没有引入其他的datasource依赖，也没有手动定义datasource bean，则Spring Boot自动装配会帮我们自动创建HikariCP data source的Bean。

  * 有了data source Bean，SpringBoot自动装配也会帮我们自动创建JdbcTemplate的Bean。

  * 因此我们可以在Repository类直接注入JdbcTemplate bean。

  * 又因为我们还引入了h2数据库依赖，这个依赖不仅包含h2依赖，还包含了h2 jdbc driver依赖。

    h2 jdbc driver会被注册为对象。

    ```xml
    <dependency>
        <groupId>com.h2database</groupId>
        <artifactId>h2</artifactId>
        <scope>runtime</scope>
    </dependency>
    ```

  * 则JdbcTemplate可以通过data source bean来获取connection，调用JDBC接口，最终由h2 jdbc driver对这些接口的实现，完成对数据库的操作。

* RowMapper是什么

  RowMapper是一个对象，通过对该对象的配置，可以指定SELECT SQL执行结果ResultSet如何和Java类进行映射。

  也就是如何将查询结果转化为目标的Java类对象。

* JDBCTemplate SELECT query steps：

  ![image-20251213141952645](asset/image-20251213141952645.png)

* Jdbc vs Jdbc Driver vs JdbcTemplate vs DriverManager vs DataSource

  * Jdbc是JDK定义的、用于和DB交互的接口

  * Jdbc Driver是各个数据库厂商对Jdbc接口的实现

  * JdbcTemplate是Spring提供的一个工具类，因为Jdbc的接口对于开发者来说并不方便使用，JdbcTemplate进行了进一步封装，对于开发者更加简单易用。【底层会从data source获取连接，调用Jdbc接口来实现和DB交互】

  * DriverManager用于管理所有的JdbcDriver，根据URL来分配请求给对应的Jdbc Driver。

    例如url为 `jdbc:mysql://localhost:3306`，就会找到MySQL的Driver来处理请求。

  * DataSource高效管理连接，尽量复用，必要时才创建新连接，提升性能。

## 12.3 自定义数据源

12.2的例子中，我们使用的是H2内存数据库，该数据库仅适用于教学，但是在真正实践中，我们通常使用MySQL、Oracle等可以进行持久化的数据库，那我们应该如何定义数据源？下面通过将12.2中的例子中的H2数据库改为使用MySQL数据库来介绍，总共分三步：

1. 引入MySQL JDBC driver
2. 在application.properties配置文件中配置连接信息（url、username、password...）
3. 让SpringBoot根据配置信息生成默认的DataSource Bean / 自定义DataSource Bean



1. 引入MySQL JDBC driver

   总共需要以下依赖

   ```xml
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-web</artifactId>
   </dependency>
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-jdbc</artifactId>
   </dependency>
   <dependency>
       <groupId>mysql</groupId>
       <artifactId>mysql-connector-java</artifactId>
       <scope>runtime</scope>
   </dependency>
   ```

2. 在application.properties配置文件中配置连接信息（url、username、password...）

   ```properties
   # defines the location to the database.
   spring.datasource.url=jdbc:mysql://localhost/spring_start_here?useLegacyDatetimeCode=false&serverTimezone=UTC
   spring.datasource.username=root
   spring.datasource.password=password
   # We set the initialization mode to 
   # “always” to instruct Spring Boot to run
   # the queries in the “schema.sql” file
   spring.sql.init.mode=always
   ```

   注意：如果是使用SpringBoot自动装配来创建DataSource Bean，则配置的前缀必须是`spring.datasource.*`，这是 Spring Boot 官方定义的标准前缀。

   

3. 让SpringBoot根据配置信息生成默认的DataSource Bean（自动装配） / 自定义DataSource Bean

   1. SpringBoot根据application.properties文件中的`spring.datasource`配置自动创建DataSource Bean（自动装配）。

      如果要使用SpringBoot默认生成的DataSource Bean，则不需要任何操作，到这里就配置好了MySQL数据源，就能够通过注入JDBCTemplate和MySQL数据库交互了。

      SpringBoot默认使用的是HikariCP DataSource。

      

   2. 自定义DataSource Bean

      *什么时候需要自定义DataSource Bean？*

      其中一个场景是多数据源场景：如果应用需要同时连多个数据库，则意味着需要多个DataSource Bean，而SpringBoot默认只会创建一个，因此在这种场景下，需要开发者手动创建。

      开发者手动创建之后，SpringBoot探测到已经有了，就不会进行自动创建。

      注意：在有多个Bean的情况下，注入的时候需要通过@Qualifier注解来指定注入的是哪个Bean。

      简单来说就是，当自动配置不能满足开发者的需求的时候，就可以自定义Bean。

      

      *如何自定义DataSource Bean？*

      ```java
      @Configuration
      public class ProjectConfig {
          @Value("${datasource.url}")
          private String datasourceUrl;
      
          @Value("${datasource.username}")
          private String datasourceUsername;
      
          @Value("${datasource.password}")
          private String datasourcePassword;
      
          //The method returns a DataSource object. If
          //Spring Boot finds a DataSource already exists in
          //the Spring context it doesn’t configure one.
          @Bean
          public DataSource dataSource() {
              //We’ll use HikariCP as the data source implementation
              // for this example. However, when you define the bean
              // yourself, you can choose other implementations if
              // your project requires something else.
              HikariDataSource dataSource = new HikariDataSource();
              dataSource.setJdbcUrl(datasourceUrl);
              dataSource.setUsername(datasourceUsername);
              dataSource.setPassword(datasourcePassword);
              dataSource.setConnectionTimeout(1000);
              return dataSource;
          }
      }
      ```

      ```properties
      # 由于是自定义Bean，这里的配置可以自定义，见明知义即可
      datasource.url=jdbc:mysql://localhost/spring_start_here?useLegacyDatetimeCode=false&serverTimezone=UTC
      datasource.username=root
      datasource.password=password
      ```

      至此，自定义MySQL数据源配置完成，可以通过注入JDBCTemplate和MySQL数据库交互了。

      自动装配的思想：引入依赖，SpringBoot会根据配置自动创建Bean，开发者可以直接注入使用。如果开发者自定义了Bean，则SpringBoot会使用这个Bean，不会再进行默认创建。也体现出了Spring convention over configuration的哲学。

# 13. 事务

## 13.1 事务是什么

事务是一组操作，这组操作要么全部执行成功，要么全部都不执行，没有中间状态。

例如A向B转账100元包含两个动作：A余额-100，B余额+100。事务就是这两个动作要么全部执行成功，要么全部不执行，不存在A扣款成功，但是B余额没有增加的情况。

## 13.2 在Spring中如何使用事务

使用@Transactional注解。

* 如果在一个方法上增加@Transactional，则这个方法组成一个事务，如果方法执行过程中抛出了运行时异常，则会进行回滚。只有整个方法执行成功，事务才会被提交。
* 如果在一个类上面增加@Transactional注解，其作用等同于在这个类中的所有方法上都增加这个注解。
* 当在方法和类上都使用了@Transactional注解，则方法上的注解优先级更高。

## 13.3 @Transactional注解的底层执行机制

Spring中@Transactional注解底层实现机制是AOP。

拦截所有增加了@Transactional注解的方法，在方法执行之前开启事务，在方法执行成功之后提交事务。如果捕获到了运行时异常，则进行回滚。

![image-20251218171046038](asset/image-20251218171046038.png)

==注意：切面必须要捕获到方法抛出的异常，才会进行回滚。如果在transferMoney方法内部就catch了异常并进行了处理，且没有将异常throw出去，则切面是捕获不到异常的，因此事务也会成功提交。==

![image-20251218171633988](asset/image-20251218171633988.png)

## 13.4 哪种类型的异常会进行回滚

Java有两种异常：`RuntimeException` 和 `Checked Exception` 

Spring的事务机制是只有当捕获到 `RuntimeException` 异常才会进行回滚。

* `RuntimeException` 和 `Checked Exception` 的区别是什么

  * `Checked Exception`：编译器强制检查。必须使用 `try-catch` 捕获，或者在方法签名中用 `throws` 声明。

    例如：这段代码会报红，无法编译通过，必须处理它可能抛出的FileNotFoundException异常

    ![image-20251218192801689](asset/image-20251218192801689.png)

    只有try-catch对该异常进行了处理，或者将该异常throw出去，才能够编译通过

    ![image-20251218193042296](asset/image-20251218193042296.png)

  * `RuntimeException`：编译器不检查。不强制要求处理，代码可以正常编译。

    例如下面这段代码，除0运行时会抛出异常，但这属于运行时异常，编译器不会检查。

    ![image-20251218193238793](asset/image-20251218193238793.png)

* 为什么仅当捕获到 `RuntimeException` 异常才会进行回滚。

  因为 `Checked Exception` 类型的异常，Spring认为开发者是在编写代码的时候就已经意识到了该种异常会有可能抛出，因此Spring认为开发者会通过代码逻辑来指定后续操作（修复/手动撤回），因此这属于可控事件。

  Spring认为就算抛出了这种异常，也是在开发者意料之中的，并不属于意料之外的异常，因此不会进行回滚。

  其设计理念是：`Checked Exception`异常被视为一种“可预测的场景”，应该由开发者的逻辑去管理，而不是由事务框架强制拦截。

  而 `RuntimeException` 异常，不是在开发者意料之中的异常，因此如果抛出了这类异常，统一进行回滚处理。

* 如何自定义对于哪些类型的异常进行回滚

  如果你希望对于任何异常，都要进行回滚，则可以使用@Transactional注解的rollbackFor属性来显式指定抛出哪些类型的异常时进行回滚。

  ```java
  // 告诉 Spring：不管是运行时异常还是受检异常，只要报错，全给我回滚！
  @Transactional(rollbackFor = Exception.class) 
  public void saveOrder() {
      // 业务逻辑
  }
  ```

## 13.5 代码实践

定义一个转账系统，该系统提供两种功能：

1. 查询所有账户信息
2. 转账

![image-20251218202826054](asset/image-20251218202826054.png)



代码实现：

* 数据库表设计

  * account表包含三列

    * id - 自增主键
    * name - 用户名
    * amount - 余额

  * 使用H2数据库

    * schema.sql 启动时自动执行该sql语句创建数据表

      ```sql
      create table account (
          id INT NOT NULL AUTO_INCREMENT PRIMARY KEY,
          name VARCHAR(50) NOT NULL,
          amount DOUBLE NOT NULL
      );
      ```

    * data.sql 启动时自动执行该sql语句插入新数据

      ```sql
      INSERT INTO account VALUES (NULL, 'Helen Down', 1000);
      INSERT INTO account VALUES (NULL, 'Peter Read', 1000);
      ```

* model

  * Account

    ```java
    public class Account {
        private long id;
        private String name;
        private BigDecimal amount;
    
        // omitted getter & setter
    }
    
    ```

* pom.xml

  ```xml
  <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-jdbc</artifactId>
  </dependency>
  <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
  </dependency>
  <dependency>
      <groupId>com.h2database</groupId>
      <artifactId>h2</artifactId>
      <scope>runtime</scope>
  </dependency>
  ```

* repository层

  * findAccountById(long id) - 根据id获取Account信息

  * updateAmount(long id, BigDecimal amount) - 更新余额信息

  * findAllAccount() - 获取所有Account信息

    ```java
    @Repository
    public class AccountRepository {
        private final JdbcTemplate jdbcTemplate;
    
        public AccountRepository(JdbcTemplate jdbcTemplate) {
            this.jdbcTemplate = jdbcTemplate;
        }
    
        public Account findAccountById(long id) {
            String sql = "SELECT * FROM account WHERE id = ?";
            return jdbcTemplate.queryForObject(sql, new AccountRowMapper(), id);
        }
    
        public void updateAmount(long id, BigDecimal amount) {
            String sql = "UPDATE account SET amount = ? WHERE id = ?";
            jdbcTemplate.update(sql, amount, id);
        }
    
        public List<Account> findAllAccount() {
            String sql = "SELECT * FROM account";
            return jdbcTemplate.query(sql, new AccountRowMapper());
        }
    }
    ```

  * rowMapper

    * AccountRowMapper

      ```java
      public class AccountRowMapper implements RowMapper<Account> {
          @Override
          public Account mapRow(ResultSet rs, int rowNum) throws SQLException {
              Account account = new Account();
              account.setId(rs.getInt("id"));
              account.setName(rs.getString("name"));
              account.setAmount(rs.getBigDecimal("amount"));
              return account;
          }
      }
      ```

      

* service层

  * getAllAccount() - 获取所有account信息

  * transferMoney(long senderId, long receiverId, double amount) - 进行转账

    ```java
    @Service
    public class AccountService {
        private final AccountRepository accountRepository;
    
        public AccountService(AccountRepository accountRepository) {
            this.accountRepository = accountRepository;
        }
    
        public List<Account> getAllAccount() {
            return accountRepository.findAllAccount();
        }
    
        @Transactional  // 涉及多次DB操作 这多个操作组成一个事务
        public void transferMoney(long senderId, long receiverId, BigDecimal amount) {
            // 1. 获取sender信息
            Account senderAccount = accountRepository.findAccountById(senderId);
            // 2. 获取receiver信息
            Account receiverAccount = accountRepository.findAccountById(receiverId);
            // 3. 计算sender最新余额
            BigDecimal senderNewAmount = senderAccount.getAmount().subtract(amount);
            // 4. 计算receiver最新余额
            BigDecimal receiverNewAmount = receiverAccount.getAmount().add(amount);
            // 3. 更新sender余额
            accountRepository.updateAmount(senderId, senderNewAmount);
            // 4. 更新receiver余额
            accountRepository.updateAmount(receiverId, receiverNewAmount);
        }
    }
    ```

* controller层

  * GET - /accounts - getAllAccount() - 获取所有account信息

  * POST - /transfer - transferMoney(@RequestBody TransferRequest transferRequest) - 进行转账

    ```java
    @RestController
    public class AccountController {
        private final AccountService accountService;
    
        public AccountController(AccountService accountService) {
            this.accountService = accountService;
        }
    
        @GetMapping("/accounts")
        public List<Account> getAllAccount() {
            return accountService.getAllAccount();
        }
    
        @PostMapping("/transfer")
        public void transferMoney(@RequestBody TransferRequest transferRequest) {
            accountService.transferMoney(transferRequest.getSenderId(),
                    transferRequest.getReceiverId(),
                    transferRequest.getAmount());
        }
    }
    ```

* DTO

  The TransferRequest object simply models the HTTP request body. Such objects, whose responsibility is to model the data transferred between two apps, are DTOs.

  * TransferRequest

    ```java
    public class TransferRequest {
        private long senderId;
        private long receiverId;
        private BigDecimal amount;
    
        // omitted getter & setter
    }
    ```

* 测试效果

  * 事务内部无异常

    转帐前

    ![image-20251218202159814](asset/image-20251218202159814.png)

    Hellen向Peter转账100元

    ![image-20251218202218715](asset/image-20251218202218715.png)

    转账成功后

    ![image-20251218202229100](asset/image-20251218202229100.png)

  * 事务内部抛出异常

    在service方法手动抛出异常

    ```java
    @Transactional  // 涉及多次DB操作 这多个操作组成一个事务
    public void transferMoney(long senderId, long receiverId, BigDecimal amount) {
        // 1. 获取sender信息
        Account senderAccount = accountRepository.findAccountById(senderId);
        // 2. 获取receiver信息
        Account receiverAccount = accountRepository.findAccountById(receiverId);
        // 3. 计算sender最新余额
        BigDecimal senderNewAmount = senderAccount.getAmount().subtract(amount);
        // 4. 计算receiver最新余额
        BigDecimal receiverNewAmount = receiverAccount.getAmount().add(amount);
        // 3. 更新sender余额
        accountRepository.updateAmount(senderId, senderNewAmount);
        // 4. 更新receiver余额
        accountRepository.updateAmount(receiverId, receiverNewAmount);
    
        throw new RuntimeException("oops..出现了异常");
    }
    ```

    转帐前

    ![image-20251218202531533](asset/image-20251218202531533.png)

    Hellen向Peter转账100元（抛出异常）

    ![image-20251218202542817](asset/image-20251218202542817.png)

    ![image-20251218202557241](asset/image-20251218202557241.png)

    转账失败后（数据进行了回滚，和事务前完全一致。）

    ![image-20251218202641183](asset/image-20251218202641183.png)



# 14. Spring Data

## 14.1 Spring Data是什么

- 在实际开发中，实现持久层可以采用多种不同的技术方案。
   对于关系型数据库，常见的访问方式包括 JDBC、Hibernate、MyBatis 等；
   对于 MongoDB 这类非关系型数据库，则需要使用另一套专门的数据访问技术；
   而对于 Neo4j 等图数据库，又会对应一套全新的访问方式。

  总体来说，不同类型的数据库通常需要调用不同库提供的 API，并采用各自不同的连接和访问方式。即使是同一种数据库，也往往存在多种访问路径，例如既可以直接通过 JDBC 连接 MySQL，也可以通过 Hibernate 或 MyBatis 这类基于 JDBC 封装的框架来进行访问。

  因此，当项目需要同时接入多种类型的数据库时，开发者通常需要分别学习并使用多套数据访问技术，持久层的复杂度和学习成本都会显著提高。

  ![image-20251221122004290](asset/image-20251221122004290.png)

- Spring Data 在各类数据库访问技术之上提供了一层统一的抽象，通过一组风格一致的 Repository 接口，屏蔽了不同数据库和不同数据访问技术之间的底层 API 差异。
   无论底层使用的是基于 JDBC 的关系型数据库访问方式，还是基于 Hibernate 的 JPA，亦或是用于 MongoDB 等非关系型数据库的访问技术，Spring Data 都提供了相似的 Repository 接口模型和使用方式。

  通过使用 Spring Data，开发者可以以统一的接口和编程模型来编写持久层代码，从而减少对具体数据库驱动和访问 API 的直接依赖。当底层数据存储发生变化时，例如从 MySQL 切换到 MongoDB，只需引入对应的 Spring Data 模块（如从 Spring Data JDBC 切换为 Spring Data MongoDB），上层基于 Repository 的代码通常可以保持较高的复用度。
   相比之下，如果不使用 Spring Data，开发者往往需要直接面对 JDBC 或各类数据库驱动提供的不同 API，在切换数据存储类型时，持久层代码的改动成本会显著增加。

  ![image-20251221145959302](asset/image-20251221145959302.png)

- Spring Data 的本质是在不同数据库访问技术之上提供了一层统一的抽象，使开发者无需直接关心底层数据库驱动和具体 API 的调用方式，只需通过 Spring Data 提供的统一接口来完成数据访问操作。
   通过这种方式，开发者可以将更多精力放在业务逻辑的实现上，而不必过多关注底层数据访问细节，从而提升开发效率和代码的可维护性。

## 14.2 Spring Data如何工作

1. Spring Data 并不是单一的一个依赖包，而是一个“伞式”项目，包含多个针对不同数据存储技术的模块。
   换句话说，并不存在一个叫做 `spring-data` 的统一依赖，而是根据具体的数据存储类型选择对应的 Spring Data 模块。

   例如：

   - 如果要访问关系型数据库，可以引入 **Spring Data JDBC** 或 **Spring Data JPA** 的依赖；
   - 如果要访问 MongoDB，则可以引入 **Spring Data MongoDB** 的依赖。

   Spring Data 支持的完整模块列表可以在官方文档网站查看`https://www.spring-doc.cn/projects/spring-data#overview`。

2. Spring Data底层是如何工作的？

   它的底层定义了三个基础接口，关系如图：![image-20251221163414463](asset/image-20251221163414463.png)这三个接口的区别是：

   * `Repository` 接口本身并不定义任何方法，它仅作为一个**标记接口（marker interface）**存在。一个接口如果继承了 `Repository`，仅表示它是一个 Spring Data Repository。通常情况下，我们不会直接继承该接口。

     ```java
     @Indexed
     public interface Repository<T, ID> {
     }
     ```

   * `CrudRepository` 在 `Repository` 的基础上定义了常用的 **CRUD（增删改查）** 方法。一个接口只要继承了 `CrudRepository`，就可以自动获得这些基础的数据访问能力。

     ```java
     @NoRepositoryBean
     public interface CrudRepository<T, ID> extends Repository<T, ID> {
         <S extends T> S save(S entity);
     
         <S extends T> Iterable<S> saveAll(Iterable<S> entities);
     
         Optional<T> findById(ID id);
     
         boolean existsById(ID id);
     
         Iterable<T> findAll();
     
         Iterable<T> findAllById(Iterable<ID> ids);
     
         long count();
     
         void deleteById(ID id);
     
         void delete(T entity);
     
         void deleteAllById(Iterable<? extends ID> ids);
     
         void deleteAll(Iterable<? extends T> entities);
     
         void deleteAll();
     }
     ```

     

   * `PagingAndSortingRepository` 又在 `CrudRepository` 的基础上，进一步提供了 **分页和排序** 相关的方法，适用于需要分页查询或排序能力的场景。

     ```java
     @NoRepositoryBean
     public interface PagingAndSortingRepository<T, ID> extends CrudRepository<T, ID> {
         Iterable<T> findAll(Sort sort);
     
         Page<T> findAll(Pageable pageable);
     }
     ```

     在使用 Spring Data 时，通常会定义一个 Repository 层的 接口，并继承 Spring Data 提供的这三个 Repository 接口之一。
      开发者无需为该接口编写具体实现，Spring Data 会在运行时自动生成对应的实现类，并将其实例注册为 Spring 容器中的 Bean，从而可以在应用的其他位置通过依赖注入的方式直接使用。

     例如，如果我们定义了一个接口 A 并继承自 `CrudRepository`，Spring Data 会自动为接口 A 生成一个实现类，该实现类会实现接口中定义的所有方法。同时，Spring Data 还会负责创建该实现类的实例并将其纳入 Spring 容器管理。在 Service 层中，我们只需注入该 Repository Bean，即可完成对数据的访问操作。

3. 当然这些接口仅仅提供了一些最简单、最常用的方法，如果有特殊需求，当然也是可以自定义方法的。

   在自定义接口中可以定义方法，通过@Query注解声明SQL语句，如果是涉及DB修改的操作，还需要增加上@Modifying注解。

   在14.3会有具体的示例介绍。

4. 对于一些数据库，它有一些更加特殊的使用方式，Spring Data也提供了对应的接口。

   例如，当我们引入 Spring Data MongoDB 时，在 Spring Data 提供的通用 Repository 接口（如 `CrudRepository`、`PagingAndSortingRepository`）之上，还会额外提供一个 `MongoRepository` 接口。
    `MongoRepository` 继承自 `PagingAndSortingRepository`，并在此基础上扩展了一些 MongoDB 特有的能力，这些能力是其他关系型数据库持久化技术所不具备的。

   在实际应用中，如果系统使用的是 MongoDB，通常会涉及到这些与 MongoDB 特性强相关的使用方式。因此，在定义持久层接口时，往往会选择继承 `MongoRepository`，以便直接使用其提供的 MongoDB 专有方法，而不是仅依赖通用的 Repository 接口。![image-20251221165058065](asset/image-20251221165058065.png)

   

==注意：==

* Spring Data JDBC、Spring Data JPA的关系是什么

  * ORM是什么

    * ORM（Object-Relational Mapping，对象关系映射）是一种编程技术，它在面向对象语言（如Java、Python）的**对象**和关系型数据库（如MySQL、PostgreSQL）的**数据表**之间建立映射关系，让开发者能用操作对象的方式来操作数据库，而无需直接写繁琐的SQL语句，从而减少重复代码、提高开发效率。它将数据库的行、列、表等概念映射到类、属性、对象，实现数据的自动持久化（增删改查）。
    * ORM的核心思想与工作原理
      1. 映射关系：定义一个类（Class）对应数据库中的一个表（Table），类的属性（Attributes）对应表中的字段（Columns），类的对象（Object）代表表中的一行数据（Row）。
      2. 自动转换：当程序操作对象时（例如，调用一个`save()`方法），ORM框架会把对象的状态转换为对应的SQL语句，然后执行它；反之，从数据库查询数据时，ORM会把结果集转换成对象。
      3. 隐藏SQL：开发者主要与对象和类打交道，ORM框架在底层负责生成和执行SQL，使得数据访问变得更加简单、安全。 

  * JPA是什么

    ORM是一种思想，JPA是Java官方定义的一套ORM规范。

    举例，JPA定义了Entity类必须要满足的要求：

    1. 必须要用@Entity注解声明
    2. 必须要有public/protected类型的无参构造器
    3. 不能是final类
    4. ...

  * Hibernate是什么

    JPA是一套ORM规范，Hibernate是JPA的实现

  * JDBC 和 JPA的区别

    * JDBC是Java定义的一系列操作DB的API抽象
    * JPA（Java Persistence API）是Java的ORM规范， JPA底层依然是使用JDBC来实现DB操作，只不过JPA在此基础上，实现了ORM思想，以提升开发效率。

  * Spring Data JDBC和Spring Data JPA的关系

    这是两种持久层的实现。

    如果你想用JDBC实现持久层，则引入Spring Data JDBC。

    如果你想用JPA实现持久层，则引入Spring Data JPA，Spring Data JPA底层默认使用Hibernate实现。

* Spring Data中的Repository接口和@Repository的关系

  没有关系

  @Repository注解是标识告知Spring要创建这个类的bean并加入Spring容器。

  Spring Data中的Repository接口就是一个接口。

* Hibernate、MyBatis是什么关系

  Hibernate 和 MyBatis 都是用于数据库访问的 Java 持久层框架，它们的共同目标都是提升开发效率，避免开发者直接使用繁琐的 JDBC API。

  * Hibernate基于ORM思想实现，思维方式是：操作对象，SQL的生成由框架负责。【面向对象的ORM思路】
  * Mybatis不基于ORM实现，思维方式是：以SQL为核心，由开发者编写SQL，框架只负责参数绑定和结果映射。【面向SQL的映射思路】
  * 选型：Mybatis适合SQL复杂、性能要求高的场景。SQL复杂，手写可控性强；性能要求高，手写SQL更容易做针对性优化。Hibernate适合以简单的CRUD为主，追求开发效率的业务场景。

## 14.3 Spring Data的使用

通过一个示例来介绍Spring Data的使用。

* 业务需求：

  实现一个电子钱包。能够实现：

  1. 转账
  2. 查询所有用户的余额信息
  3. 根据name查询用户的余额信息

* 代码设计

  * 数据库设计

    以教学为目的，使用h2。

    * schema.sql

      ```sql
      create table account (
           id INT NOT NULL AUTO_INCREMENT PRIMARY KEY,
           name VARCHAR(50) NOT NULL,
           amount DOUBLE NOT NULL
      );
      ```

    * data.sql 初始时插入两条数据便于演示

      ```sql
      INSERT INTO account VALUES (NULL, 'Jane Down', 1000);
      INSERT INTO account VALUES (NULL, 'John Read', 1000);
      ```

  * pom.xml

    引入三个依赖：

    1. web因为我们要提供restful接口供外界调用 
    2. spring-data-jdbc 因为我们要用jdbc来实现和db的连接
    3. h2数据库的依赖 内部已包含h2 jdbc driver

    ```xml
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jdbc</artifactId>
    </dependency>
    <dependency>
        <groupId>com.h2database</groupId>
        <artifactId>h2</artifactId>
        <scope>runtime</scope>
    </dependency>
    ```

  * model

    Account（和account表对应）

    ```java
    public class Account {
        @Id
        private long id;
        private String name;
        private BigDecimal amount;
        
        // omitted getters & setters
    }
    ```

    ==使用SpringData，需要用@Id注解标识哪一个字段对应表的主键。==

  * repository层

    继承CrudRepository接口，需要提供两个泛型类型：<表对应的model类, 主键类型> --> \<Account, id>

    

    * CrudRepository接口已经提供了的方法，我们可以直接使用。
    * CrudRepository接口没有提供的方法，我们可以自定义。
      * 通过@Query注解来标识SQL语句；
      * 如果是增删改这类数据库的更改操作，需要进一步增加@Modifying注解。
      * 注意SQL语句中`:name` 的`:`和`name`之间没有空格
      * @Param负责将Java参数和SQL语句中的占位符进行对应

    ```java
    public interface AccountRepository extends CrudRepository<Account, Long> {
    
        @Query("SELECT * FROM account WHERE name = :name")
        public List<Account> findByName(@Param("name") String name);
    
        @Modifying
        @Query("UPDATE account SET amount = :amount where id = :id")
        public void updateAmount(@Param("id") long id, @Param("amount") BigDecimal amount);
    
    }
    ```

    继承了SpringData提供的接口之后，SpringData会自动创建实现类，并且创建一个Bean添加到Spring容器中。

  * service层

    ```java
    @Service
    public class TransferService {
        private final AccountRepository accountRepository;
    
        public TransferService(AccountRepository accountRepository) {
            this.accountRepository = accountRepository;
        }
    
        /**
         * 转账
         *
         * @param senderId 转出者Id
         * @param receiverId 接收者Id
         * @param amount 转账金额
         */
        @Transactional
        public void transferMoney(long senderId, long receiverId, BigDecimal amount) {
            // 1. 查询转出者和接收者的信息
            Account sender = accountRepository.findById(senderId)
                    .orElseThrow(() -> new AccountNotExistException());
            Account receiver = accountRepository.findById(receiverId)
                    .orElseThrow(() -> new AccountNotExistException());
            // 2. 计算转出者和接收者的最新余额信息
            BigDecimal senderNewAmount = sender.getAmount().subtract(amount);
            BigDecimal receiverNewAmount = sender.getAmount().add(amount);
            // 3. 更新两者的余额
            accountRepository.updateAmount(senderId, senderNewAmount);
            accountRepository.updateAmount(receiverId, receiverNewAmount);
        }
    
        /**
         * 获取所有Account信息
         * @return Account信息列表
         */
        public Iterable<Account> findAllAccounts() {
            return accountRepository.findAll();
        }
    
        /**
         * 根据用户名查询Account信息
         * @param name 用户名
         * @return Account信息列表
         */
        public List<Account> findAccountsByName(String name) {
            return accountRepository.findByName(name);
        }
    }
    ```

    ```java
    public class AccountNotExistException extends RuntimeException{
    }
    ```

  * controller层

    ```java
    @RestController
    public class AccountController {
        private final TransferService transferService;
    
        public AccountController(TransferService transferService) {
            this.transferService = transferService;
        }
    
        @PostMapping("/transfer")
        public void transferMoney(@RequestBody TransferRequest transferRequest) {
            transferService.transferMoney(transferRequest.getSenderId(),
                    transferRequest.getReceiverId(),
                    transferRequest.getAmount());
        }
    
        @GetMapping("/accounts")
        public Iterable<Account> findAccounts(@RequestParam(required = false) String name) {
            if (name == null) {
                return transferService.findAllAccounts();
            } else {
                return transferService.findAccountsByName(name);
            }
        }
    }
    ```

  * dto

    ```java
    public class TransferRequest {
        private long senderId;
        private long receiverId;
        private BigDecimal amount;
    
        // Omitted getters and setters
    }
    
    ```

* 实现效果：

  转帐前

  ![image-20251221182352548](asset/image-20251221182352548.png)

  转账

  ![image-20251221182412052](asset/image-20251221182412052.png)

  转账后

  ![image-20251221182422421](asset/image-20251221182422421.png)

  根据用户名查询

  ![image-20251221182440282](asset/image-20251221182440282.png)

==注意：==

* SpringData能够根据方法名自动推测SQL语句

  在上面的例子中，如果我们想在CrudRepository接口定义的方法之外再自定义一些方法，我们是通过定义方法，然后通过@Query注解来指定对应的SQL语句。

  其实我们还可以通过规范的方法名，让Spring Data明白我们的意图，这样甚至连SQL语句都不用写了。

  例如定义一个通过Name来查找Account信息的方法

  ```java
  public interface AccountRepository
  extends CrudRepository<Account, Long> {
      List<Account> findAccountsByName(String name);
  }
  ```

  通过定义这个方法名，Spring Data能够自动解析出对应的SQL语句

  ![image-20251221203728801](asset/image-20251221203728801.png)

  这种方法虽然很简单，但是并不推荐使用：

  * 如果我们想实现的操作比较复杂，则方法名会非常长且复杂，可读性不佳
  * 如果有人不小心修改了方法名，对背后机制的影响并不明显可见，可能导致出现潜在的bug
  * 拖慢启动的速度，启动时Spring Data需要将方法名翻译为SQL语句，影响性能

  因此，最好的方式还是我们显式地将SQL语句写出来，这样可读性也更加好。



# 15. Testing your Spring App

## 15.1 测试名词

* Regression Testing 回归测试

  * 是什么：确保在修改代码/修复bug/增加新功能之后，旧功能依然能够正常工作，没有被“改坏”。

  * 策略：由于项目越来越大，每次改动都把成千上万个测试用例跑一遍是不现实的。

    * **完全回归测试**：跑完所有的测试用例。安全但耗时极长，通常只在发布大版本前执行。
    * **选择性回归测试**：只跑那些“可能会受到影响”的相关模块。
    * **基于优先级的回归测试**：优先跑核心业务流程（如登录、下单、支付），次要功能排后。

  * 自动化：回归测试的一个显著特点是 **“重复性”**。同样的用例，每次改代码都要跑一遍。

    * **人工回归**：极其枯燥且容易出错（人会疲劳）。

      **自动化回归**：编写测试脚本（如 JUnit, Selenium 等）。一旦代码提交，机器几分钟内就能跑完所有核心流程。

* 单元测试：验证最小单位的功能块（通常是方法）是否正确运行

* 冒烟测试：验证系统最核心的功能（能不能启动、能不能登录）是否可用。

* Integration Testing 集成测试

  * 是什么：验证不同模块（组件）组合在一起后，它们之间的交互和接口是否正确。如果说单元测试是检查“每个零件”是否合格，那么集成测试就是检查这些零件“拼在一起”后能不能正常运转。
  * 为什么要进行集成测试：即使每个单元（类、方法）单独测试都是完美的，但它们组合在一起时仍可能出错，常见原因包括：
    - **数据流丢失**：模块 A 传给模块 B 的参数格式不对（例如 A 传了 String，B 接收 Long类型）。
    - **接口不匹配**：开发者对接口定义的理解不一致。
    - **外部依赖问题**：无法与数据库、缓存或第三方 API 正确通信。
    - **框架配置错误**：比如 Spring 的 Bean 注入失败、事务配置没起作用。

* Spring Integration Test：在Spring容器环境下进行的集成测试。

  * 特点：
    * **加载上下文**：它会启动 Spring 的 `ApplicationContext`。
    * **依赖注入**：它会自动把 `@Service`、`@Repository` 等组件装配在一起，就像真实运行程序一样。
    * **真实性**：你可以测试真实的数据库连接、事务管理（`@Transactional`）和 Bean 的生命周期。
  * 作用：验证当所有的代码组件（Beans）组合在一起并运行在 Spring 环境中时，它们是否能正确地协同工作。

* CI/CD

  * CI（Continuous Integration）持续集成：

    * 是什么：开发人员每天会多次将代码推送到主干仓库。每当代码更新时，CI 系统会自动执行以下操作：

      * **自动化构建**：编译代码，确保没有语法错误。
      * **自动化测试**：运行单元测试和集成测试。
      * **代码质量检查**：检查代码风格、潜在漏洞。

      ![image-20251223112245761](asset/image-20251223112245761.png)

    * 为什么：

      * **集成（Integration）**：在过去，程序员各自写代码，几个月才合并一次。合并时会发现成千上万个冲突，这个痛苦的过程被称为“集成地狱”。
      * **持续（Continuous）**：现在，我们不再等几个月，而是每写完一个小功能、甚至一个修复，就立刻合并。通过这种高频率，让“集成”变成一件日常的小事。

  * CD (Continuous Delivery) 持续交付：**确保代码随时可以发布。**在 CI 完成并测试通过后，持续交付会将代码自动部署到“类生产环境”（如测试环境或预发布环境）。

  * CD (Continuous Deployment) 持续部署：全自动化发布到生产环境，无需人工干预。



## 15.2 单元测试的正确思路

1. **单元测试代码写在哪里**

   在test文件夹中编写

   ![image-20251223114809217](asset/image-20251223114809217.png)

2. **单元测试代码怎么写（理论）**（实践在15.3）

   对于每一个要测试的方法，编写一个test class来专门负责这个方法的测试

   每一个要测试的方法，根据输入的不同，可能会有不同的执行情况。对于所有的执行情况，单元测试都应该要全部覆盖到。

   对于每一种情况，用一个测试方法验证。

   例如ch14中的转账方法transferMoney，执行逻辑包含4步：

   1. 根据senderId在Account表中查询sender信息
   2. 根据receiverId在Account表中查询receiver信息
   3. 分别sender和receiver转账后最新余额
   4. 更新sender和receiver的余额信息

   针对这一个要测试的方法，就可以找到以下多种测试场景：

   1. 查询不到sender信息
   2. 查询不到receiver信息
   3. sender转账余额不足
   4. 更新sender余额信息失败
   5. 更新receiver余额信息失败
   6. 转账执行成功

   那一个完整的单元测试，应该是一个类TransferMoneyUnitTest负责TransferMoney()方法的测试。这个类中包含6个方法，分别验证上面的6中场景的代码执行是否符合预期。

   ![image-20251223115736886](asset/image-20251223115736886.png)

## 15.3 编写单元测试

以编写transferMoney方法的单元测试为例：

transferMoney方法

```java
@Service
public class TransferService {
    private final AccountRepository accountRepository;

    public TransferService(AccountRepository accountRepository) {
        this.accountRepository = accountRepository;
    }

    @Transactional
    public void transferMoney(long senderId,
                              long receiverId,
                              BigDecimal amount) {
        // 1. 获取sender和receiver信息
        Account sender = accountRepository.findById(senderId)
                .orElseThrow(() -> new AccountNotFoundException());
        Account receiver = accountRepository.findById(receiverId)
                .orElseThrow(() -> new AccountNotFoundException());
        // 2. 计算sender和receiver转账后的余额信息
        BigDecimal senderNewAmount = sender.getAmount().subtract(amount);
        BigDecimal receiverNewAmount = receiver.getAmount().add(amount);
        // 3. 更新sender和receiver余额信息
        accountRepository.changeAmount(senderId, senderNewAmount);
        accountRepository.changeAmount(receiverId, receiverNewAmount);
    }
}
```

有下列6中情形，每一种情形都需要写一个对应的单测方法。

1. 查询不到sender信息
2. 查询不到receiver信息
3. sender转账余额不足
4. 更新sender余额信息失败
5. 更新receiver余额信息失败
6. 转账执行成功

### happy flow

先以【转账执行成功】这个最容易想到的情形来编写单测，编写一个单测方法总共需要三步：

1. given: 准备前提条件。
   1. 这个方法的input参数
   2. Object instances the method uses but that are not created by it
2. when: 执行这个方法。
3. then:验证结果是否符合预期



1. given: 准备前提条件。

   1. 这个方法的input参数：senderId、receiverId、amount
   2. Object instances the method uses but that are not created by it：accountRepository

   ![image-20251223163441180](asset/image-20251223163441180.png)

   这里我们只想测试service层的transferMoney()这个方法，但这个方法本身依赖accountRepository这个bean执行findById这个方法的结果。

   我们并不关注accountRepository.findById这个方法的执行如何。因为单元测试是关注最小单元，当前我们关注的最小单元就是service层的transferMoney()这个方法。

   因此我们不用注入真实的accountRepository bean，而是mock一个bean，这个mocked bean的行为能够被完全控制，因此这个单测方法可以仅专注于transferMoney这个方法的执行。

   ![image-20251223165438946](asset/image-20251223165438946.png)

   mock出了accountRepository这个bean之后，怎么控制它的行为？

   我们可以通过==given()方法==，来控制其行为

   ![image-20251223165542394](asset/image-20251223165542394.png)

2. when: 执行这个方法

3. then: 预期结果是怎么样的。

   预期的结果是accountRepository会以正确的参数调用changeAmount方法，用==verify()方法==进行校验校验

   ![image-20251223170508341](asset/image-20251223170508341.png)

   感觉这里似乎并不符合直觉，直觉应该是校验sender的amount是否变成了900以及receiver的amount是否变成了1100。

   但实际上，如果我们执行`assertEquals(new BigDecimal(900), sender.getAmount());`结果一定是失败的，因为

   第一，在transferMoney中，我们是获取sender信息-->计算最新余额-->直接更新DB中sender这条记录的最新余额。所以本质上，代码中并没有更新sender对象的amount属性值。

   ```java
   @Transactional
   public void transferMoney(long senderId,
                             long receiverId,
                             BigDecimal amount) {
       // 1. 获取sender和receiver信息
       Account sender = accountRepository.findById(senderId)
           .orElseThrow(() -> new AccountNotFoundException());
       Account receiver = accountRepository.findById(receiverId)
           .orElseThrow(() -> new AccountNotFoundException());
       // 2. 计算sender和receiver转账后的余额信息
       BigDecimal senderNewAmount = sender.getAmount().subtract(amount);
       BigDecimal receiverNewAmount = receiver.getAmount().add(amount);
       // 3. 更新sender和receiver余额信息
       accountRepository.changeAmount(senderId, senderNewAmount);
       accountRepository.changeAmount(receiverId, receiverNewAmount);
   }
   ```

   第二，因为我们用的accountRepository对象是mock出来的，是一个假的对象，因此它也不会去真正更新DB，也不会反过来更新sender对象。

   第三，`AccountRepository` 是 Service 的下级。 Service 的职责不是“改变账户余额”，而是“计算出余额并下达更新指令”。所以我们验证service，只需要验证service是否成功下达了正确的指令，而不是关注指令是否正确地被执行了，这是AccountRepository的单元测试需要关注的事。

   

总结一下最后的happyflow 单元测试代码：

```java
public class TransferMoneyUnitTest {
    @Test
    @DisplayName("Test the amount is transferred from one account to another if no exception occurs.")
    public void transferMoneyHappyFlow() {
        // 1. given
        AccountRepository accountRepository = mock(AccountRepository.class);
        TransferService transferService = new TransferService(accountRepository);

        Account sender = new Account();
        sender.setId(1);
        sender.setAmount(new BigDecimal(1000));

        Account receiver = new Account();
        receiver.setId(2);
        receiver.setAmount(new BigDecimal(1000));

        // 如果调用accountRepository.findById(sender.getId()) -> 就返回sender对象
        given(accountRepository.findById(sender.getId()))
                .willReturn(Optional.of(sender));
        given(accountRepository.findById(receiver.getId()))
                .willReturn(Optional.of(receiver));

        // 2. when
        transferService.transferMoney(sender.getId(), receiver.getId(), new BigDecimal(100));

        // 3. then
        verify(accountRepository)
                .updateAmount(sender.getId(), new BigDecimal(900));
        verify(accountRepository)
                .updateAmount(receiver.getId(), new BigDecimal(1100));

    }
}
```

除了调用Mock()方法之外，还能够通过注解的方式创建mock对象。

@Mock注解，mockito框架会自动创建一个mock对象，并注入对应的变量中。

@InjectMocks注解，mockito框架会自动创建要test的对象，并将被@Mock注解标注的对象都注入到被@InjectMocks注解标注的对象的属性中。按下面的例子，也就是把accountRepository注入到transferService的属性中去。

如果要@Mock和InjectMocks注解生效，还需要在单测类上增加@ExtendWith(MockitoExtension.class)注解。

```java
@ExtendWith(MockitoExtension.class)
public class TransferMoneyUnitTestWithAnnotation {
    @Mock
    private AccountRepository accountRepository;

    @InjectMocks
    private TransferService transferService;

    @Test
    public void transferMoneyHappyFlow() {

        // 1. given
        Account sender = new Account();
        sender.setId(1);
        sender.setAmount(new BigDecimal(1000));

        Account receiver = new Account();
        receiver.setId(2);
        receiver.setAmount(new BigDecimal(1000));

        // 如果调用accountRepository.findById(sender.getId()) -> 就返回sender对象
        given(accountRepository.findById(sender.getId()))
                .willReturn(Optional.of(sender));
        given(accountRepository.findById(receiver.getId()))
                .willReturn(Optional.of(receiver));

        // 2. when
        transferService.transferMoney(sender.getId(), receiver.getId(), new BigDecimal(100));

        // 3. then
        verify(accountRepository)
                .updateAmount(sender.getId(), new BigDecimal(900));
        verify(accountRepository)
                .updateAmount(receiver.getId(), new BigDecimal(1100));
        
    }
}
```



总结测试的三步走策略：

1. given: 准备前提条件。
   1. 这个方法的input参数
   2. Object instances the method uses but that are not created by it
2. when: 执行这个方法。
3. then:验证结果是否符合预期

![image-20251223174131241](asset/image-20251223174131241.png)

### exception flow

以【查询不到sender信息】为场景编写exception flow单测。

`assertThrows()`方法会调用目标方法，并验证是否抛出了符合预期的异常。

```java
@ExtendWith(MockitoExtension.class)
public class TransferMoneyUnitTestWithAnnotation {
    @Mock
    private AccountRepository accountRepository;

    @InjectMocks
    private TransferService transferService;

    @Test
    public void transferMoneyExceptionFlow() {
        //1. 控制mock的对象的行为
        Account sender = new Account();
        sender.setId(1);
        sender.setAmount(new BigDecimal(1000));

        given(accountRepository.findById(sender.getId()))
                .willReturn(Optional.of(sender));
        given(accountRepository.findById(2L))
                .willReturn(Optional.empty());

        // 2. 调用单测目标方法
        // 预期抛出AccountNotExistException.class异常
        assertThrows(AccountNotExistException.class,
                () -> transferService.transferMoney(sender.getId(), 2L, new BigDecimal(100))
        );

        // 3. 验证结果
        // We use the verify() method with the never() conditional to assert that
        // the changeAmount() method hasn't been called
        verify(accountRepository, never())
                .updateAmount(anyLong(), any());

    }
}
```

### 验证方法返回值符合预期

`assertEquals(Object excepted, Object actual);`这个方法能够校验方法返回值是否符合预期。

示例代码：为LoginController的loginPost方法编写单测

```java
@Controller
public class LoginController {

    private final LoginProcessor loginProcessor;

    LoginController(LoginProcessor loginProcessor) {
        this.loginProcessor = loginProcessor;
    }

    @PostMapping("/")
    public String loginPost(
            @RequestParam String username,
            @RequestParam String password,
            Model model
    ) {
        loginProcessor.setUsername(username);
        loginProcessor.setPassword(password);
        boolean loggedIn = loginProcessor.login();
        if (loggedIn) {
            model.addAttribute("message", "you are now logged in.");
        } else {
            model.addAttribute("message", "Login failed!");
        }
        return "login.html";
    }
}
```

1. given: 准备前提条件。

   1. 这个方法的input参数

      username, password

   2. Object instances the method uses but that are not created by it

      * LoginProcessor bean，需要控制 `loginProcessor.login();` 的执行结果
      * model 对象

2. when: 执行这个方法。

3. then:验证结果是否符合预期

   * 登陆成功
     * 执行了 `model.addAttribute("message", "you are now logged in.");`
     * loginPost方法返回了`login.html`字符串
   * 登陆失败
     * 执行了 ` model.addAttribute("message", "Login failed!");`
     * loginPost方法返回了`login.html`字符串

单测代码：

```java
@ExtendWith(MockitoExtension.class)
public class LoginControllerLoginPostUnitTests {
    @Mock
    private Model model;  // given
    @Mock
    private LoginProcessor loginProcessor;  // given
    @InjectMocks
    private LoginController loginController;  // given

    // 登陆成功
    @Test
    public void loginPostLoginSuccessTest() {
        // 1. given
        given(loginProcessor.login())
                .willReturn(true);
        // 2. when
        String result = loginController.loginPost("username", "password", model);
        // 3. then
        assertEquals("login.html", result);
        verify(model)
                .addAttribute("message", "you are now logged in.");
    }

    // 登陆失败
    @Test
    public void loginPostLoginFailTest() {
        // 1. given
        given(loginProcessor.login())
                .willReturn(false);
        // 2. when
        String result = loginController.loginPost("username", "password", model);
        assertEquals("login.html", result);
        // 3. then
        verify(model)
                .addAttribute("message", "Login failed!");
    }
}
```



## 15.4 编写集成测试

单元测试是用于测试代码逻辑的最小单元的（通常是一个方法）。

但是有的时候我们A单元和B单元分别测试通过，而A单元和B单元共同工作的时候，程序出现异常，这就是因为A单元和B单元没有正确地interact with each other。

集成测试就是用于确保多个组件/单元之间，能够正确地interact with each other。



在Spring项目中，我们会通过Spring integration test来测试Spring提供的功能是否和业务代码能够正常协作。

举例来说就是：依赖注入是否成功、事务是否生效...

我们依然以转账的例子，来写一个Spring integration test，以学习集成测试的思想。

通过这个SpringBoot集成测试，我们可以验证：

1. TransferService Bean被正确注入Spring容器
2. TransferService的依赖注入配置正确（AccountRepository bean被正确注入TransferService的属性）
3. 在Spring环境下TtransferMoney方法成功执行

```java
// 开启SpringBoot测试框架。
// 添加这个注解，等同于告诉JUnit，不要只运行这个类，请启动整个 Spring 容器（ApplicationContext），
// 就像我平时运行 main 方法启动程序那样。
@SpringBootTest
public class TransferMoneySpringIntegrationTest {
    // 在 Spring 容器中创建一个 Mock 对象，并用它替换掉容器中原本真实的 Bean
    // 如果Spring容器中没有AccountRepository类型Bean，则将新的直接放入容器
    // 如果已经有了，则将其踢出，放入这个Mock的假Bean
    // 容器中所有依赖这个Bean的地方，拿到的都会是这个mock假bean
    @MockBean
    private AccountRepository accountRepository;

    // 注入Spring容器中，真正的transferService Bean
    // 其属性，被注入的是上面的mock accountRepository bean
    @Autowired
    private TransferService transferService;

    @Test
    @DisplayName("Test the amount is transferred " +
"from one account to another if no exception occurs.")
    public void transferMoneyHappyFlow() {
        // 1. given
        Account sender = new Account();
        sender.setId(1);
        sender.setAmount(new BigDecimal(1000));

        Account receiver = new Account();
        receiver.setId(2);
        receiver.setAmount(new BigDecimal(1000));

        given(accountRepository.findById(sender.getId()))
                .willReturn(Optional.of(sender));

        given(accountRepository.findById(receiver.getId()))
                .willReturn(Optional.of(receiver));

        // 2. when
        transferService.transferMoney(sender.getId(), receiver.getId(), new BigDecimal(100));

        // 3. then
        verify(accountRepository)
                .updateAmount(sender.getId(), new BigDecimal(900));
        verify(accountRepository)
                .updateAmount(receiver.getId(), new BigDecimal(1100));
    }
}
```



为什么要mock一个假的AccountRepository，而不是用Spring容器中那个真的？

* 这个测试的重点在于，TransferService是否和AccountRepository正确协作，也就是TransferService是否向AccountRepository发出正确的指令。只要AccountRepository成功接受到正确指令即可，至于AccountRepository执行什么逻辑，是否执行正确，是AccountRepository自己的单元测试需要关注的内容。
* 如果用真的AccountRepository，则必须在数据库准备完整的测试数据。Mock可以完全控制AccountRepository的行为，让测试更加简单。
* 真实Bean很难模拟出一些极端场景，例如网络超时、连接断开等异常。Mock可以轻松控制mock对象抛出异常，测试回滚逻辑是否有效。



那集成测试代码和单元测试的代码，都验证了 `TransferService` 是否调用了 `AccountRepository`，那都能够验证两者是否正确集成交互，为什么还需要集成测试这样写？

答：关键的区别在于：**“谁”负责把这两个对象连接在一起。**

* 代码 B (@ExtendWith - 纯 Mockito 单元测试)
  * 谁在连接对象？ 是 Mockito 框架通过反射硬塞进去的。
  * 它能证明什么？ 它只能证明：如果 transferService 里有这个 repository，那么它的业务逻辑（加减法）是对的。
  * 它漏掉了什么？ 它无法证明你的 Spring 配置是否正确。例如，如果你漏写了 @Service 注解，或者构造函数写错了导致 Spring 无法启动，这个测试依然能通过，但你的程序在生产环境会崩溃。

* 代码 A (@SpringBootTest - Spring 集成测试)
  * 谁在连接对象？ 是真实的 Spring 容器。
  * 它能证明什么？ 它证明了：
    * Spring 能成功扫描到TransferService这个Bean。
    * Spring 能将AccountRepository这个Bean注入到TransferService Bean中。
    * 如果你有 AOP（切面），比如 @Transactional，Spring 会为 transferService 创建代理对象。这个测试运行的是带代理的对象。因此也就能够验证事务是否正确运行。
  * 它漏掉了什么？ 它没有真正测试数据库存取（因为 Repository 被 Mock 了）。但是没关系，我们这里只关注SpringBoot和TransferService是否正确协作，并不关注AccountRepository是否正确实现数据库存取。



@DisplayName是JUint5提供的注解，其作用是让开发者可以用人类可读的语言命名测试，则测试报告会显式这个名字，可读性会更加好，一眼就能看出来这是在测什么。

![image-20251223220233743](asset/image-20251223220233743.png)



如果我们还想验证事务管理是否生效，可以用内存数据库H2，进行真实测试，验证抛出异常之后，数据库的值是否不变。

==注意：==

* 可测试性和可维护性 相辅相成。 the app’s testability decreases when you fail to separate the different responsibilities into small and easy-to-read methods. 如果一个方法很臃肿，塞了太多逻辑，一方面，这个方法的可维护性不会好，只会越写越长。另一方面， 也不好测试，因为为了覆盖所有路径，必须构造极其复杂的 Mock 数据和前置条件，导致单测编写举步维艰。相反，如果方法只负责最原子的一件小事，则方法不会臃肿，可维护性好。单侧也很好写，因为输入输出极其明确，我们可以用极小的代价实现全场景覆盖。
