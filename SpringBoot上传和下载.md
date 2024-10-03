SpringBoot对于文件上传提供了自动配置的方式，使得配置文件上传与下载没有ssm中那样繁琐。

默认情况下springboot通过`MultipartAutoConfiguration` 来自动配置文件上传功能，只需要在配置文件中对文件进行基本设置即可。

# MultipartFile



# 配置文件

在springboot的全局配置文件中，我们可以设置上传文件的大小，请求总数据的大小以及临时存储文件路径。

```
spring:
  application:
    name: demochapter03
  servlet:
    multipart:
      max-file-size: 10MB
      max-request-size: 50MB
#临时文件存储路径
#     location=/tmp
```

# 相关接口解释

## org.springframework.core.io.Resource

### 1. `Resource` 接口的主要作用

`Resource` 接口将不同类型的资源（如文件系统中的文件、类路径资源、URL 资源等）抽象为一个统一的接口，从而使得应用程序可以通过统一的方式加载和操作这些资源。

常见的使用场景包括：

- 加载配置文件（如 XML、properties 文件）。
- 加载静态资源（如图片、HTML 文件）。
- 读取外部文件（如远程 URL 资源）。

### 2. `Resource` 的常用实现类

`Resource` 是一个接口，Spring 提供了多个具体的实现类，用来处理不同类型的资源：

- `FileSystemResource`：用于访问文件系统中的文件。
- `ClassPathResource`：用于访问类路径下的资源（例如 `.properties` 文件）。
- `UrlResource`：用于访问基于 URL 的资源（如 HTTP、FTP 资源）。
- `ServletContextResource`：用于访问 Web 应用上下文中的资源。
- `ByteArrayResource`：基于字节数组的资源，通常用于生成资源数据而不是从文件或 URL 读取。
- `InputStreamResource`：基于 `InputStream` 的资源，表示直接从流中读取的资源。

这些不同的实现类隐藏了资源的具体细节，你只需要根据需求选择合适的实现类来访问资源。

### 3. `Resource` 接口的常用方法

以下是 `Resource` 接口中常用的方法，所有实现类都必须实现这些方法：

- `InputStream getInputStream()`：用于获取资源内容的 `InputStream`，你可以通过流读取资源内容。
- `boolean exists()`：检查资源是否存在，返回 `true` 表示资源存在。
- `boolean isReadable()`：检查资源是否可读。
- `boolean isOpen()`：检查资源是否已经打开（主要用于 `InputStream` 类型的资源）。
- `URL getURL()`：获取资源的 `URL` 表示形式（如果适用）。
- `File getFile()`：将资源转换为文件对象（如果适用）。
- `long contentLength()`：获取资源内容的字节数。
- `String getFilename()`：获取资源的文件名（如果适用）。
- `String getDescription()`：返回资源的描述信息，通常是资源的路径或 URL。

## org.springframework.core.io.UrlResource;

`org.springframework.core.io.UrlResource` 是 Spring 框架中的一个类，它实现了 `org.springframework.core.io.Resource` 接口，用于表示基于 URL 的资源。`UrlResource` 允许你通过统一的方式来访问任何可以通过 URL 标识的资源，比如 HTTP、FTP 资源，也可以用于访问本地文件（`file://` 协议）。

### UrlResource的构造方法

#### 通过url对象

```
UrlResource resource =  new UrlResource(new URL("https://example.com/resource.txt"))
```

####  通过字符串url

```
UrlResource resource = new UrlResource("https://example.com/resource.txt")
```

#### 通过URI

```
UrlResource urlResource = new UrlResource(new URI("https://example.com/resource.txt"));
```

## org.springframework.http.ResponseEntity

`org.springframework.http.ResponseEntity` 是 Spring 框架中用于构建 HTTP 响应的一个泛型类。它不仅包含响应的主体（body），还包含 HTTP 状态码和响应头，允许开发者灵活地控制返回给客户端的完整 HTTP 响应信息。

### 1. `ResponseEntity` 的作用

`ResponseEntity` 通常用于 Spring MVC 或 Spring WebFlux 中的控制器方法返回类型，用来封装 HTTP 响应数据。与返回简单对象不同，`ResponseEntity` 允许你精确地设置：

- **HTTP 状态码**：例如 `200 OK`, `404 NOT FOUND`, `500 INTERNAL SERVER ERROR` 等。
- **响应头**：例如设置 `Content-Type`、`Location`、自定义的 HTTP 头等。
- **响应体**：即返回给客户端的数据。

### 2. **构造方法**

`ResponseEntity` 提供了多个构造方法和静态工厂方法，方便创建带有不同状态码、头部信息以及主体的响应。

- **直接构造**： 你可以使用 `ResponseEntity` 的构造函数直接创建一个实例：

```
ResponseEntity<String> response = new ResponseEntity<>("hello world",HttpStatus.OK);
```

**使用静态方法**： Spring 提供了一系列静态方法帮助构建 `ResponseEntity`，更直观简便：

- `ResponseEntity.ok()`：表示 200 OK 响应。
- `ResponseEntity.status()`：用于指定任意状态码。
- `ResponseEntity.badRequest()`：表示 400 Bad Request 响应。
- `ResponseEntity.notFound()`：表示 404 Not Found 响应。

### 3.常见用法

#### **返回带状态码的响应**

`ResponseEntity` 允许你设置 HTTP 状态码，确保客户端能收到正确的响应状态：

```
@GetMapping("/status") 
public ResponseEntity<String> getStatus() 
{ return new ResponseEntity<>("Success", HttpStatus.OK); }
```

这将返回一个 200 OK 响应，带有 `"Success"` 作为响应体。

#### 设置响应头

可以通过 `ResponseEntity` 来设置自定义 HTTP 头。`HttpHeaders` 类用于定义头部信息：

```
@GetMapping(/headers)
public ResponseEntity<String> getHeader(){
HttpHeaders headers = new HttpHeaders();
headers.add("Custom-Header","CustomValue");
return new ResonseEntity<"header set",headers,HttpStatus.OK>;
}
```

自定义了一个名为 `"Custom-Header"` 的响应头，返回给客户端。

#### **返回 JSON 或其他对象**

`ResponseEntity` 不仅支持返回简单的字符串或基本类型，还可以返回 JSON 或其他对象。Spring MVC 默认会根据内容协商机制将对象序列化为 JSON 格式：

```
@GetMapping("/user") 
public ResponseEntity<User> getUser() 
{ User user = new User("John", "Doe"); return ResponseEntity.ok(user); }
```



在这种情况下，`User` 对象会被自动序列化为 JSON 格式并返回给客户端，状态码为 200 OK。

#### **设置 HTTP 状态码**

有时你可能需要返回一个非 200 的状态码，例如返回 404 Not Found 或 500 Internal Server Error。你可以使用 `ResponseEntity.status()` 方法指定状态码：

```
j
```



这将返回一个 404 响应，并带有 `"Resource not found"` 作为响应体。

#### **重定向响应**

通过 `ResponseEntity`，你还可以返回一个重定向响应，比如 302 Found：

```
@GetMapping("/redirect")
public ResponseEntity<Void> redirect() {
    HttpHeaders headers = new HttpHeaders();
    headers.setLocation(URI.create("https://example.com"));
    return new ResponseEntity<>(headers, HttpStatus.FOUND);
}
```

此示例会返回一个 302 重定向响应，并将客户端重定向到 `https://example.com`。

#### 4. **常用静态方法**

Spring 提供了一些静态工厂方法来简化 `ResponseEntity` 的创建。

##### `ok()`

返回状态码 200 OK 的 `ResponseEntity`，并包含响应体：

```
return ResponseEntity.ok("Success");
```

##### `status()`

返回指定状态码的 `ResponseEntity`，可以指定响应体：

```
return ResponseEntity.status(HttpStatus.CREATED).body("Resource created");
```

##### `badRequest()`

返回状态码 400 Bad Request：

```
return ResponseEntity.badRequest().body("Invalid request");
```

##### `notFound()`

返回状态码 404 Not Found：

```
return ResponseEntity.notFound().build();
```

##### `noContent()`

返回状态码 204 No Content，表示没有返回内容：

```
return ResponseEntity.noContent().build()
```



### 5. **与** `@ResponseBody` 的区别

在 Spring MVC 中，`@ResponseBody` 也用于返回 HTTP 响应体。与 `ResponseEntity` 不同，`@ResponseBody` 只处理响应体的内容，而不涉及 HTTP 状态码和头部信息。`ResponseEntity` 则允许你同时控制状态码、头部和响应体的内容。

#### **简单示例：**`@ResponseBody`

```
@GetMapping("/simple") @ResponseBody public String simple() { return "Simple Response"; }
```

这将返回 200 OK 的响应，并包含 `"Simple Response"` 作为响应体。要控制更多的响应细节，比如状态码或头部信息，应该使用 `ResponseEntity`。

### 6. **应用场景**

- **返回自定义 HTTP 状态码**：比如创建资源后返回 `201 Created`，或者请求失败时返回 `400 Bad Request`。
- **设置 HTTP 响应头**：如文件下载时设置 `Content-Disposition`，或者自定义头部信息。
- **返回 JSON 数据**：可以与 `ResponseEntity` 一起使用，返回复杂的 JSON 对象。
- **重定向或自定义响应格式**：通过 `ResponseEntity` 返回 302 重定向或 204 无内容响应。

### 7. **总结**

- `ResponseEntity` 是 Spring 提供的一个强大的类，用于构建和返回 HTTP 响应，允许控制响应体、状态码、以及响应头。
- 通过 `ResponseEntity`，你可以精确地控制 HTTP 响应的各个方面，适用于需要返回复杂响应、控制状态码和自定义头部的场景。
- 静态工厂方法如 `ok()`、`status()` 等，简化了 `ResponseEntity` 的创建，使其更具可读性和易用性。

## java.nio.file.Files

`Files` 类是 Java NIO (New I/O) 的一部分，提供了许多静态方法，用于操作文件和目录，比如创建、读取、写入、删除文件，处理文件的复制、移动，以及设置文件的属性等。

常见的操作方法包括：

- `copy(Path source, Path target, CopyOption... options)`：用于复制文件或目录。
- `move(Path source, Path target, CopyOption... options)`：用于移动或重命名文件。
- `readAllBytes(Path path)`：读取文件的所有字节并返回一个字节数组。
- `write(Path path, byte[] bytes, OpenOption... options)`：将字节数组写入文件。
- `delete(Path path)`：删除指定的文件或目录

```
//在这个例子中，Files.copy 方法会将 source.txt 文件复制为 target.txt，并使用 StandardCopyOption.REPLACE_EXISTING 选项替换目标文件
//(若存在)
Path source = Paths.get("source.txt");
Path target = Paths.get("target.txt");
Files.copy(source, target, StandardCopyOption.REPLACE_EXISTING);
```

## java.nio.file.Path

`Path` 是 Java NIO 文件系统的核心接口，用于表示文件或目录的路径。它支持相对路径和绝对路径，路径可以是文件系统中的实际文件，也可以是虚拟路径。

`Path` 类通过 `Paths` 类中的静态方法进行创建，它提供了许多方法用于解析路径、获取文件名、文件父目录等。

常见的方法包括：

- `getFileName()`：获取路径的文件名。
- `getParent()`：获取路径的父目录。
- `toAbsolutePath()`：将路径转换为绝对路径
- `resolve(String other)` :将路径拼接到当前路径的末尾，若other时相对路径，则拼接，若是绝对路径，则直接返回other路径。

```
Path filePath = Paths.get("example.txt");
System.out.println(filePath.getFileName());  // 输出 example.txt
System.out.println(filePath.toAbsolutePath());  // 输出文件的绝对路径
```

## java.nio.file.Paths

`Paths` 是一个工具类，用于将字符串转换为 `Path` 对象。通过它的静态方法 `get()`，可以根据提供的路径字符串（如文件系统路径、URL 路径等）创建 `Path` 对象。

```
Path path = Paths.get("data/file.txt");
```

## java.nio.file.StandardCopyOption

`StandardCopyOption` 是 Java NIO 中的一个枚举，定义了标准的文件复制或移动的行为选项。它用于指定文件复制和移动的行为，比如是否替换目标文件、是否复制文件的属性等。

常用选项包括：

- `REPLACE_EXISTING`：如果目标文件存在，则替换它。
- `COPY_ATTRIBUTES`：复制文件的所有属性，如修改时间、权限等。
- `ATOMIC_MOVE`：执行一个原子移动操作，保证文件在移动过程中不会损坏。

```
Files.copy(sourcePath, targetPath, StandardCopyOption.REPLACE_EXISTING);
```

# 上传下载的具体实现

```
package com.example.demochapter02.Controller;

import org.springframework.core.io.Resource;
import org.springframework.core.io.UrlResource;
import org.springframework.http.HttpHeaders;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.multipart.MultipartFile;

import java.io.IOException;
import java.net.MalformedURLException;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.Paths;
import java.nio.file.StandardCopyOption;

@RestController
public class FileUploadController {
    // 文件上传保存的目标目录路径
    private final String UPLOAD_DIR = "D:/images/";  // 可修改为其他绝对路径
    //MultipartFile是Spring框架提供的一个用于处理文件上传的类
    @PostMapping("/upload")
    public ResponseEntity<String> handleFileUpload(@RequestParam("file") MultipartFile file) {
        //检查文件是否为空
        if (file.isEmpty()) {
            return ResponseEntity.status(HttpStatus.BAD_REQUEST).body("文件为空");
        }

        try {
            // 确保上传目录存在，如果不存在则创建
            Path uploadPath = Paths.get(UPLOAD_DIR);//获取目标目录的Path对象
            if (!Files.exists(uploadPath)) {
                Files.createDirectories(uploadPath);  // 创建文件夹
            }

            // 文件完整路径
            Path path = uploadPath.resolve(file.getOriginalFilename());
            // 复制文件，StandardCopyOption.REPLACE_EXISTING:如果存在那么就替换
            Files.copy(file.getInputStream(), path, StandardCopyOption.REPLACE_EXISTING);

            return ResponseEntity.ok("文件上传成功: " + file.getOriginalFilename());
        } catch (IOException e) {
            return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body("文件上传失败");
        }
    }
    @GetMapping("/download")
    public ResponseEntity<Resource> downloadFile(@RequestParam("filename") String filename){
        try{
            Path path=Paths.get(UPLOAD_DIR).resolve(filename).normalize();
            Resource resource= new UrlResource(path.toUri());
            if(resource.exists()){
                return ResponseEntity.ok()
                        .header(HttpHeaders.CONTENT_DISPOSITION, "attachment; filename=\"" + resource.getFilename() + "\"")
                        .body(resource);
            }else{
                return ResponseEntity.status(HttpStatus.NOT_FOUND).body(null);
            }
        } catch (IOException e) {
            return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body(null);
        }
    }
}
```

## 上传

Spring Boot 提供了对文件上传的原生支持，开发者可以使用 `MultipartFile` 来处理上传的文件。

需要注意的是，对于文件上传来说，提交表单时的数据的编码方式必须是multipart/form-data，

如enctype="multipart/form-data"， 否则无法正确传输文件。

前端以表单形式将文件发送到后端，后端获取文件后在指定的上传文件夹复制文件。

```
 @PostMapping("/upload")
    public ResponseEntity<String> handleFileUpload(@RequestParam("file") MultipartFile file) {
        //检查文件是否为空
        if (file.isEmpty()) {
            return ResponseEntity.status(HttpStatus.BAD_REQUEST).body("文件为空");
        }

        try {
            // 确保上传目录存在，如果不存在则创建
            Path uploadPath = Paths.get(UPLOAD_DIR);//获取目标目录的Path对象
            if (!Files.exists(uploadPath)) {
                Files.createDirectories(uploadPath);  // 创建文件夹
            }

            // 文件完整路径
            Path path = uploadPath.resolve(file.getOriginalFilename());
            // 复制文件，StandardCopyOption.REPLACE_EXISTING:如果存在那么就替换
            Files.copy(file.getInputStream(), path, StandardCopyOption.REPLACE_EXISTING);

            return ResponseEntity.ok("文件上传成功: " + file.getOriginalFilename());
        } catch (IOException e) {
            return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body("文件上传失败");
        }
    }
```

## 下载

前端向后端发送下载请求，后端根据文件名判断文件夹中是否有这个文件，有则正常响应，否则报异常。

```
@GetMapping("/download")
    public ResponseEntity<Resource> downloadFile(@RequestParam("filename") String filename){
        try{
            Path path=Paths.get(UPLOAD_DIR).resolve(filename).normalize();
            Resource resource= new UrlResource(path.toUri());
            if(resource.exists()){
                return ResponseEntity.ok()
                        .header(HttpHeaders.CONTENT_DISPOSITION, "attachment; filename=\"" + resource.getFilename() + "\"")
                        .body(resource);
            }else{
                return ResponseEntity.status(HttpStatus.NOT_FOUND).body(null);
            }
        } catch (IOException e) {
            return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body(null);
        }
    }
```