---
title: nginx crush (1)
date: 2022-03-01 22:55:05
tag: [nignx, Nginx, 엔진엑스]
category: Nginx
---


## 1. Nginx Docker Setting

Nginx를 공부하기 위해 nginx docker image를 사용하였다.

```shell
docker container run --name webserver -d -p 80:80 nginx
```

위의 명령어로 nginx 컨테이너를 실행하였고,

<img width="640" alt="스크린샷 2022-03-01 오후 11 04 17" src="https://user-images.githubusercontent.com/33755241/156183570-87f61b52-d66f-4af9-ab39-7be66786fe01.png">

localhost:80 으로 nginx가 잘 작동하고 있는것을 확인 하였다.



더 자세하게 Nginx docker안으로 들어가서 어떻게 동작하고 있는지를 확인하기 위해서 컨테이너안으로 접속했다.

분홍색 형광팬은 명령어, 초록색은 docker container id 이다.

![image](https://user-images.githubusercontent.com/33755241/156184710-f6c39e52-cbd7-4959-90a9-fd39105070e7.png)

Nginx Version 1.21.6이 설치 된 것을 마지막 nginx -v 명령어로 확인 할 수 있다.

> 혹시나 같은 에러를 만나시는 분들을 위해서

<img width="765" alt="스크린샷 2022-03-01 오후 11 14 48" src="https://user-images.githubusercontent.com/33755241/156185669-06546008-915c-445e-aeb6-be7981b1f1cc.png">

ps -ef | grep nginx 명령어를 통해서 nginx를 포함하는 프로세스를 찾으려고 했는데 ps 명령어를 찾을 수 없다 하여 [여기](https://zetawiki.com/wiki/Bash:_ps:_command_not_found)  를 통하여 해당 에러를 해결한 후 진행했다. 



<img width="765" alt="스크린샷 2022-03-01 오후 11 21 01" src="https://user-images.githubusercontent.com/33755241/156186132-5d96be2c-f4e6-458f-8792-b54b5ac61a3e.png">

여기서 주목해야 할 것은 엔진엑스가 정상적으로 실행 중이라면 항상 마스터 프로세스가 한 개, 워커 프로세스가 한 개 이상 있다는 점이다. 엔진엑스가 제대로 동작하려면 권한 상승이 필요하므로 마스터 프로세스가 root 권한으로 실행 중이라는 점을 알고 가자!



## 2. 주요 설정 파일, 디렉터리, 명령어

### 엔진엑스 주요 설정 파일과 디렉터리

밑에 설명 외 더 자세한 정보는 [nginx 홈페이지에서 full example](https://www.nginx.com/resources/wiki/start/topics/examples/full/) 을 확인 할 수 있다.

- /etc/nginx

  엔진엑스 서버가 사용하는 기본 설정이 저장된 루트 디렉터리. 엔진엑스는 이곳에 저장된 설정 파일의 내용에 따라 동작.

  <img width="504" alt="스크린샷 2022-03-01 오후 11 25 07" src="https://user-images.githubusercontent.com/33755241/156186805-cd5bca49-b060-44e3-8b00-27a451cc198e.png">

- /etc/nginx/nginx.conf

  엔젠엑스의 기본 성정 파일로, 모든 설정에 대한 진입점. 워커 프로세스 개수, 튜닝, 동적 모듈 적재와 같은 글로벌 설정 항목을 포함하며, 다른 엔진엑스 세부 설정 파일에 대한 참조를 지정. 뿐만 아니라 이어서 설명할 디렉터리에 위치한 모든 설정 파일을 포함하는 최상위 http 블록을 갖고 있음.

  <img src="https://user-images.githubusercontent.com/33755241/156189300-02dd803c-48d6-4649-8337-03ac39fadeca.png" width="50%">

- /etc/nginx/conf.d/

  기본 http 서버 설정 파일을 포함(default.conf). default.conf 파일 내의 server 블록은 엔진엑스를 사용하면서 몇번 작성해보았던 익숙한 내용들이다.

  <img width="50%" alt="스크린샷 2022-03-01 오후 11 53 32" src="https://user-images.githubusercontent.com/33755241/156191562-b7b4d2a5-bf7f-4649-a087-c94042eeb629.png">



- /var/log/nginx

  엔진엑스의 로그가 저장되는 디렉터리이다. access.log 파일과 error.log파일이 있다. 엔진엑스 설정을 통해 debug 모듈을 활설화 했다면 디버그 정보도 오류 로그 파일에 기록된다.



### 엔진엑스 명령어

- nginx -T

  위의 설정 파일을 시험하고 결과를 화면에 보여줌. 

- nginx -s reload

  설정에 문제가 없다면 엔진엑스가 변경된 설정을 읽어드리도록 reload

  해당 명령어를 통해서는 패킷 손실 없이 설정을 리로드 할 수 있다.(무중단 설정 리로드)

- nginx -s signal

  s 매개변수는 지정된 신호(stop, quit, reload, reopen)를 엔진엑스 마스터 프로세스에 전송. 

  - stop은 엔진엑스 프로세스가 즉시 동작을 멈춤.
  - quit은 현재 진행 중인 요청을 모두 처리한 뒤 엔진엑스 프로세스를 종료
  - reload는 엔진엑스가 설정을 다시 읽어드리게 함.
  - reopen은 지정된 로그 파일을 다시 열도록 명령.







