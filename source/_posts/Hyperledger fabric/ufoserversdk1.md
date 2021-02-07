---
title: UFO Project 5 - Hyperledger Fabric SDK + application (1)
date: 2020-12-02 15:45:26
tag: [블록체인, raft, blockchain,hyperledger fabric, 하이퍼레저 패브릭, fabric-client, fabric sdk]
category: [hyperledger fabric]
---





체인코드를 기반으로 모바일과 통신하기 위한 서버가 필요하다. 

### 하이퍼레저 패브릭 SDK

 하이퍼레저 패브릭 SDK를 통해 외부에서 하이퍼레저 패브릭 네트워크에 접속할 수 있다. 

![image](https://user-images.githubusercontent.com/33755241/101305979-248ea280-3887-11eb-9de1-f83bb7438454.png)

하이퍼레저 패브릭 SDK 는 크게 3가지 핵심모듈로 구성되는데.

- **Fabric-client** : 하이퍼레저 패브릭 기반 블록체인 네트워크와 통신을 가능하게 하는 핵심 구성요소다. 피어, 오더러 관리 및 이벤트 처리 등 다양한 API를 제공한다. 새로운 채널 생성, 피어 노드의 채널 참여, 피어에 체인코드 설치 및 인스턴스화, 트랜잭션 제출, 트랜잭션 또는 블록의 원장 조회등.
- **Fabric-CA-Client**: 사용자 관리에 사용된다. 새로운 사용자 등록, 하이퍼레저 패브릭 서버에서 서명한 등록 인증서 발급, 기존 사용자 인증서 폐기등이 있다.
- **Fabric-Network(API)** : 플러그할 수 있는 구성 요소에 대한 API를 제공한다. SDK에서 사용하는 주요 인터페이스인 CryptoSuite, key, keyValueStore를 기본적으로 내장하고 있다.

하이퍼레저 패브릭 SDK는 하이퍼레저 패브릭 네트워크와 gRPC를 통해 통신하는데, gRPC는 구글에서 개발한 HTTP 기반 RPC 프레임워크로, 더 적은 리소스를 통해 네트워크 통신의 효율성을 극대화해 성능을 강화한 통신 프로토콜이다.



먼저 hyperledger fabric server도 node js 로 작성 할 것이기 때문에, package.json을 작성해준다.

```
{
  "name": "server",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "start": "./node_modules/nodemon/bin/nodemon.js src/app.js",
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "dependencies": {
    "body-parser": "^1.18.3",
    "cors": "^2.8.5",
    "express": "^4.16.4",
    "fabric-ca-client": "~1.4.0",
    "fabric-network": "~1.4.0",
    "handlebars": "^4.5.3",
    "js-yaml": "^3.13.1",
    "morgan": "^1.9.1",
    "nodemon": "^1.18.9",
    "sed": "0.0.1",
    "tar": ">=2.2.2"
  }
}
```

그리고 난뒤 package.json이 있는 폴더에서 npm install을 하게되면 node_modules가 설치된다.



먼저 관리자를 등록해줘야 한다. 필요한 정보를 위해서 connection.yaml 과 config.json을 작성한다.

- config.json

```json
{
    "connection_file": "connection.yaml",
    "appAdmin": "admin",
    "appAdminSecret": "adminpw",
    "orgMSPID": "SalesMSP",
    "caName": "ca.sales.ufo.com",
    "userName": "admin",
    "gatewayDiscovery": { "enabled": false, "asLocalhost": true }
}
```

- connetcion.yaml

  해당 파일에는 채널, 조직, 피어들의 정보가 담겨있다. 전체 코드는 깃에서 확인 할 수 있다.

  ```yaml
  channels:
    ufochannel:
      orderers:
        - orderer.ufo.com
        ...
        ...
       
  ```

  

enrollAdmin.js라는 파일을 만든다.

```js
/*
 * SPDX-License-Identifier: Apache-2.0
 */

'use strict';

const FabricCAServices = require('fabric-ca-client');
const { FileSystemWallet, X509WalletMixin } = require('fabric-network');
const fs = require('fs');
const path = require('path');
const yaml = require('js-yaml')

// capture network variables from config.json
const configPath = path.join(process.cwd(), 'config.json');
const configJSON = fs.readFileSync(configPath, 'utf8');
const config = JSON.parse(configJSON);
var connection_file = config.connection_file;
var appAdmin = config.appAdmin;
var appAdminSecret = config.appAdminSecret;
var userName = config.userName;
var orgMSPID = config.orgMSPID;
var caName = config.caName;

const filePath = path.join(process.cwd(), '/connection.yaml');
let fileContents = fs.readFileSync(filePath, 'utf8');
let connectionFile = yaml.safeLoad(fileContents);

async function main() {
    try {
        // Create a new CA client for interacting with the CA.
        const caURL = connectionFile.certificateAuthorities[caName].url;
        const ca = new FabricCAServices(caURL);
        // Create a new file system based wallet for managing identities.
        const walletPath = path.join(process.cwd(), 'wallet');
        const wallet = new FileSystemWallet(walletPath);

        // Check to see if we've already enrolled the admin user.
        const adminExists = await wallet.exists(userName);
        if (adminExists) {
            console.log('An identity for the admin user "admin" already exists in the wallet');
            return;
        }

        // Enroll the admin user, and import the new identity into the wallet.

        const enrollment = await ca.enroll({ enrollmentID: appAdmin, enrollmentSecret: appAdminSecret });
        const identity = X509WalletMixin.createIdentity(orgMSPID, enrollment.certificate, enrollment.key.toBytes());
        wallet.import(userName, identity);
        console.log('msg: Successfully enrolled admin user ' + userName + ' and imported it into the wallet');

    } catch (error) {
        console.error(`Failed to enroll admin user: ${userName} ${error}`);
        process.exit(1);
    }
}

main();
```



처음 서버를 키고나서 generate 후에 만들어지는 key들을 위에서 만든 connection.yaml에 복사해서 가져와야 한다. 그것을 위해 sh 파일을 만들어 주었다.

```sh
  
#!/bin/bash

echo 'printing keystore for Sales'

ORG_1_KEYSTORE=$(ls ../../ufo-network/crypto-config/peerOrganizations/sales.ufo.com/users/Admin\@sales.ufo.com/msp/keystore/)
ORG_2_KEYSTORE=$(ls ../../ufo-network/crypto-config/peerOrganizations/customer.ufo.com/users/Admin\@customer.ufo.com/msp/keystore/)

ORG_1_PATH_TO_KEYSTORE="Admin@sales.ufo.com/msp/keystore/"
ORG_2_PATH_TO_KEYSTORE="Admin@customer.ufo.com/msp/keystore/"

UPDATED_KEYSTORE_ORG_1="$ORG_1_PATH_TO_KEYSTORE$ORG_1_KEYSTORE"
UPDATED_KEYSTORE_ORG_2="$ORG_2_PATH_TO_KEYSTORE$ORG_2_KEYSTORE"

echo $UPDATED_KEYSTORE_ORG_1

# sed -i "s|keystore/.*|${UPDATED_KEYSTORE}|g" connection.yaml
# .* is regex-ese for "any character followed by zero or more of any character(s)"

echo 'updating connection.yaml Sales adminPrivateKey path with' ${UPDATED_KEYSTORE_ORG_1}

sed -i -e "s|Admin@sales.ufo.com/msp/keystore/.*|$UPDATED_KEYSTORE_ORG_1|g" connection.yaml

echo 'updating connection.yaml Customer adminPrivateKey path with' ${UPDATED_KEYSTORE_ORG_2}

sed -i -e "s|Admin@customer.ufo.com/msp/keystore/.*|$UPDATED_KEYSTORE_ORG_2|g" connection.yaml
```



그 후에 node enrollAdmin.js 를 하면, 지갑이 생성되었다는 log과 함께 wallet/admin이라는 폴더가 생성이 된다.