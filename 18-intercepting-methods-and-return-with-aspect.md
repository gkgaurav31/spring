# ALTERING THE INTERCEPTED METHOD'S PARAMETERS AND THE RETURNED VALUE

We created a logging aspect in the previous example which could add the logs when a method is executed. Here, I would like to show, how we can further intercept and even change the parameters and return values passed by the caller.

We will modify the method `publishComment()` in `CommentService` so that it returns a String.

```java
package services;

import models.Comment;
import org.springframework.stereotype.Service;

import java.util.logging.Logger;

@Service
public class CommentService {

    private Logger logger = Logger.getLogger(CommentService.class.getName());

    public String publishComment(Comment comment){
        logger.info("publishing comment: " + comment.getText() + " by " + comment.getAuthor());
        return "PUBLISHED!";
    }

    public void validateComment(Comment comment){
        if(comment.getText().isEmpty())
            logger.warning("invalid comment");
        else
            logger.info("valid comment");
    }

}
```

Let us also add a `toString()` method to the `Comment` class:

```java
package models;

public class Comment {

    private String author;
    private String text;

    public String getAuthor() {
        return author;
    }

    public void setAuthor(String author) {
        this.author = author;
    }

    public String getText() {
        return text;
    }

    public void setText(String text) {
        this.text = text;
    }

    @Override
    public String toString() {
        return "Comment{" +
                "author='" + author + '\'' +
                ", text='" + text + '\'' +
                '}';
    }
}
```

Now, let's update the aspect to log the method name which is supposed to be called and its parameters.

We can get the get the actual method name using: `String methodName = joinPoint.getSignature().getName()`
We can get the parameters passed while calling this method in the aspect method using: `Object[] args = joinPoint.getArgs()`

```java
package aspects;

import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.springframework.stereotype.Component;

import java.util.Arrays;
import java.util.logging.Logger;

@Aspect
@Component
public class LoggingAspect {

    private Logger logger = Logger.getLogger(LoggingAspect.class.getName());

    @Around("execution(* services.*.*(..))")
    public Object log(ProceedingJoinPoint joinPoint) throws Throwable {

        //get the method name to be called
        String methodName = joinPoint.getSignature().getName();
        //get the args list which were passed while calling the method
        Object[] args = joinPoint.getArgs();

        //log them
        logger.info("Intercepting: " + methodName + ". Arguments: " + Arrays.asList(args));

        logger.info("------ starting execution ------");

        //call the actual method from the aspect proxy
        Object returnedByMethod = joinPoint.proceed();

        //log the return value
        logger.info("Returned by method: " + returnedByMethod);

        logger.info("------ execution complete ------");

        //here, instead of returning "returnedByMethod", we change the return value to COMPLETE
        return "COMPLETE";

    }

}
```

In the above code, we have logged the method called and the parameters passed. The actual return should have been `PUBLISHED!`. However, we changed it using aspect to `COMPLETE`.

**OUTPUT:**

```text
Dec 23, 2023 8:38:45 PM aspects.LoggingAspect log
INFO: Intercepting: publishComment. Arguments: [Comment{author='Scooby', text='Scooby Doobey Doo!'}]
Dec 23, 2023 8:38:45 PM aspects.LoggingAspect log
INFO: ------ starting execution ------
Dec 23, 2023 8:38:45 PM services.CommentService publishComment
INFO: publishing comment: Scooby Doobey Doo! by Scooby
Dec 23, 2023 8:38:45 PM aspects.LoggingAspect log
INFO: Returned by method: PUBLISHED!
Dec 23, 2023 8:38:45 PM aspects.LoggingAspect log
INFO: ------ execution complete ------
COMPLETE
```
