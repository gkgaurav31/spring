# AUTO-WIRE

Using the `@Autowired` annotation, we mark an object’s property where we want Spring to inject a value from the context, and we mark this intention directly in the class that defines the object that needs the dependency.

There are mainly three ways how we can use `@Autowired` annotation:

- Injecting the value in the field of the class
- Injecting the value through the constructor paramters of the class (most commonly used)
- Injecting the value through the setter

We will see the example by using `@Autowired` annotation in the field of the class. However, this is generally not used in production code since it makes the code difficult to maintain and test.

Also, this does not compile, since we cannot define final field without an initial value.

```java
 @Autowired
 private final Parrot parrot;
```

## USING AUTO WIRE WITH STEREOTYPE ANNOTATION

Here we will use the stereotype annotation to create the `Parrot` and `Person` beans, but we can also do it using `@Bean` way.

- Create the `Parrot` class and annotate it with `@Component`

```java
package beans;

import org.springframework.stereotype.Component;

@Component
public class Parrot {

    private String name = "Koko";

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

}
```

- Create the `Person` class and annotate it with `@Component`. Here, we will use the annotation `@Autowired` for the parrot field. This tells Spring to automatically add the depdendency from the spring context.

```java
package beans;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

@Component
public class Person {

    private String name = "Alan";

    @Autowired
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

- Next, since we are using `@Component` annotation, we need to tell Spring where to find these classes using `@ComponentScan` annotation in our Spring Configuration class.

```java
package config;

import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

@Configuration
@ComponentScan(basePackages = "beans")
public class AppConfig {

}
```

- That's it, we can now run our main app as usual.

```java
package main;

import beans.Person;
import config.AppConfig;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class App {

    public static void main(String[] args) {

        var context = new AnnotationConfigApplicationContext(AppConfig.class);

        Person person = context.getBean(Person.class);

        System.out.println("Person Name: " + person.getName());
        System.out.println("Parrot Name: " + person.getParrot().getName());

    }

}
```

## DOING IT THE `@Bean` WAY

- The main change will be in the `@Configuration` class. Remember to remove `@Component` annotation from the `Person` and `Parrot` classes. Rest of the code stays the same.

```java
package config;

import beans.Parrot;
import beans.Person;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
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
    public Person person(Parrot parrot){
        Person person = new Person();
        person.setName("Alan");
        person.setParrot(parrot);
        return person;
    }


}
```

## AUTOWIRE THROUGH CONSTRUCTOR AKA CONSTRUCTOR INJECTION

- Recommended approach mostly
- It enables you to define the fields as final, ensuring no one can change their value after Spring initializes them.
- The possibility to set the values when calling the constructor also helps you when writing specific unit tests where you don’t want to rely on Spring making the field injection for you. (We will see this later)

The only change needed will be in `Person` class. Instead of using `@Autowire` for the parrot field, we will use constructor to inject the dependency for us.

```java
package beans;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

@Component
public class Person {

    private String name = "Alan";

    private Parrot parrot; // we can also mark it final now!

    @Autowired
    public Person(Parrot parrot){
        this.parrot = parrot;
    }

    //..setters and getters

}
```

## DEPENDENCY INJECTION USING SETTERS

- Not a recommended way
- We cannot use final. Makes testing difficult

All we need to do is mark the setter method `setParrot` with `@Autowired`

```java
package beans;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

@Component
public class Person {

    private String name = "Alan";

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

    @Autowired
    public void setParrot(Parrot parrot) {
        this.parrot = parrot;
    }

}
```
