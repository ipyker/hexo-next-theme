layout: post
title: Elastic Stack之X-Pack7.0破解
author: Pyker
categories: elastic
img: /images/bar/xpack.png
tags:
  - x-pack
top: false
cover: false
date: 2019-03-13 16:30:00
---
>**说明： elastic官方在elastic stack 6.4.2版本后就在elasticsearch中内置了X-Pack工具，因此下文破解X-Pack7.0.1的版本也是对应elastic stack7.0.1的版本。而X-Pack内置在elasticsearch包中，以下所有操作都是针对elasticsearch7.0.1包中进行的。**

## X-Pack是什么
X-pack是elasticsearch的一个扩展包，将安全，警告，监视，图形和报告功能捆绑在一个易于安装的软件包中，虽然x-pack被设计为一个无缝的工作，但是你可以轻松的启用或者关闭一些功能。
>目前6.2及以下版本只能使用免费版，然而免费版的功能相当少。X-pack 的破解基本思路是先安装正常版本，之后替换破解的jar包来实现，目前只能破解到白金版，但已经够用了。更多版本功能介绍请查看[官方版本订阅文档](https://www.elastic.co/cn/subscriptions)

## 下载elasticsearch
上面提到X-Pack自6.4.2版本后已经内置到elasticsearch中，因此我们需要下载elasticsearch7.0.1最新版。
```bash
# 下载elasticsearch.tar.gz
$ mkdir /elk && cd /elk
$ wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.0.1-linux-x86_64.tar.gz

# 解压elasticsearch.tar.gz
$ tar zxvf elasticsearch-7.0.1-linux-x86_64.tar.gz
```
下载完成并且解压后，我们可以查看自带x-pack的模版。
```bash
# 查看x-pack相关的模块

$ cd elasticsearch-7.0.1
$ ls modules/ | grep x-pack
x-pack-ccr
x-pack-core
x-pack-deprecation
x-pack-graph
x-pack-ilm
x-pack-logstash
x-pack-ml
x-pack-monitoring
x-pack-rollup
x-pack-security
x-pack-sql
x-pack-watcher
```
我们需要破解的`x-pack-core.7.0.1.jar`也就位于x-pack-core目录下
```bash
$ ls /elk/elasticsearch-7.0.1/modules/x-pack-core | grep x-pack
x-pack-core-7.0.1.jar 
```
## 下载反编译工具Luyten
破解X-Pack-core-7.0.1.jar需要反编译工具Luyten，我们可以前往[下载地址](https://github.com/deathmarine/Luyten/releases)下载Luyten工具。
我们这里下载Luyten.exe windows版本，下载下来后打开，并将`x-pack-core.7.0.1.jar`文件拖进去，即可展开jar包的源代码了。

## 修改X-Pack源码文件
在Luyten工具中我们需要把2个文件提取出来进行修改。`org.elasticsearch.license.LicenseVerifier`和`org.elasticsearch.xpack.core.XPackBuild`。
### 修改LicenseVerifier.java
`LicenseVerifier`中有两个静态方法，这就是验证授权文件是否有效的方法，我们把它修改为全部返回true.
```java
/*如下代码为修改完后的代码,我们这里使用注释将不需要的代码注释掉*/

package org.elasticsearch.license;

import java.nio.*;
import org.elasticsearch.common.bytes.*;
import java.security.*;
import java.util.*;
import org.elasticsearch.common.xcontent.*;
import org.apache.lucene.util.*;
import org.elasticsearch.core.internal.io.*;
import java.io.*;

public class LicenseVerifier
{
    public static boolean verifyLicense(final License license, final byte[] publicKeyData) {
 /*       
        byte[] signedContent = null;
        byte[] publicKeyFingerprint = null;
        try {
            final byte[] signatureBytes = Base64.getDecoder().decode(license.signature());
            final ByteBuffer byteBuffer = ByteBuffer.wrap(signatureBytes);
            final int version = byteBuffer.getInt();
            final int magicLen = byteBuffer.getInt();
            final byte[] magic = new byte[magicLen];
            byteBuffer.get(magic);
            final int hashLen = byteBuffer.getInt();
            publicKeyFingerprint = new byte[hashLen];
            byteBuffer.get(publicKeyFingerprint);
            final int signedContentLen = byteBuffer.getInt();
            signedContent = new byte[signedContentLen];
            byteBuffer.get(signedContent);
            final XContentBuilder contentBuilder = XContentFactory.contentBuilder(XContentType.JSON);
            license.toXContent(contentBuilder, (ToXContent.Params)new ToXContent.MapParams((Map)Collections.singletonMap("license_spec_view", "true")));
            final Signature rsa = Signature.getInstance("SHA512withRSA");
            rsa.initVerify(CryptUtils.readPublicKey(publicKeyData));
            final BytesRefIterator iterator = BytesReference.bytes(contentBuilder).iterator();
            BytesRef ref;
            while ((ref = iterator.next()) != null) {
                rsa.update(ref.bytes, ref.offset, ref.length);
            }
            return rsa.verify(signedContent);
        }
        catch (IOException ex) {}
        catch (NoSuchAlgorithmException ex2) {}
        catch (SignatureException ex3) {}
        catch (InvalidKeyException e) {
            throw new IllegalStateException(e);
        }
        finally {
            if (signedContent != null) {
                Arrays.fill(signedContent, (byte)0);
            }
        }
*/
        return true;
    }
    
    public static boolean verifyLicense(final License license) {
        /*
        byte[] publicKeyBytes;
        try {
            final InputStream is = LicenseVerifier.class.getResourceAsStream("/public.key");
            try {
                final ByteArrayOutputStream out = new ByteArrayOutputStream();
                Streams.copy(is, (OutputStream)out);
                publicKeyBytes = out.toByteArray();
                if (is != null) {
                    is.close();
                }
            }
            catch (Throwable t) {
                if (is != null) {
                    try {
                        is.close();
                    }
                    catch (Throwable t2) {
                        t.addSuppressed(t2);
                    }
                }
                throw t;
            }
        }
        catch (IOException ex) {
            throw new IllegalStateException(ex);
        }
        //return verifyLicense(license, publicKeyBytes);
        */
        return true;
    }
}

```
### 修改XPackBuild.java
`XPackBuild`中最后一个静态代码块中 try的部分全部删除，这部分会验证jar包是否被修改.
```java
/*如下代码为修改完后的代码,我们这里使用注释将不需要的代码注释掉*/

package org.elasticsearch.xpack.core;

import org.elasticsearch.common.io.*;
import java.net.*;
import org.elasticsearch.common.*;
import java.nio.file.*;
import java.io.*;
import java.util.jar.*;

public class XPackBuild
{
    public static final XPackBuild CURRENT;
    private String shortHash;
    private String date;
    
    @SuppressForbidden(reason = "looks up path of xpack.jar directly")
    static Path getElasticsearchCodebase() {
        final URL url = XPackBuild.class.getProtectionDomain().getCodeSource().getLocation();
        try {
            return PathUtils.get(url.toURI());
        }
        catch (URISyntaxException bogus) {
            throw new RuntimeException(bogus);
        }
    }
    
    XPackBuild(final String shortHash, final String date) {
        this.shortHash = shortHash;
        this.date = date;
    }
    
    public String shortHash() {
        return this.shortHash;
    }
    
    public String date() {
        return this.date;
    }
    
    static {
        final Path path = getElasticsearchCodebase();
        String shortHash = null;
        String date = null;
        Label_0109: {
/*            if (path.toString().endsWith(".jar")) {
                try {
                    final JarInputStream jar = new JarInputStream(Files.newInputStream(path, new OpenOption[0]));
                    try {
                        final Manifest manifest = jar.getManifest();
                        shortHash = manifest.getMainAttributes().getValue("Change");
                        date = manifest.getMainAttributes().getValue("Build-Date");
                        jar.close();
                    }
                    catch (Throwable t) {
                        try {
                            jar.close();
                        }
                        catch (Throwable t2) {
                            t.addSuppressed(t2);
                        }
                        throw t;
                    }
                    break Label_0109;
                }
                catch (IOException e) {
                    throw new RuntimeException(e);
                }
            }
*/
            shortHash = "Unknown";
            date = "Unknown";
        }
        CURRENT = new XPackBuild(shortHash, date);
    }
}


```
## 生成.class文件
上述`LicenseVerifier.java`和`XPackBuild.java`两个文件在本地电脑windows修改完成后，我们需要将其复制到elasticsearch服务器上并编译成class文件，然后打包到x-pack-core-7.0.1.jar中。我们这里将这2个文件放到了/root目录下。
```bash
# 编译LicenseVerifier.java
$ javac -cp "/elk/elasticsearch-7.0.1/lib/elasticsearch-7.0.1.jar:/elk/elasticsearch-7.0.1/lib/lucene-core-8.0.0.jar:/elk/elasticsearch-7.0.1/modules/x-pack-core/x-pack-core-7.0.1.jar:/elk/elasticsearch-7.0.1/modules/x-pack-core/netty-common-4.1.32.Final.jar:/elk/elasticsearch-7.0.1/lib/elasticsearch-core-7.0.1.jar" /root/LicenseVerifier.java

# 编译XPackBuild.java
$ javac -cp "/elk/elasticsearch-7.0.1/lib/elasticsearch-7.0.1.jar:/elk/elasticsearch-7.0.1/lib/lucene-core-8.0.0.jar:/elk/elasticsearch-7.0.1/modules/x-pack-core/x-pack-core-7.0.1.jar:/elk/elasticsearch-7.0.1/modules/x-pack-core/netty-common-4.1.32.Final.jar:/elk/elasticsearch-7.0.1/lib/elasticsearch-core-7.0.1.jar" /root/XPackBuild.java

# 查看编译后的文件
$ ls /root | grep .class
LicenseVerifier.class
XPackBuild.class
```

## 替换LicenseVerifier.class和XPackBuild.class
我们把`/elk/elasticsearch-7.0.1/modules/x-pack-core`目录下的`x-pack-core-7.0.1.jar`提取出来，放到一个临时的`/elk/x-pack`目录中。
```bash
$ cp /elk/elasticsearch-7.0.1/modules/x-pack-core/x-pack-core-7.0.1.jar /elk/x-pack
$ cd /elk/x-pack
# 解压x-pack-core-7.0.1.jar
$ jar -xvf x-pack-core-7.0.1.jar

# 替换.class文件
$ cp /root/XPackBuild.class /elk/x-pack/org/elasticsearch/xpack/core/
$ cp /root/LicenseVerifier.class /elk/x-pack/org/elasticsearch/license/
```
## 打包新x-pack-core-7.0.1.jar文件
```bash
$ cd /elk/x-pack
$ rm -rf x-pack-core-7.0.1.jar   # 删除临时拷贝过来的源文件
$ jar cvf x-pack-core-7.0.1.jar .
```
至此在/elk/x-pack目录下会新生成一个`x-pack-core-7.0.1.jar`文件。也就是破解后的文件。

## 替换x-pack-core-7.0.1.jar文件
我们将新生成的x-pack-core-7.0.1.jar文件文件替换掉源x-pack-core-7.0.1.jar文件。
```bash
cp /elk/x-pack/x-pack-core-7.0.1.jar /elk/elasticsearch-7.0.1/modules/x-pack-core/
rm -rf /elk/x-pack  # 完成文件替换后该目录既可以删除了
``` 
## 申请License
完成以上步骤后，我们还需要去elastic官网申请一个license, [License申请地址](https://license.elastic.co/registration)，申请完成后，下载下来的License格式为json格式。并将该License的`type`、`expiry_date_in_millis`、`max_nodes`分别修改成`platinum`、`2524579200999`、`1000`。如下：
```json
{"license":
    {
        "uid":"537c5c48-c1dd-43ea-ab69-68d209d80c32",
        "type":"platinum",
        "issue_date_in_millis":1558051200000,
        "expiry_date_in_millis":2524579200999,
        "max_nodes":1000,
        "issued_to":"pyker",
        "issuer":"Web Form",
        "signature":"AAAAAwAAAA3fIq7NLN3Blk2olVjbAAABmC9ZN0hjZDBGYnVyRXpCOW5Bb3FjZDAxOWpSbTVoMVZwUzRxVk1PSmkxaktJRVl5MUYvUWh3bHZVUTllbXNPbzBUemtnbWpBbmlWRmRZb25KNFlBR2x0TXc2K2p1Y1VtMG1UQU9TRGZVSGRwaEJGUjE3bXd3LzRqZ05iLzRteWFNekdxRGpIYlFwYkJiNUs0U1hTVlJKNVlXekMrSlVUdFIvV0FNeWdOYnlESDc3MWhlY3hSQmdKSjJ2ZTcvYlBFOHhPQlV3ZHdDQ0tHcG5uOElCaDJ4K1hob29xSG85N0kvTWV3THhlQk9NL01VMFRjNDZpZEVXeUtUMXIyMlIveFpJUkk2WUdveEZaME9XWitGUi9WNTZVQW1FMG1DenhZU0ZmeXlZakVEMjZFT2NvOWxpZGlqVmlHNC8rWVVUYzMwRGVySHpIdURzKzFiRDl4TmM1TUp2VTBOUlJZUlAyV0ZVL2kvVk10L0NsbXNFYVZwT3NSU082dFNNa2prQ0ZsclZ4NTltbU1CVE5lR09Bck93V2J1Y3c9PQAAAQCjNd8mwy8B1sm9rGrgTmN2Gjm/lxqfnTEpTc+HOEmAgwQ7Q1Ye/FSGVNIU/enZ5cqSzWS2mY8oZ7FM/7UPKVQ4hkarWn2qye964MW+cux54h7dqxlSB19fG0ZJOJZxxwVxxi8iyJPUSQBa+QN8m7TFkK2kVmP+HnhU7mGUrqXt3zTk5d3pZw3QBQ/Rr3wmSYC5pxV6/o2UHFgu1OPDcX+kEb+UZtMrVNneR+cEwyx7o5Bg3rbKC014T+lMtt69Y080JDI5KfHa7e9Ul0c3rozIL975fP45dU175D4PKZy98cvHJgtsCJF3K8XUZKo2lOcbsWzhK2mZ5kFp0BMXF3Hs",
        "start_date_in_millis":1558051200000
    }
}
```
我们将过期时间写到2050年，type改为platinum 白金版，这样我们就会拥有全部的x-pack功能。
## 配置elasticsearch安全协议
完成以上所有操作在启动elasticsearch前，我们需要配置elasticsearch的SSL/TLS安全协议,如果不配置的话，需要禁止security才能配置License。当License配置完成后我们需要再开启security，并开启SSL\TLS。
```bash
# 加载License到elasticsearch之前操作
$ echo "xpack.security.enabled: false" >> /elk/elasticsearch-7.0.1/config/elasticsearch.yml
$ ./bin/elasticsearch -d   # 后台方式启动elasticsearch

# 加载License到elasticsearch之后操作
$ echo "xpack.security.transport.ssl.enabled: true" >> /elk/elasticsearch-7.0.1/config/elasticsearch.yml
$ sed -i 's/xpack.security.enabled: false/xpack.security.enabled: true/g' /elk/elasticsearch-7.0.1/config/elasticsearch.yml
$ kill -9 13023 && ./bin/elasticsearch -d   # 重启elasticsearch
```
## 加载License到elasticsearch
```bash
$ curl -XPUT -u elastic 'http://192.168.20.210:9200/_xpack/license' -H "Content-Type: application/json" -d @license.json
Enter host password for user 'elastic':           # 提示输入elastic用户密码，当前无密码，所以直接回车
{"acknowledged":true,"license_status":"valid"}    # license写入成功
```
## 查看License
```bash
$ curl -XGET -u elastic:tWbWZc7NE3wYqS6DvSu4 http://192.168.20.210:9200/_license
{
  "license" : {
    "status" : "active",
    "uid" : "537c5c48-c1dd-43ea-ab69-68d209d80c32",
    "type" : "platinum",
    "issue_date" : "2019-05-17T00:00:00.000Z",
    "issue_date_in_millis" : 1558051200000,
    "expiry_date" : "2049-12-31T16:00:00.999Z",
    "expiry_date_in_millis" : 2524579200999,
    "max_nodes" : 1000,
    "issued_to" : "pyker",
    "issuer" : "Web Form",
    "start_date_in_millis" : 1558051200000
  }
}
```
由结果可以看出x-pack到期时间为2049-12-31，破解完成。也可以在kibana web页面`管理`中查看破解详情。
![](/images/pic/x-pack.png)

## 设置密码
现在我们可以使用`x-pack铂金版`的所有功能了，例如密码安全验证功能。
```bash
$ ./bin/elasticsearch-setup-passwords auto

Initiating the setup of passwords for reserved users elastic,apm_system,kibana,logstash_system,beats_system,remote_monitoring_user.
The passwords will be randomly generated and printed to the console.
Please confirm that you would like to continue [y/N]y

Changed password for user apm_system
PASSWORD apm_system = 24UtJKbNI1UqHUQkKPZY

Changed password for user kibana
PASSWORD kibana = 8SSZMisIY0NZFMCS6wv9

Changed password for user logstash_system
PASSWORD logstash_system = rFhWkYzayIUZVl8VIunJ

Changed password for user beats_system
PASSWORD beats_system = U1B4O5SKrSEatqDQRsQz

Changed password for user remote_monitoring_user
PASSWORD remote_monitoring_user = zdpj7HqO02yRXZR9Bwa2

Changed password for user elastic
PASSWORD elastic = tWbWZc7NE3wYqS6DvSu4
```