# PROTOTYPE BEAN SCOPE

Every time you request a reference to a prototype-scoped bean, Spring creates a new object instance. For prototype beans, Spring doesn’t create and manage an object instance directly. The framework manages the object’s type and creates a new instance every time someone requests a reference to the bean.

## EXAMPLE

- We will make use of our `CommentService` class. First, let's create the `CommentRepository`

```java
package repository;

import org.springframework.stereotype.Repository;

@Repository
public class CommentRepository {

}
```

- Use the `CommentRepository` as a dependency in `CommentService`. The default scope is singleton. To change it to prototype scope, we will need to add the annotation `@Scope(BeanDefinition.SCOPE_PROTOTYPE)`

```java
package service;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.config.BeanDefinition;
import org.springframework.context.annotation.Scope;
import org.springframework.stereotype.Service;
import repository.CommentRepository;

@Service
@Scope(BeanDefinition.SCOPE_PROTOTYPE)
public class CommentService {

    @Autowired
    private CommentRepository commentRepository;

}
```

- The usual configuration class:

```java
package config;

import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

@Configuration
@ComponentScan(basePackages = {"repository","service"})
public class AppConfig {

}
```

- To check if we get a different object each time we request for CommentService from Spring, we will use the following main App:

```java
package main;

import config.AppConfig;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import service.CommentService;

public class App {

    public static void main(String[] args) {

        var context = new AnnotationConfigApplicationContext(AppConfig.class);

        CommentService commentService1 = context.getBean(CommentService.class);
        CommentService commentService2 = context.getBean(CommentService.class);

        System.out.println(commentService1 == commentService2); //output: false

    }

}
```

The output will be true if we remove `@Scope(BeanDefinition.SCOPE_PROTOTYPE)`.

## THE @BEAN WAY

- Repository:

```java
package repository;

public class CommentRepository {
}
```

- Service:

```java
package service;

import repository.CommentRepository;

public class CommentService {

    private CommentRepository commentRepository;

    public CommentRepository getCommentRepository() {
        return commentRepository;
    }

    public void setCommentRepository(CommentRepository commentRepository) {
        this.commentRepository = commentRepository;
    }
}
```

- Configuration:

This is where we specify the `@Bean` and set the scope using `@Scope(BeanDefinition.SCOPE_PROTOTYPE)`

```java
package config;

import org.springframework.beans.factory.config.BeanDefinition;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Scope;
import repository.CommentRepository;
import service.CommentService;

import javax.xml.stream.events.Comment;

@Configuration
public class AppConfig {

    @Bean
    public CommentRepository commentRepository(){
        CommentRepository commentRepository = new CommentRepository();
        return commentRepository;
    }

    @Bean
    @Scope(BeanDefinition.SCOPE_PROTOTYPE)
    public CommentService commentService(CommentRepository commentRepository){
        CommentService commentService = new CommentService();
        commentService.setCommentRepository(commentRepository);
        return commentService;
    }

}
```

- main app:

```java
package main;

import config.AppConfig;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import service.CommentService;

public class App {

    public static void main(String[] args) {

        var context = new AnnotationConfigApplicationContext(AppConfig.class);

        CommentService commentService1 = context.getBean(CommentService.class);
        CommentService commentService2 = context.getBean(CommentService.class);

        System.out.println(commentService1 == commentService2); //output: false

    }

}
```

## UNDERSTAND HOW TO USE IT WITH REAL EXAMPLE

Let's say, we need to have a `CommentProcessor` class that processes the comments and validates them. The `CommentService` class needs to use `CommentProcessor` to implement a use-case. The `CommentProcessor` stores the comment to be processed as an attribute and its methods change this attribute (so we cannot make it immutable)

The `CommentProcessor` could look like this:

```java
package util;

import model.Comment;

public class CommentProcessor {

    private Comment comment;

    public Comment getComment() {
        return comment;
    }

    public void setComment(Comment comment) {
        this.comment = comment;
    }

    public void processComment(){
        comment.setMessage(comment.getMessage().toLowerCase());
    }

    public boolean validateComment(){
        if(comment.getMessage().isBlank() || comment.getMessage().isEmpty())
            return false;
        else
            return true;
    }
}
```

The `CommentService` could be using this in the following manner:

```java
package service;

import model.Comment;
import org.springframework.stereotype.Service;
import util.CommentProcessor;

@Service
public class CommentService {

    public void sendComment(Comment comment){

        CommentProcessor commentProcessor = new CommentProcessor();

        commentProcessor.setComment(comment);
        commentProcessor.processComment();
        commentProcessor.validateComment();

        comment = commentProcessor.getComment();
        //do something with the modified comment

    }

}
```

At this point, `CommentProcessor` is not even a bean in the spring context and it could work. However, it's important to think if we want to make it a bean.

What if the `CommentProcessor` class had to save the comment in a DB (which means, it had a dependency on `@CommentRepository`). To use dependency injection with spring in that case, `CommentProcessor` would need to be set as a bean as well.

## Should it be a singleton?

No. If we define this bean as singleton and multiple threads use it concurrently, we get into a race condition. We would not be sure which comment provided by which thread is processed and if the comment was processed correctly. In this scenario, we want each method call to get a different instance of the `CommentProcessor` object (and set it as a prototype bean).

## DOING IT THE WRONG WAY (IMPORTANT)

Don’t make the mistake of injecting the `CommentProcessor` directly in the `CommentService` bean. The `CommentService` bean is a singleton, which means that Spring creates only an instance of this class. As a consequence, Spring will also inject the dependencies of this class just once when it creates the `CommentService` bean itself. In this case, you’ll end up with only an instance of the `CommentProcessor`. Each call of the `sendComment()` method will use this unique instance, so with multiple threads you’ll run into the same race condition issues as with a singleton bean.

```java
package service;

import model.Comment;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import util.CommentProcessor;

@Service
public class CommentService {

    @Autowired
    private CommentProcessor commentProcessor;
    //Spring injects this bean when creating the CommentService bean.
    //But because CommentService is singleton, Spring will also create and inject the CommentProcessor just once.


    public void sendComment(Comment comment){
        commentProcessor.processComment();
        comment = commentProcessor.getComment();
        //do something more with updated comment
    }

}
```

## FIXING IT

What we need is a new `CommentProcessor` instance for each "usage" when we get it from the spring context.

Let's start with the simple classes, like the `Comment` class:

```java
package model;

public class Comment {

    private String author;
    private String message;

    public String getAuthor() {
        return author;
    }

    public void setAuthor(String author) {
        this.author = author;
    }

    public String getMessage() {
        return message;
    }

    public void setMessage(String message) {
        this.message = message;
    }
}
```

We'll also create a `CommentRespository` for the `CommentProcessor` class to use if it needs to save to a DB.

```java
package repository;

import model.Comment;
import org.springframework.stereotype.Repository;

@Repository
public class CommentRepository {

    public void saveComment(Comment comment){
        System.out.println("saving the comment to DB. By: " + comment.getAuthor() + " with message: " + comment.getMessage());
    }


}
```

The `CommentProcessor` class itself which should be prototype scope as we discussed previously.

```java
package util;

import model.Comment;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.config.BeanDefinition;
import org.springframework.context.annotation.Scope;
import org.springframework.stereotype.Component;
import repository.CommentRepository;

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

    public void processComment(){
        comment.setMessage(comment.getMessage().toLowerCase());
    }

    public void saveComment(){
        commentRepository.saveComment(comment);
    }

}
```

Now the interesting part. In the `CommentService`, instead of having `CommentProcessor` attribute autowired, we will autowire an application context. How does that help? We will use this in the `sendComment()` method of the `CommentService`. This `getBean()` method will run each time we call `sendComment()`. Since we have marked `CommentProcessor` as prototype scoped, we will get a difference instance of `CommentProcessor` each time!

```java
package service;

import model.Comment;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.ApplicationContext;
import org.springframework.stereotype.Service;
import util.CommentProcessor;

@Service
public class CommentService {

    @Autowired
    private ApplicationContext applicationContext;

    public void sendComment(Comment comment){

        CommentProcessor commentProcessor = applicationContext.getBean(CommentProcessor.class);

        commentProcessor.setComment(comment);
        commentProcessor.processComment();

        comment = commentProcessor.getComment();
        System.out.println(comment.getMessage());
        //further processing

    }

}
```

Now the usual configuration class:

```java
package config;

import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.stereotype.Component;

@Configuration
@ComponentScan(basePackages = {"repository", "service", "util"})
public class AppConfig {
}
```

Testing our app:

```java
package main;

import config.AppConfig;
import model.Comment;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import service.CommentService;

public class App {

    public static void main(String[] args) {

        var context = new AnnotationConfigApplicationContext(AppConfig.class);

        Comment comment = new Comment();
        comment.setAuthor("Alan Wake");
        comment.setMessage("Let's Play a Game.");

        CommentService commentService = context.getBean(CommentService.class);

        commentService.sendComment(comment);

    }

}
```

The `CommentService` is still singleton. So the dependencies of it are also initialized once. This is why we did not autowire `CommentProcessor` directly in `CommentService`, as we need a different bean for each call.

## COMPARISON - SINGLETON VS PROTOTYPE

| Singleton                                                                                           | Prototype                                                                                             |
| --------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------- |
| The framework associates a name with an actual object instance.                                     | A name is associated with a type.                                                                     |
| Every time you refer to a bean name you’ll get the same object instance.                            | Every time you refer to a bean name, you get a new instance.                                          |
| You can configure Spring to create the instances when the context is loaded or when first referred. | The framework always creates the object instances for the prototype scope when you refer to the bean. |
| Singleton is the default bean scope in Spring.                                                      | You need to explicitly mark a bean as a prototype.                                                    |
| It’s not recommended that a singleton bean to have mutable attributes.                              | A prototype bean can have mutable attributes.                                                         |
