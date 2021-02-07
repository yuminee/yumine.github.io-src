---
title: Build Blockchain with TypeScript 1
date: 2020-12-30 15:00:01
tags: [typescript, Blockchain, 블록체인]
category: Blockchain
mathjax: true
---

타입스크립트로 블록체인을 간단하게 만들어보기 위해서 타입스크립트의 기초 문법을 공부했다.

아래 명령어로 typescript와 tsc-watch를 설치해준다.

```
yarn global add typescript
yarn add tsc-watch --dev //nodem.
```

```
|-- dist/
|	-- index.js //(컴파일 후 생김)
|	-- index.js.map //(컴파일 후 생김)
|-- src/
|	-- index.ts
|-- package.json
|-- tsconfig.json
```



### 1. yarn init

자바스크립트의 프로젝트와 마찬가지로 yarn init(npm init)등을 이용해서 package.json을 만들어준다.  



### 2. tsconfig.json

작업 디렉토리에서 타입스크립트 설정을 위해 tsconfig.json이라는 파일을 만들어주고,

```json
{
    "compilerOptions": {
        "module": "commonjs",
        "target": "ES2015",
        "sourceMap": true,
        "outDir": "dist"
    },
    "include": [
        "src/**/*"
    ],
    "exclude": [
        "node_modules"
    ]
}
```

먼저 `compilerOptions` 부분은 컴파일로 옵션이다.  

[컴파일러 옵션]: https://typescript-kr.github.io/pages/compiler-options.html

**module** : 묘듈 코드 생성 지정.

**target** : ECMAScript 대상 버전 지정. 기본값은 `ES3`

**sourceMap** : 해당하는 .map파일을 생성할지 여부. true니까 .map파일을 생성한다.

**outDir** : 출력 구조를 디렉토리로 리다이렉트 함. 해당 프로젝트에서는 dist안에 출력된 .map 과 js 파일이 생성됨.

**include** : src 파일아래에서 작업을 할 것 이고, 그 밑으로 전부 컴파일.

**exclude** : 해당 프로젝트에서는 node_modules를 사용하지 않을 것 이므로, 제외함.



### 3. package.json 수정

```json
{
  "name": "typeChain",
  "version": "1.0.0",
  "main": "index.js",
  "repository": "https://github.com/yuminee/typeChain.git",
  "author": "yuminee <ayuminee2@gmail.com>",
  "license": "MIT",
  "scripts": {
    "start": "tsc-watch --onSuccess \"node dist/index.js\" "
  },
  "devDependencies": {
    "tsc-watch": "^4.2.9"
  },
  "dependencies": {
    "typescript": "^4.1.3"
  }
}

```

위의 script 부분을 주목하자.   앞으로는 `yarn start`를 실행하면 수정되는 상황이 바로바로

컴파일 되어 결과를 확인 할 수 있다. 

만약 tsc 관련해서 오류가 난다면, `yarn add typescript`로 설치를 하고 실행하면 될 것 이다.



### 4. index.ts

기초 문법을 학습하기 위해 index.ts를 생성했다. js를 할 줄 안다는 가정하에 typescript를 간단하게만 설명할 예정이다.



- alert("aaa")

  이런식으로 프린트문을 작성한다.

- class를 만들 수 있다.

  - Human 이라는 클래스를 만든다면,

    ```typescript
    class Human {
        
       public name :string;
       public age : number;
       public gender : string;
        
       constructor(name: string, age:number, gender:string){
          this.name = name;
          this.age = age;
          this.gender = gender;
       } 
    }
    ```

    class안의 접근 지정자를 설정해 줄 수 있다. public으로 선언하면 클래스 밖에서도 접근이 가능하지만 만약에 private으로 선언하면 class안에서만 접근이 가능 할 것 이다.

    또한 타입스크립트라는 이름에 맞게, 타입도 설정해 줄 수 있다.

    constructor 부분은 생성자다.

  - class를 기반으로 객체 생성

    ```typescript
    const youmin = new Human("youmin", 26,"famale")
    ```

    이런식으로 Human 클래스의 객체를 생성한다. 만약 저기서 파라메타들의 타입이 틀리거나 파라메타의 갯수가 다르다면 에러가 난다.

  - 함수로 만들어서 사용해보기!

    ```typescript
    const sayHi = (person :Human):string => {
       return `Hello ${person.name}, you are ${person.age} a ${person.gender}`
    }
    
    ```

    sayHi라는 함수를 만들었다. Human 클래스의 타입인 person을 인자로 받고, string을 리턴하는 모습이다. 

  - (다른 상황) 인자를 받거나 안받거나!

    ```typescript
    const sayHi = (name, age, gender?):string => {
    
    return `Hello ${person.name}, you are ${person.age} a ${person.gender}`
    
    }
    ```

    위의 함수에서는 gender뒤에 ?가 붙은것을 볼 수 있다. 정말 좋은 기능중에 하나라고 생각이 드는데 ?를 뒤에 붙이면 인자로 받지 않아도 된다.

- export{}

  마지막에 export{}  를 해서 내보내야한다.



### 5. 전체 코드

```typescript
class Human {
   public name :string;
   public age : number;
   public gender : string;
   constructor(name: string, age:number, gender:string){
      this.name = name;
      this.age = age;
      this.gender = gender;
   } 
}

const youmin = new Human("youmin", 26,"famale")


const sayHi = (person :Human):string => {
   return `Hello ${person.name}, you are ${person.age} a ${person.gender}`
}

console.log(sayHi(youmin));

export {};

```



다음 포스트에서는 타입스크립트를 이용하여 블록체인을 만들어보겠다.