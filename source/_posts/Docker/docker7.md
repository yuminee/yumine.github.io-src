---
title: 도커 호스트 볼륨 다른 컨테이너에 덮어쓰기
date: 2021-02-11 15:55:16
tags: [docker, 도커]
category: Docker
---

# 도커 호스트 볼륨 다른 컨테이너에 덮어쓰기

```
docker run -i -t --name volume_dummy alicek106/volume_test
```

위의 명령어로 volume_test 라는 컨테이너를 만들었다.

```
root@d44885d7827e:/# ls /home/testdir_2/
test
```

위의 컨테이너에는 test 라는 폴더가 들어있다.

```
docker run -i -t --name volume_overide -v /Users/yuminkuu/Desktop/wordpress_db:/home/testdir_2 alicek106/volume_test
```

위의 명령어로 이미지에 원래 존재하던 디렉터리에 호스트의 볼륨을 공유하였고,

```
root@092d0af651c7:/# ls /home/testdir_2/
aria_log.00000001  aria_log_control  ib_buffer_pool  ib_logfile0  ibdata1  multi-master.info  mysql  performance_schema  wordpress
```

위와 같은 결과를 얻었다(전 포스터에서 만든 wordpress_db를 그대로 가져온것을 알 수 있다.) 즉, 컨테이너 디렉터리내용이 덮혀 사라지고, 호스트 볼륨에 있던 내용이 쓰여진다.



