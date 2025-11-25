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

