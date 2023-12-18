# USING STEREOTYPE ANNOTATION TO ADD BEANS TO SPRING CONTEXT

Out of multiple Stereotype Annotations, we will make use of `@Component` stereotype annotation. This gives us another way to add beans to the spring context with lesser code. Each way has its advantages and disadvantages which we will look at later.

## STEPS

- Mark the class for which a bean needs to be add using `@Component` stereotype annotation. In our case, it will be the `Parrot` class.

```java
package model;

import org.springframework.stereotype.Component;

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

- By default, Spring doesn’t search for classes annotated with stereotype annotations, so if we just leave the code as-is, Spring won’t add a bean of type Parrot in its context. To tell Spring it needs to search for classes annotated with stereotype annotations, we use the @ComponentScan annotation over the configuration class. Also, with the @ComponentScan annotation, we tell Spring where to look for these classes.

```java
package config;

import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

@Configuration
@ComponentScan(basePackages = "model")
public class AppConfig {

}
```

That's all for the Configuration class. No code needed. The important part is the annotation `@ComponentScan` which has the basePackages parameter to look for classes annotated with stereotype annotation.

- Using it in our main app is identical.

```java
package main;

import config.AppConfig;
import model.Parrot;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class App {

    public static void main(String[] args) {

        var context = new AnnotationConfigApplicationContext(AppConfig.class);

        Parrot parrot = context.getBean(Parrot.class);

        System.out.println(parrot);
        System.out.println(parrot.getName()); //this will be null in our case since we never set the parrot name

    }

}
```
