---
title: A tale of performance — ECS, Boto3 & IAM
date: 2022-03-06 15:00:01
tags: [AWS, ECS, boto3, IAM]
category: AWS
mathjax: true
---

https://engineering.instawork.com/a-tale-of-performance-ecs-boto3-iam-dd22a2624398



위의 글을 보고 정리한 내용입니다.



## 문제

- 몇몇 API들의 퍼포먼스가 좋지 않는것을 발견했는데, 해당 API들이 S3를 콜할때 5000ms이상 걸린다는 사실을 알게됨.

  

## 원인

- IAM을 통해서 S3에 접근하는 권한을 얻는데 시간이 너무 오래 걸림.

  

## 문제 해결을 위해 알아야 할 개념
### AWS IAM?

위의 문제를 잘 이해하기 위해서는 AWS의 IAM이 어떻게 동작하는지를 알아야 한다. AWS에서는 특정 리소스에 접근할 경우, 해당 수준의 권한이 필요하다. IAM을 통해 액세스를 할 수 있는데, 아래 두가지 방법이 일반적이다.

1. IAM user 생성. 코드안에 access key와 access key secret 을 넣어서 사용. 주로 로컬에서 돌릴때 해당 방법을 쓴다.
2. IAM role 생성. 돌아가는 서버에 해당 롤을 주는것. 이 방법을 더 추천하는데 그 이유는 보안적으로 관리하기가 쉽기 때문이다(access key와 access key secret를 제공 안해도 되기 때문)



### EC2 Server에 role을 붙일 경우 생기는 일.

boto3를 이용하여 s3을 사용할때 생기는일

1. Boto3 call에 대한 권한이 있는지 알아보기 위해 access key와 access key secret을 확인. 하지만 키를 제공하지 않았으므로 실패
2. ~/.aws/credentials file 확인. 하지만 해당 파일을 제공하지 않았으므로 실패
3. instance metadata service (IMDS)을 사용하여 ec2 metadata endpoint를 콜하므로써 s3의임시 권한을 얻음.

### 그러면 왜 느리냐?

권한만 있으면 되는건데 왜 그냥 한번으로 해결되지 않는걸까?

<img width="468" alt="스크린샷 2022-03-07 오전 1 02 44" src="https://user-images.githubusercontent.com/33755241/156931202-60185eb8-90f5-4fb3-8b63-17f428a0acdf.png">




## 해결



### 그래서 뭘 해야함?

- hop count를 2로 늘린다.
- ecs task role을 이용한다.



AWS에서 권한을 얻는 우선순위는 아래와 같다.

1. command line options
2. Environment vairables
3. CLI credentials file
4. CLI configuration file
5. Container Credentials(ECS task role)
6. Instance profile credentials





## 결론

AWS에서 서비스에 접근하는 권한이 너무 느리다면(특히 컨테이너 환경에서) 다음 3가지를 고려해서 사용해 보기.

1. hot limit을 2이상으로 늘리기
2. Boto3의 session사용(요청마다 권한을 얻을 필요 없이 session을 유지하여 사용하는 옵션)
3. ECS task role 사용하기



