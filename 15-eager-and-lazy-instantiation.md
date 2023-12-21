# USING EAGER AND LAZY INSTANTIATION

Spring creates all singleton beans when it initializes the context—this is
Spring's default behavior. We can use lazy initialization using the annotation `@Lazy`.

## EXAMPLE: EAGER INSTANTIATION (DEFAULT)

- Let's start with the usual `CommentRepository`. Nothing special here. We will use this as a dependency for `CommentService`. Let's add a constuction which prints a line which an instance of this class gets created.

```java
package repository;

import org.springframework.stereotype.Repository;

@Repository
public class CommentRepository {

    public CommentRepository(){
        System.out.println("CommentRepository instance created!");
    }

}
```

- We will add a similar constructor for `CommentService` as well.

```java
package service;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Lazy;
import org.springframework.stereotype.Service;
import repository.CommentRepository;

@Service
public class CommentService {

    @Autowired
    private CommentRepository commentRepository;

    public CommentService(){
        System.out.println("CommentService instance created!");
    }

}
```

- Since we are using stereotype annotation, let's have the configuration file with `@ComponentScan` annotation.

```java
package config;

import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

@Configuration
@ComponentScan(basePackages = {"repository","service"})
public class AppConfig {

}
```

### TEST WITH DEFAULT EAGER INITIALIZATION

We have not added any `@Lazy` annotation, so the beans should get initialized during app startup. This means, as soon as we run our app, we should see the beans get create. We will track this using the print statements that we added in the constructor of the service and repository classes.

```java
package main;

import config.AppConfig;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import service.CommentService;

public class App {
    public static void main(String[] args) {

        var context = new AnnotationConfigApplicationContext(AppConfig.class);

        System.out.println("BEFORE");
        CommentService commentService = context.getBean(CommentService.class);
        System.out.println("AFTER");

    }
}
```

**OUTPUT:**

Notice that the print statements from the constructor are present already before "BEFORE" when the spring context is initialized.

```text
CommentRepository instance created!
CommentService instance created!
BEFORE
AFTER
```

### TEST WITH LAZY INITIALIZATION

For this, let's add `@Lazy` annotation to our `CommentService`

```java
package service;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Lazy;
import org.springframework.stereotype.Service;
import repository.CommentRepository;

@Service
@Lazy
public class CommentService {

    @Autowired
    private CommentRepository commentRepository;

    public CommentService(){
        System.out.println("CommentService instance created!");
    }

}
```

- Now let's run the same main App. No changes to the code here.

```java
package main;

import config.AppConfig;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import service.CommentService;

public class App {
    public static void main(String[] args) {

        var context = new AnnotationConfigApplicationContext(AppConfig.class);

        System.out.println("BEFORE");
        CommentService commentService = context.getBean(CommentService.class);
        System.out.println("AFTER");

    }
}
```

**OUTPUT:**

The CommentService instance got created at the time of its use `context.getBean(CommentService.class)`!

```text
CommentRepository instance created!
BEFORE
CommentService instance created!
AFTER
```

## WHICH ONE SHOULD WE CHOOSE

In most cases, eager initialization should work well for us. If we use lazy initialization, spring will have to keep a check if the bean exists, which adds an overhead. Lazy initialization could potentially reduce the memory footprints for monolithic apps, however, generally there is a better solution in those scenarios.
