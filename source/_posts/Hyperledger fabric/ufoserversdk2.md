---
title: UFO Project 6 - Hyperledger Fabric SDK + application (2)
date: 2020-12-03 15:45:26
tag: [블록체인, raft, blockchain,hyperledger fabric, 하이퍼레저 패브릭, fabric-client, fabric sdk]
category: [hyperledger fabric]
---
# UFO PROJECT 6 - Hyperledger Fabric SDK + application (2)

체인코드 각각의 함수들을 불러 올 수 있게 app.js를 먼저 작성한다.

```js
const express = require('express')
const bodyParser = require('body-parser')
const cors = require('cors')
const morgan = require('morgan')

var network = require('./fabric/network.js');
const { response } = require('express');

const app = express()
app.use(morgan('combined'))
app.use(bodyParser.json())
app.use(cors())

const PORT = 8080;
const HOST = 'localhost';

```

기본적으로 express로 port는 8080으로 열어서 테스트를 할 것 이다.

- app.js 의 initWallet

```js
app.post('/initWallet', (req, res) => {
  
    network.initWallet(req.body.id)
    .then((response) => {
      res.send(response)
    })
  })  
```

이렇게 initWallet를 app.js에서 불러오면, 

- network.js 의 initWallet 함수를 불러온다. 모바일 서버에서 id 값을 받으면 그 id 값에 대해 initWallet를 하는 것이므로 인자는 id 하나이다.

  ```js
  exports.initWallet = async function(id) {
  
      try {
  
          var response = {};
  
          // Create a new file system based wallet for managing identities.
          const walletPath = path.join(process.cwd(), '/wallet');
          const wallet = new FileSystemWallet(walletPath);
          console.log(`Wallet path: ${walletPath}`);
  
          // Check to see if we've already enrolled the user.
          const userExists = await wallet.exists(userName);
          if (!userExists) {
              console.log('An identity for the user ' + userName + ' does not exist in the wallet');
              console.log('Run the registerUser.js application before retrying');
              response.error = 'An identity for the user ' + userName + ' does not exist in the wallet. Register ' + userName + ' first';
              return response;
          }
  
          // Create a new gateway for connecting to our peer node.
          console.log('we here in initWallet')
  
          const gateway = new Gateway();
          await gateway.connect(connectionFile, { wallet, identity: userName, discovery: gatewayDiscovery });
  
          // Get the network (channel) our contract is deployed to.
          const network = await gateway.getNetwork('ufochannel');
  
          // Get the contract from the network.
          const contract = network.getContract('ufo');
  
          // Submit the specified transaction.
          // initWallet transaction - requires 1 argument, ex: ('initWallet', '11111111')
  
          await contract.submitTransaction('initWallet', id);
          console.log('Transaction has been submitted');
  
          // Disconnect from the gateway.
          await gateway.disconnect();
  
          response.msg = 'initWallet Transaction has been submitted';
          return response;        
  
      } catch (error) {
          console.error(`Failed to submit transaction: ${error}`);
          response.error = error.message;
          return response; 
      }
  }
  ```

  다른 chaincode의 함수들도 비슷한 방식으로 불러올 수 있다.

  <br>

  <br>

  - app.js 의 getHistoryWallet이다.

  ```js
  app.post('/getHistoryWallet', (req,res) => {
    network.getHistoryWallet(req.body.id)
    .then((response) => {
      var walltHistory = JSON.parse(response);        
      res.send(Buffer.from(walltHistory).toString())
    
    
    })
  
  })
  ```

  

  - network.js의 getHistoryWallet 이다.

  ```js
  exports.getHistoryWallet = async function(id) {
  
      try {
  
          var response = {};
  
          // Create a new file system based wallet for managing identities.
          const walletPath = path.join(process.cwd(), '/wallet');
          const wallet = new FileSystemWallet(walletPath);
          console.log(`Wallet path: ${walletPath}`);
  
          // Check to see if we've already enrolled the user.
          const userExists = await wallet.exists(userName);
          if (!userExists) {
              console.log('An identity for the user ' + userName + ' does not exist in the wallet');
              console.log('Run the registerUser.js application before retrying');
              response.error = 'An identity for the user ' + userName + ' does not exist in the wallet. Register ' + userName + ' first';
              return response;
          }
  
          // Create a new gateway for connecting to our peer node.
          console.log('we here in getHistoryWallet')
  
          const gateway = new Gateway();
          await gateway.connect(connectionFile, { wallet, identity: userName, discovery: gatewayDiscovery });
  
          // Get the network (channel) our contract is deployed to.
          const network = await gateway.getNetwork('ufochannel');
  
          // Get the contract from the network.
          const contract = network.getContract('ufo');
  
          // Submit the specified transaction.
          // initWallet transaction - requires 1 argument, ex: ('initWallet', '11111111')
  
          const result = await contract.evaluateTransaction('getHistory', id);
         
  
          // Disconnect from the gateway.
          await gateway.disconnect();
       
          console.log( Buffer.from(result).toString());
        
          return result
  
      } catch (error) {
          console.error(`Failed to submit transaction: ${error}`);
          response.error = error.message;
          return response; 
      }
  ```

  이 함수는 다른 initWallet, deleteWallet 등등과의 차이점이 있는데 result를 return 한다는 것이다. 체인코드에서 데이터를  buffer로 받아오는데 단순히 데이터를 parse 하니 읽을 수 없는 형태로 데이터가 반환되었다.

  그래서 찾은것이

  **Buffer.from(result).toString()** 이다.  

  전체 코드는 [GIT](https://github.com/yuminee/UFO_FabricNet_raft/tree/main/wep-app/server)에서 확인 할 수 있다.

  

---

모바일에서 보기 편하게 md 파일을 작성 했다.

  

## HFB Server

### ChainCode invoke 

#### Json key:value 형식

- initWallet

  - POST /initWallet 

  | key  | value  | Description                                        |
  | ---- | ------ | -------------------------------------------------- |
  | id   | String | 받은 id 값으로 Token을 0으로 초기화 하여 지갑 생성 |



- getBalance

  - POST /getBalance

    | key  | value  | Description               |
    | ---- | ------ | ------------------------- |
    | id   | String | 받은 id 값의 Token return |



- deleteWallet

  - POST /deleteWallet

    | key  | value  | Description             |
    | ---- | ------ | ----------------------- |
    | id   | String | 받은 id값의 wallet 지움 |

  

- chargeMoney

  - POST /chargeMoney

    | key    | value  | Description        |
    | ------ | ------ | ------------------ |
    | id     | String | 충전할 Wallet의 id |
    | amount | String | 충전할 금액        |

  



- transferMoney

  - POST /transferMoney

    | key      | value  | Description                     |
    | -------- | ------ | ------------------------------- |
    | sender   | String | 보내는 wallet Id                |
    | receiver | String | 받은 wallet Id                  |
    | amount   | String | sender 가 보내는 Token의 amount |

    

