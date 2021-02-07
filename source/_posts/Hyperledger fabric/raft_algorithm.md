---
title: 'UFO Project 1 - understanding raft '
date: 2020-11-17 13:59:44
tag: [블록체인, raft, blockchain,hyperledger, 하이퍼레저]
category: [hyperledger fabric]
---
# 하이퍼레저 패브릭 개발 
 
[1. Raft](#Raft)    
[2. Raft Algorithm](#Raft_Algorithm)    
[3. Leader Election](#Leader_Election)       
[4. Log Replication](#Log_Replication)
[5. Election timeout ](#Election_timeout )

 
 **중요한 파일**
 
 * byfn.sh : 간단한 하이퍼레저 패브릭 네트워크를 구축하고 이를 테스트 할 수 있는 BYFN 예제를 위한 쉘 스크립트 파일. byfn.sh를 실행해 하이퍼레저 패브릭 네트워크를 쉽게 구축 할 수 있다.
 
 ## Raft
 
 <span style="display:block;text-align:center">![image](https://user-images.githubusercontent.com/33755241/99322868-3e564e80-28b4-11eb-94e2-356226253980.png)</span>

 
 - Consensus : 분산 시스템에서 노드 간의 상태를 공유하는 알고리즘.
 
 - Consensus Problem(distributed consensus problem) : 여러 노드로 이루어진 분산 서버에서 합의를 이뤄내는문제
                       
 - Paxos, Zookeeper의 zab가 있음. Raft는 이해하기 어려운 기존의 알고리즘과 달리 쉽고 구현이 용이하게 설계.
                       
 ## Raft Algorithm
 
 - 뗏목처럼 운용 중인 여러 서버들 중 일부에 장애가 발생하더라도 제 기능을 유지하도록 하는 **합의알고리즘** (합의 프로토콜)
 
 <span style="display:block;text-align:center">![image](https://user-images.githubusercontent.com/33755241/99323002-8bd2bb80-28b4-11eb-8829-5281d2990ed1.png)</span>
 
 - Raft의 node는 **Follower, Candidate, Leader** 라는 3가지 상태를 가짐.
 - 모든 노드는 처음에 Follower state 가지고 시작. Follower가 Leader의 응답을 받지 못하면 Candidate
 상태로 전환될 수 있음.
 
 <span style="display:block;text-align:center">![image](https://user-images.githubusercontent.com/33755241/99322373-27632c80-28b3-11eb-8581-68e1fc25bf47.png)</span>
 
 - Candidate는 다른 노드들에게 투표를 요청하고 노드들은 투표 결과를 응답으로 전달.
 - 가장 많은 표를 얻은 노드는 Leader가 될 수 있음. (이러한 프로세스를 Leader Election)
 
 
###### Election timeout 

- - Election timeout : Follower에서 Candidate로 전환되기 위해 기다리는 시간.
                      일반적으로 150ms 에서 300ms 사이의 값으로 랜덤하게 설정.

 ## Leader Election
 
 - 모든 팔로워 노드들은 리더가 될 수 있음. 단 먼저 후보자 노드가 되어야 리더로 산출될 수 있는 자격이 주    어짐. 만약 팔로워 노드가 리더로 부터 아무소식도 듣지못하면 후보자 노드가 될 수 있음.
 <span style="display:block;text-align:center">![image](https://user-images.githubusercontent.com/33755241/99323739-2a135100-28b6-11eb-80f5-a80c7945293b.png)</span>
<span style="display:block;text-align:center">![image](https://user-images.githubusercontent.com/33755241/99323766-38616d00-28b6-11eb-8ec5-2e50d1b8422d.png)</span>
 1. Election timeout이 끝나면 Follwer는 Candidate가 되고 Election term을 시작.
 2. Candidate는 본인에게 투표를 하고 다른 노드들에게 투표 요청 메세지를 전달.
 3. 만일 메세지를 받는 노드가 해당 Election term에서 아직 투표를 하지 않았다면, 먼저 메세지를 전달해준 Candidate에게 투표.
 4. 투표를 마친 해당 노드는 Election timeout이 초기화
 5. 가장 많은 표를 받은 노드가 Leader로 선정.
 
 
  ## Log Replication
 
 <span style="display:block;text-align:center">![image](https://user-images.githubusercontent.com/33755241/99323838-59c25900-28b6-11eb-9937-198e5a0a508f.png)</span>

 1. Green node가 시스템에 "5"라는 값을 업데이트 하라는 요청을 보냄.
 2. 클라이언트의 요청은 가장 먼저 리더에게 수신.
 3. 업데이트 요청을 받은 리더는 우선 **자신의 로그 인트리에 해당 값을 기록**
    (변경된 사항은 즉시 커밋되지 않음)
 4. 변경된 사항을 전체 시스템에 커밋하기 위해 리더는 자신의 로그를 팔로워들의 노드에 복제.
 5. 리더는 과반수 이상의 팔로워 노드들이 각자의 로그 엔트리에 변경사항을 기록 할 때까지 기다림.
 6. 과반수 이상의 노드들로부터 각자의 로그 엔트리가 기록이 완료 되었다는 응답을 받으면, 리더는 자신의 로그를 커밋하고 값을 "5"로 업데이트. 팔로워들에게 변경 사항이 커밋되었음을 알림.
 7. 리더의 알림을 받은 팔로워들의 상태값은 모두 "5"로 변경
 
 <span style="display:block;text-align:center">![image](https://user-images.githubusercontent.com/33755241/99324159-fd136e00-28b6-11eb-8f89-08dc58c077b9.png)</span>

## Election timeout 

- - Election timeout : Follower에서 Candidate로 전환되기 위해 기다리는 시간.
(일반적으로 150ms 에서 300ms 사이의 값으로 랜덤하게 설정.)
                   
<span style="display:block;text-align:center">![image](https://user-images.githubusercontent.com/33755241/99324625-0bae5500-28b8-11eb-8818-42130f880fe7.png)</span>
                      
1. 만일 두 개의 노드가 동시에 Election term을 시작하고 메세지가 동시에 Follwer에게 도달한다고 가정
2. 이러한 경우 노드A, 노드B는 2표씩 얻게 되고, 표가 동일하므로 해당 Election term에는 Leader가 선정
3. Leader가 선정되지 않았으므로 Election timeout에 따라 새로운 Election term을 시작하게됨.

<span style="display:block;text-align:center">![image](https://user-images.githubusercontent.com/33755241/99324741-41533e00-28b8-11eb-8603-46db4a630866.png)</span>
