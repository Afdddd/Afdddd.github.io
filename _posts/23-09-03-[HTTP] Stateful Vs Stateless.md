--- 
layout: post 
categories: [HTTP]
tags: [http,stateful,stateless]
---

# [HTTP] Stateflu vs Stateless

---

<br><br><br>

## Stateful(상태유지)

---

<br>

> 서버가 클라이언트의 상태를 유지.

<br>

클라이언트의 정보를 서버의 세션(Session) 메모리에 저장한다.

대표적인 Stateful 구조는 TCP 프로토콜이 있다.

TCP 3 way hadsahake 과정을 보자

<br>

![출처:[](/assets/img/09-03-img/Untitled.png)



<br>

1. 클라이언트에서 접속요청(SYN)을 전송한다.
2. 서버에선 요청을 수락(ACK)하고 요청(SYN)을 다시 클라이언트로 전송한다.
3. 클라이언트에서 수락(ACK)을 보내면 서버의 상태는 ESTABLISHED  상태가 된다.
4. 이제부터 클라이언트와 서버간에 데이터가 송수신이 가능해진다.

<br>

이렇게 **상태**에 따라 서버의 응답이 달라지므로 TCP는 Stateful 구조를 가지고 있다.

<br><br>

### Stateful의 단점

---

<br>

![출처:[](/assets/img/09-03-img/Untitled1.png)



![출처:[](/assets/img/09-03-img/Untitled2.png)



<br>

- 클라이언트A가 서버1에 접속해 노트북2개를 주문하려고 장바구니에 담아둔 상태였다.
- 만약 서버1에서 장애가 나서 연결이 끊겨 서버2 와 연결한다면 서버2는 클라이언트A가 무엇을 주문할지 모른다. 클라이언트A의 상태는 서버1이 가지고 있기 때문이다.

<br>

이렇게 서버가 클라이언트의 상태를 유지한다면 서버 증설에 어려움이 있다.(클라이언트A는 서버1에만 고정되어있기 때문이다.)

<br><br><br>

## Stateless(무상태)

---

<br>

> 서버가 클라이언트의 상태를 유지하지 않음.

<br>

HTTP, 토큰 등 Stateless 구조를 가지고있다.

서버는 클라이언트의 상태를 유지하지 않으므로 단순히 요청이 온다면 응답만 해주면 될뿐이다.

서버와 클라이언트의 통신에 필요한 모든 정보는 클라이언트가 가지고있다가 통신할때 마다 매번 클라이언트는 가지고 있는 정보를 모두 서버로 전송한다.

<br>

![출처:[](/assets/img/09-03-img/Untitled3.png)


![출처:[](/assets/img/09-03-img/Untitled4.png)


<br>

클라이언트A가 요청할때마다 노트북, 2개 라는 정보를 매번 서버로 실어 보낸다.

그렇기 떄문에 서버1에서 장애가 발생해도 클라이언트A가 요청에 정보를 실어 서버2와 연결하면  문제없이 통신을 할 수 있다.

서버는 상태를 유지하지 않기 때문에 아무 서버와 연결해도 되고, 서버가 장애가 나도 다른서버와 연결하면 아무 문제없이 진행할수있다.평확장

<br><br>

### 수평확장

---

<br>

![출처:[](/assets/img/09-03-img/Untitled5.png)

<br>

갑자기 클라이언트 요청이 증가해도 서버를 대거 투입할수있다.

무상태는 서버를 쉽게 바꿀수 있으므로  무한한 서버 증설이 가능하다.

단, 클라이언트가 보내야할 데이터의 양이 많아진다.


###출처

출처:[https://www.inflearn.com/course/http-웹-네트워크](https://www.inflearn.com/course/http-%EC%9B%B9-%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC)
