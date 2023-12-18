# ADDING MULTIPLE BEANS OF SAME TYPE TO SPRING CONTEXT

- Let us define our Spring Configuration first, in which we will create two methods which will add the objects of `Parrot` class.

```java
package config;

import main.Parrot;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class ProjectConfig {

    @Bean
    Parrot parrot1(){
        Parrot p = new Parrot();
        p.setName("Parrot 1");
        return p;
    }

    @Bean
    Parrot parrot2(){
        Parrot p = new Parrot();
        p.setName("Parrot 2");
        return p;
    }

}
```

- If we now try to get the bean of type `Parrot`, we will get a `NoUniqueBeanDefinitionException` exception.

```java
package main;

import config.ProjectConfig;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class Main {

    public static void main(String[] args) {

        var context = new AnnotationConfigApplicationContext(ProjectConfig.class);

        Parrot p = context.getBean(Parrot.class); //this will cause an exception

    }

}
```

```text
Exception in thread "main" org.springframework.beans.factory.NoUniqueBeanDefinitionException: No qualifying bean of type 'main.Parrot' available: expected single matching bean but found 2: parrot1,parrot2
    at org.springframework.beans.factory.support.DefaultListableBeanFactory.resolveNamedBean(DefaultListableBeanFactory.java:1310)
    at org.springframework.beans.factory.support.DefaultListableBeanFactory.resolveBean(DefaultListableBeanFactory.java:484)
    at org.springframework.beans.factory.support.DefaultListableBeanFactory.getBean(DefaultListableBeanFactory.java:339)
    at org.springframework.beans.factory.support.DefaultListableBeanFactory.getBean(DefaultListableBeanFactory.java:332)
    at org.springframework.context.support.AbstractApplicationContext.getBean(AbstractApplicationContext.java:1191)
    at main.Main.main(Main.java:12)

Process finished with exit code 1
```

This seems obvious because now there are two methods which return an object of type Parrot.

- To solve this, we will need to use something which identifies these objects uniquely. Remember that the name of the method will be the bean name by default. We will use this as the identifier.

```java
getBean("the name of the bean to retrieve", "type the bean must match; can be an interface or superclass")
```

```java
package main;

import config.ProjectConfig;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class Main {

    public static void main(String[] args) {

        var context = new AnnotationConfigApplicationContext(ProjectConfig.class);

        Parrot p1 = context.getBean("parrot1", Parrot.class);
        System.out.println(p1.getName());

        Parrot p2 = context.getBean("parrot2", Parrot.class);
        System.out.println(p2.getName());
    }

}
```

- You can also define a different name than the method name itself.

  - @Bean(name = "myparrot")
  - @Bean(value = "myparrot")
  - @Bean("myparrot")

```java
@Bean("myparrot") //or @Bean(name="myparrot") or @Bean(value="myparrot")
Parrot parrot3(){
    Parrot p = new Parrot();
    p.setName("Koko");
    return p;
}
```

- We can also set one of the beans as primary! So, if multiple matches are found, like in our first example, it will choose the primary one instead of throwing an exception. To mark the bean as primary, just annotate it with `@Primary`.

```java

package config;

import main.Parrot;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;

@Configuration
public class ProjectConfig {

    //.. other beans which return a Parrot

    @Bean(name="myparrot")
    @Primary
    Parrot parrot3(){
        Parrot p = new Parrot();
        p.setName("Koko");
        return p;
    }

}

```

```java
package main;

import config.ProjectConfig;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class Main {

    public static void main(String[] args) {

        var context = new AnnotationConfigApplicationContext(ProjectConfig.class);

        Parrot p = context.getBean(Parrot.class);
        System.out.println(p.getName()); //this will choose "myparrot" bean since its primary

    }

}
```

Obviously, we can have one primary only otherwise you will get the following exception when Spring tries to initialize its context:

```text
Exception in thread "main" org.springframework.beans.factory.NoUniqueBeanDefinitionException: No qualifying bean of type 'main.Parrot' available: more than one 'primary' bean found among candidates: [parrot1, parrot2, myparrot]
    at org.springframework.beans.factory.support.DefaultListableBeanFactory.determinePrimaryCandidate(DefaultListableBeanFactory.java:1753)
    at org.springframework.beans.factory.support.DefaultListableBeanFactory.resolveNamedBean(DefaultListableBeanFactory.java:1295)
    at org.springframework.beans.factory.support.DefaultListableBeanFactory.resolveBean(DefaultListableBeanFactory.java:484)
    at org.springframework.beans.factory.support.DefaultListableBeanFactory.getBean(DefaultListableBeanFactory.java:339)
    at org.springframework.beans.factory.support.DefaultListableBeanFactory.getBean(DefaultListableBeanFactory.java:332)
    at org.springframework.context.support.AbstractApplicationContext.getBean(AbstractApplicationContext.java:1191)
    at main.Main.main(Main.java:12)
```
