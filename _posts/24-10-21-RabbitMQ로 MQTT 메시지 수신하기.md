--- 
layout: post 
categories: [Project]
tags: [RabbitMQ, MQTT, AMQP]
---

<br>


진행중인 프로젝트에서 RabbitMQ를 사용해서 MQTT 프로토콜을 통해 데이터를 받을 일이 있어 정리한다. 들어가기전에 RabbitMQ와 MQTT에 대해 간단하게 정리하고 가겠다.

<br><br>

## RabbitMQ란?

![image.png](/assets/img/rabbitMQ_MQTT/rabbitMQ_MQTT(1))

<br>

메시지 브로커(Message Broker)로, 서로 다른 애플리케이션 간에 데이터를 주고받을 수 있도록 중개 역할을 하는 소프트웨어이다. 일반적으로 어플리케이션(Producer)이 메시지를 큐(queue)에 저장하고, 해당 메시지를 다른 애플리케이션(Consumer)이  꺼내 가는 방식으로 동작한다.

Producer는 메시지를 큐에 계속 쌓고 Consumer는 쌓인 메시지를 꺼내가는 방식이기 때문에 비동기로 데이터를 처리할 수 있다.

RabbitMQ는 주로 **AMQP(Advanced Message Queuing Protocol)**를 기반으로 한다.

<br><br>

## AMQP? MQTT?

<br>

AMQP, MQTT 모두 메시지 기반 프로토콜이다.

우리가 잘아는 HTTP 프로토콜은 클라이언트가 특정 서버에 요청을 보내면 서버가 응답을 반환하는 방식으로 동작하지만 메시지 기반 통신에서는 **브로커**를 통해 데이터를 주고받는다. 클라이언트가 메시지를 특정 **큐**에 발행(publish)하면, 이를 구독(subscribe)하고 있는 소비자(다른 서비스나 애플리케이션)가 해당 메시지를 받아 처리하는 방식이다.


<br>

먼저 **AMQP (Advanced Message Queuing Protocol)란**
메시지 기반 통신을 위한 **표준 프로토콜**로, 다양한 시스템 간의 메시지를 안정적이고 효율적으로 전달하기 위해 사용된다. 서비스 간의 비동기 통신을 가능하게 하여 시스템의 확장성과 유연성을 증가시켜준다.

- **발행-구독**, **포인트-투-포인트**, **라우팅 키 기반 전달** 등 다양한 방식을 지원한다.
- 표준이기 때문에 메시지 라우팅, 큐잉, transaction 등의 기능을 모두 지원한다.
- 비교적 **무거운 프로토콜**로, 높은 데이터 신뢰성과 복잡한 메시지 처리가 필요한 상황 적합하다.
- 대표 메시지 브로커 : `RabbitMQ`

<br>

**MQTT(Message Queuing Telemetry Transport)** 는 경량의 **메시지 프로토콜**로, **사물인터넷(IoT)** 환경에서 사용하기 위해 설계되었다. MQTT는 저전력, 낮은 대역폭, 제한된 자원 등과 같은 상황에서도 안정적으로 동작할 수 있도록 설계되었기 때문에 IoT 디바이스, 센서 등 다양하게 사용된다.

- 발행/구독 (Publish/Subscribe) 모델에 중심
- QoS (Quality of Service) 를 통해 메시지 전달의 신뢰성을 보장한다.
    - **QoS 0**: "At most once" - 메시지가 한 번만 전달되며, 전송 실패 시 재전송하지 않음
    - **QoS 1**: "At least once" - 메시지가 적어도 한 번 전달되며, 전송 성공이 확인될 때까지 재전송
    - **QoS 2**: "Exactly once" - 메시지가 정확히 한 번만 전달되도록 보장
- 리소스가 제한된 환경에 적합하다.
- 메시지 브로커 : `Mosquitto`

<br><br>

## RabbitMQ로 MQTT 통신

RabbitMQ는 위에서 봤다싶이 AMQP를 기반으로 한다. 하지만 RabbitMQ는 Plugin을 통해 MQTT를 통신할수있게 지원 해준다. 

RabbitMQ는 Docker를 통해 쉽게 실행 시킬 수 있다.

```bash
docker run -d --name rabbitmq -p 5672:5672 -p 15672:15672 -p 1883:1883 rabbitmq:management
```

- `-p 5672:5672` : AMQP 프로토콜 통신 포트는 5672번 포트이다.  **5672** 포트를 호스트와 컨테이너에서 동일하게 사용할 수 있도록 매핑
- `-p 15672:15672` : 15672번 포트는 RabbitMQ Management 에 접속하는 데 사용.  마찬가지로 15672 포트를 호스트와 컨테이너에서 동일하게 사용할 수 있도록 매핑
- `-p 1883:1883` : 1883번 포트는 **MQTT 프로토콜** 통신 포트이다. 호스트와 컨테이너에서 동일하게 매핑

<br>

### RabbitMQ MQTT 플러그인 설정

```bash
docker exec -it rabbitmq rabbitmq-plugins enable rabbitmq_mqtt
```

<br>

![image.png](/assets/img/rabbitMQ_MQTT/rabbitMQ_MQTT(2))

<br>

- `docker exec`  : 실행 중인 Docker 컨테이너 내에서 명령어를 실행할 수 있게 해주는 명령어
- `-it` :  터미널에서 명령어를 입력하는 것처럼 컨테이너 내에서 명령어를 실행할 수 있게해준다.
- `rabbitmq-plugins` : RabbitMQ 플러그인을 관리하기 위한 명령어
- `enable rabbitmq_mqtt` : RabbitMQ에서 MQTT 프로토콜을 지원하는 플러그인을 활성화

<br>

플러그인을 설정이 되었는지 확인해보자

<br>

### MQTT 플러그인이 활성화되어 있는지 확인

```bash
docker exec -it rabbitmq bash
rabbitmq-plugins list
```

<br>

- `rabbitmq bash` : rabbitmq라는 컨테이너 내에서 Bash 셀을 실행하는 명령어
- `rabbitmq-plugins list` :활성화된 플러그인과 비활성화된 플러그인의 전체 목록을 보여주는 명령어

<br>

![image.png](/assets/img/rabbitMQ_MQTT/rabbitMQ_MQTT(3))

[E] : 플러그인이 설치되었음을 의미

[*] : 활성화 되었음을 의미

<br>

플러그인이 설정이 되었다면 RabbitMQ Management에서 MQTT 메시지를 AMQP로 변환하여 Queue에 메지시를 담아보자.

<br>

### RabbitMQ 서버에 MQTT 메시지를 보관할 Queue 만들기

<br>

RabbitMQ Management([http://localhost:15672/](http://localhost:15672/))에 접속한다.

![image.png](/assets/img/rabbitMQ_MQTT/rabbitMQ_MQTT(4))

- Username : guest

- Password : guest

<br>

### Queue 생성

Queues and Streams 탭의 Add a new queue

![image.png](/assets/img/rabbitMQ_MQTT/rabbitMQ_MQTT(5))

![image.png](/assets/img/rabbitMQ_MQTT/rabbitMQ_MQTT(6))

- Virtual host : 큐가 속할 가상 호스트(Virtual Host)를 설정
- Type : 큐 유형 설정
- Name : 큐의 이름
- Durability : 큐의 내구성을 설정
    - Durable : 서버가 재시작 된 후에도 큐가 유지
    - Transient : 서버가 재시작되면 큐를 삭제
- Arguments : 큐의 추가 설정을 정의

MQTT 의 통신이 목적이므로 이름만 설정해주고 큐를 만들어주자. 

그러면 이렇게 Queue가 생성이 된다.

<br>

![image.png](/assets/img/rabbitMQ_MQTT/rabbitMQ_MQTT(7))

RabbitMQ에서 메시지를 처리할 때, 기본적으로 메시지는 교환기(Exchange)로 라우팅된다.  클라이언트에서 메시지를 보내면 RabbitMQ는 메시지를 내부 교환기로 보내고, 그 교환기로부터 지정된 큐에 바인딩을 통해 메시지를 전달하는 매커니즘이다.

- **MQTT 기본 교환기**는 기본적으로 `amq.topic`이다.
- MQTT 메시지는 특정 주제(Topic)로 전송되며, 해당 주제는 교환기를 통해 큐에 바인딩된다.

<br>

### 큐의 바인딩 설정

큐에 MQTT 메시지를 저장하려면 교환기와 큐 사이에 바인딩을 설정해야 한다.

RabbitMQ 관리 콘솔에서 **Exchanges** 탭으로 이동해  `amq.topic` 교환기를 선택

![image.png](/assets/img/rabbitMQ_MQTT/rabbitMQ_MQTT(8))

<br>

**Bindings** 섹션에서 `MQTT_Queue` 큐를 특정 주제(예: `mqtt.topic`)와 바인딩한다.

![image.png](/assets/img/rabbitMQ_MQTT/rabbitMQ_MQTT(9))

<br>

바인딩을 설정하면 해당 주제로 전송된 모든 메시지가 `MQTT_Queue`로 전달이 될것이다.

<br>

### POSTMAN을 사용해 MQTT 전송

PostMan에서 MQTT를 호환해준다. MQTT 메시지를 RabbitMQ에 보내 큐에 메시지가 쌓이는지 확인해보자.

메시지를 전송할 URL을 입력하자.

![image.png](/assets/img/rabbitMQ_MQTT/rabbitMQ_MQTT(10))

- `mqtt://` : **MQTT 프로토콜**을 사용하여 메시지를 전송하겠다는 것을 나타낸다.
- [`localhost:1833`](http://localhost:1833) : 로컬 에서 실행 중인 MQTT 브로커(docker로 띄운 RabbitMQ 서버)에 연결

<br>

Connect 버튼을 눌러 브로커와 연결

![image.png](/assets/img/rabbitMQ_MQTT/rabbitMQ_MQTT(11))

<br>

해당 메시지가 어떤 Queue로 갈지 Binding할 Topic을 설정해야한다.

![image.png](/assets/img/rabbitMQ_MQTT/rabbitMQ_MQTT(12))

- QOS : Quality of Service로 메시지 전달의 신회성을 보장하기 위한 메커니즘
    - QOS 0 : 메시지가 최대 1번 전송, 메시지 손실 가능성이 있다. (중복X, 손실 O)
    - QOS 1 : 메시지가 최소 1번 전송, 메시지 중복 가능성이 있지만, 적어도 한 번은 도착하도록 보장 (중복O, 손실X)
    - QOS 2 : 메시지가 정확히 한 번만 전달(중복X, 손실X)

<br>

메시지를 입력하고 전송해보자

<br>

![image.png](/assets/img/rabbitMQ_MQTT/rabbitMQ_MQTT(13))

메시지가 성공적으로 전송되었다. 

<br>

![image.png](/assets/img/rabbitMQ_MQTT/rabbitMQ_MQTT(14))

이제 RabbitMQ Management에서 메시지가 Queue에 저장되어있는지 확인해보자.

<br>

![image.png](/assets/img/rabbitMQ_MQTT/rabbitMQ_MQTT(15))

Ready에 1이 올라간게 보일텐데 성공적으로 메시지가 도착한것이다.

- Ready : 현재 큐에 대기중인 메시지의 수
- Unacked : 큐에서 소비자에게 전달되었지만 아직 소비자로부터 확인이 되지 않은 메시지의 수
- Total: Ready와 Unacked 메시지를 합친값, 큐에 존재하는 총 메시지 수.

<br>

메시지를 확인해보자.

<br>

Get messages 탭에서 내가 보낸 메시지를 확인할 수 있다.

![image.png](/assets/img/rabbitMQ_MQTT/rabbitMQ_MQTT(16))

내가 보낸 메시지가 맞다!

<br>

이렇게 RabbitMQ(Management)에서 MQTT를 받아 AMQP로 변환해 Queue에 담는것 까지 해보았다. 다음에는 Spring AMQP를 사용해 큐에 저장된 메시지를 사용하는것을 해보겠다.