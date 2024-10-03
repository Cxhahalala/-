# 什么是异常处理

在日常的 Web 开发中，项目中难免会出现各种异常，为了使客户端能接收较为友好的提示，通常开发者会对异常进行统一处理。为了便于开发者处理异常，Spring Boot通过自动装配提供了一套默认的异常处理机制，一旦程序中出现了异常，Spring Boot会根据该机制进行默认的异常处理。除了默认的异常处理，Spring Boot也支持自定义异常处理。

# Spring Boot自动配置异常处理原理

Spring Boot 提供了开箱即用的异常处理自动配置，通过 `ErrorMvcAutoConfiguration` 类将多个组件注入到容器中。

- **ErrorPageCustomizer**：该类内部会定义错误响应规则，通常会配置一个 `/error` 路径，并将所有异常转发到这个路径。我们可以通过覆盖 `/error` 路径的处理逻辑，自定义错误页面。
- **BasicErrorController**：Spring Boot 提供的默认控制器，处理 `/error` 路径。通过自定义 `MyExceptionController` 来替代 `BasicErrorController`，处理异常并返回自定义错误视图。
- **DefaultErrorViewResolver**：该类会根据状态码、路径等信息来解析到具体的错误视图指定了返回 `error.html`，而 `DefaultErrorViewResolver` 会将 `Model` 中的数据传递到视图中。
- **DefaultErrorAttributes**：在 Spring Boot 中，错误信息默认会存储在 `DefaultErrorAttributes` 中，比如 `status`, `error`, `message`, `path` 等。我们可以从 `HttpServletRequest` 中获取这些错误信息，传递给视图渲染。

# @ControllerAdvice+@ExceptionHandler+@ResponseStatus

**@ControllerAdvice**和**@ExceptionHandler**是由Spring FrameWork提供的注解，是一种全局异常处理机制，可以让我们**自定义处理不同类型的异常并返回自定义的响应或者视图**。

Spring Boot是基于Spring Framework的拓展，因此也能使用**@ControllerAdvice**和**@ExceptionHandler。**

使用注解的方式配置异常处理较为简单。

## **@ControllerAdvice**

**作用**

- `@ControllerAdvice` 是一个全局控制器增强注解，它允许你将全局的异常处理逻辑、数据绑定和模型属性注入提取到一个单独的类中，应用于整个应用的所有控制器。这样，你不需要在每个控制器中编写重复的异常处理代码。
- 它通常用来处理全局范围内的异常，可以与 `@ExceptionHandler` 配合使用

**用法**

- 使用 `@ControllerAdvice` 可以定义一个全局异常处理类，该类中的 `@ExceptionHandler` 方法可以捕获应用中所有控制器抛出的异常。
- 它不仅可以处理异常，还能在所有的控制器执行前后，执行一些公共的逻辑，比如全局数据绑定、模型属性处理等。

## **@ExceptionHandler**

**作用：**

- `@ExceptionHandler` 是一个局部异常处理注解，它可以用于单个控制器类或者与 `@ControllerAdvice` 结合使用，用来处理特定类型的异常。
- 当控制器中的某个方法抛出异常时，Spring 会寻找与该异常类型匹配的 `@ExceptionHandler` 方法，并调用它来处理异常。

**用法：**

- `@ExceptionHandler` 注解可以标注在控制器类中的一个方法上，该方法会捕获指定类型的异常并处理它。
- 可以处理局部控制器类中的异常，或者通过 `@ControllerAdvice` 来处理全局范围的异常。

## @ResponseStatus

`@ResponseStatus` 注解可以标记在控制器方法或者异常处理方法上，用于指定该方法返回的 HTTP 状态码。它通常与 `@ExceptionHandler` 一起使用，处理异常时返回指定的状态码，而不是默认的 500 内部服务器错误。

## **创建异常类**

ResourceNotFoundException.java

```
package com.example.demochapter02.exception;
//继承自RuntimeException
@ResponseStatus(value = HttpStatus.NOT_FOUND, reason = "Resource Not Found")
public class ResourceNotFoundException extends RuntimeException{
    public ResourceNotFoundException(String message) {
        super(message);
    }
}
```

## 全局异常处理类

GlobalExceptionHandler.java

### 返回自定义响应

自定义响应可以在前后端分离项目中使用，前端根据响应决定渲染的页面或内容。

```
package com.example.demochapter02.error;

import com.example.demochapter02.exception.ResourceNotFoundException;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.context.request.WebRequest;
import org.springframework.web.servlet.ModelAndView;

import java.util.HashMap;
import java.util.Map;

@ControllerAdvice
public class GlobalExceptionHandler {
//捕获ResourceNotFoundException并作出处理
    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<Map<String,Object>> handleResourceNotFoundException(ResourceNotFoundException ex, WebRequest request){
        // 自定义返回的 JSON 数据
        Map<String, Object> body = new HashMap<>();
        body.put("message", ex.getMessage());
        body.put("status", HttpStatus.NOT_FOUND.value());

        return new ResponseEntity<>(body, HttpStatus.NOT_FOUND);
    }
//捕获RuntimeException.class并进行处理
    @ExceptionHandler(RuntimeException.class)
    public ResponseEntity<Map<String, Object>> handleGlobalException(RuntimeException ex, WebRequest request) {
        // 自定义返回的 JSON 数据
        Map<String, Object> body = new HashMap<>();
        body.put("message", "服务器内部错误");
        body.put("status", HttpStatus.INTERNAL_SERVER_ERROR.value());

        return new ResponseEntity<>(body, HttpStatus.INTERNAL_SERVER_ERROR);
    }
}
```

### 返回自定义页面

返回的页面需要由thymeleaf渲染，因此需要引入thymeleaf依赖。

thymeleaf默认寻找html在/resource/templates目录下，会自动匹配路径和后缀，因此只需要设置视图名称即可。

如设置视图名为login,那么就会自动匹配到/resource/templates/login.html

```
package com.example.demochapter02.error;

import com.example.demochapter02.exception.ResourceNotFoundException;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.context.request.WebRequest;
import org.springframework.web.servlet.ModelAndView;

import java.util.HashMap;
import java.util.Map;

@ControllerAdvice
public class GlobalExceptionHandler {
//// 处理自定义的 ResourceNotFoundException 异常，返回 HTML 页面
@ExceptionHandler(ResourceNotFoundException.class)
public ModelAndView handleResourceNotFoundException(ResourceNotFoundException ex, WebRequest request) {
    ModelAndView mav = new ModelAndView();
    mav.setViewName("error404");  // 指定返回的 HTML 页面模板
    mav.setStatus(HttpStatus.NOT_FOUND);  // 设置状态码为404
    mav.addObject("message", ex.getMessage());  // 将错误信息传递到页面
// 即/resource/templates/error404.html
    return mav;
}

    // 处理所有 RuntimeException 异常，返回 HTML 错误页面
    @ExceptionHandler(RuntimeException.class)
    public ModelAndView handleRuntimeException(RuntimeException ex, WebRequest request) {
        ModelAndView mav = new ModelAndView();
        mav.setViewName("error500");  // 返回 error500.html 页面
        mav.setStatus(HttpStatus.INTERNAL_SERVER_ERROR);  // 设置状态码为500
        mav.addObject("message", ex.getMessage());  // 将异常消息传递到页面
// 即/resource/templates/error500.html
        return mav;
    }
}
```

## 测试

TestExceptionController.java

```
package com.example.demochapter02.Controller;

import com.example.demochapter02.exception.ResourceNotFoundException;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

//@RestController
@Controller
public class TestExceptionController {
    // 访问这个端点会触发 ResourceNotFoundException
    @GetMapping("/notfound")
    public String trigger404() {
        throw new ResourceNotFoundException("资源未找到");
    }

    // 访问这个端点会触发一个一般的运行时异常
    @GetMapping("/trigger-error")
    public String triggerError() {
        System.out.println("进入了trigger-error");
        throw new RuntimeException("服务器内部错误");
    }
}
```

当访问这些路径时，就会抛出异常然后由全局异常处理器捕获并处理。