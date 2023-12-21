# BASE PACKAGE VS BASE PACKAGE CLASSES

When defining our configuration class, we use `@ComponentScan` to provide the location for classes which use stereotype annotations like `@Component`.

There are two ways in which we can provide this location:

- Using `basePackages`

Here we just provide the package name which has the classes with stereotype annotation. It can come in handy if we have a lot of such classes.

```java
package config;

import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

@Configuration
@ComponentScan(basePackages = "proxy,repository,service")
public class AppConfig {

}
```

- Using `basePackageClasses`

We would need to provide the classes with the stereotype annotations directly. This can potentially help in avoiding issues later in case the package name is changed (as the code will stop compiling if this case)

```java
package config;

import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import proxy.EmailCommentNotificationProxy;
import repository.DBCommentRepository;
import service.CommentService;

@Configuration
@ComponentScan(basePackageClasses = {EmailCommentNotificationProxy.class, DBCommentRepository.class, CommentService.class})
public class AppConfig {

}
```

We can also use the complete path for the class names:

```java
package config;

import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

@Configuration
@ComponentScan(basePackageClasses = {proxy.EmailCommentNotificationProxy.class, repository.DBCommentRepository.class, service.CommentService.class})
public class AppConfig {

}
```
