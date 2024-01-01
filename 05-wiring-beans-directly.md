# WIRING THE BEANS TO ESTABLISH RELATIONSHIP (DIRECT WAY)

Let us say, we need to establish a HAS-A relationship between a `Person` and a `Parrot`. We can have two methods in `@Configuration` annotated class -

1. To add `Parrot` bean to spring context
2. To add `Person` bean to spring context. Also, it needs to "link" it with the parrot bean while creating the person bean.

We will use a direct way to achieve this "wiring" -

- Create a `Parrot` class as usual

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

    @Override
    public String toString() {
        return "ParrotName: " + name;
    }
}
```

- Create a `Person` class which HAS-A `Parrot`

```java
package model;

public class Person {

    private String name;
    private Parrot parrot;

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

- In the `@Configuration` annotated class, we define our beans

```java
package config;

import model.Parrot;
import model.Person;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class AppConfig {

    @Bean
    public Parrot parrot(){
        Parrot parrot = new Parrot();
        parrot.setName("Koko");
        return parrot;
    }

    @Bean
    public Person person(){
        Person person = new Person();
        person.setName("Alan");
        person.setParrot(parrot()); //we wire the person with the parrot here
        return person;
    }
}
```

- Now we should be able to get the Person bean from the spring context and its corresponding parrot bean

```java
package main;

import config.AppConfig;
import model.Person;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class App {

    public static void main(String[] args) {

        var context = new AnnotationConfigApplicationContext(AppConfig.class);

        Person person = context.getBean(Person.class);

        System.out.println(person.getParrot()); //this will output "Koko"

    }
}
```

> **IMPORTANT**: We might expect to have two `Parrot` beans - one created by spring while initializing the spring context and another due to `parrot()` method call while creating the `Person` bean. However, that is NOT the case.
> When two methods annotated with `@Bean` call each other, Spring knows we want to create a link between the beans. If the bean already exists in the context, Spring returns the existing bean without forwarding the call to the `@Bean` method. If the bean doesn't exist, Spring creates the bean and returns its reference.

> **TIP**: We can prove the above behavior by adding a no-args constructor with a print statement in the `Parrot` class. It will be called only once when we run our app.

- Alternative way to achieve this wiring, but without directly calling the `parrot()` method. All we need to do is update the method which creates the `Person` bean.

The interesting thing here is that now Spring will automatically **_inject_** the bean for parrot while creating the person bean. (Dependency Injection)

```java
@Bean
public Person person(Parrot parrot){ //will be injected by spring!
    Person person = new Person();
    person.setName("Alan");
    person.setParrot(parrot); //no direct call to parrot() here
    return person;
}
```
