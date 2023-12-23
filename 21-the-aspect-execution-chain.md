# THE ASPECT EXECUTION CHAIN

There can be multiple aspects tier to the same method. In certain cases, it becomes important to ensure that they are executed in a certain manner. For example: Let's say, we have an aspect which handles logging and another which handles security. If security aspect gets executed first and chooses the deny the call further (does not proceed), it may not get logged. So, we need a way to make sure that the logging aspect gets executed first. We can achieve this using `@Order` annotation for the aspects.

We will create two aspects, one to mimic logging and another for security. We will not really implement these behavior as we are more interested in the flow of calls here. To keep it simpler, we will skip using any model. Just a basic service. Let's make use of the custom annotation which we had created earlier as well.

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

```java
package services;

import aspects.ToLog;
import org.springframework.stereotype.Service;

import javax.xml.stream.events.Comment;
import java.util.logging.Logger;

@Service
public class CommentService {

    Logger logger = Logger.getLogger(CommentService.class.getName());

    @ToLog
    public void publishComment(){
        logger.info("publishing comment");
    }

}
```

Let's create the two aspects. Both of them are supposed to run for all methods annotated with the custom annotation `@ToLog`.

```java
package aspects;

import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.springframework.core.annotation.Order;
import org.springframework.stereotype.Component;

import java.util.logging.Logger;

@Aspect
@Component
public class LoggingAspect {

    Logger logger = Logger.getLogger(LoggingAspect.class.getName());

    @Around("@annotation(ToLog)")
    public void log(ProceedingJoinPoint joinPoint) throws Throwable {
        logger.info("in logging aspect");
        joinPoint.proceed();
    }

}
```

```java
package aspects;

import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.springframework.core.annotation.Order;
import org.springframework.stereotype.Component;

import java.util.logging.Logger;

@Aspect
@Component
public class SecurityAspect {

    Logger logger = Logger.getLogger(LoggingAspect.class.getName());

    @Around("@annotation(ToLog)")
    public void validate(ProceedingJoinPoint joinPoint) throws Throwable {
        logger.info("in validation aspect");
        joinPoint.proceed();
    }
}
```

The usual configuration class:

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

A simple main App:

```java
package main;

import config.AppConfig;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import services.CommentService;

public class App {

    public static void main(String[] args) {

        var context = new AnnotationConfigApplicationContext(AppConfig.class);

        CommentService commentService = context.getBean(CommentService.class);

        commentService.publishComment();

    }

}
```

The `publishComment()` method has been annotated with our custom annotation `@ToLog`. Both the aspects `LoggingAspect` and `SecurityAspect` have a method which should run when the method with `@ToLog` annotation gets executed. This has been done using the advice `@Around("@annotation(ToLog)")`. So, now the question is which one should run first? Well, spring does not guarantee the order here. Since we want to ensure that the `LoggingAspect` runs first, we will give it an earlier "Order". Here's how we can do it:

```java
package aspects;

import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.springframework.core.annotation.Order;
import org.springframework.stereotype.Component;

import java.util.logging.Logger;

@Aspect
@Component
@Order(1)
public class LoggingAspect {

    Logger logger = Logger.getLogger(LoggingAspect.class.getName());

    @Around("@annotation(ToLog)")
    public void log(ProceedingJoinPoint joinPoint) throws Throwable {
        logger.info("in logging aspect");
        joinPoint.proceed();
    }

}
```

Similar, for the `SecurityAspect`, but with a larger value for `@Order`:

```java
package aspects;

import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.springframework.core.annotation.Order;
import org.springframework.stereotype.Component;

import java.util.logging.Logger;

@Aspect
@Component
@Order(2)
public class SecurityAspect {

    Logger logger = Logger.getLogger(LoggingAspect.class.getName());

    @Around("@annotation(ToLog)")
    public void validate(ProceedingJoinPoint joinPoint) throws Throwable {
        logger.info("in validation aspect");
        joinPoint.proceed();
    }
}
```

Now, we can be sure that `LoggingAspect` will run after `SecurityAspect` which can then call the intercepted method. The intercepted method returns to the SecurityAspect, which returns further to the LoggingAspect
