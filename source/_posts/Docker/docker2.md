---
title: Docker Engine (도커 엔진)?
date: 2021-02-07 15:55:16
tags: [docker, 도커, docker run, docker rename, 도커 컨테이너 생성]
category: Docker
---

# Docker Engine (도커 엔진)?

### 도커 이미지와 컨테이너

도커 엔진에서 사용하는 기본 단위는 있다 **이미지**와 **컨테이너**이며, 이 두가지가 도커 엔진의 핵심입니다.



### 도커 이미지

**이미지는 컨테이너를 생성할 때 필요한 요소**이며, 가상 머신을 생성할 때 사용하는 iso 파일과 비슷한 개념입니다. 이미지는 여러 개의 계층으로 된 바이너리 파일로 존재하고, 컨테이너를 생성하고 실행할 때 **있는 읽기 전용**으로 사용됩니다. 이미지는 도커 명령어로 내려받을 수 있으므로 별도로 설치할 필요는 없습니다.

도커에서 사용하는 이미지의 이름은 기본적으로 [저장소 이름]/[이미지 이름]:[태그]의 형태로 구성되어 있습니다.

![이미지이름의구성](https://user-images.githubusercontent.com/33755241/107135224-326eab80-693c-11eb-8578-c06a5f6aba43.png)

- **저장소(Repository ) 이름**은 이미지가 저장된 장소를 의미. 저장소 이름이 명시되지 않은 이미지는 도커에서 기본적으로 제공하는 이미지 저장소인 도커 허브(docker hub)의 공식(official) 이미지 입니다.
- **이미지 이름**은 해당 이미지가 어떤 역활을 하는지 나타냅니다. 위 예시에는 우분투 컨테이너를 생성하기 위한 이미지라는 것을 알 수 있습니다. 
- **태그**는 이미지의 버전 관리, 혹은 리버전(Revision) 관리에 사용합니다. 일반적으로는 버전을 위와 같이 명시하지만, 적지 않는다면 이미지 태그는 latest로 인식합니다.



### 도커 컨테이너

도커 이미지는 우분투, CentOS등 기본적인 리눅스 운영체제에서 부터 아파치 웹 서버, Mysql 데이터베이스 등등 여러가지 종류가 있습니다. 이러한 **이미지로 컨테이너를 생성하면 해당 이미지의 목적에 맞는 파일이 들어 있는 파일 시스템과 격리된 시스템 자원 및 네트워크를 사용할 수 있는 독립된 공간이 생성**되고 이것이 **도커 컨테이너*입니다. 

컨테이너는 이미지를 읽기 전용으로 사용하되 이미지에서 변경된 사항만 컨테이너 계층에 저장하므로 컨테이너는 무엇을 하든지 원래의 이미지에는 영향을 받지 않습니다. 

### 도커 컨테이너 다루기

저는 지금 mac m1 을 사용하고 있어서 m1 버전으로 릴리즈된 docker를 최근에 설치하였습니다. 

[docker for m1 docs ](https://docs.docker.com/docker-for-mac/apple-m1/)에서 설치 방법을 확인 하실 수 있습니다!

#### 컨테이너 생성

![도커버전확인](https://user-images.githubusercontent.com/33755241/107135920-1f5eda00-6942-11eb-8004-e5b87da0a4c2.png){:.alignleft}

위의 명령어로 도커의 버전을 확인 할 수 있고, 저렇게 버전이 나온다면 잘 설치가 된 것 입니다.

<img width="565" alt="스크린샷 2021-02-07 오후 3 01 20" src="https://user-images.githubusercontent.com/33755241/107138058-5ee2f180-6955-11eb-83ec-ad662a465313.png">

docker run 명령어를 입력하면 보여지는 화면입니다. 

`docker run -i -t ubuntu:14.04`

라는 명령어를 통해 컨테이너를 생성 및 샐행과 동시에 컨테이너 내부로 들어오게 됩니다. 컨테이너에서 기본 사용자는 root 이고, 호스트 이름은 무작위의 16진수 해시값입니다. 무작위의 16진수 해쉬값은 컨테이너의 고유한 ID의 앞 일부분이며, 위 예시에서는 65d9daa47df6 입니다.

docker run 명령어로 컨테이너를 생성할때 **-i 명령어로 상호 입출력을, -t 명령어로 tty를 활설화해서 배시(bash) 셸을 사용하도록** 컨테이너를 설정했습니다.

docker run 명령어에서 이 두 옵션중 하나라도 사용하지 않으면 셸을 정상적으로 사용할 수 없습니다.

<img width="680" alt="스크린샷 2021-02-07 오후 3 07 51" src="https://user-images.githubusercontent.com/33755241/107138178-41625780-6956-11eb-9ca4-6ad49394de3f.png">

ls 명령어로 파일시스템을 확인해보면 아무것도 살치되지 않은 상태임을 확인할 수 있습니다.

컨테이너 내부에서 빠져 나오는 방법은

- exit 를 입력하거나,
- Ctrl +D를 동시에 입력

하는 것입니다.

<img width="680" alt="스크린샷 2021-02-07 오후 3 11 46" src="https://user-images.githubusercontent.com/33755241/107138260-cea5ac00-6956-11eb-95d5-ffb2033ff737.png">

다른 방법으로 컨테이너를 정지하지 않고 빠져나오는 것으로 `Ctrl +P,Q` 를 입력하는 것입니다. `exit` 명령어는 배시 셸을 종료함으로써 컨테이너를 정지시킴과 동시에 컨테이너에서 빠져나오지만, Ctrl+ P,Q를 사용하면 단순히 컨테이너의 셸에서만 빠져나오기 때문에 컨테이너 애플리케이션을 개발하는 목적으로 컨테이너를 사용할 때는 이 방법을 많이 씁니다.

docker pull 명령어로 centos 이미지를 다운받아 보겠습니다.

<img width="680" alt="스크린샷 2021-02-07 오후 3 21 46" src="https://user-images.githubusercontent.com/33755241/107138515-33153b00-6958-11eb-9ff4-2558c2c2d04a.png">

이미지가 잘 받아졌는지 확인하기 위해서 **docker images** 명령어를 사용합니다. 이 명령어는 도커 엔진에 존재하는 이미지의 목록을 출력합니다.

<img width="680" alt="스크린샷 2021-02-07 오후 3 23 15" src="https://user-images.githubusercontent.com/33755241/107138563-68ba2400-6958-11eb-91c6-65f193954e80.png">

방금 내려받은 centos 이미지와 아까 전에 받은 ubuntu 이미지를 둘다 확인 할 수 있습니다.

`docker create -i -t --name mycentos centos:7`

<img width="680" alt="스크린샷 2021-02-07 오후 3 24 39" src="https://user-images.githubusercontent.com/33755241/107138586-9b641c80-6958-11eb-8c2d-59329e5eab22.png">

위의 명령어는 다운받은 centos:7 이미지로 컨테이너를 생성하며, --name 옵션으로 컨테이너의 이름을 설정합니다. 저는 mycentos로 설정해 보았습니다.

<img width="439" alt="스크린샷 2021-02-07 오후 3 35 09" src="https://user-images.githubusercontent.com/33755241/107138820-124de500-695a-11eb-8930-b9bad235d11b.png">

docker create 명령어는 컨테이너를 생성만 해주는 명령어 입니다. 그래서 start -> attach를 사용하여 컨테이너를 시작하고 내부로 들어갑니다.

- start : 컨테이너 시작.
- attach : 컨테이너 내부로 들어감.

그럼 컨테이너를 생성하기 위해서, run , create, start 명령어를 사용했습니다. run 명령어는 pull,create,start 명령어를 일괄적으로 실행한 후 attach가 가능한 컨테이너라면 컨테이너 내부로 들어갑니다.

- run 명령어 : docker pull(이미지 없을 때) ->  docker create -> docker start -> docker attach(-i -t 옵션을 사용했을 때)
- create 명령어 : docker pull(이미지 없을 때) ->  docker create

---

### 컨테이너 이름 바꾸기

컨테이너의 이름을 설정해주지 않으면 랜덤으로 이름이 설정되는데,

<img width="433" alt="스크린샷 2021-02-07 오후 3 27 00" src="https://user-images.githubusercontent.com/33755241/107138656-ee3dd400-6958-11eb-964e-119c17b69ccb.png">

도커 dastboard를 보면 이름을 정해준 centos:7 로만든 컨테이너는 mycentos라고 되어있지만, 아까 그냥 만들어준 ubuntu는 awesome_mclean이라는 이름으로 정해져 있습니다(~~개발자 이름일까요..?~~) 

위의 이름은 쉽게 바꾸어 줄 수 있습니다.

<img width="626" alt="스크린샷 2021-02-07 오후 3 29 59" src="https://user-images.githubusercontent.com/33755241/107138720-5a203c80-6959-11eb-9bf7-73fbb95bf05f.png">

docker ps -a 로 컨테이너의 id를 조회를 한 뒤, 

**docker rename [컨테이너id 앞4자리] [바꾸고싶은 이름]**

로 바꾸어 주면 됩니다. 꼭 컨테이너 id 앞 4자리 일 필요는 없고, 다른 컨테이너들과 구분 할 수 있을 정도만 입력해주면 됩니다.

<img width="439" alt="스크린샷 2021-02-07 오후 3 32 12" src="https://user-images.githubusercontent.com/33755241/107138776-a8354000-6959-11eb-9806-87f839b62507.png">

위의 명령어로 우분투 컨테이너의 이름도 바꾸어 보았습니다.

---

위키 북스의 시작하세요 도커/쿠버네티스를 읽고 작성하였습니다.