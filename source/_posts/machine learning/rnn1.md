---
title: RNN -  순환 신경망(Recurrent Neural Network) ①
date: 2020-11-21 23:05:36
tags: [rnn, Recurrent Neural, 머신러닝, 알고리즘, 데이터 사이언스, 자연어 처리]
category: [머신러닝]
---
 <br/>

#  RNN -  순환 신경망(Recurrent Neural Network) ①

[1. 순서(sequence)가 있는 데이터](#순서-sequence-가-있는-데이터)

[2. 시간 개념을 포함한 RNN 구조](#시간-개념을-포함한-RNN-구조)

[3. RNN 동작 원리](#RNN-동작-원리)

- 내부적으로 순환(Recurrent) 되는 구조를 이용하여,
- 순서(Sequence)가 있는 데이터를 처리하는 데 강점을 가진 신경망

   <br/>
  
![rnn](https://user-images.githubusercontent.com/33755241/99877365-01d27c00-2c41-11eb-936b-352da70056b2.png)

### 순서(sequence)가 있는 데이터

- 문장이나 음성 같은 연속적인 데이터. 이런 데이터는 문장에서 놓여진 위치(순서)에 따라 의미가 달라지는 것을 알 수 있음.
- 즉,  현재 데이터 의미를 알기 위해서는 이전에 놓여 았는 과거 데이터도 알고 있어야 함.
- 그래서 RNN은 이러한 과거의 데이터를 알기 위해서 **1. 은닉층내에 순환(Recurrent)구조를 이용하여 과거의 데이터를 기억** 해두고 있다가 **2. 새롭게 읩력으로 주어지는 데이터와 은낙층에서 기억하고 있는 과거 데이터를 연결 시켜서 그 의미를 알아내는** 기능을 가지고 있음.


   <br/>

### 시간 개념을 포함한 RNN 구조

![image](https://user-images.githubusercontent.com/33755241/99877694-3f380900-2c43-11eb-8d4b-d455a7a254cb.png)

- 순환 구조를 ① 은닉층에서 기억하는 과거의 데이터(붉은색 화살표)와 ② 일정 시간이 지난후에 입력되는 데이터와 연결되는 구조.
- 시간 경과에 따라서 데이터가 순차적으로 들어온다는 것과 같은 의미

  
 <br/>

### RNN 동작 원리

![image](https://user-images.githubusercontent.com/33755241/99878891-d5702d00-2c4b-11eb-88b4-9e19a3599c22.png)

- 바이어스 : 각 층마다 하나씩 
  - 은닉층 :  bh
  - 출력층  : bo
- 가중치
  - 은닉층 
    - 입력 데이터 A1의 가중치 = Wih
    - 내부적 순환 구조를 이용하여 기억되는 과거 데이터 H에 적용되는 가중치 = Whh
  - 출력층
    - 입력 데이터 A2에 적용되는 가중치 = Who

   <br/>


#### 첫번째 입력 데이터 A1에서의 RNN 동작 원리

Liner Regression ②  `A1(입력데이터) · Wih = Z2`

Summation ③ `Z2 + (H·Whh) + bn = R2`

Tanh ④ `tanh(R2) = A2` -> `next H = A2, current H = 0` 

###### A2 는 순환되어 저장되는 값, 첫번째니까 H값이 존재안함.

Liner Regression ⑤ `A2·Who = Z3` 

Out put ⑥ softmax(Z3·bo) = A3


 <br/>



#### 두번째 입력 데이터 A1에서의 RNN 동작 원리

Liner Regression ②  `A1(입력데이터) · Wih = Z2`

Summation ③ `Z2 + (H·Whh) + bn = R2`

Tanh ④ `tanh(R2) = A2` -> `next H = A2, current H = A2prev`

###### A2 는 순환되어 저장되는 값, 두번째니까 H값이 존재.

Liner Regression ⑤ `A2·Who = Z3` 

Out put ⑥ softmax(Z3·bo) = A3



 <br/>


즉,

> **시간 개념을 포함한  Current state Ht**
>
> **Ht = A2 = tanh(A1·Wih + Ht-1 ·Whh + bh)**

Ht : 현재 입력데이터 A1에 대한 state

A1:현재입력데이터

Ht-1 : 과거 입력 데이터 A1에 대한 state

bh : 은닉층 바이어스

