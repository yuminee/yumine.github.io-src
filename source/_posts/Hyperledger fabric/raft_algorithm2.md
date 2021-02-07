---
title: UFO Project 2 - Hyperledger Fabric generate implementation
date: 2020-11-27 19:36:57
tag: [블록체인, raft, blockchain,hyperledger fabric, 하이퍼레저 패브릭, ]
category: [hyperledger fabric]
---



지난 포스트에서는 raft algorithom에 대해 간단히 공부를 했으니 이제 본격적으로 구현을 시작해 봅니다.

### 개발 환경

- ubuntu 

- Docker & Docker Compose
  Fabric Network 구동을 위한 컨테이너

- chain code : node js

  ![image](https://user-images.githubusercontent.com/33755241/100456009-05637880-3103-11eb-9910-1dcbdbc7a974.png)

  지금 개발할려는 raft 기반의 Hyperledger Fabric Network는 위의 그림과 같다.

  <br>

  <br>

   ### Generate 

  hyperledfer fabric network를 구축할때 가장 먼저 하는것이 generate 이다.

  generate을 하기 위해 나는

  - ccp-generate.sh
  - ccp-template.josn
  - ccp-template.yaml
  - configtx.yaml : 네트워크의 channel과 genesis block을 만들고 anchor peer를 설정한다. 파일 이름에서 유츄할 수 있듯이 네트워크 전체의 설정 내용을 담고 있다.
  - cryto-config.yaml : cryptogen이 이 파일을 사용한다. 이 파일을 통해서 organization과 그 구성원들에게 각각 certificate를 발급한다. 그래서 각각의 organization들이 독자적인 CA를 가지고 있는 것처럼 보이게 할 수 있다.
  - connection - customer.json
  - connection-sales.yaml
  - connection - sales.json
  - connection-sales.yaml

  파일들을 만들어 줄 것이다. 

  맨 처음에 네트워크를 구성할때에는 connection.json파일 하나로 모든 정보를 담았었는데, 하이퍼레저  raft의 파일구조를 참고하여, 팀원들과 이야기를 해본 결과 저렇게 조직별로 나누기로 하였다.

  

  #### ./byfn.sh generate 를 실행하면

  - generateCerts
  - replacePrivateKey
  - generateChannelArtifacts

  이렇게 세 가지의 함수를 실행한다.

  <br>

  <br>

  #### 1. genetateCerts

     ```shell
     function generateCerts() {
       which cryptogen
       if [ "$?" -ne 0 ]; then
         echo "cryptogen tool not found. exiting"
         exit 1
       fi
       echo
       echo "##########################################################"
       echo "##### Generate certificates using cryptogen tool #########"
       echo "##########################################################"
     
       if [ -d "crypto-config" ]; then
         rm -Rf crypto-config
       fi
       set -x
       cryptogen generate --config=./crypto-config.yaml
       res=$?
       set +x
       if [ $res -ne 0 ]; then
         echo "Failed to generate certificates..."
         exit 1
       fi
       echo
       echo "Generate CCP files for Sales and Customer"
       ./ccp-generate.sh
     
       mv crypto-config/peerOrganizations/sales.ufo.com/ca/*_sk crypto-config/peerOrganizations/sales.ufo.com/ca/ca.sales.ufo.com_sk
       mv crypto-config/peerOrganizations/customer.ufo.com/ca/*_sk crypto-config/peerOrganizations/customer.ufo.com/ca/ca.customer.ufo.com_sk
     }
     ```

     <br>
     <br>

    ##### - crypto-config.yaml 

       - OrderOrgs

         ```yaml
         OrderOrgs:
         	- Name: Orderer
         	  Domain : ufo.com
         	  EnableNodeOUs : true
         	  
         	  
         	  Specs:
         	  	- Hostname : orderer
         	  	- Hostname : orderer2
         	  	- Hostname : orderer3
         	  	- Hostname : orderer4
         	  	- Hostname : orderer5  #raft를 구현 할 것이므로 5개.
         ```
         <br>
         <br>

       - PeerOrgs

         ```yaml
         PeerOrgs:
         	- Name : Sales
         	  Domain : sales.ufo.com
         	  EnableNodeOUs : true
         	 
         	  Template :
         	 	 Count : 2 #조직의 peer node의 수
         	 
         	  Users : 
         	 	 Count : 1 #admin의 수
         	 	
         	 - Name : Customer
         	   Domain : customer.ufo.com
         	   EnableNodeOUs : true
         	   
         	   Template :
         	 	 Count : 2 #조직의 peer node의 수
         	 
         	  Users : 
         	 	 Count : 1 #admin의 수	
         ```

       - Sales 와 Customer 2개의 조직에 각각 2개의 peer node , 1개의 admin이 있는 구조이다.

       <br>

       <br>

    ##### - ccp-generate.sh

       ```shell
       #!/bin/bash
       
       function one_line_pem {
           echo "`awk 'NF {sub(/\\n/, ""); printf "%s\\\\\\\n",$0;}' $1`"
       }
       
       function json_ccp {
           local PP=$(one_line_pem $6)
           local CP=$(one_line_pem $7)
           sed -e "s/\${SORG}/$1/" \
               -e "s/\${LORG}/$2/" \
               -e "s/\${P0PORT}/$3/" \
               -e "s/\${P1PORT}/$4/" \
               -e "s/\${CAPORT}/$5/" \
               -e "s#\${PEERPEM}#$PP#" \
               -e "s#\${CAPEM}#$CP#" \
               ccp-template.json 
       }
       
       function yaml_ccp {
           local PP=$(one_line_pem $6)
           local CP=$(one_line_pem $7)
           sed -e "s/\${SORG}/$1/" \
               -e "s/\${LORG}/$2/" \
               -e "s/\${P0PORT}/$3/" \
               -e "s/\${P1PORT}/$4/" \
               -e "s/\${CAPORT}/$5/" \
               -e "s#\${PEERPEM}#$PP#" \
               -e "s#\${CAPEM}#$CP#" \
               ccp-template.yaml | sed -e $'s/\\\\n/\\\n        /g'
       }
       
       
       SORG=sales
       LORG=Sales 
       P0PORT=7051
       P1PORT=8051
       CAPORT=7054
       PEERPEM=crypto-config/peerOrganizations/sales.ufo.com/tlsca/tlsca.sales.ufo.com-cert.pem
       CAPEM=crypto-config/peerOrganizations/sales.ufo.com/ca/ca.sales.ufo.com-cert.pem
       
       echo "$(json_ccp $SORG $LORG $P0PORT $P1PORT $CAPORT $PEERPEM $CAPEM)" > connection-sales.json
       echo "$(yaml_ccp $SORG $LORG $P0PORT $P1PORT $CAPORT $PEERPEM $CAPEM)" > connection-sales.yaml
       
       
       SORG=customer
       LORG=Customer
       P0PORT=9051
       P1PORT=10051
       CAPORT=8054
       PEERPEM=crypto-config/peerOrganizations/customer.ufo.com/tlsca/tlsca.customer.ufo.com-cert.pem
       CAPEM=crypto-config/peerOrganizations/customer.ufo.com/ca/ca.customer.ufo.com-cert.pem
       
       echo "$(json_ccp $SORG $LORG $P0PORT $P1PORT $CAPORT $PEERPEM $CAPEM)" > connection-customer.json
       echo "$(yaml_ccp $SORG $LORG $P0PORT $P1PORT $CAPORT $PEERPEM $CAPEM)" > connection-customer.yaml
       
       ```

       - SORG, LORG ,P0PORT, P1PORT, CAPORT, PEERPEM, CAPEM설정해주고 그 변수들을 ccp-template.json, ccp-template.yaml 형식으로 connection-sales.json, connection-sales.yaml, connection-customer.json, connection-customer.yaml에 보내준다.

       <br>

       <br>

    ##### - ccp-template.json

         ```json
         {
             "name": "ufo-network-${LORG}Org",
             "version": "1.0.0",
             "client": {
                 "organization": "${LORG}",
                 "connection": {
                     "timeout": {
                         "peer": {
                             "endorser": "300"
                         }
                     }
                 }
             },
             "organizations": {
                 "${ORG}": {
                     "mspid": "${LORG}MSP",
                     "peers": [
                         "peer0.${SORG}.ufo.com",
                         "peer1.${SORG}.ufo.com"
                     ],
                     "certificateAuthorities": [
                         "ca.${SORG}.ufo.com"
                     ]
                 }
             },
             "peers": {
                 "peer0.${SORG}.ufo.com": {
                     "url": "grpcs://localhost:${P0PORT}",
                     "tlsCACerts": {
                         "pem": "${PEERPEM}"
                     },
                     "grpcOptions": {
                         "ssl-target-name-override": "peer0.${SORG}.ufo.com",
                         "hostnameOverride": "peer0.${SORG}.ufo.com"
                     }
                 },
                 "peer1.${SORG}.ufo.com": {
                     "url": "grpcs://localhost:${P1PORT}",
                     "tlsCACerts": {
                         "pem": "${PEERPEM}"
                     },
                     "grpcOptions": {
                         "ssl-target-name-override": "peer1.${SORG}.ufo.com",
                         "hostnameOverride": "peer1.${SORG}.ufo.com"
                     }
                 }
             },
             "certificateAuthorities": {
                 "ca.${SORG}.ufo.com": {
                     "url": "https://localhost:${CAPORT}",
                     "caName": "ca-${SORG}",
                     "tlsCACerts": {
                         "pem": "${CAPEM}"
                     },
                     "httpOptions": {
                         "verify": false
                     }
                 }
             }
         }
         
         ```

    ##### - ccp-template.yaml

         ```yaml
         ---
         name: first-network-${LORG}Org
         version: 1.0.0
         client:
           organization: ${LORG}Org
           connection:
             timeout:
               peer:
                 endorser: '300'
         organizations:
           Org${SORG}:
             mspid: ${LORG}MSP
             peers:
             - peer0.${SORG}.ufo.com
             - peer1.${SORG}.ufo.com
             certificateAuthorities:
             - ca.${SORG}.ufo.com
         peers:
           peer0.${SORG}.ufo.com:
             url: grpcs://localhost:${P0PORT}
             tlsCACerts:
               pem: |
                 ${PEERPEM}
             grpcOptions:
               ssl-target-name-override: peer0.${SORG}.ufo.com
               hostnameOverride: peer0.${SORG}.ufo.com
           peer1.${SORG}.ufo.com:
             url: grpcs://localhost:${P1PORT}
             tlsCACerts:
               pem: |
                 ${PEERPEM}
             grpcOptions:
               ssl-target-name-override: peer1.${SORG}.ufo.com
               hostnameOverride: peer1.${SORG}.ufo.com
         certificateAuthorities:
           ca.${SORG}.ufo.com:
             url: https://localhost:${CAPORT}
             caName: ca-${SORG}
             tlsCACerts:
               pem: |
                 ${CAPEM}
             httpOptions:
               verify: false
         ```
      <br>
      <br>

      ![image](https://user-images.githubusercontent.com/33755241/100534864-09a0aa80-3257-11eb-8e33-f51205efffef.png)
      그러고 나면, 이렇게 4개의 파일이 생성이 된다.
      ![image](https://user-images.githubusercontent.com/33755241/100534990-4c16b700-3258-11eb-939b-3d9c3c9231eb.png)
      그리고 crypto-config 파일까지 생성

      <br>
      <br>



  #### 2. replacePrivateKey

     ```shell
     function replacePrivateKey() {
       # sed on MacOSX does not support -i flag with a null extension. We will use
       # 't' for our back-up's extension and delete it at the end of the function
       ARCH=$(uname -s | grep Darwin)
       if [ "$ARCH" == "Darwin" ]; then
         OPTS="-it"
       else
         OPTS="-i"
       fi
     
       # Copy the template to the file that will be modified to add the private key
       cp docker-compose-e2e-template.yaml docker-compose-e2e.yaml
     
       # The next steps will replace the template's contents with the
       # actual values of the private key file names for the two CAs.
       CURRENT_DIR=$PWD
       cd crypto-config/peerOrganizations/sales.ufo.com/ca/
       PRIV_KEY=$(ls *_sk)
       cd "$CURRENT_DIR"
       sed $OPTS "s/CA1_PRIVATE_KEY/${PRIV_KEY}/g" docker-compose-e2e.yaml
       cd crypto-config/peerOrganizations/customer.ufo.com/ca/
       PRIV_KEY=$(ls *_sk)
       cd "$CURRENT_DIR"
       sed $OPTS "s/CA2_PRIVATE_KEY/${PRIV_KEY}/g" docker-compose-e2e.yaml
       # If MacOSX, remove the temporary backup of the docker-compose file
       if [ "$ARCH" == "Darwin" ]; then
         rm docker-compose-e2e.yaml
       fi
     }
     ```

  ​      만들어진 ca의 key값을 docker-compose-e2e.yaml 파일에 복사함.

  <br>

  <br>

#### 3. generateChannelArtifacts

   ```shell
   function generateChannelArtifacts() {
     which configtxgen
     if [ "$?" -ne 0 ]; then
       echo "configtxgen tool not found. exiting"
       exit 1
     fi
   
     echo "##########################################################"
     echo "#########  Generating Orderer Genesis block ##############"
     echo "##########################################################"
     # Note: For some unknown reason (at least for now) the block file can't be
     # named orderer.genesis.block or the orderer will fail to launch!
     echo "CONSENSUS_TYPE="$CONSENSUS_TYPE
     set -x
     if [ "$CONSENSUS_TYPE" == "solo" ]; then
       configtxgen -profile TwoOrgsOrdererGenesis -channelID $SYS_CHANNEL -outputBlock ./channel-artifacts/genesis.block
     elif [ "$CONSENSUS_TYPE" == "kafka" ]; then
       configtxgen -profile SampleDevModeKafka -channelID $SYS_CHANNEL -outputBlock ./channel-artifacts/genesis.block
     elif [ "$CONSENSUS_TYPE" == "etcdraft" ]; then
       configtxgen -profile SampleMultiNodeEtcdRaft -channelID $SYS_CHANNEL -outputBlock ./channel-artifacts/genesis.block
     else
       set +x
       echo "unrecognized CONSESUS_TYPE='$CONSENSUS_TYPE'. exiting"
       exit 1
     fi
     res=$?
     set +x
     if [ $res -ne 0 ]; then
       echo "Failed to generate orderer genesis block..."
       exit 1
     fi
     echo
     echo "#################################################################"
     echo "### Generating channel configuration transaction 'channel.tx' ###"
     echo "#################################################################"
     set -x
     configtxgen -profile TwoOrgsChannel -outputCreateChannelTx ./channel-artifacts/channel.tx -channelID $CHANNEL_NAME
     res=$?
     set +x
     if [ $res -ne 0 ]; then
       echo "Failed to generate channel configuration transaction..."
       exit 1
     fi
   
     echo
     echo "#################################################################"
     echo "#######    Generating anchor peer update for SalesMSP   ##########"
     echo "#################################################################"
     set -x
     configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/SalesMSPanchors.tx -channelID $CHANNEL_NAME -asOrg SalesMSP
     res=$?
     set +x
     if [ $res -ne 0 ]; then
       echo "Failed to generate anchor peer update for SalesMSP..."
       exit 1
     fi
   
     echo
     echo "#################################################################"
     echo "#######    Generating anchor peer update for CustomerMSP   ##########"
     echo "#################################################################"
     set -x
     configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate \
       ./channel-artifacts/CustomerMSPanchors.tx -channelID $CHANNEL_NAME -asOrg CustomerMSP
     res=$?
     set +x
     if [ $res -ne 0 ]; then
       echo "Failed to generate anchor peer update for CustomerMSP..."
       exit 1
     fi
     echo
   }
   
   ```

   Channel configuration들을 만드는 부분.

   configtxgen을 이용하여 Genesis Block을 만들고, genesis block 설정관련은 configtx.yaml파일의 Profiles에 적어두었다.

   ```shell
     configtxgen -profile TwoOrgsChannel -outputCreateChannelTx ./channel-artifacts/channel.tx -channelID $CHANNEL_NAME
   ```

   위의 명령어로 channel configuration 파일을 만들고,

   ```shell
   configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate \
       ./channel-artifacts/CustomerMSPanchors.tx -channelID $CHANNEL_NAME -asOrg CustomerMSP
   ```

   위의 명령어로 각 조직의 앵커피어들의 정보를 transaction으로 만든다.

   여기서 만들어진 네 파일들은 모두 channel을 만드는 설정 파일이다. (.block, .tx)

   각각은 모두 block이거나 transaction들로 channel이 만들어지고 그 channel의 peer들의 ledger에 모두 기록된다.

