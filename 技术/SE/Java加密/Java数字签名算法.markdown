## Java 数字签名算法

数字签名算法可以看成是带秘钥的消息摘要算法，用于验证数据完整性、认证数据来源，并起到抗否认的作用。遵循私钥加签，公钥验签的规则，数字签名算法是非对称加密算法和消息摘要算法的结合体。数字签名算法主要包括RSA和DSA。这节记录下这两种算法在JDK8下的实现。

数字签名加签验签流程分为以下几步：

1. A在本地构建秘钥对，并将公钥发布给B；
2. A使用私钥对数据进行签名；
3. A发送签名和数据给B；
4. B使用公钥对签名和数据进行验签。

## RSA

RSA数字签名算法主要分为MD系列和SHA系列两大类。MD系列主要包括MD2withRSA和MD5withRSA共2种数字签名算法；SHA系列主要包括SHA1withRSA、SHA224withRSA、SHA256withRSA、SHA384withRSA和SHA512withRSA共5种数字签名算法。

代码示例

```java
import org.junit.Test;

import java.security.*;
import java.util.Base64;

public class RsaSignatureDemo {

    @Test
    public void test() throws Exception {
        String value = "mrbird's blog";
        // 非对称加密算法
        String algorithm = "RSA";
        // 签名算法，RSA+SHA
        String signAlgorithm = "SHA256withRSA";

        // ----- 公私钥生成 --------
        // 实例化秘钥对生成器
        KeyPairGenerator keyPairGenerator = KeyPairGenerator.getInstance(algorithm);
        // 初始化，秘钥长度512~16384位，64倍数
        keyPairGenerator.initialize(512);
        // 生成秘钥对
        KeyPair keyPair = keyPairGenerator.generateKeyPair();
        // 公钥
        PublicKey publicKey = keyPair.getPublic();
        System.out.println("RSA公钥: " + Base64.getEncoder().encodeToString(publicKey.getEncoded()));
        // 私钥
        PrivateKey privateKey = keyPair.getPrivate();
        System.out.println("RSA私钥: " + Base64.getEncoder().encodeToString(privateKey.getEncoded()));

        // ----- 私钥加签 ---------
        // 获取签名对象
        Signature signature = Signature.getInstance(signAlgorithm);
        signature.initSign(privateKey);
        signature.update(value.getBytes());
        byte[] sign = signature.sign();
        System.out.println("签名值: " + Base64.getEncoder().encodeToString(sign));

        // ----- 公钥验签 ---------
        signature.initVerify(publicKey);
        signature.update(value.getBytes());
        System.out.println("验签结果: " + signature.verify(sign));
        
    }
}
```

秘钥对生成过程和上篇RSA介绍的无异，主要关注加签和验签操作即可，程序输出如下：

```
RSA公钥: MFwwDQYJKoZIhvcNAQEBBQADSwAwSAJBAKTTlw+zyhGzmTmhT5w9vEP1ejOcVfM2rHbz8jUae7InAh42R9ZaYUk1c3q0uqmTv8xKOnszU/vrdV52zoFM+OMCAwEAAQ==RSA私钥: MIIBUwIBADANBgkqhkiG9w0BAQEFAASCAT0wggE5AgEAAkEApNOXD7PKEbOZOaFPnD28Q/V6M5xV8zasdvPyNRp7sicCHjZH1lphSTVzerS6qZO/zEo6ezNT++t1XnbOgUz44wIDAQABAkBsZQXz+p2J1J2Qq8fqDSNxYc8Sf956SttSgw0m5Rqxxiw10cgHt67uocu3qK6UeMuJuaOiN3YT48kvFp6Joc75AiEA4L7R1zDWcOdWf2BE/k3yxJ4Uv0vbIZ9vWLuJGBR3xK0CIQC7v5f2fcedBWbJ/kR7CvbFE91ivM55dvWZMe/JrjXVzwIgFUn+FqRJq+g+CVLVNkGr/XP8AyLsXwL7SSx6kA1gSwECIHysAn4VEftr/dC+Pr0yD6HYyhbp53XzD6214lQbkfYzAiB1b2wNi0Y3N+D/OIrGUHlwgA0vkX82NP3V8qMDmRbCTQ==签名值: HVN5WkhND0hy/xY43h8r3+AVt6oxMSv1Ug/Y+bv1tGxw4ePQtIgzFwK0lQQbhIlwts2d2STwQBews4dXCfEMmA==验签结果: true
```

