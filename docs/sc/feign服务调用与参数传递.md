### 服务调用
!>默认拥有已经注册到服务中心的service-hello（服务提供者）、client-hello（服务消费者）两个服务了，client-hello的端口号为8762。client-hello服务要调用service-hello服务。

#### 服务器中Eureka注册中心的地址为：192.168.2.137:8761

##### service-hello服务
在controller中创建restful接口：

```
@GetMapping("/service-hello/{str}")  
public String serviceHello(@PathVariable String str){
    return str; //返回接收到的字符串
}
```

##### client-hello服务
1.引入feign依赖：

```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
    <version>2.1.0.RELEASE</version>
</dependency>
```
2.在启动类上添加注解：

```
@EnableFeignClients
```

3.创建一个interface：

```
@FeignClient(name = "service-hello")  //name值为被调用服务注册到服务中心的名字
public interface ClientService {
    @GetMapping("/service-hello/{str}") //被调用服务的Restful接口
    String echo(@PathVariable(value = "str") String str);
}
```
4.在controller中调用：

```
@GetMapping("/client-hello/{str}")
public String clientHello(@PathVariable(value = "str") String str){
    return clientService.echo(str);
}
```

在浏览器访问client-hello的接口，如：http://localhost:8762/client-hello/helloWorld ,可以在浏览器中看见打印出了 helloWorld 。说明我们通过client-hello服务调用了service-hello服务的接口，并且还传递了一个String类型的参数。

!> 目前spring cloud集成了dubbo但是不稳定，还在测试阶段中。

### 参数传递
?>在服务调用时，通常会传递一些参数。下文使用service-hello和client-hello两个服务针对传递对象和文件这两种情况分别举了例子。
#### 传递对象
1.接收端需使用POST请求，即使发送端使用的是GET请求，Feign依然会发送POST请求。

2.接收端需要使用@RequestBody注解来获取发送端传递来的对象。

**例子**：  
> 说明：    
1.本例中是client-hello向service-hello传递对象。  
2.两服务都已有User类并重写了toString方法。

**service-hello服务**		
在controller中，创建接口：

```
@PostMapping("/service-hello") //如果这里使用GET请求，client-hello服务会报异常
public String serviceHello(@RequestBody User user){
    return user.toString(); //返回接收到的user对象的内容
}
```

**client-hello服务**		
1.在ClientService中，使用POST请求向service-hello传递对象user：

```
@FeignClient(name = "service-hello")
public interface ClientService {
    @PostMapping("/service-hello")
    String echo(User user);
}
```
2.在controller中调用echo：

```
@GetMapping("/client-hello")
public String clientHello(){
    User user = new User();
	user.setName("小明");
	user.setGender("男");
	user.setAge(10);
	user.setAddress("tfswx");
    return clientService.echo(user);
}
```

访问 http://localhost:8762/client-hello :

![](../_img/feign.png)		


#### 传递文件
在Spring Cloud 的Feign组件中并不支持文件的传输，但是我们可以通过使用Feign的扩展包实现这个功能。

本例是client-hello向service-hello传递文件。

**client-hello服务**：

1.添加扩展包依赖：

```
<dependency>
    <groupId>io.github.openfeign.form</groupId>
    <artifactId>feign-form</artifactId>
    <version>3.0.3</version>
</dependency>
<dependency>
    <groupId>io.github.openfeign.form</groupId>
    <artifactId>feign-form-spring</artifactId>
    <version>3.0.3</version>
</dependency>
<dependency>
    <groupId>commons-fileupload</groupId>
    <artifactId>commons-fileupload</artifactId>
    <version>1.3.3</version>
</dependency>
```

2.编写方法，用以调用service-hello服务。

```
@FeignClient(name = "service-hello")
public interface ClientService {

    @PostMapping(value = "/receiveFile",consumes = MediaType.MULTIPART_FORM_DATA_VALUE)
    byte[] fileUpload(MultipartFile file);
}
```

3.在controller中调用fileUpload方法

```
@GetMapping("/sendFile")
public byte[] sendFile(){
    FileItem fileItem = createFileItem(); 
    MultipartFile multipartFile = new CommonsMultipartFile(fileItem);
    return clientService.fileUpload(multipartFile);
}
```

```
private FileItem createFileItem(){
    File file = new File("C:/Users/Administrator/Desktop/upload.txt");//要传递的文件
    DiskFileItem fileItem = (DiskFileItem) new DiskFileItemFactory().createItem("file",
            MediaType.TEXT_PLAIN_VALUE, true, file.getName());

    try (InputStream input = new FileInputStream(file); OutputStream os = fileItem.getOutputStream()) {
        IOUtils.copy(input, os);
    } catch (Exception e) {
        throw new IllegalArgumentException("Invalid file: " + e, e);
    }
    return fileItem;
}
```

**service-hello服务**：     
编写接口：

```
@PostMapping("/receiveFile")
public byte[] handleFileUpload(@RequestPart("file") MultipartFile file){
    byte[] result = null;
    try {
        //返回文件的内容作为一个字节数组
        result = file.getBytes(); 
        //使用transferTo方法，将接收到的文件转移到给定的目标文件
        file.transferTo(new File("D:/"+file.getOriginalFilename()));
    } catch (IOException e) {
        e.printStackTrace();
    }
    return result;
}
```

访问 http://localhost:8762/sendFile ， 我们可以在浏览器中看见文件中的内容，也能在本地看见文件了。

