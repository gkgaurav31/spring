# USING SPRING IN A REAL WORLD APP

We will start by creating a simple app which does not use Spring. Then, use what we have learnt previously to _Springify_ it.

## USE-CASE

Say you are implementing an app a team uses to manage their tasks. One of the
app's features is allowing the users to leave comments for the tasks.

When a user publishes a comment, it is stored somewhere (e.g., in a database), and the app sends an email to a specific address configured in the app.

We need to design the objects and find the right responsibilities and abstractions
for implementing this feature.

## IMPLEMENTING IT THE USUAL WAY (WITHOUT SPRING)

- We can start by creating a `Comment` class to model the comment which needs to be stored in the Databases or used during notification.

```java
package model;

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
}
```

- Next, we need a `CommentRepository` interface and its implementation `DBCommentRepository`. As we have seen, it's better to code to interfaces as the implementation can change later.

```java
package repository;

import model.Comment;

public interface CommentRepository {
    public void storeComment(Comment comment);
}
```

```java
package repository;

import model.Comment;

public class DBCommentRepository implements  CommentRepository{


    @Override
    public void storeComment(Comment comment) {
        System.out.println("Storing comment to DB. Author: " + comment.getAuthor() + " commented: " + comment.getText());
    }
}
```

- Similarly, we would need a notification interface `CommentNotificationProxy` and its implementation `EmailCommentNotificationProxy`.

```java
package proxy;

import model.Comment;

public interface CommentNotificationProxy {
    public void sendComment(Comment comment);
}
```

```java
package proxy;

import model.Comment;

public class EmailCommentNotificationProxy implements CommentNotificationProxy{

    @Override
    public void sendComment(Comment comment) {
        System.out.println("Notifying by e-mail. Author: " + comment.getAuthor() + " commented: " + comment.getText());
    }
}
```

- The `CommentRepository` and `CommentNotificationProxy` will be the dependencies for the `CommentService` class this will be responsible for publishing the comment `publishComment()`. We will add a constructor to help initialize this service.

```java
package service;

import model.Comment;
import proxy.CommentNotificationProxy;
import repository.CommentRepository;

public class CommentService {

    private final CommentRepository commentRepository;
    private final CommentNotificationProxy commentNotificationProxy;

    //We would have to use @Autowired if the class had more than one constructor
    //Spring uses this constructor to create the bean and injects references from its context in the parameters when creating the instance.
    public CommentService(CommentRepository commentRepository, CommentNotificationProxy commentNotificationProxy){
        this.commentRepository = commentRepository;
        this.commentNotificationProxy = commentNotificationProxy;
    }

    public void publishComment(Comment comment){
        commentRepository.storeComment(comment);
        commentNotificationProxy.sendComment(comment);
    }

}
```

- Now that we have all the required classes, we can run our app from a main class.

```java
package main;

import model.Comment;
import proxy.EmailCommentNotificationProxy;
import repository.DBCommentRepository;
import service.CommentService;

public class App {

    public static void main(String[] args) {

        Comment comment = new Comment();
        comment.setAuthor("Alan Wake");
        comment.setText("Have you played it yet?");

        CommentService commentService = new CommentService(new DBCommentRepository(), new EmailCommentNotificationProxy());

        commentService.publishComment(comment);

    }

}
```

- OUTPUT

```text
Storing comment to DB. Author: Alan Wake commented: Have you played it yet?
Notifying by e-mail. Author: Alan Wake commented: Have you played it yet?
```

## SPRINGIFY IT

- The first step is to identify the beans that we would need the spring to maintain for us. In our case, we don't really need to have a bean for `Comment` to be maintained by Spring since it does not depend on any other class or vice versa. Rest all the concrete classes would be needed. We don't need to mark the interfaces with `@Component` since they cannot have objects directly.

- So, we would need to add `@Component` annotation to the following classes:

  - `CommentService`
  - `DBCommentRepository`
  - `EmailCommentNotificationProxy`

- We would need a `@Configuration` class as well to tell spring the package location to look for `@Component` annotated classes. We would need to create a new class for this.

```java
package config;

import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

@Configuration
@ComponentScan(basePackages = "proxy,repository,service")
public class AppConfig {

}
```

- And that's majorly it. Obviously, we would change the way we run our main App now. Instead of creating the object for `CommentService`, we can get it from spring context!

```java
package main;

import config.AppConfig;
import model.Comment;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import service.CommentService;

public class App {

    public static void main(String[] args) {

        var context = new AnnotationConfigApplicationContext(AppConfig.class);

        CommentService commentService = context.getBean(CommentService.class);

        Comment comment = new Comment();
        comment.setAuthor("Alan Wake");
        comment.setText("Hope you have a great time!");

        commentService.publishComment(comment);

    }

}
```

- In the `CommentService` class, instead of using constructor to injected the dependencies, we can make use of `@Autowired` annotation. The code would look like this:

```java
package service;

import model.Comment;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;
import proxy.CommentNotificationProxy;
import repository.CommentRepository;

@Component
public class CommentService {

    @Autowired
    private CommentRepository commentRepository; //removed final

    @Autowired
    private CommentNotificationProxy commentNotificationProxy; //removed final

    public void publishComment(Comment comment){
        commentRepository.storeComment(comment);
        commentNotificationProxy.sendComment(comment);
    }

}
```

Spring uses the default constructor to create the instance of the
`CommentService` class and then injects the two dependencies from its context.

- Instead of using `@Component` annotation, we can also make use of `@Bean` in configuration class. So, we can remove `@Component` and `@ComponentScan` from the corresponding classes. And we can create the beans as we have done before using `@Bean` still using abstraction through interfaces.

```java
package config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import proxy.CommentNotificationProxy;
import proxy.EmailCommentNotificationProxy;
import repository.CommentRepository;
import repository.DBCommentRepository;
import service.CommentService;

@Configuration
public class AppConfig {
    @Bean
    public CommentRepository commentRepository(){
        return new DBCommentRepository();
    }

    @Bean
    public CommentNotificationProxy commentNotificationProxy(){
        return new EmailCommentNotificationProxy();
    }

    @Bean
    public CommentService commentService(CommentRepository commentRepository, CommentNotificationProxy commentNotificationProxy){
        return new CommentService(commentRepository, commentNotificationProxy);
    }
}
```

```java
package service;

import model.Comment;
import org.springframework.stereotype.Component;
import proxy.CommentNotificationProxy;
import repository.CommentRepository;

@Component
public class CommentService {

    private CommentRepository commentRepository;
    private CommentNotificationProxy commentNotificationProxy;

    public CommentService(CommentRepository commentRepository, CommentNotificationProxy commentNotificationProxy) {
        this.commentRepository = commentRepository;
        this.commentNotificationProxy = commentNotificationProxy;
    }

    public void publishComment(Comment comment){
        commentRepository.storeComment(comment);
        commentNotificationProxy.sendComment(comment);
    }

}
```

### Benefits of Using Spring and Dependency Injection

1. **Decoupling of Components:**

   - Spring's dependency injection decouples the `CommentService` from concrete implementations (`DBCommentRepository` and `EmailCommentNotificationProxy`). The service relies on abstractions (interfaces) rather than specific implementations, enhancing flexibility and allowing easier changes or substitutions in the future.

2. **Easier Management of Dependencies:**

   - Spring manages the creation and lifecycle of beans (`CommentService`, `DBCommentRepository`, `EmailCommentNotificationProxy`). It handles object instantiation, thus relieving the developer from manually managing dependencies.

3. **Simplified Configuration:**

   - With `@ComponentScan` and `@Configuration`, Spring auto-discovers components and wires them together based on annotations. It reduces the need for explicit object creation, enabling a more concise and maintainable codebase.
