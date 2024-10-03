# 



# 全局配置文件

SpringBoot提供了全局配置文件，使得我们编写配置文件更加轻松。

SpringBoot的全局配置文件主要有两种写法，分别是

**applications.yml**和**applications.propertis**

需要注意的是，全局配置文件的名称是不能随意改变的。

下面我将分别介绍这两种配置文件的区别

## application.properties

applications.properties是SpringBoot的一种配置文件写法，配置是以**键值对**的形式存在的，比如**key=value;**

同时，这种写法也支持分级的形式，比如`key1.key2.key3=value` 

下面是一个简单的配置文件格式，指定了springboot程序的服务端口，数据库连接等信息。

```
server.port=8080
spring.datasource.url=jdbc:mysql://localhost:3306/mydb
spring.datasource.username=root
spring.datasource.password=password
```

但是，从这个配置文件上也不难看出，如果前缀相同的话，那我们每次都要重新写这个前缀，就显的有些太麻烦了。

因此，我将介绍另外一种配置文件的形式，`yaml`

## application.yaml

首先我们应该要知道，什么是yaml格式。

YAML 允许通过 **缩进** 和 **层次结构** 来表达配置项，能够更直观地表示复杂的嵌套关系和数组结构，避免重复书写相同的前缀。

下面是一个示例

```yaml
server:
  port: 8080
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/mydb
    username: root
    password: password
```

可以看到，yaml格式的配置文件是使用缩进控制分级的，相比于properties格式的配置文件，层次结构更加分明，并且无需再写重复的前缀。

所以对于springboot项目来说，本人是更加推荐使用yaml这种格式的配置文件的。

## 配置文件的位置

已经知道了配置文件的写法，那么配置文件的位置应该在哪？

当我们创建完springboot项目后，springboot的配置文件应统一放置在resource目录下。

# 全局配置文件属性注入

spring的默认配置，如服务端口，数据库连接信息，会被springboot的自动配置类自动读取并应用。

那我们个人的配置，该如何让springboot知道呢？

这就要涉及到注入配置文件了，springboot为我们提供了多种注入的方式，我将介绍其中较常用的方式。

## @Compont+@ConfigurationProperties

@Compont标签会把我们的类标记为Spring的bean，并将这个bean交给ioc容器进行管理。

@ConfigurationProperties允许我们将外部配置文件内容注入到我们的Java类中。

那么既然ConfigurationProperties标签的功能就是注入外部配置文件，那么在springboot中，是否只用ConfigurationProperties就可以完成对外部配置文件的注入呢？

答案是否定的！

\1. `@ConfigurationProperties` **只是一个绑定工具**

首先，我们要明确：`@ConfigurationProperties` 只是为了将外部配置文件中的内容绑定到 Java 类中的一个工具。通过它，确实可以把配置文件中的属性值绑定到类的对应字段中，从而实现外部配置的注入。

但是，`@ConfigurationProperties` 仅仅负责属性绑定，它不会自动将这个类注册为 Spring IoC 容器中的 Bean。这意味着即使我们可以成功地将外部配置绑定到类中，其他类和组件也无法访问这个类中的属性，因为 Spring 容器并没有管理这个类。

\2. **类未注册为 Bean，其他组件无法访问**

Spring 的 IoC 容器会管理所有标记为 Bean 的类，并允许这些类被其他组件通过依赖注入的方式引用。然而，`@ConfigurationProperties` 本身并不负责将类注册为 Bean，也不会告诉 Spring Boot 这个类应该被管理。因此，**如果一个类仅仅使用了** `@ConfigurationProperties`，但没有被注册为 Bean，它就不会进入 Spring IoC 容器，其他类自然也无法访问这个类及其配置。

那么如何把@ConfigurationProperties交给ioc容器管理呢？

@ConfigurationProperties通常与@Component或@EnableConfigurationProperties配合使用。

@Component会告诉spring把此标签标记的类注册为ioc的bean,@EnableConfigurationProperties会显式的告诉spring将对应的类注册为bean。

下面是一个@Component和@ConfigurationProperties的示例

**application.yaml**

```yaml
spring:
  application:
    name: demochapter02

person:
  hobby:
    - play
    - read
    - sleep
  family:
    - father
    - mother
  map:
    k1: v1
    k2: v2
  id: 111
  name: chenxin
```

**Person.java**

```
package com.example.demochapter02.domain;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.stereotype.Component;

import java.util.Arrays;
import java.util.List;
import java.util.Map;

@Data
@AllArgsConstructor
@NoArgsConstructor
//@ConfigurationProperties配合@Component使用，指定前缀为person
//类中的属性要与配置文件中的
@ConfigurationProperties(prefix = "person")
@Component
public class Person {

    private int id;

    private String name;

    private List hobby;

    private String[] family;

    private Map map;

}
```

启动单元测试检测，可以观测到已经属性绑定成功

**可以看出，若配置文件中不包含相应的配置，那么创建的bean相应的属性为null**

![img](/upload/image.png)

## @Component+@Value

前面已经描述过，使用@ConfigurationProperties可以为类注入相应的属性，这是属于SpringBoot中的注解。

Spring也为我们提供了@Value标签供我们绑定属性。

同样的，@Value也需要配合@Component使用，具体原因不再介绍(见@Component+@ConfigurationProperties)。

**application.yaml(与@ConfigurationProperties中一致)**

```yaml
spring:
  application:
    name: demochapter02

person:
  hobby:
    - play
    - read
    - sleep
  family:
    - father
    - mother
  map:
    k1: v1
    k2: v2
  pet:
    type: dog
    name: kity
  id: 111
```

**Person.java**

```java
package com.example.demochapter02.domain;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.stereotype.Component;

import java.util.Arrays;
import java.util.List;
import java.util.Map;

@Data
@AllArgsConstructor
@NoArgsConstructor
@Component
public class Person {
    @Value("${person.id}")
    private int id;
    @Value("${person.name}")
    private String name;
    @Value("${person.hobby}")
    private List hobby;
    @Value("${person.family}")
    private String[] family;
    @Value("${person.map}")
    private Map map;
    @Value("${person.pet}")
    private Pet pet;

}
```

**使用了@Value标签，运行单元测试，会发现单元测试无法正常启动。**

![img](/upload/image-wpff.png)

在控制台中可以看到这样的报错，原因是因为我们的配置文件中没有person.name这一项配置，所以会直接报错

这也是@Value与@ConfigurationProperties不同的地方，

后者若配置文件无，那么bean可以正常创建，相应配置为null；

而前者会导致bean无法正常创建。

那么，是否我们把@Value("${person.name}")去除就可以完成测试呢？

重新启动单元测试，结果如下

![img](/upload/image-ltka.png)

但我们的配置文件中是有这个配置项的，原因就在于`@Value` **并不适合处理复杂数据结构（如** `List`**、**`Map` **和嵌套对象）。**`@Value` **更适合简单的属性注入。**

因此更加推荐使用@ConfigurationProperties去注入配置

## 两种配置注入的对比

![img](/upload/image-awew.png)

# 自定义配置文件

除了使用SpringBoot默认提供给我们的配置文件，我们也可以自定义自己的配置文件。

自定义配置文件为了统一路径，也建议放在resource目录下。

## 自定义配置文件

**my.properties**

```
my.name=chenxin
my.courseName=JavaEE
```

## 导入自定义配置文件

导入配置文件有多种方法，下面介绍使用@PropertyResource加载配置文件和使用@ImportResource加载配置文件。

### @PropertyResource+@Configuration导入自定义配置文件

@PropertyResource需要配合@Configuration使用

MyProperties.java

```java
package com.example.demochapter02.domain;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.PropertySource;
import org.springframework.stereotype.Component;

@Data
@AllArgsConstructor
@NoArgsConstructor
@Configuration
//@Component
@PropertySource("classpath:my.properties")
@EnableConfigurationProperties(MyProperties.class)
@ConfigurationProperties(prefix = "my")
public class MyProperties {
    private String name;

    private String courseName;
}
```

运行单元测试后，打印MyProperties会发现已经把属性绑定到了MyProperties中。

下面分别解释这几个标签的作用：

#### @Configuration

@Configuration通常用来定义Spring的配置类，这种类通常包含一组@Bean标签，来创建配置应用的各种bean。

`@Configuration` 可以用于任何需要通过 `@Bean` 方法显式声明和创建 Bean 的场景，而不仅仅是用于绑定配置文件。

标记为 `@Configuration` 的类，Spring 会将其视为一个 Java 配置类，里面定义的 `@Bean` 方法会将 Bean 注册到 Spring 容器中，供其他组件依赖。

#### @PropertyResource

@PropertySource会告诉spring要加载一个外部的配置文件，并指定这个配置文件的位置。

但是，指定了位置，是否就能将自定义的配置文件绑定到配置类中呢？@ConfigurationProperties默认是绑定全局配置文件的，我们虽然使用了@PropertySource导入了自定义配置文件，但@ConfigurationProperties是不会与@PropertySource协作的，我们需要把当前配置类交给spring ioc容器进行管理，才能绑定自定义配置文件。这也就是为什么要使用@Configuration标签的原因。

#### @EnableConfigurationProperties(MyProperties.class)

@EnableConfigurationProperties会告诉SpringBoot自动启用带有@ConfigurationProperties注解的类，并将**其注册为Spring的Bean**，这是显式启用@ConfigurationProperties绑定的一种方式。

那既然@EnableConfigurationProperties已经可以配置类注册为bean交给spring ioc容器管理，为什么还要使用@Configuration标签来注册bean呢？

下面做下实验，如果我们把@Configuration标签去掉，观察是否能注册为bean

**MyProperties.java**

```java
package com.example.demochapter02.domain;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.PropertySource;
import org.springframework.stereotype.Component;

@Data
@AllArgsConstructor
@NoArgsConstructor
//@Configuration
//@Component
@EnableConfigurationProperties(MyProperties.class)
@PropertySource("classpath:my.properties")
@ConfigurationProperties(prefix = "my")
public class MyProperties {
    private String name;

    private String courseName;
}
```

![img](/upload/image-eyid.png)

可以看到，Spring已经为我们提示，是没有这个Bean的。

那如果我们把**@EnableConfigurationProperties(MyProperties.class)**放到启动类呢？

**MyProperties.java**

```java
package com.example.demochapter02.domain;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.PropertySource;
import org.springframework.stereotype.Component;

@Data
@AllArgsConstructor
@NoArgsConstructor
//@Configuration
//@Component
@PropertySource("classpath:my.properties")
@ConfigurationProperties(prefix = "my")
public class MyProperties {
    private String name;

    private String courseName;
}
```

**Demochapter02Application.java**

```
package com.example.demochapter02;

import com.example.demochapter02.domain.MyProperties;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.context.annotation.ImportResource;

@ImportResource("classpath:beans.xml")
@EnableConfigurationProperties(MyProperties.class)
@SpringBootApplication
public class Demochapter02Application {

    public static void main(String[] args) {
        SpringApplication.run(Demochapter02Application.class, args);
    }

}
```

![img](/upload/image-dhyv.png)

此时，Spring已经不会报没有这个bean，执行单元测试。

![img](/upload/image-tpya.png)

可以发现，虽然有这个bean，但却没有正常绑定。

真正的原因是，`@EnableConfigurationProperties`注解确实是用来注册带有`@ConfigurationProperties`注解的类为Spring管理的Bean。这个注解本身确保了属性绑定的类被创建并且其属性能被Spring环境中的全局配置文件（如application.properties或application.yml）中的相应属性自动填充。

但是，这个注解本身并不涉及到从特定的属性文件（通过`@PropertySource`指定的文件）加载配置。这意味着，如果`@PropertySource`加载的属性文件中的属性想要被绑定到`@ConfigurationProperties`类中，需要确保类也是通过一种方式（如`@Configuration`）被Spring识别为配置类。

**解决方案**

- **使用**`@Configuration`在配置类上：这是最简单的方法。通过将`@Configuration`注解添加到`MyProperties`类上，你不仅注册了该类为一个Bean，同时`@PropertySource`也能确保属性文件被加载并且其中的属性绑定到类的字段上。
- **将属性加载逻辑移至启动类**：如果不想在`MyProperties`类上使用`@Configuration`，可以考虑在启动类中使用`@PropertySource`来加载属性文件，然后通过`@EnableConfigurationProperties`注册`MyProperties`。

SpringBoot程序启动后，会默认把全局配置文件加入到一个名为**Environment**的抽象中管理配置，然后才会去创建bean。如果配置类中导入了配置文件，那就需要在配置类也使用`@Configuration` 。
或者我们可以把导入配置直接放到启动类，这样在启动的时候，配置以及加载到了**Environment**中了。

## 自定义XML配置文件

在Resource目录下自定义XML配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                      http://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean id="myService" class="com.example.demochapter02.domain.MyService" />
</beans>
```

### @ImportResource导入xml配置文件

```java
package com.example.demochapter02;

import com.example.demochapter02.domain.MyProperties;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.context.annotation.ImportResource;
import org.springframework.context.annotation.PropertySource;

//使用ImportResource注解，将xml注入到SpringIoc中
@ImportResource("classpath:beans.xml")
@EnableConfigurationProperties(MyProperties.class)
@SpringBootApplication
public class Demochapter02Application {

    public static void main(String[] args) {
        SpringApplication.run(Demochapter02Application.class, args);
    }

}
```

启动单元测试

![img](/upload/image-svtw.png)

可以看到，我们在xml文件中定义的bean已经存在与ioc容器中。

# 自定义配置类

上面介绍的是，如何导入全局配置文件之外的配置文件到Spring中，下面介绍如何自己定义配置类。

这个注解在上文中已经提到，就是**@Configuration**注解。

详细介绍@Configuration注解。

@Configuration是@Component的特例，它不仅能把一个类标记为Bean交给IoC容器管理，还表明该类是用作配置信息的源。

**MyConfig.java**

```java
package com.example.demochapter02.domain;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class MyConfig {
//需要注意，使用@Bean注解标记Bean，Bean的id是方法名字
    @Bean
    public MyService test(){
        return new MyService();
    }

}
```

运行单元测试

![img](/upload/image-twwv.png)

# 多环境配置

如果每次换一个环境，我们都需要修改大量的配置文件，就有点太过于复杂。

我们可以多定义几个全局配置文件，在相应的环境，我们只需要在开发环境，测试环境或者生产环境下，激活相应的配置文件就可以了。

要明确的是，application.yaml才是真正的全局配置文件，其余配置文件可以在application.yaml中激活。

多环境开发配置文件应该严格遵循

`application-{XXX}.yaml` 这种格式

**application.yaml**

```
spring:
  profiles:
    active: dev #此处的active的值就是需要激活的配置文件，例如application-dev.yaml
```

激活对应的环境就可以了

@Profile注解不再赘述，@Profile标签作用于类，如果激活的环境与@Profile中指定的值相同，那么被@Profile标记的类，那么就激活这个类并加载相关配置。

# 随机值设置与参数引用

## 随机值

Spring Boot 提供了一些用于生成随机值的内置占位符。常见的随机值占位符包括：

- `${random.int}`：生成一个随机整数。
- `${random.int(min,max)}`：生成一个在 `[min, max]` 范围内的随机整数。
- `${random.long}`：生成一个随机的长整数。
- `${random.uuid}`：生成一个随机的 UUID。
- `${random.value}`：生成一个随机的浮点数。

```
app.random.int=${random.int}
app.random.int.range=${random.int(1000,2000)}
app.random.long=${random.long}
app.random.uuid=${random.uuid}
app.random.float=${random.value}
```

## 参数引用

可以在 Spring Boot 配置文件中通过 `${}` 来引用其他的配置属性。这在某些场景下非常有用，例如当多个配置项共享一个通用值时，可以使用参数引用来避免重复定义。

```
server.port=8080
app.name=MyApp
app.description=${app.name} is a sample application running on port ${server.port}.
```