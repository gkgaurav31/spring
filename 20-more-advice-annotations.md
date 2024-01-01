# MORE ADVICE ANNOTATIONS THAT WE CAN USE

`@Around` annotation that we have seen is the most common advice annotation that is used. There are some other advice annotation, which may not be as powerful, but they can make the code/logic simpler, thereby promoting maintainability of the app. Let's take a look at them:

- `@Before`: Calls the method defining the aspect logic before the execution of the intercepted method.

  ```java
  @Before("execution(* services.CommentService.publishComment(..))")
  public void log(){
      logger.info("executing aspect!");
  }
  ```

  ```text
  Dec 23, 2023 11:38:34 PM aspects.LoggingAspect log
  INFO: executing aspect!
  Dec 23, 2023 11:38:34 PM services.CommentService publishComment
  INFO: publishing comment: Comment{author='Scooby', message='Scooby Doobey Doo'}
  ```

- `@AfterReturning`: Calls the method defining the aspect logic after the method successfully returns, and provides the returned value as a parameter to the aspect method. The aspect method isn't called if the intercepted method throws an exception.

  ```java
  @AfterReturning("execution(* services.CommentService.publishComment(..))")
  public void log(){
      logger.info("executing aspect!");
  }
  ```

  ```text
  Dec 23, 2023 11:41:56 PM services.CommentService publishComment
  INFO: publishing comment: Comment{author='Scooby', message='Scooby Doobey Doo'}
  Dec 23, 2023 11:41:56 PM aspects.LoggingAspect log
  INFO: executing aspect!
  ```

- `@AfterThrowing`: Calls the method defining the aspect logic if the intercepted method throws an exception, and provides the exception instance as a parameter to the aspect method.

  In Service:

  ```java
  public void publishComment(Comment comment) throws Exception {
      logger.info("publishing comment: " + comment);
      throw new Exception("test exception");
  }
  ```

  In aspect method:

  ```java
  @AfterThrowing(value = "execution(* services.CommentService.publishComment(..))", throwing = "error")
  public void log(JoinPoint jp, Throwable error){
      logger.info("executing aspect! " + error);
  }
  ```

  In main App:

  ```java
  try {
      commentService.publishComment(comment);
  } catch (Exception e) {
      System.out.println("handled exception: " + e.getMessage());
  }
  ```

  ```text
  Dec 23, 2023 11:48:33 PM services.CommentService publishComment
  INFO: publishing comment: Comment{author='Scooby', message='Scooby Doobey Doo'}
  Dec 23, 2023 11:48:33 PM aspects.LoggingAspect log
  INFO: executing aspect! java.lang.Exception: test exception
  handled exception: test exception
  ```

- `@After`: Calls the method defining the aspect logic only after the intercepted method execution, whether the method successfully returned or threw an exception.

  ```java
  @After(value = "execution(* services.CommentService.publishComment(..))")
  public void log(JoinPoint jp){
      logger.info("executing aspect! ");
  }
  ```

  ```text
  Dec 23, 2023 11:51:35 PM services.CommentService publishComment
  INFO: publishing comment: Comment{author='Scooby', message='Scooby Doobey Doo'}
  Dec 23, 2023 11:51:35 PM aspects.LoggingAspect log
  INFO: executing aspect!
  handled exception: test exception
  ```
