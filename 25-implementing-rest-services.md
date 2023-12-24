# IMPLEMENTING REST SERVICES

When implementing REST endpoints, the Spring MVC flow changes. The app no longer needs a view resolver because the client needs the data returned by the controller's action directly. Once the controller's action completes, the dispatcher servlet returns the HTTP response without rendering any view.

![rest-services-flow](images/rest-services-flow.png)

Here's an example:

The annotation `@ResponseBody` tells the dispatcher servlet that the controller's action doesn't return a view name but the data sent directly in the HTTP response. To avoid adding this annotation one each method, spring provides another way, which is to annotated the class with `@RestController`.

```java
package com.example.demo.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.ResponseBody;

@Controller
public class HelloController {

    @GetMapping("/hello")
    @ResponseBody
    public String hello(){
        return "hello!";
    }

}
```

Using `@RestController`:

```java
package com.example.demo.controller;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {

    @GetMapping("/hello")
    public String hello(){
        return "hello!";
    }

}
```

TEST:

```bash
curl -X GET http://localhost:8080/hello
hello!
```
