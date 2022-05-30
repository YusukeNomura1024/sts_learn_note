# Map の参照
```java
package sample.thymeleaf.web;

import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;

import java.util.HashMap;

@Controller
public class HelloController {

    @GetMapping("/hello")
    public String hello(Model model) {
        HashMap<String, String> map = new HashMap<>();
        map.put("message", "Hello World!!");

        model.addAttribute("map", map);
        return "hello";
    }
}
```
```html
<!doctype html>
<html xmlns:th="http://www.thymeleaf.org">
    <head>
        <meta charset="UTF-8" />
        <title>Hello Thymeleaf</title>
    </head>
    <body>
        <div th:text="${map.message}"></div>
        <div th:text="${map['message']}"></div>
    </body>
</html>
```
* Map の場合、 map.キー で値を参照できる

* また map['キー'] のように角括弧で参照することもできる