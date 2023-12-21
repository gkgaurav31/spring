# BEAN SCOPES AND LIFE CYCLE - SINGLETON

Spring has multiple different approaches for creating beans and managing their life cycle, and in the Spring world we name these approaches scopes.

The default scope is singleton. However, this is different from the singleton design pattern that we often read about in which there can be only a single instance in the app.

> Unique per name but not unique per app

Singleton bean scope means that there will be a single bean for a specific unique name/ID. (There can be multiple beans of the same type though)

## EXAMPLE

```text
        +-----------------------------+
        | @Service: CommentService    |
        +-----------------------------+
                 |
                 v
+--------------------------------+
| @Repository: CommentRepository |
+--------------------------------+
                 ^
                 |
        +-----------------------------+
        | @Service: UserService       |
        +-----------------------------+
```

Let's say, there are two services `@CommentService` and `@UserService` which depend on a repository `@CommentRepository`. The classes would look like this:

- First we create the `@Repository` dependency

```java
package repository;

import org.springframework.stereotype.Repository;

@Repository
public class CommentRepository {

    public void saveComment(){
        System.out.println("saving the comment to db.");
    }

}
```

(We are not using interfaces here since this example is just to prove singleton nature of the beans)

- Next we define the `@Service` classes: `CommentService` and `UserService` which has a dependency on `CommentRepository`. We will use `@Autowired` to inject the dependency.

```java
package service;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import repository.CommentRepository;

@Service
public class CommentService {

    @Autowired
    private CommentRepository commentRepository;

    public CommentRepository getCommentRepository(){
        return this.commentRepository;
    }

}
```

`UserService` class:

```java
package service;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import repository.CommentRepository;

@Service
public class UserService {

    @Autowired
    private CommentRepository commentRepository;

    public CommentRepository getCommentRepository(){
        return commentRepository;
    }

}
```

We have added a getter method in the above classes so that we can use that to get the object corresponding to the commmentRepository and compare them.

- Now that we have all the classes in place, we will run our app and check if the commentRepository in `CommentService` and `UserService` refer to the same object or different:

```java
package main;

import config.AppConfig;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import service.CommentService;
import service.UserService;

public class App {

    public static void main(String[] args) {

        var context = new AnnotationConfigApplicationContext(AppConfig.class);

        CommentService commentService = context.getBean(CommentService.class);
        UserService userService = context.getBean(UserService.class);

        System.out.println(commentService.getCommentRepository() == userService.getCommentRepository()); //output: true

    }

}
```

The output `true` shows us that both services have a reference to the same object for the `CommentRepository` dependency.

## EXAMPLE TO SHOW THAT MULTIPLE BEANS CAN STILL EXIST OF THE SAME TYPE

- We will make use of the existing `CommentRepository` class and create multiple beans using `@Bean` annotation in our App Config class.

```java
package repository;

import org.springframework.stereotype.Repository;

@Repository
public class CommentRepository {

    public void saveComment(){
        System.out.println("saving the comment to db.");
    }

}
```

```java
package config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import repository.CommentRepository;

@Configuration
public class AppConfig {

    @Bean
    public CommentRepository commentRepository1(){
        CommentRepository commentRepository = new CommentRepository();
        return commentRepository;
    }

    @Bean
    public CommentRepository commentRepository2(){
        CommentRepository commentRepository = new CommentRepository();
        return commentRepository;
    }

}
```

- Now if we get the beans using the name of the above methods "commentRepository1" and "commentRepository2", they will have refer to different objects! This shows that there is a single bean for a specific name/ID.

```java
package main;

import config.AppConfig;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import repository.CommentRepository;
import service.CommentService;
import service.UserService;

public class App {

    public static void main(String[] args) {

        var context = new AnnotationConfigApplicationContext(AppConfig.class);

        CommentRepository commentRepository1 = context.getBean("commentRepository1", CommentRepository.class);
        CommentRepository commentRepository2 = context.getBean("commentRepository2", CommentRepository.class);

        System.out.println(commentRepository1 == commentRepository2); //output: false

    }

}
```

## SINGLETON IN REAL WORLD

Because the singleton bean scope assumes that multiple components of the app
can share an object instance, the most important thing to consider is that these beans must be **_immutable_**. Most often, a real-world app executes actions on multiple threads and can share the same object instance. If these threads change the instance, we encounter a race-condition scenario.

If we want mutable singleton beans (whose attributes change), we need to make these beans concurrent by ourselves (mainly by employing thread synchronization). But singleton beans aren't designed to be synchronized. They're commonly used to define an app's backbone class design and delegate responsibilities one to another. Technically, synchronization is possible, but it's not a good practice.

**IMPORTANT:**

A better design while using dependency injection with singleton scope would be to ensure that the instances are immutable. We could do this by making the bean's fields as final and then use constructor dependency injection!

```java
package service;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import repository.CommentRepository;

@Service
public class CommentService {

    private final CommentRepository commentRepository; //make this final

    //use constructor dependency injection
    public CommentService(CommentRepository commentRepository){
        this.commentRepository = commentRepository;
    }

    public CommentRepository getCommentRepository(){
        return this.commentRepository;
    }

}
```

**Remember:**

- Make an object bean in the Spring context only if you need Spring to manage it so that the framework can augment that bean with a specific capability. If the object doesn't need any capability offered by the framework, you don't need to make it a bean.

- If you need to make an object bean in the Spring context, it should be singleton only if it's immutable. Avoid designing mutable singleton beans.

- If a bean needs to be mutable, an option could be to use the prototype scope which will be convered next.
