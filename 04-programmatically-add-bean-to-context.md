# ADD BEANS TO SPRING CONTEXT PROGRAMATICALLY

This is available in Spring 5 and later. Using the `registerBean()` method enables you to implement custom logic for adding beans to the Spring context.

```java
if (condition) {
 registerBean(b1);

} else {
 registerBean(b2);
}
```

- The `Configuration` class will be simple:

```java
package config;

import org.springframework.context.annotation.Configuration;

@Configuration
public class AppConfig {

}
```

- No `@Component` annotation needed in `Parrot` class:

```java
package model;

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

- Using `registerBean()` method to add the bean programatically. This method has 4 parameters:

  - `String beanName`
    Name of the bean that we need to add. Can be null if we don't want to give it a name.
  - `Class<T> beanClass`
    Class that defines that the bean we are adding
  - `Supplier<T> supplier`
    Serves as a factory for creating bean instances. Functional interface which provides a single method, `get()` which returns as instance of `T`. This allows for lazy instantiation and more control over the creation of the bean, as the `Supplier<T>` can contain custom logic to produce the bean instance.
    Ref: [educative.io](https://www.educative.io/answers/what-is-the-supplier-functional-interface-in-java)
  - `BeanDefinitionCustomizer... customizers`
    Interface to configure different characteristics of the bean, like making it primary. Since this is varargs, we can omit it.

    Example:

    ```java
    context.registerBean("myparrot", Parrot.class, supplier, bc -> bc.setPrimary(true));
    ```

```java
package main;

import config.AppConfig;
import model.Parrot;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

import java.util.function.Supplier;

public class App {

    public static void main(String[] args) {

        var context = new AnnotationConfigApplicationContext(AppConfig.class);

        Parrot x = new Parrot();
        x.setName("Koko");

        Supplier<Parrot> supplier = () -> x;

        /*
        alternative way:

        Supplier<Parrot> supplier = new Supplier<Parrot>() {
            @Override
            public Parrot get() {
                return x;
            }
        };
        */

        context.registerBean("myparrot", Parrot.class, supplier);

        Parrot myparrot = context.getBean(Parrot.class);

        System.out.println(myparrot.getName());

    }

}
```
