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

## Comparison of Adding Beans to Spring Context

| Using @Bean Annotation                                                                                                                                                                                                                                              | Using Stereotype Annotations                                                                                                                                                                                                                        |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1. You have full control over the instance creation you add to the Spring context. It is your responsibility to create and configure the instance in the body of the method annotated with @Bean. Spring only takes that instance and adds it to the context as-is. | 1. You only have control over the instance after the framework creates it.                                                                                                                                                                          |
| 2. You can use this method to add more instances of the same type to the Spring context.                                                                                                                                                                            | 2. This way, you can only add one instance of the class to the context.                                                                                                                                                                             |
| 3. You can use the @Bean annotation to add to the Spring context any object instance. The class that defines the instance doesn’t need to be defined in your app. Like, we can add a String and an Integer to the Spring context.                                   | 3. You can use stereotype annotations only to create beans of the classes your application owns. For example, you cannot add a bean of type String or Integer because you don’t own these classes to change them by adding a stereotype annotation. |
| 4. You need to write a separate method for each bean you create, which adds boilerplate code to your app. For this reason, we prefer using @Bean as a second option to stereotype annotations in our projects.                                                      | 4. Using stereotype annotations to add beans to the Spring context doesn’t add boilerplate code to your app.                                                                                                                                        |

## @PostConstruct

As we saw earlier, when using `@Component` annotation, the bean which was added to the spring context, did not have any values for its instance variables. Basically, we did not have a way to initialize the object/bean during its creation, like we could do when using `@Bean` annotated method. `@PostConstruct` helps us solve this problem - execute some instructions right after Spring creates the bean.

- We will need to add the following dependency for Java 11 and later.

```xml
<dependency>
    <groupId>javax.annotation</groupId>
    <artifactId>javax.annotation-api</artifactId>
    <version>1.3.2</version>
</dependency>
```

- Now, we can add a method in the Parrot class annotated with `@PostConstruct` to initialize the attributes.

```java
package model;

import org.springframework.stereotype.Component;
import javax.annotation.PostConstruct;

@Component
public class Parrot {

    private String name;

    @PostConstruct
    public void init(){ //any method name is ok
        this.name = "Koko";
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```

- That's all. If we run our existing app, we can now see the actual name getting printed instead of null.

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
        System.out.println(parrot.getName()); //this will print "Koko"

    }

}
```

> `PreConstruct`: Very similarly, but less encountered in real-world apps, you can use an annotation
> named @PreDestroy. With this annotation, you define a method that Spring calls immediately before closing and clearing the context.
