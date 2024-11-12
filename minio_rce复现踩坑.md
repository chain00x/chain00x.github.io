# minio rce复现踩坑

在参考

https://pow1e.github.io/2023/12/26/%E4%BB%A3%E7%A0%81%E5%AE%A1%E8%AE%A1/minio%E4%BB%8E%E4%BF%A1%E6%81%AF%E6%B3%84%E6%BC%8F%E5%88%B0RCE/

复现minio rce的时候发现无论如何都成功不了，一直报错 mc: <ERROR> Unable to update the server.

<img width="717" alt="image" src="https://github.com/user-attachments/assets/edde6750-d00b-471f-8438-6234fe81ea33">

可以用以下docker-compose.yml搭建容器复现

```
version: '3.7'

services:
  minio:
    image: minio/minio:RELEASE.2022-10-20T00-55-09Z
    ports:
      - "9000:9000"
    environment:
      MINIO_ROOT_USER: minioadmin
      MINIO_ROOT_PASSWORD: minioadmin-vulhub
      MINIO_UPDATE_MINISIGN_PUBKEY:
    command: server /data
    volumes:
      - ./data:/data
    restart: always
```
