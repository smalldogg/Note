## 一.REST架构的主要原则

- 对网络上所有的资源都有一个资源标志符。
- 对资源的操作不会改变标识符。
- 同一资源有多种表现形式（xml、json）
- 所有操作都是无状态的（Stateless）

符合上述REST原则的架构方式称为RESTful

## 二.资源事先定义出来

首先需要定义出来资源的提供方,是一种面向资源的

像对于到数据库的一张表

针对于单表修改不再重复 crud

SpringDataRest 可以帮我们生成出来重复的代码

### 1.RESTful资源操作

| **http方法** | **资源操作** | **幂等** | **安全** |
| ------------ | ------------ | -------- | -------- |
| GET          | SELECT       | 是       | 是       |
| POST         | INSERT       | 否       | 否       |
| PUT          | UPDATE       | 是       | 否       |
| DELETE       | DELETE       | 是       | 否       |

幂等性：对同一REST接口的多次访问，得到的资源状态是相同的。

安全性：对该REST接口访问，不会使服务器端资源的状态发生改变。

### 2.接口示例：

#### 2.1.传统URL请求格式：

*http://127.0.0.1/user/query/1* GET 根据用户id查询用户数据

http://127.0.0.1/user/save POST 新增用户

http://127.0.0.1/user/update POST 修改用户信息

http://127.0.0.1/user/delete GET/POST 删除用户信息

#### 2.2.RESTful请求格式：

*http://127.0.0.1/user/1* GET 根据用户id查询用户数据

*http://127.0.0.1/user* POST 新增用户

*http://127.0.0.1/user* PUT 修改用户信息

*http://127.0.0.1/user* DELETE 删除用户信息

### 3.响应设计

> 原则：数据接收到即可使用，无需拆箱。

在一次请求中，content body仅仅用于传输数据。Header中存放描述请求或请求的元数据，例如 X-Result-Fields。

### 4.http响应状态码

根据http响应码，判断请求状态，进而做出提醒。
![这里写图片描述](https://img-blog.csdn.net/20180721093338663?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3g1NDEyMTExOTA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)****

## 三.实例代码

