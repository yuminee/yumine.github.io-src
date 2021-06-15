---
title: Apple Sillicon/M1 Mysql error, Wordpress-mariadb 연결
date: 2021-02-09 15:55:16
tags: [docker, 도커]
category: Docker
---

# Apple Sillicon/M1 Mysql error, Wordpress-mariadb 연결

도커를 학습하면서 mysql 컨테이너를 다운받아 워드 프레스에 연결하려는데, 다음과 같은 에러가 났다.

```
# docker run -d \
--name wordpressdb \
-e MYSQL_ROOT_PASSWORD=password \
-e MYSQL_DATABASE=wordpress \
mysql:5.7
```

라고 명령어를 쳤고,

받은 에러는

**Unable to find image 'mysql:latest' locally
latest: Pulling from library/mysql
docker: no matching manifest for linux/arm64/v8 in the manifest list entries.
See 'docker run --help'.**

즉, linux/arm64/v8 이 manifest 에 없다 뭐 그런 이야기인거 같은데 애플 실리콘 m1칩을 쓰면서 여러가지 산전수전을 많이 겪어서 stackoverflow에서 여러가지를 찾다가 찾는글이다.

https://stackoverflow.com/questions/65456814/docker-apple-silicon-m1-preview-mysql-no-matching-manifest-for-linux-arm64-v8

어찌되었든 결론적으로 mysql 대신에 마리아db를 쓰라는건데,

https://docs.docker.com/docker-for-mac/apple-m1/

M1칩에 관한 docker docs에 보면 위와 같은 이슈가 있으니, mysql 대신에 mariadb이미지를 쓰라고 되어있다. 

> Not all images are available for ARM64. You can add `--platform linux/amd64` to run an Intel image under emulation.
>
> In particular, the [mysql](https://hub.docker.com/_/mysql?tab=tags&page=1&ordering=last_updated) image is not available for ARM64. You can work around this issue by using a [mariadb](https://hub.docker.com/_/mariadb?tab=tags&page=1&ordering=last_updated) image.



```
# docker run -d \
name wordpressdb \
-e MYSQL_ROOT_PASSWORD=password \
-e MYSQL_DATABASE=wordpress \
mariadb
```

위의 명령어로 mariadb를 다운받고 실행시켜 준다.

```
# docker run -d \
-e WORDPRESS_DB_PASSWORD=password \
--name wordpress \
--link wordpressdb:mysql \
-p 80 \
wordpress
```

위의 명령어로 wordpress를 다운받고 실행시켜 준다.

그 다음, docker ps 명령어로 확인해보면,

```
# docker ps
CONTAINER ID   IMAGE       COMMAND                  CREATED          STATUS          PORTS                   NAMES
65eb2de5ed25   wordpress   "docker-entrypoint.s…"   12 seconds ago   Up 10 seconds   0.0.0.0:49157->80/tcp   wordpress
0660deed1975   mariadb     "docker-entrypoint.s…"   29 seconds ago   Up 28 seconds   3306/tcp                wordpressdb
```

위의 결과를 보면 PORTS에서 0.0.0.0:49157 -> 80으로 호스트의 port 49157로 접근을 하면, 80으로 포트포워딩을 해준다고 나와있다. 

확인해 보기 위해서 위의 주소로 들어가면

![스크린샷 2021-02-10 오전 2.46.10](/Users/yuminkuu/Desktop/스크린샷 2021-02-10 오전 2.46.10.png)

이렇게 워드 프레스 초기 설정화면이 뜨는것을 확인 할 수 있다.

