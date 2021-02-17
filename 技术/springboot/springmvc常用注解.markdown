## 一：@RequestMapping

@RequestMapping
RequestMapping是一个用来处理请求地址映射的注解（将请求映射到对应的控制器方法中），可用于类或方法上。用于类上，表示类中的所有响应请求的方法都是以该地址作为父路径。

**RequestMapping的属性**
**value：指定请求的实际url**
(1)普通的具体值。如前面的value="/book"。
(2)含某变量的一类值。

路径中的bookId可以当变量，@PathVariable注解即提取路径中的变量值。

@RequestMapping(value={"/get","/fetch"} )即 /get或/fetch都会映射到该方法上。

**method：指定请求的method类型， GET、POST、PUT、DELETE等；**
@RequestMapping(value="/get/{bookid}",method={RequestMethod.GET,RequestMethod.POST})

**params：指定request中必须包含某些参数值是，才让该方法处理。**
@RequestMapping(params="action=del")，请求参数包含“action=del”,如：http://localhost:8080/book?action=del

**headers：指定request中必须包含某些指定的header值，才能让该方法处理请求。**
@RequestMapping(value="/header/id", headers = "Accept=application/json")：表示请求的URL必须为“/header/id 且请求头中必须有“Accept =application/json”参数即可匹配。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```java
@Controller
@RequestMapping("/owners/{ownerId}")
public class RelativePathUriTemplateController {

@RequestMapping(value = "/pets", method = RequestMethod.GET, headers="Referer=http://www.ifeng.com/")
  public void findPet(@PathVariable String ownerId, @PathVariable String petId, Model model) {    
    // implementation omitted
  }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 仅处理request的header中包含了指定“Refer”请求头和对应值为“`http://www.ifeng.com/`”的请求。

**consumes：指定处理请求的提交内容类型（Content-Type），例如application/json, text/html。**

```java
@Controller
@RequestMapping(value = "/pets", method = RequestMethod.POST, consumes="application/json")
public void addPet(@RequestBody Pet pet, Model model) {    
    // implementation omitted
}
```

 方法仅处理request Content-Type为“application/json”类型的请求。
**produces: 指定返回的内容类型，仅当request请求头中的(Accept)类型中包含该指定类型才返回。** 

```java
@Controller
@RequestMapping(value = "/pets/{petId}", method = RequestMethod.GET, produces="application/json")
@ResponseBody
public Pet getPet(@PathVariable String petId, Model model) {    
    // implementation omitted
}
```

 方法仅处理request请求中Accept头中包含了"application/json"的请求，同时暗示了返回的内容类型为application/json;



**@RequestBody 将HTTP请求正文转换为适合的HttpMessageConverter对象。
@ResponseBody 将内容或对象作为 HTTP 响应正文返回，并调用适合HttpMessageConverter的Adapter转换对象，写入输出流。**

**@RequestBody**
作用：
 i) 该注解用于读取Request请求的body部分数据，使用系统默认配置的HttpMessageConverter进行解析，然后把相应的数据绑定到要返回的对象上；
 ii) 再把HttpMessageConverter返回的对象数据绑定到 controller中方法的参数上。
使用时机：
A) GET、POST方式提时， 根据request header Content-Type的值来判断:
application/x-www-form-urlencoded， 可选（即非必须，因为这种情况的数据@RequestParam, @ModelAttribute也可以处理，当然@RequestBody也能处理）；
multipart/form-data, 不能处理（即使用@RequestBody不能处理这种格式的数据）；
其他格式， 必须（其他格式包括application/json, application/xml等。这些格式的数据，必须使用@RequestBody来处理）；

B) PUT方式提交时， 根据request header Content-Type的值来判断:
application/x-www-form-urlencoded， 必须；
multipart/form-data, 不能处理；
其他格式， 必须；
说明：request的body部分的数据编码格式由header部分的Content-Type指定；

**@ResponseBody**
作用：
 该注解用于将Controller的方法返回的对象，通过适当的HttpMessageConverter转换为指定格式后，写入到Response对象的body数据区。
使用时机：
 返回的数据不是html标签的页面，而是其他某种格式的数据时（如json、xml等）使用；

## 使用postman传递数组

以springboot两个接收参数的注解为例：**@RequestBody和@RequestParam**

***一、先简单的写一下springboot的注解@RequestBody和@RequestParam在后台是如何接收数组***

1. **@RequestParam接收数组**

请注意@RequestParam括号里的名称一定得带中括号[ ]，后边定义的参数名则不需要

```java
    @GetMapping
    public String getAll(@RequestParam("ids[]") List<Integer> ids) {
        System.out.println(ids);
        return String.valueOf(ids);
    }
```

![](D:\MyWork\MarkDownPicture\postman\传入数组_requestparam.png)

***二、进入正题，使用postman来传递@RequestBody和@RequestParam注解的数组参数**

1. **用postman传递@RequestBody注解的数组**

看图操作便是（**注意微操，格式需选择JSON格式**）

```java
    @PostMapping
    public String getAll1(@RequestBody String[] ids) {
        System.out.println(ids);
        Stream.of(ids).forEach(System.out :: println);
        return String.valueOf(ids);
    }
```



![](D:\MyWork\MarkDownPicture\postman\传入数组_requestbody.png)

1. **用postman传递@RequestParam注解的数组**

与@RequestBody不同，@RequestParam传递的数组中有多少个值，便排排下来写便是
（**注意微操，参数名需为key的名称为@RequestParam括号里的名称，而不是定义的数组名**）



