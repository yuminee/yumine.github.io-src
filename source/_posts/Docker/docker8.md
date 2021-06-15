---
title: 도커 볼륨으로 데이터 보존하기
date: 2021-02-12 15:55:16
tags: [docker, 도커]
category: Docker
---
# 도커 볼륨으로 데이터 보존하기

도커 자체에서 제공하는 볼륨 기능을 활용해 데이터를 보존할 수 있다.

```
# docker volume create --name myvolume
# docker volume ls
DRIVER    VOLUME NAME     
local     myvolume
```

위의 명령어로 도커 볼륨을 생성하고, 아래의 명령어로 myvolume이라는 볼륨을 사용하는 컨테이너를 생성한다.

```
# docker run -i -t --name myvolume_1 -v myvolume:/root/ ubuntu
root@f333e2b6b071:/# echo hello, volume! >> /root/volume
```

위의 예시는 볼륨을 컨테이너의 /root/ 디렉터리에 마운트하므로 /root 디렉터리에 파일을 쓰면 해당 파일이 볼륨에 저장된다.  /root 디렉터리에 volume이라는 파일을 생성했다. 다른 컨테이너도 myvolume 볼륨을 쓰면 볼륨을 활용한 디렉터리에 volume 파일이 존재할 것 이다. 