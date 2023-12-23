# INTERCEPTING CUSTOM ANNOTATED METHODS

We will define a custom annotation and log only the execution of methods we mark using this custom annotation using AOP. We would mainly do the following:

- Define a custom annotation, and make it accessible at runtime. We'll call this annotation `@ToLog`.
- Use am AspectJ pointcut expression for the aspect method to tell the aspect to intercept the methods annotated with the custom annotation.

Let's start by creating the basic classes that we are going to need first:

`Comment` class:

```java
package models;

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

    @Override
    public String toString() {
        return "Comment{" +
                "author='" + author + '\'' +
                ", message='" + message + '\'' +
                '}';
    }
}
```

A custom annotation `@ToLog`:

> The definition of the retention policy with `@Retention(RetentionPolicy.RUNTIME)` is critical. By default, in Java annotations cannot be intercepted at runtime. We need to explicitly specify that someone can intercept annotations by setting the retention policy to `RUNTIME`. The `@Target` annotation specifies which language elements we can use this annotation for. By default, we can annotate any language elements, but it's always a good idea to restrict the annotation to only what you make it for — in our case, methods.

```java
package aspects;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ToLog {

}
```

`CommentService` class with three methods. We will annotate only one of them with our custom annotation. This is the one which should be intercepted using aspect.

```java
package services;

import models.Comment;
import aspects.ToLog;
import org.springframework.stereotype.Service;
import java.util.logging.Logger;

@Service
public class CommentService {

    Logger logger = Logger.getLogger(CommentService.class.getName());

    public void publishComment(Comment comment){
        logger.info("publishing comment by: " + comment.getAuthor() + " with message: " + comment.getMessage());
    }

    @ToLog
    public void deleteComment(Comment comment){
        logger.info("deleting comment by: " + comment.getAuthor() + " with message: " + comment.getMessage());
    }

    public void editComment(Comment comment){
        logger.info("editing comment by: " + comment.getAuthor() + " with message: " + comment.getMessage());
    }

}
```

Let us now created the `LoggingAspect` which will have a method to intercept the real calls. Rememeber that we need to add a bean of this class to spring context as well.

```java
package aspects;

import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.springframework.stereotype.Component;
import java.util.logging.Logger;

@Aspect
@Component
public class LoggingAspect {

    Logger logger = Logger.getLogger(LoggingAspect.class.getName());

    @Around("@annotation(ToLog)")
    public void log(ProceedingJoinPoint joinPoint) throws Throwable {

        logger.info("--------BEGIN EXECUTION--------");

        joinPoint.proceed();

        logger.info("--------END EXECUTION--------");

    }

}
```

The configuration class. Enable Aspects using `@EnableAspectJAutoProxy`.

```java
package config;

import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.EnableAspectJAutoProxy;

@Configuration
@ComponentScan(basePackages = {"services","aspects"})
@EnableAspectJAutoProxy
public class AppConfig {

}
```

Time to test using our main app:

```java
package main;

import config.AppConfig;
import models.Comment;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import services.CommentService;

public class App {

    public static void main(String[] args) {

        var context = new AnnotationConfigApplicationContext(AppConfig.class);

        Comment comment = new Comment();
        comment.setAuthor("Scooby");
        comment.setMessage("Scooby Doobey Doo!");

        CommentService commentService = context.getBean(CommentService.class);

        commentService.publishComment(comment);
        commentService.deleteComment(comment); //only this method will be intercepted since we have added `@ToLog` annotation to this method
        commentService.editComment(comment);

    }

}
```

**OUTPUT:**

Notice that the logs for the aspect method is printed only for `deleteComment()`.

```plaintext
Dec 23, 2023 11:16:42 PM services.CommentService publishComment
INFO: publishing comment by: Scooby with message: Scooby Doobey Doo!
Dec 23, 2023 11:16:42 PM aspects.LoggingAspect log
INFO: --------BEGIN EXECUTION--------
Dec 23, 2023 11:16:42 PM services.CommentService deleteComment
INFO: deleting comment by: Scooby with message: Scooby Doobey Doo!
Dec 23, 2023 11:16:42 PM aspects.LoggingAspect log
INFO: --------END EXECUTION--------
Dec 23, 2023 11:16:42 PM services.CommentService editComment
INFO: editing comment by: Scooby with message: Scooby Doobey Doo!
```
