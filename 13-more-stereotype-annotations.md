# MORE STEREOTYPE ANNOTATION IN SPRING

We have seen the use of `@Component` annotation. There are two other common stereotype annotations which can be used inplace of this generic type:

- `@Service`: The services are the objects with the responsibility of implementing the use cases like `CommentService` class in our examples.

  ```java
  package service;

  import model.Comment;
  import org.springframework.beans.factory.annotation.Qualifier;
  import org.springframework.stereotype.Service;
  import proxy.CommentNotificationProxy;
  import repository.CommentRepository;

  @Service
  public class CommentService {

      private final CommentRepository commentRepository;

      private final CommentNotificationProxy commentNotificationProxy;

      public CommentService(CommentRepository commentRepository, @Qualifier("EMAIL") CommentNotificationProxy commentNotificationProxy){
          this.commentRepository = commentRepository;
          this.commentNotificationProxy = commentNotificationProxy;
      }

      public void publishComment(Comment comment){
          commentRepository.storeComment(comment);
          commentNotificationProxy.sendComment(comment);
      }

  }
  ```

- `@Repository`: Objects managing the data persistence like `CommentRepository` class.

  ```java
  package repository;

  import model.Comment;
  import org.springframework.stereotype.Repository;

  @Repository
  public class DBCommentRepository implements  CommentRepository{


      @Override
      public void storeComment(Comment comment) {
          System.out.println("Storing comment to DB. Author: " + comment.getAuthor() + " commented: " + comment.getText());
      }
  }
  ```
