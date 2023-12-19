# CHOOSING FROM MULTIPLE BEANS DURING DEPENDENCY INJECTION

Let's say, the `Person` has a dependency on `Parrot` class. If we have multiple beans for `Parrot`, which one will Spring choose from the context?

## EXAMPLE TO SHOW THE PROBLEM

- The usual `Parrot` class

```java
package beans;

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

- `Person` class which depends on `Parrot` like we have previously done. Marked with `@Component` to let spring create a bean for it. This means, we should use `@ComponentScan` annotation in spring configuration class.

```java
package beans;

import org.springframework.stereotype.Component;

@Component
public class Person {

    private String name = "Alan";

    private Parrot parrot;

    public  Person(Parrot parrot){
        this.parrot = parrot;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Parrot getParrot() {
        return parrot;
    }

    public void setParrot(Parrot parrot) {
        this.parrot = parrot;
    }
}
```

- We will use `@Bean` annotation to create two beans for `Parrot` in the class marked with `@Configuration`.

```java
package config;

import beans.Parrot;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

@Configuration
@ComponentScan(basePackages = "beans")
public class AppConfig {

    @Bean
    public Parrot parrot1(){
        Parrot parrot = new Parrot();
        parrot.setName("PARROT 1");
        return parrot;
    }

    @Bean
    public Parrot parrot2(){
        Parrot parrot = new Parrot();
        parrot.setName("PARROT 2");
        return parrot;
    }

}
```

- Which `Parrot` bean will be injected to the `Person` bean if we run this app?

```java
package main;


import beans.Person;
import config.AppConfig;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class App {

    public static void main(String[] args) {

        var context = new AnnotationConfigApplicationContext(AppConfig.class);

        Person person = context.getBean(Person.class);

        System.out.println(person.getParrot().getName());

    }
}
```

It will actually fail as the spring framework cannot decide.

```text
Caused by: org.springframework.beans.factory.NoUniqueBeanDefinitionException: No qualifying bean of type 'beans.Parrot' available: expected single matching bean but found 2: parrot1,parrot2
    at org.springframework.beans.factory.config.DependencyDescriptor.resolveNotUnique(DependencyDescriptor.java:218)
    at org.springframework.beans.factory.support.DefaultListableBeanFactory.doResolveDependency(DefaultListableBeanFactory.java:1418)
    at org.springframework.beans.factory.support.DefaultListableBeanFactory.resolveDependency(DefaultListableBeanFactory.java:1348)
    at org.springframework.beans.factory.support.ConstructorResolver.resolveAutowiredArgument(ConstructorResolver.java:911)
    at org.springframework.beans.factory.support.ConstructorResolver.createArgumentArray(ConstructorResolver.java:789)
    ... 14 more
```

## WHAT CAN BE DONE?

We have the following cases:

- The identifier of the parameter matches the name of one of the beans from the
  context (which, remember, is the same as the name of the method annotated
  with `@Bean` that returns its value). In this case, Spring will choose the bean for
  which the name is the same as the parameter.
- If there is no match in the previous case:

  - If a bean is marked as `@Primary`, it's chosen.

  ```java
  @Bean
  @Primary
  public Parrot parrot1(){
      Parrot parrot = new Parrot();
      parrot.setName("PARROT 1");
      return parrot;
  }
  ```

  - Use `@Qualifier` annotation to explicitly select a bean.

  ```java
  @Bean
  public Person person(@Qualifier("parrot2") Parrot parrot2){
      Person person = new Person();
      person.setName("Alan");
      person.setParrot(parrot2);
      return person;
  }
  ```

  - Without `@Primary` or `@Qualifier`, an exception is thrown for ambiguous bean type.
