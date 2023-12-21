# CHOOSING WHAT TO AUTO-WIRE FROM MULTIPLE IMPLEMENTATIONS OF AN ABSTRACTION

In our previous example, we had a single implementation of each type of dependency for `CommentService` class. So, we did not have to do anything special while using `@Autowired` for the dependencies `CommentRepository` and `CommentNotificationProxy`.

Suppose, we create another implementation of `CommentNotificationProxy` and mark it as a `@Component`.

```java
package proxy;

import model.Comment;
import org.springframework.stereotype.Component;

@Component
public class CommentPushNotificationProxy implements CommentNotificationProxy{

    @Override
    public void sendComment(Comment comment) {
        System.out.println("Push Notification by: " + comment.getAuthor() + " . The message is: " + comment.getText());
    }

}
```

If we run the same App again, it will fail due to `NoUniqueBeanDefinitionException` since we now have two beans for `CommentNotificationProxy` in the `CommentService` class and we need to tell Spring which one to choose.

A couple of ways which we have seen previously:

- Using the `@Primary` annotation to mark one of the beans for implementation as the default. For example, we can mark `EmailCommentNotificationProxy` as `@Primary` to avoid conflict.

  ```java
  package proxy;

  import model.Comment;
  import org.springframework.context.annotation.Primary;
  import org.springframework.stereotype.Component;

  @Component
  @Primary
  public class EmailCommentNotificationProxy implements CommentNotificationProxy{

      @Override
      public void sendComment(Comment comment) {
          System.out.println("Notifying by e-mail. Author: " + comment.getAuthor() + " commented: " + comment.getText());
      }
  }
  ```

- Using the `@Qualifier` annotation to name a bean and then refer to it by its name for Dependency Injection.

First, we add a `@Qualifier` annotation to each implementation to give it a unique name/id.

```java
package proxy;

import model.Comment;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.context.annotation.Primary;
import org.springframework.stereotype.Component;

@Component
@Qualifier("EMAIL")
public class EmailCommentNotificationProxy implements CommentNotificationProxy{

    @Override
    public void sendComment(Comment comment) {
        System.out.println("Notifying by e-mail. Author: " + comment.getAuthor() + " commented: " + comment.getText());
    }
}
```

```java
package proxy;

import model.Comment;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.stereotype.Component;

@Component
@Qualifier("PUSH")
public class CommentPushNotificationProxy implements CommentNotificationProxy{

    @Override
    public void sendComment(Comment comment) {
        System.out.println("Push Notification by: " + comment.getAuthor() + " . The message is: " + comment.getText());
    }

}
```

Next, we make use this of identifier in our `CommentService`:

```java
package service;

import model.Comment;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.stereotype.Component;
import proxy.CommentNotificationProxy;
import proxy.CommentPushNotificationProxy;
import repository.CommentRepository;

@Component
public class CommentService {

    private final CommentRepository commentRepository;

    private final CommentNotificationProxy commentNotificationProxy;

    public CommentService(CommentRepository commentRepository, @Qualifier("PUSH") CommentNotificationProxy commentNotificationProxy){
        this.commentRepository = commentRepository;
        this.commentNotificationProxy = commentNotificationProxy;
    }

    public void publishComment(Comment comment){
        commentRepository.storeComment(comment);
        commentNotificationProxy.sendComment(comment);
    }

}
```

Spring will now choose the implementation based on `@Qualifier` annotation provided for the parameters in the constructor of `CommentService`.
