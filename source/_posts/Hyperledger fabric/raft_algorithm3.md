---
title: UFO Project 3 - Hyperledger Fabric channel implementation
date: 2020-11-29 15:45:26
tag: [블록체인, raft, blockchain,hyperledger fabric, 하이퍼레저 패브릭, ]
category: [hyperledger fabric]
---



generate 를 하고나면, 

```
./byfn.sh up -o etcdraft -s couchdb -l node
```

위의 명령어를 통해서 fabric network 를 up 합니다. 

db는 couch db를 이용하고,  chaincode는 node js 로 작성할거다.



### BYFN의 프로세스

1. 5개의 오더링 서비스 노드, 4개의 피어 노드 총 9개의 노드 컨테이너., CLI 컨테이너, 각 조직의 ca 컨테이너( 총 2개), 4개의 couchdb 컨테이너 가 실행되며, 총 16개의 컨테이너가 먼저 실행된다.
2.  네트워크 내부의 CLI 컨테이너에 접속해 생성된 채널 트랜잭션 파일인 channel.tx를 가지고 채널을 생성하고 채널 ufochannel을 생성하고 모든 피어노드를 가입시킨다.
3. 두 조직의 peer0을 앵커로 가입한다



위의 과정을 구현하기 위해서 필요한 파일은

- docker-compose-ca.yaml
- docker-compose-cli.yaml
- docker-compose-couch.yaml
- docker-compose-etcdraft2.yaml
- docker-compose-base.yaml
- peer-base.yaml

들을 만질 것 이다.

#### docker-compose-ca.yaml

- CA는 MSP에서 암호화 인증을 위해 필요한 인증기관이며, 공개키 인증서 및 이에 대응하는 개인키를 발급한다. 블록체인 네트워크를 구성하는 조직에게는 루트 인증서를, 블록체인 네트워크에 접속하는 사용자에게는 신원등록 인증서를 발급한다.

```yaml

version: '2'

networks:
  byfn:

services:
  ca0:
    image: hyperledger/fabric-ca:$IMAGE_TAG
    environment:
      - FABRIC_CA_HOME=/etc/hyperledger/fabric-ca-server
      - FABRIC_CA_SERVER_CA_NAME=ca-sales
      - FABRIC_CA_SERVER_TLS_ENABLED=true
      - FABRIC_CA_SERVER_TLS_CERTFILE=/etc/hyperledger/fabric-ca-server-config/ca.sales.ufo.com-cert.pem
      - FABRIC_CA_SERVER_TLS_KEYFILE=/etc/hyperledger/fabric-ca-server-config/${BYFN_CA1_PRIVATE_KEY}
      - FABRIC_CA_SERVER_PORT=7054
    ports:
      - "7054:7054"
    command: sh -c 'fabric-ca-server start --ca.certfile /etc/hyperledger/fabric-ca-server-config/ca.sales.ufo.com-cert.pem --ca.keyfile /etc/hyperledger/fabric-ca-server-config/${BYFN_CA1_PRIVATE_KEY} -b admin:adminpw -d'
    volumes:
      - ./crypto-config/peerOrganizations/sales.ufo.com/ca/:/etc/hyperledger/fabric-ca-server-config
    container_name: ca_peerSales
    networks:
      - byfn

  ca1:
    image: hyperledger/fabric-ca:$IMAGE_TAG
    environment:
      - FABRIC_CA_HOME=/etc/hyperledger/fabric-ca-server
      - FABRIC_CA_SERVER_CA_NAME=ca-customer
      - FABRIC_CA_SERVER_TLS_ENABLED=true
      - FABRIC_CA_SERVER_TLS_CERTFILE=/etc/hyperledger/fabric-ca-server-config/ca.customer.ufo.com-cert.pem
      - FABRIC_CA_SERVER_TLS_KEYFILE=/etc/hyperledger/fabric-ca-server-config/${BYFN_CA2_PRIVATE_KEY}
      - FABRIC_CA_SERVER_PORT=8054
    ports:
      - "8054:8054"
    command: sh -c 'fabric-ca-server start --ca.certfile /etc/hyperledger/fabric-ca-server-config/ca.customer.ufo.com-cert.pem --ca.keyfile /etc/hyperledger/fabric-ca-server-config/${BYFN_CA2_PRIVATE_KEY} -b admin:adminpw -d'
    volumes:
      - ./crypto-config/peerOrganizations/customer.ufo.com/ca/:/etc/hyperledger/fabric-ca-server-config
    container_name: ca_peerCustomer
    networks:
      - byfn
```

- network는 byfn이고 services 로 ca0, ca1이 있다.
- ca0 은 sales 조직을 서비스하고, ca1은 customer 조직을 서비스한다.



#### docker-compose-cli.yaml

- volumes

```yaml
volumes:
  orderer.ufo.com:
  peer0.sales.ufo.com:
  peer1.sales.ufo.com:
  peer0.customer.ufo.com:
  peer1.customer.ufo.com:
  ca.sales.ufo.com:
  ca.customer.ufo.com:

```

먼저 docker-compose-cli파일의 volumes 이다.

- docker volume이란?
  - container의 데이터 휘발성 때문에 데이터를 container가 아닌 호스트에 저장할 때,  또는 container 끼리 데이터를 공유할때 Volume을 사용.
- ca

```yaml
ca.sales.ufo.com:
    image: hyperledger/fabric-ca
    environment:
      - FABRIC_CA_SERVER_TLS_ENABLED=true
      - FABRIC_CA_HOME=/etc/hyperledger/fabric-ca-server
      - FABRIC_CA_SERVER_CA_NAME=ca.sales.ufo.com
      - FABRIC_CA_SERVER_CA_CERTFILE=/etc/hyperledger/fabric-ca-server-config/ca.sales.ufo.com-cert.pem
      - FABRIC_CA_SERVER_CA_KEYFILE=/etc/hyperledger/fabric-ca-server-config/ca.sales.ufo.com_sk
    ports:
      - "7054:7054"
    command: sh -c 'fabric-ca-server start -b admin:adminpw'
    volumes:
      - ./crypto-config/peerOrganizations/sales.ufo.com/ca/:/etc/hyperledger/fabric-ca-server-config
    container_name: ca.sales.ufo.com
    networks:
      - byfn
```

위는 ca.sales.ufo.com을 설정하는 부분이다.  ca.customer.ufo.com도 위에서 sales를 customer로 바꿔주면 된다.

- order

  ```yaml
  orderer.ufo.com:
      extends:
        file:   base/docker-compose-base.yaml
        service: orderer.ufo.com
      container_name: orderer.ufo.com
      networks:
        - byfn
  ```

  raft 의 핵심! 오더러 설정 부분이다. 나는 다섯개의 오더러를 설정했으므로, 

  ```yaml
   orderer2.ufo.com:
      extends:
        file:   base/docker-compose-base.yaml
        service: orderer2.ufo.com
      container_name: orderer2.ufo.com
      networks:
        - byfn
  ```

  이런식으로 orderer3, orderer4, orderer5 를 더 만들어 준다.

- peer

  ```yaml
   peer0.sales.ufo.com:
      container_name: peer0.sales.ufo.com
      extends:
        file:  base/docker-compose-base.yaml
        service: peer0.sales.ufo.com
      networks:
        - byfn
  
    peer1.sales.ufo.com:
      container_name: peer1.sales.ufo.com
      extends:
        file:  base/docker-compose-base.yaml
        service: peer1.sales.ufo.com
      networks:
        - byfn
  
  ```

  sales  조직에 대한 peer0, peer1을 설정하고 위 설정에서 sales를 customer로 바꾸면 customer 설정이다.

- cli

  ```yaml
  cli:
      container_name: cli
      image: hyperledger/fabric-tools:$IMAGE_TAG
      tty: true
      stdin_open: true
      environment:
        - SYS_CHANNEL=$SYS_CHANNEL
        - GOPATH=/opt/gopath
        - CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
        - FABRIC_LOGGING_SPEC=DEBUG
        - FABRIC_LOGGING_SPEC=INFO
        - CORE_PEER_ID=cli
        - CORE_PEER_ADDRESS=peer0.sales.ufo.com:7051
        - CORE_PEER_LOCALMSPID=SalesMSP
        - CORE_PEER_TLS_ENABLED=true
        - CORE_PEER_TLS_CERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/sales.ufo.com/peers/peer0.sales.ufo.com/tls/server.crt
        - CORE_PEER_TLS_KEY_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/sales.ufo.com/peers/peer0.sales.ufo.com/tls/server.key
        - CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/sales.ufo.com/peers/peer0.sales.ufo.com/tls/ca.crt
        - CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/sales.ufo.com/users/Admin@sales.ufo.com/msp
      working_dir: /opt/gopath/src/github.com/hyperledger/fabric/peer
      command: /bin/bash
      volumes:
          - /var/run/:/host/var/run/
          - ./../chaincode/:/opt/gopath/src/github.com/chaincode
          - ./crypto-config:/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/
          - ./scripts:/opt/gopath/src/github.com/hyperledger/fabric/peer/scripts/
          - ./channel-artifacts:/opt/gopath/src/github.com/hyperledger/fabric/peer/channel-artifacts
      depends_on:
        - orderer.ufo.com
        - orderer2.ufo.com
        - orderer3.ufo.com
        - orderer4.ufo.com
        - orderer5.ufo.com
        - peer0.sales.ufo.com
        - peer1.sales.ufo.com
        - peer0.customer.ufo.com
        - peer1.customer.ufo.com
        - ca.sales.ufo.com
        - ca.customer.ufo.com
      networks:
        - byfn
  ```

  위에는 cli 설정이다. cli 컨테이너는 위에서 설정해준 모든 컨테이너에 의존한다.



#### docker-compose-couch.yaml

couchdb 설정 yaml 파일이다. 여기서 구성한 네트워크에서는 총 4개의 couchdb 컨테이너가 있는데,

각각의 컨테이너는

- couchdb0 : peer0.sales.ufo.com

- couchdb1 : peer1.sales.ufo.com

- couchdb2 : peer0.customer.ufo.com

- couchdb3 : peer1.customer.ufo.com

  에 대응한다.

예시로 여기서는 couchdb0의 설정만 올리겠다.

```yaml
services:
  couchdb0:
    container_name: couchdb0
    image: hyperledger/fabric-couchdb
    # Populate the COUCHDB_USER and COUCHDB_PASSWORD to set an admin user and password
    # for CouchDB.  This will prevent CouchDB from operating in an "Admin Party" mode.
    environment:
      - COUCHDB_USER=
      - COUCHDB_PASSWORD=
    # Comment/Uncomment the port mapping if you want to hide/expose the CouchDB service,
    # for example map it to utilize Fauxton User Interface in dev environments.
    ports:
      - "5984:5984"
    networks:
      - byfn

  peer0.sales.ufo.com:
    environment:
      - CORE_LEDGER_STATE_STATEDATABASE=CouchDB
      - CORE_LEDGER_STATE_COUCHDBCONFIG_COUCHDBADDRESS=couchdb0:5984
      # The CORE_LEDGER_STATE_COUCHDBCONFIG_USERNAME and CORE_LEDGER_STATE_COUCHDBCONFIG_PASSWORD
      # provide the credentials for ledger to connect to CouchDB.  The username and password must
      # match the username and password set for the associated CouchDB.
      - CORE_LEDGER_STATE_COUCHDBCONFIG_USERNAME=
      - CORE_LEDGER_STATE_COUCHDBCONFIG_PASSWORD=
    depends_on:
      - couchdb0
```

​		

#### docker-compose-etcdraft2.yaml

여기서는 raft를 위해 orderer를 제외하고 나머지 orderer2, orderer3, orderer4, orderer5를 설정한다.

```yaml
volumes:
  orderer2.ufo.com:
  orderer3.ufo.com:
  orderer4.ufo.com:
  orderer5.ufo.com:
```



이 파일 또한 예시로 orderer2.ufo.com 만 첨부 하겠다.

```yaml
orderer2.ufo.com:
    extends:
      file: base/peer-base.yaml
      service: orderer-base
    container_name: orderer2.ufo.com
    networks:
    - byfn
    volumes:
        - ./channel-artifacts/genesis.block:/var/hyperledger/orderer/orderer.genesis.block
        - ./crypto-config/ordererOrganizations/ufo.com/orderers/orderer2.ufo.com/msp:/var/hyperledger/orderer/msp
        - ./crypto-config/ordererOrganizations/ufo.com/orderers/orderer2.ufo.com/tls/:/var/hyperledger/orderer/tls
        - orderer2.ufo.com:/var/hyperledger/production/orderer
    ports:
    - 8050:7050
```



#### docker-compose-base.yaml

이름 그대로, docker-compose의 base가 되는 yaml 설정 파일이다.

```yaml
orderer.ufo.com:
    container_name: orderer.ufo.com
    extends:
      file: peer-base.yaml
      service: orderer-base
    volumes:
        - ../channel-artifacts/genesis.block:/var/hyperledger/orderer/orderer.genesis.block
        - ../crypto-config/ordererOrganizations/ufo.com/orderers/orderer.ufo.com/msp:/var/hyperledger/orderer/msp
        - ../crypto-config/ordererOrganizations/ufo.com/orderers/orderer.ufo.com/tls/:/var/hyperledger/orderer/tls
        - orderer.ufo.com:/var/hyperledger/production/orderer
    ports:
      - 7050:7050
```

orderer.ufo.com 을 예시로 들었다. 전체 코드는 git에서 확인 할 수 있다.

#### peer-base.yaml

이 파일에서는 peer-base, orderer-base의 환경변수 설정등을 한다.

자세한 파일 내용은 git에서 확인 할 수 있다.



---

- 전체 코드 GIT : https://github.com/yuminee/UFO_FabricNet_raft

