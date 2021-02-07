---
title: Build Blockchain with TypeScript 2
date: 2020-12-31 15:00:01
tags: [typescript, Blockchain, 블록체인]
category: Blockchain
---

# Build Blockchain with TypeScript 2

이제 본격적으로 blockchain을 만들어 보자!



### 1. Blockchain 구조

Blockchain의 구조는 기본적으로,

- index
- previous hash
- hash
- timestamp
- data

이렇게 5가지를 놓을 수 있다.



```typescript
class Block {
   public index : number;
   public hash : string;
   public previoushash :string;
   public data :string;
   public timestamp : number;
   constructor(
      index:number,
      hash:string,
      previoushash:string,
      data:string,
      timestamp:number 
      ){
         this.index =index;
         this.hash=hash;
         this.previoushash=previoushash;
         this.data = data;
         this.timestamp=timestamp     
   }
}
```

그것을 바탕으로 Block class와 생성자를 만들어주었다.



### 2. gensisBlock 생성

블록체인에서 가장 첫번째 블럭인 `genesisBlock` 이라고 한다. 여기서는 하드코딩으로 제네시스블럭을 만들어줄려고 한다. 

```typescript
const gensisBlock:Block = new Block(0, "202020202020", "", "this is a gensisBlock", 123456);

```

들어가는 인자는 `class Block`의 생성자의 순서대로 `index, hash, previoushash, data,timestamp`순이다.

```typescript
let blockchain:Block[] = [gensisBlock]

```

blockchian 이라는 이름의 변수를 Block 클래스의 배열타입으로 선언해주면서, 아까 만들어두었던 gensisBlock을 첫번째로 넣어준다.

![image](https://user-images.githubusercontent.com/33755241/103353357-fb5edc00-4aeb-11eb-8dac-24636176a071.png)

만든 블럭을 console.log로 찍어보면 이렇게 나온다!



### 3. Hash 값 계산!

블록체인의 블럭은 직전 블록의 해쉬값과, 현재 블록의 정보로 만들어진 해쉬값이 존재한다.

나는 블록의 해쉬값을 계산하는 함수를 static으로 block 클래스 안에 만들어주었다.



먼저 SHA256을 계산하기 위해

```
yarn add crypto-js
```

로 설치를 해준후에, 

```
import * as CryptoJS from "crypto-js"
```

파일의 제일 윗 부분에 crypto-js를 import 해준다.

그리고 난뒤에 아래 함수처럼 hash값을 계산한다.

```typescript
   static calculateBlockHash = (
      index:number,
      previoushash:string,
      timestamp:number,
      data:string
   ):string=>
      CryptoJS.SHA256(index + previoushash + timestamp + data).toString();

```



### 4. 새로운 블록 생성하기

새로운 블록을 생성하기 위해서는 직전 블록의 정보가 필요하다.

그래서 직전 블록을 얻어올 수 있는 함수를 먼저 하나 만들고, 타임스탬프를 찍어주는 함수도 만들것이다.

아래와 같다.

```
const getLatesBlock = () :Block => blockchain[blockchain.length-1];
//진전 블록의 정보를 얻어온다. 
```

```
const getNewTimeStamp = ():number => Math.round(new Date().getTime()/1000);
//타임스탬프값을 계산한다.
```

그리고 블록을 생성하는 함수를 작성한다.

```typescript
const createNewBlock = (data:string) : Block => {
   const previousBlock : Block = getLatesBlock();
   const newIndex : number = previousBlock.index + 1;
   const newTimestamp : number = getNewTimeStamp();
   const newtHash : string = Block.calculateBlockHash(
      newIndex,
      previousBlock.hash,
      newTimestamp,
      data
      );
   const newBlock : Block = new Block(
      newIndex,
      newtHash,
      previousBlock.hash,
      data,
      newTimestamp
      );
   
   return newBlock;
};
```

getLatesBlock()을 이용하여 직전블록을 가져오고, 그 정보를 토대로 인덱스값을 증가시킨다.

getNewTimeStamp()를 이용하여 타임스탬프를 찍고 

calculateBlockHash( newIndex, previousBlock.hash,newTimestamp,data)를 이용하여 hash값을 계산한다.



### 5. 블록의 인증

블록을 생성했다고해서 그냥 바로 체인에 넣을 수는 없다. 이 블록이 맞는 블록인지 확인을 해야한다.

그것을 위해 isBlockVaild()라는 함수를 만들었다. 

```typescript
const isBlockValid = (candidateBlock : Block, previousBlock : Block):boolean => {
   if(!Block.vaildBlockStructure(candidateBlock)){
      return false
   } else if(previousBlock.index + 1 !== candidateBlock.index){
      return false
   } else if(previousBlock.hash !== candidateBlock.previoushash){
      return false
   } else if(getHashForBlock(candidateBlock) !== candidateBlock.hash ){
      return false
   } else{
      return true;
   }
};
```

위의 함수는 블럭 클래스의 candidateBlock과 preivousBlock 두개를 인자로 받는다. 그리고 boolean값을 리턴한다.

체크하는 부분은,

1. 해당 함수가 블럭의 구조를 가지고 있는가?
2. 직전 블럭의 인덱스보다 +1 된 값의 인덱스를 가지고 있는가?
3. 후보 블럭의 직전해쉬값은 직전블록의 해쉬값과 같은가?
4. 후보 블럭의 해쉬값은 후보 블럭의 데이터로 해쉬를 계산했을때 같은 값인가?

이렇게 4가지를 체크한다.

1번 부분을 체크하기 위해서

```typescript
static vaildBlockStructure = (aBlock:Block) : boolean =>
    typeof aBlock.index === "number" && 
    typeof aBlock.hash ==="string" && 
    typeof aBlock.previoushash === "string" && 
    typeof aBlock.timestamp === "number" &&
    typeof aBlock.data === "string";
   constructor(
      index:number,
      hash:string,
      previoushash:string,
      data:string,
      timestamp:number 
      ){

         this.index =index;
         this.hash=hash;
         this.previoushash=previoushash;
         this.data = data;
         this.timestamp=timestamp     
   }
```

해당 함수를 Block class안에 만들어주었다. 

3번 부분을 체크하기 위해서

```typescript
const getHashForBlock = (aBlock:Block) :string =>
 Block.calculateBlockHash(
    aBlock.index, 
    aBlock.previoushash, 
    aBlock.timestamp, 
    aBlock.data)
```

해당 함수를 만들어주었다.



### 6. Add Block!

블록 인증을 마치고나서는 체인에 블록을 추가해야한다.

```typescript
const addBlock = (candiateBlock : Block) : void=> {
   if(isBlockValid(candiateBlock, getLatesBlock())){
      blockchain.push(candiateBlock);
   }
};
```

위의 함수로 블록을 추가하고 위의 함수는,

createNewBlock(data:string):Block에 아래와 같이 추가한다.

```typescript
const createNewBlock = (data:string) : Block => {
   const previousBlock : Block = getLatesBlock();
   const newIndex : number = previousBlock.index + 1;
   const newTimestamp : number = getNewTimeStamp();
   const newtHash : string = Block.calculateBlockHash(
      newIndex,
      previousBlock.hash,
      newTimestamp,
      data
      );
   const newBlock : Block = new Block(
      newIndex,
      newtHash,
      previousBlock.hash,
      data,
      newTimestamp
      );
   
   addBlock(newBlock);
   return newBlock;
};
```



### 7. 블록 만들고 console.log!

```typescript
createNewBlock("second block");
createNewBlock("third block");
createNewBlock("fourth block");
```



그 다음 `console.log(bolckchain)` 으로 결과를 확인하면, 

![image](https://user-images.githubusercontent.com/33755241/103410544-1cd4cc00-4baf-11eb-93d7-f9dc7f5223f4.png)

다음과 같은 결과를 확인 할 수 있다!

[전체 코드 링크]: https://github.com/yuminee/typeChain

