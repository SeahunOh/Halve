---
title: Why use setbuf() in C
description: pwnable 문제를 풀면서 느꼇던 setbuf()의 유무에 따른 메모리 영역의 변화에 대한 연구
categories:
 - kou
tags:
 - c
 - pwn
 - setbuf
---

<!-- more --> 

## Motivation

문제들 풀다보면 가끔 힙에 이상한 더미가 생성되어 공격을 방해했던 적이 있었으나, 그때는 어찌저찌 잘 넘어갔었다.

하지만 최근 pwnable.tw 문제를 풀다가 대체 왜 생기나 궁금해 지기 시작하였다. (with king god @hackability..)

~~웹 하는사람한테 포너블을..근데 포너블 꿀잼..~~

## TL;DR;

제목은 `왜 setbuf()를 사용하는가`라고  거창하게 써있긴 하지만, 사실 그렇게 거창하지 않을뿐더러 이미 다들 아는 내용일 것이다.

그래서 결론부터 말하자면 <u>**setbuf()**가 걸려있지 않은 상태</u><u>에서 **scanf()**를 사용하게 되면</u> **Heap영역에 데이터가 들어간다**.

자 그럼 이제 디버깅을 통해 삽질을 시작해보자.





작성중 - - - -