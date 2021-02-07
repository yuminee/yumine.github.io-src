---
title: UFO Project 4 - Hyperledger Fabric Chain Code implementation in Node js (1)
date: 2020-12-01 15:45:26
tag: [블록체인, raft, blockchain,hyperledger fabric, 하이퍼레저 패브릭,ChainCode, ChainCode in node js]
category: [hyperledger fabric]
---

지난 포스트에서 Node JS로 체인코드를 작성하는 방법에 대해 공부 했다. 그럼 이제 진짜 체인코드를 작성할 시간이다.

---

### Directory Structure for chain code implementation

```
cd UFO_Fabric_raft #작업 디렉터리로 이동
mkdir chaincode/ufo/javascript #체인코드 작업할 폴더 만들기
cd chaincode/ufo/javascript # 체인코드 작업 폴더로 이동

```



### Package.json

먼저, `npm init`을 통하여 package.json을 만들어 준다. 

- package.json

```json
{
  "name": "ufo",
  "version": "1.0.0",
  "description": "ufo chaincode in node js",
  "main": "index.js",
  "scripts": {
    "start": "fabric-chaincode-node start"
  },
  "dependencies": {
    "fabric-contract-api": "~1.4.0",
    "fabric-shim": "~1.4.0"
  },
  "author": "",
  "license": "ISC"
}

```

먼저 여기서 

- **"dependencies" : { "fabric-contract-api",  "fabric-shim" }**

- **"main" : "index.js"** 

- **"scripts" : { start:"fabric-chaincode-node start}** 

  위 세가지를 추가해준다.

### index.js

- index.js

```js
'use strict';


const Ufo = require('./ufo');

module.exports.Ufo = Ufo;
module.exports.contracts = [ Ufo ];
```

위와 같은 방식으로 index.js를 작성해준다.



### npm install

위의 두개가 작성이 되었으면 package.json이 있는 폴더안에서 `npm install`을 진행한다. 



### ufo.js

이제 본격적으로 ufo.js 라는 js파일을 만들어서 체인코드를 작성을 해볼 것이다. 지금 진행하고 있는 프로젝트는 UFO(Uni Festival in One)이라고 하여 전국 대학 축제를 소개하고, 하이퍼레저를 이용하여 결제 장부를 저장한다. 

그래서 이 체인코드에서 필요한 함수들은,

- Init() : 체인코드 instantiate을 위해 넣어주었다. Transaction Id 를 console.info로 보여준다.

- initWallet(id) : 사용자(id)가 가입을 하면 지갑을 생성하고,  balance를 0으로 초기화 해준다.

- chargeMoney(id, amount) : 가입한 사용자의 id에 amount만큼의 돈을 충전한다.

- getBalance(id) : 사용자(id)로 현재 잔액을 조회한다.

- transferMoney(sender, receiver, amount) : sender는 receiver에게 amount만큼의 금액을 보낸다. 

- deleteWallet(id) : id로 지갑정보를 지운다(회원 탈퇴)

- (작성중)changeOrg(파라메터 미정) : 이 프로젝트에는 sales와 customer 두개의 조직이 있는데,

  - sales :  축제에서 판매를 하는 조직(노점, 주점등)
  - customer : 그 이외에 축제를 즐기는 모든 사람들

  이 어플리케이션에서는 A라는 축제에서는 sales였다가, B라는 축제에서는 customer일 경우인 사람이 있다. 이럴 경우 Org를 바꿔줘야 하기 때문에 추가했다.

- (작성중)getWallet(id) : 사용자 id로 계좌의 내역을 확인한다.



먼저 ufo.js의 require은,

```js
'use strict';

const shim = require('fabric-shim');
const { Contract } = require('fabric-contract-api');
```



class 는 아래와 같이 작성한다.

```js
class Ufo extends Contract{
...
}
```



#### Init()

```js
async Init(ctx) {
        console.info('=========== Instantiated Chaincode ===========');
        console.info('Transaction ID: ' + ctx.stub.getTxID());
        return shim.success();
    }
```



	#### initWallet(id)

```js
 async initWallet(ctx, id){

        console.info('========== START : initWallet ===========')
        
        //Declare wallet
        let wallet = {

            ID:id,
            Token:"0",
           }
        
        await ctx.stub.putState(id,Buffer.from(JSON.stringify(wallet)));
        console.log(wallet);
        console.info('========== END : initWallet ===========')
        
       }
```

json 형식으로 wallet을 선언했다. 선언한 wallet에는 ID,Token 이 들어가있고 

인자로 들어오는 id를 ID 값에, Token 은 0으로 초기화 시켜준다. 



#### chargeMoney(id, amount) 

```js
async chargeMoney(ctx, id, amount){

        console.info('========== START : chargeMoney ===========')

        let charger = id
        const walletAsBytes = await ctx.stub.getState(charger);

        if (!walletAsBytes || walletAsBytes.toString().length <= 0) {
            throw new Error(`${id} does not exist`);
        }
        
        let walletInfo = {};
        walletInfo = JSON.parse(walletAsBytes.toString());
        console.log(walletInfo)

        let walletToken = parseInt(walletInfo.Token);
        amount = parseInt(amount);

        walletInfo.Token = (walletToken + amount).toString();
        
        let updatedWalletAsBytes = JSON.stringify(walletInfo);

        await ctx.stub.putState(charger, Buffer.from(updatedWalletAsBytes));
        

        console.info('========== END : chargeMoney ===========')

    }
```



#### getBalance(id)

```js
async getBalance(ctx, id){

        console.info('========== START : getBalance ===========')

        let walletId = id;

        const walletAsBytes = await ctx.stub.getState(walletId);

        if (!walletAsBytes || walletAsBytes.toString().length <= 0) {
            throw new Error(`${id} does not exist`);
        }

        console.log(walletAsBytes.toString());
        walletInfo = JSON.parse(walletAsBytes.toString());
        walletToken = walletInfo.Token;
        console.info('========== END : getBalance ===========')
        return walletToken;
    }
```



#### transferMoney(sender, receiver, amount) 

```js
 async transferMoney(ctx, senderId, receiverId, amount){
       
        console.info('========== START : transferMoney ===========')
        
        let sender = senderId
        let receiver = receiverId

        console.log("Sender :" + sender + ", Receiver :" + receiver + ", amount : " + amount);

       
        let receiverAsBytes = await ctx.stub.getState(receiver)
        if (!receiverAsBytes || receiverAsBytes.toString().length <= 0) {
            throw new Error(`${receiver} does not exist`);
        }

        let receiverInfo = {};
        receiverInfo = JSON.parse(receiverAsBytes.toString());
        let receiverToken = parseInt(receiverInfo.Token)
            

        let senderAsBytes = await ctx.stub.getState(sender)
        if (!senderAsBytes || senderAsBytes.toString().length <= 0) {
            throw new Error(`${senderId} does not exist`);
        }

        let senderInfo = {};
        senderInfo = JSON.parse(senderAsBytes.toString());
        let senderToken = parseInt(senderInfo.Token)
        
        amount = parseInt(amount)
        if (amount < 0) {
            return shim.Error("you can't transfer less than 0")
        }
        
        if (amount > senderToken) {
            return shim.Error(`${senderId} doesn't have enough money to send`)
        }
        
        receiverInfo.Token = (receiverToken + amount).toString();
        senderInfo.Token = (senderToken - amount).toString();
    
        var updatedSenderAsBytes = JSON.stringify(senderInfo);
        var updatedReceiverAsBytes = JSON.stringify(receiverInfo);
    
        await ctx.stub.putState(sender, Buffer.from(updatedSenderAsBytes));
        await ctx.stub.putState(receiver,Buffer.from(updatedReceiverAsBytes));
       
    }
```



#### deleteWallet(id)

```js
async deleteWallet(ctx, id){

        console.info('========== START : deleteWallet ===========')


        await ctx.stub.deleteState(id); 
   
        console.info('========== END : deleteWallet ===========')


    }
```



마지막으로,

```js
module.exports = Ufo;
```

해주면 된다.



### Package ChainCode

```
docker exec cli peer chaincode package -n ufo -l node -p /opt/gopath/src/github.com/chaincode/ufo/javascript -v 0 -s -S ccpack.out
```

위의 명령어를 통해서 체인코드를 패키징 해준다.



### ChainCode install

우리는 총 2개의 조직에 각각 2개씩 4개의 피어가 있으므로 각각의 피어에 체인코드를 install 해준다.



- peer0.sales.ufo.com:7051

  ```
  docker exec -e CORE_PEER_LOCALMSPID=SalesMSP -e CORE_PEER_ADDRESS=peer0.sales.ufo.com:7051 -e CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/sales.ufo.com/users/Admin@sales.ufo.com/msp -e CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/sales.ufo.com/peers/peer0.sales.ufo.com/tls/ca.crt cli peer chaincode install ccpack.out -l node
  
  ```

  

- peer1.sales.ufo.com:8051

  ```
  docker exec -e CORE_PEER_LOCALMSPID=SalesMSP -e CORE_PEER_ADDRESS=peer1.sales.ufo.com:8051 -e CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/sales.ufo.com/users/Admin@sales.ufo.com/msp -e CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/sales.ufo.com/peers/peer0.sales.ufo.com/tls/ca.crt cli peer chaincode install ccpack.out -l node
  
  ```



- peer0.customer.ufo.com:9051

  ```
  docker exec -e CORE_PEER_LOCALMSPID=CustomerMSP -e CORE_PEER_ADDRESS=peer0.customer.ufo.com:9051 -e CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/customer.ufo.com/users/Admin@customer.ufo.com/msp -e CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/customer.ufo.com/peers/peer0.customer.ufo.com/tls/ca.crt cli peer chaincode install ccpack.out -l node
  
  ```



- peer1.customer.ufo.com:10051

  ```
  docker exec -e CORE_PEER_LOCALMSPID=CustomerMSP -e CORE_PEER_ADDRESS=peer1.customer.ufo.com:10051 -e CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/customer.ufo.com/users/Admin@customer.ufo.com/msp -e CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/customer.ufo.com/peers/peer0.customer.ufo.com/tls/ca.crt cli peer chaincode install ccpack.out -l node
  
  ```



### ChainCode instantiate

```
docker exec -e CORE_PEER_LOCALMSPID=SalesMSP -e CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/sales.ufo.com/users/Admin@sales.ufo.com/msp cli peer chaincode instantiate -o orderer.ufo.com:7050 -C ufochannel -n ufo -l node -v 0 -c '{"Args":[]}' -P 'AND('\''SalesMSP.member'\'','\''CustomerMSP.member'\'')' --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/ufo.com/orderers/orderer.ufo.com/msp/tlscacerts/tlsca.ufo.com-cert.pem 

```



이렇게 체인코드를 작성하고, 설치, 인스턴스화까지 했다. 

### Invoke ChainCode

- initWallet

  ```
  docker exec -e CORE_PEER_LOCALMSPID=SalesMSP -e CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/sales.ufo.com/users/Admin@sales.ufo.com/msp cli peer chaincode invoke -o orderer.ufo.com:7050 -C ufochannel -n ufo -c '{"function":"initWallet","Args":["2015116581"]}' --waitForEvent --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/ufo.com/orderers/orderer.ufo.com/msp/tlscacerts/tlsca.ufo.com-cert.pem --peerAddresses peer0.sales.ufo.com:7051 --peerAddresses peer1.sales.ufo.com:8051 --peerAddresses peer0.customer.ufo.com:9051 --peerAddresses peer1.customer.ufo.com:10051 --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/sales.ufo.com/peers/peer0.sales.ufo.com/tls/ca.crt --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/sales.ufo.com/peers/peer0.sales.ufo.com/tls/ca.crt --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/customer.ufo.com/peers/peer0.customer.ufo.com/tls/ca.crt --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/customer.ufo.com/peers/peer0.customer.ufo.com/tls/ca.crt
  
  ```

  위에서 function에 initWallet를 Arg에 id 값을 넣어준다.

  **'{"function":"initWallet","Args":["2015116581"]}'**

  - Couchdb 화면

  ![image](https://user-images.githubusercontent.com/33755241/101023683-751bac80-35b6-11eb-837a-9b125b1f0b56.png)

- chargeMoney

  ```
  docker exec -e CORE_PEER_LOCALMSPID=SalesMSP -e CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/sales.ufo.com/users/Admin@sales.ufo.com/msp cli peer chaincode invoke -o orderer.ufo.com:7050 -C ufochannel -n ufo -c '{"Args":["chargeMoney","2015116581","5000"]}' --waitForEvent --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/ufo.com/orderers/orderer.ufo.com/msp/tlscacerts/tlsca.ufo.com-cert.pem --peerAddresses peer0.sales.ufo.com:7051 --peerAddresses peer1.sales.ufo.com:8051 --peerAddresses peer0.customer.ufo.com:9051 --peerAddresses peer1.customer.ufo.com:10051 --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/sales.ufo.com/peers/peer0.sales.ufo.com/tls/ca.crt --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/sales.ufo.com/peers/peer0.sales.ufo.com/tls/ca.crt --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/customer.ufo.com/peers/peer0.customer.ufo.com/tls/ca.crt --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/customer.ufo.com/peers/peer0.customer.ufo.com/tls/ca.crt
  
  ```

  **'{"Args":["chargeMoney","2015116581","5000"]}'**

  방금 가입한 "2015116581" 이라는 id에 "5000"만큼 충전한다.

  - couchDB : Token이 0에서 5000으로 증가했다.

    ![image](https://user-images.githubusercontent.com/33755241/101023874-b3b16700-35b6-11eb-941a-d4b3e77df735.png)

- transferMoney

  ```
  docker exec -e CORE_PEER_LOCALMSPID=SalesMSP -e CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/sales.ufo.com/users/Admin@sales.ufo.com/msp cli peer chaincode invoke -o orderer.ufo.com:7050 -C ufochannel -n ufo -c '{"Args":["transferMoney","2015116581","20131111","1000"]}' --waitForEvent --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/ufo.com/orderers/orderer.ufo.com/msp/tlscacerts/tlsca.ufo.com-cert.pem --peerAddresses peer0.sales.ufo.com:7051 --peerAddresses peer1.sales.ufo.com:8051 --peerAddresses peer0.customer.ufo.com:9051 --peerAddresses peer1.customer.ufo.com:10051 --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/sales.ufo.com/peers/peer0.sales.ufo.com/tls/ca.crt --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/sales.ufo.com/peers/peer0.sales.ufo.com/tls/ca.crt --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/customer.ufo.com/peers/peer0.customer.ufo.com/tls/ca.crt --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/customer.ufo.com/peers/peer0.customer.ufo.com/tls/ca.crt
  
  ```

  **'{"Args":["transferMoney","2015116581","20131111","1000"]}'**

  아까 가입한 "2015116581" 에 추가적으로 "20131111" 이라는 id를 initWallet 해주었고

  "20131111"에게 "2015116581" 이 "1000" 만큼 보낸다.

  - CouchDB : 5000이 있었던 "2015116581" 은 1000을 보내서 4000이 되었고, 

    "20131111" 은 1000을 받아서 Token이 1000이 되었다.

  ![image](https://user-images.githubusercontent.com/33755241/101024091-fa06c600-35b6-11eb-9b92-3f75b303abf9.png)

