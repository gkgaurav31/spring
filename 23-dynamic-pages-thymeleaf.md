# CREATING DYNAMIC PAGE USING THYMELEAF

In the previous example, we create a simple Web Application using Spring Boot which return a static HTML page. Most applications today are dynamic. In this example, we will use a `template engine` called `Thymeleaf` to create dynamic content in Spring Boot.

![dynamic-page-spring](images/dynamic-page-spring.png)

## DEPENDENCY

To use Thymeleaf, we would need to following dependency in pom.xml:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

- Create a `home.html` file under `resources/templates` (not resources/static) like below:

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
  <head>
    <meta charset="UTF-8" />
    <title>Hello Spring</title>
  </head>
  <body>
    <h1>
      Welcome <span th:style="'color:' + ${color}" th:text="${username}"></span>
    </h1>
  </body>
</html>
```

Here we have used a namespace for thymeleaf `xmlns:th="http://www.thymeleaf.org"`. This is similar to import statements in Java.

- Create a controller to handle requests to /home. We also add a parameter `Model`. We can add attributes to this model object as key-value pairs. This will be passed on to the view for rendering. Notice that we have use these keys `username` and `color` in the HTML template.

```java
package com.example.demo;

import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestMapping;

@Controller
public class MainController {

    @RequestMapping("/home")
    public String home(Model model){

        model.addAttribute("username", "Katy");
        model.addAttribute("color", "green");

        return "home.html";
    }

}
```
