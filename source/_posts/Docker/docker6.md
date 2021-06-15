---
title: 도커 호스트랑 볼륨 공유하기
date: 2021-02-10 15:55:16
tags: [docker, 도커]
category: Docker
---

# 도커 호스트랑 볼륨 공유하기

전 포스터에서는 컨테이너로 워드프레스와 디비를 연결해서 만들어 보았다. 컨테이너로 데이터베이스 작업을 하였을때 가장 큰 단점은 mariadb 컨테이너를 삭제하면 컨테이너의 계층에 저장되어 있던 데이터베이스의 정보도 삭제 된다는 것이다.

도커의 컨테이너는 생성과 삭제가 매우 쉬우므로 실수로 컨테이너를 삭제하면 데이터를 복구할 수 없게 된다. **이를 방지하기 위해 컨테이너의 데이터를 영속적 데이터로 활용할 수 있는 방법이 몇가지 있는데, 그중 가장 쉬운것이 볼륨을 활용하는 것이다.**

볼륨을 활용하는 방법은,

- 호스트와 볼륨 공유하기
- 볼륨 컨테이너 활용
- 도커과 관리하는 볼륨 생성

이 포스팅에서는 첫번째 방법인 `호스트와 볼륨 공유하기` 를 해볼 것이다. 이 방식으로 데이터 베이스 컨테이너를 삭제해도 데이터는 삭제되지 않도록 설정 해보자.



```
# sudo docker run -d \
--name wordpressdb_volume \
-e MYSQL_ROOT_PASSWORD=password \
-e MYSQL_DATABASE=wordpress \
-v /Users/wordpress_db:/var/lib/mysql \
mariadb
```

위의 명령어로 마리아 디비 컨테이너를 생성할려고 하였다.  원래라면 -v 옵션으로 해당 위치에 파일이 생성되어야 하는데,

```
docker: Error response from daemon: error while creating mount source path '/Users/wordpress_db': mkdir /Users/wordpress_db: operation not permitted.
```

하는 에러가 자꾸 떴다. 

그래서 찾다가 그냥 docker desktop으로,

Preferences -> Resources -> File Sharing 부분에서 path를 추가시켜주고

```
# sudo docker run -d \
--name wordpressdb_volume \
-e MYSQL_ROOT_PASSWORD=password \
-e MYSQL_DATABASE=wordpress \
-v /Users/yuminkuu/Desktop/wordpress_db:/var/lib/mysql \
mariadb
```

생성한 path로 볼륨을 설정하여 만들어주니 에러없이 잘 작동하였다.

위와 같은 방식으로 wordpress 컨테이너는 아래와 같이 생성하였다.

```
# docker run -d \
> -e WORDPRESS_DB_PASSWORD=password \
> --name wordpress_volume \
> --link wordpressdb_volume:mariadb \
> -p 80 \
> wordpress
```

이렇게 호스트와 볼륨을 공유하는 방식으로 생성된 컨테이너는 컨테이너를 지워도 해당 폴더의 데이터들이 그대로 남아있게 된다. 

```
-> wordpress_db ls
aria_log.00000001  ib_buffer_pool     ibdata1            multi-master.info  performance_schema
aria_log_control   ib_logfile0        ibtmp1             mysql              wordpress
```

ls 명령어로 확인하였을때 컨테이너의 파일이 복사되어 있는것을 확인 할 수 있다.

하지만 컨테이너를 지우고 다시 생성하면서 같은 폴더를 볼륨으로 생성하면, 파일이 덮어지는것 같으니 조심하자!



