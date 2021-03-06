## Java 消息摘要算法

消息摘要算法是单向不可逆的，无法通过加密后的散列值反推原始值，相同的内容用同样的摘要算法获得的散列值是一样的，所以常用于验证数据的完整性。

该算法主要分为三大类：MD（Message Digest，消息摘要算法）、SHA（Secure HashAlgorithm，安全散列算法）和MAC（Message Authentication Code，消息认证码算法）。MD系列算法包括MD2、MD4和MD5共3种算法；SHA算法主要包括其代表算法SHA-1和SHA-1算法的变种SHA-2系列算法（包含SHA-224、SHA-256、SHA-384和SHA-512）；MAC算法综合了上述两种算法，主要包括HmacMD5、HmacSHA1、HmacSHA256、HmacSHA384和HmacSHA512算法。这节主要记录下JDK8对这些算法的支持情况。



消息摘要算法的结果我们一般将其转换为16进制字符串，方便阅读传输。

## MD系列算法

MD5算法是MD系列算法的代表，由MD2、MD4等算法演变而来。无论采用哪种MD算法，结果都是32字节的16进制字符串。JDK8只支持MD2和MD5两种MD算法。

新建一个maven项目，引入common-codec依赖（包含一些加解密工具类，用于增强JDK或者简化JDK相关加解密API）：

```xml
<dependency>
    <groupId>commons-codec</groupId>
    <artifactId>commons-codec</artifactId>
    <version>1.14</version>
</dependency>
```

JDK8中，MD系列算法的实现是通过MessageDigest类来完成的，下面演示下使用JDK8原生API实现MD2和MD5加密（算法名称不区分大小写）。

MD2：

```java
import org.apache.commons.codec.binary.Hex;
import org.junit.Test;

import java.security.MessageDigest;

public class Demo {

    @Test
    public void test() throws Exception {
        String value = "hello";
        String algorithm = "md2";
        // 获取MessageDigest实例
        MessageDigest messageDigest = MessageDigest.getInstance(algorithm);
        // 调用digest方法生成数字摘要
        byte[] digest = messageDigest.digest(value.getBytes());
        // 结果转换为16进制字符串（借助common-codec Hex类）
        String md2Hex = Hex.encodeHexString(digest);
        System.out.println(md2Hex);
        System.out.println(md2Hex.length());
    }
}
```



运行结果：

```
a9046c73e00331af68917d3804f70655
32
```



MD5（只需要将算法改为md5即可）：

```java
import org.apache.commons.codec.binary.Hex;
import org.junit.Test;

import java.security.MessageDigest;

public class Demo {

    @Test
    public void test() throws Exception {
        String value = "hello";
        String algorithm = "md5";
        // 获取MessageDigest实例
        MessageDigest messageDigest = MessageDigest.getInstance(algorithm);
        // 调用digest方法生成数字摘要
        byte[] digest = messageDigest.digest(value.getBytes());
        // 结果转换为16进制字符串（借助common-codec Hex类）
        String md5Hex = Hex.encodeHexString(digest);
        System.out.println(md5Hex);
        System.out.println(md5Hex.length());
    }
}
```



运行结果：

```
5d41402abc4b2a76b9719d911017c592
32
```



common-codec的DigestUtils也提供了MD2和MD5算法相关方法：

```java
import org.apache.commons.codec.digest.DigestUtils;
import org.junit.Test;

public class Demo {

    @Test
    public void test() {
        String value = "hello";
        String md5Hex = DigestUtils.md5Hex(value.getBytes());
        System.out.println(md5Hex);

        String md2Hex = DigestUtils.md2Hex(value.getBytes());
        System.out.println(md2Hex);
    }
}
```



common-codec并没有自己实现相关算法，而是对JDK原生API进行封装，使用起来更方便。

## SHA系列算法

SHA算法是基于MD4算法实现的，作为MD算法的继任者，成为了新一代的消息摘要算法的代表。SHA与MD算法不同之处主要在于摘要长度，SHA算法的摘要更长，安全性更高。

SHA算法家族目前共有SHA-1、SHA-224、SHA-256、SHA-384和SHA-512五种算法，通常将后四种算法并称为SHA-2算法。

JDK8支持SHA-1、SHA-256、SHA-384、SHA-224和SHA-512五种算法，其中SHA的写法等价于SHA-1。JDK8中，SHA系列算法的实现也是通过MessageDigest类来完成的：

```java
import org.apache.commons.codec.binary.Hex;
import org.junit.Test;

import java.security.MessageDigest;
import java.security.NoSuchAlgorithmException;

public class Demo {

    @Test
    public void test() throws NoSuchAlgorithmException {
        String value = "hello";
        String algorithm = "SHA-1";
        MessageDigest messageDigest = MessageDigest.getInstance(algorithm);
        byte[] digest = messageDigest.digest(value.getBytes());
        System.out.println(Hex.encodeHexString(digest));
    }
}
```

运行结果：

```
aaf4c61ddcc5e8a2dabede0f3b482cd9aea9434d
```



剩下的几种算法可以自己尝试。

common-codec同样也提供了SHA相关算法的API：

```java
import org.apache.commons.codec.digest.DigestUtils;
import org.junit.Test;

public class Demo {

    @Test
    public void test() {
        String value = "hello";

        String sha1Hex = DigestUtils.sha1Hex(value);
        System.out.println(sha1Hex);

        String sha256Hex = DigestUtils.sha256Hex(value);
        System.out.println(sha256Hex);
        
        String sha384Hex = DigestUtils.sha384Hex(value);
        System.out.println(sha384Hex);

        String sha512Hex = DigestUtils.sha512Hex(value);
        System.out.println(sha512Hex);
    }
}
```



## MAC系列算法

MAC算法结合了MD和SHA算法的优势，并加入秘钥的支持，是一种更为安全的消息摘要算法。因为MAC算法融合了秘钥散列函数（keyed-Hash），通常我们也把MAC称为HMAC（keyed-Hash Message Authentication Code）。

MAC算法主要集合了MD和SHA两大系列消息摘要算法。MD系列算法有HmacMD2、HmacMD4和HmacMD5三种算法；SHA系列算法有HmacSHA1、HmacSHA224、HmacSHA256、HmacSHA384和HmacSHA512五种算法。

JDK8支持了HmacMD5、HmacSHA1、HmacSHA224、HmacSHA256、HmacSHA384和HmacSHA512这六种MAC算法，通过Mac类实现。

```java
import org.apache.commons.codec.binary.Base64;
import org.apache.commons.codec.binary.Hex;
import org.junit.Test;

import javax.crypto.KeyGenerator;
import javax.crypto.Mac;
import javax.crypto.SecretKey;
import javax.crypto.spec.SecretKeySpec;

public class Demo {

    @Test
    public void test() throws Exception {
        String value = "hello";
        // JDK支持 HmacMD5、HmacSHA1、HmacSHA224、HmacSHA256、HmacSHA384和HmacSHA512六种算法
        String algorithm = "HmacMD5";
        // 初始化KeyGenerator
        KeyGenerator keyGenerator = KeyGenerator.getInstance(algorithm);
        // 构建秘钥
        SecretKey secretKey = keyGenerator.generateKey();
        // 获得秘钥
        byte[] key = secretKey.getEncoded();
        // 还原秘钥
        SecretKeySpec secretKeySpec = new SecretKeySpec(key, algorithm);
        // 打印下秘钥
        System.out.println(Base64.encodeBase64String(secretKeySpec.getEncoded()));
        // 实例化Mac
        Mac mac = Mac.getInstance(algorithm);
        // 初始化Mac
        mac.init(secretKeySpec);
        // 获取消息摘要
        byte[] bytes = mac.doFinal(value.getBytes());
        // 转换为16进制
        System.out.println(Hex.encodeHexString(bytes));
    }
}
```

输出结果：

```
PCg3+Q7i/C0ahZ74Vo3Nl/2wBvHnsdycoSmoAXzuxSwc5DVc1rWyHKHdt1XzlanT5GdJiKkcKhCwXGm7iN+udA==
6c40b8d59b9d0f818e94b9028b789892
```



## 实际应用

在Tomcat下载页面：https://tomcat.apache.org/download-70.cgi中，我们可以查看相关文件的摘要：

![QQ20200820-102253@2x](https://mrbird.cc/img/QQ20200820-102253@2x.png)

我们将32-bit Windows zip这个文件下载下来，计算出这个文件的sha512值：

```java
import org.apache.commons.codec.binary.Hex;
import org.junit.Test;

import java.io.ByteArrayOutputStream;
import java.io.FileInputStream;
import java.security.MessageDigest;

public class Demo {

    @Test
    public void test() throws Exception {
        String algorithm = "SHA-512";
        MessageDigest messageDigest = MessageDigest.getInstance(algorithm);

        String filePath = "files/apache-tomcat-7.0.105-windows-x86.zip";
        FileInputStream fis = new FileInputStream(filePath);
        int len;
        byte[] buffer = new byte[1024];
        ByteArrayOutputStream stream = new ByteArrayOutputStream();
        while ((len = fis.read(buffer)) != -1) {
            stream.write(buffer, 0, len);
        }

        byte[] digest = messageDigest.digest(stream.toByteArray());
        System.out.println(Hex.encodeHexString(digest));
    }
}
```

结果：

```
b7a3b0629dad0d9684bc57a5d18251e38bafa172fcffeac06f7e3c40884f2afc099e7c0143a0471639887b8294c8135c35d1f1ac24f1637dee3c
```