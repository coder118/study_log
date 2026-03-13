# Tcp(transmission control protocol) UDP(user datagram protocol)

osi의 transport 계층은 end point간에 **신뢰성**있는 데이터 **전송**을 담장하는 계층이다.

신뢰성 - 데이터를 순차적, 안정적으로 전달 

전송 - 포트 번호에 해당하는 프로세스에 데이터를 전달.

### 만약 transport 계층이 존재하지 않는다면,

1. 데이터의 순차전송이 원활히 되지 않는다.
순서가 뒤죽박죽 섞인다.

2. flow (흐름 문제) 
원인 : 송수신자간의 데이터 처리 속도 차이 수신자가 처리할 수 있는 데이터량을 초과한 경우.

3. congestion(혼잡 문제)
원인 : 네트워크의 데이터 처리 속도(ex. 라우터) 가 혼잡할때 


# TCP

- 신뢰성있는 데이터 통신을 가능하게 해주는 프로토콜

- 특징 : connection 연결(3-way-handshake) -양방향 통신

- 데이터의 순차 전송을 보장

- flow control (흐름제어)

- congestion control(혼잡 제어)

- error detection(오류 감지)

## 세그먼트

보내는 데이터를 쪼개서 보내는 단위.

## tcp의 3 - way handshake(connection연결)

![alt text](/CS/network_images/image.png)

1. syn 비트를 1로 설정해 패킷을 송신

2. syn ack 비트를 1로 설정해 패킷 송신

3. ack 비트를 1로 설정해 패킷 송신


### tcp의 데이터 전송 방식

1. client 가 패킷을 송신한다.

2. server에서 패킷을 잘 받았다고 ack를 송신한다.

3. 만약, client가 ack를 받지 못할경우 잠시 wait했다가 다시 packet을 재전송을 해준다.

위의 과정처럼 tcp는 신뢰성있는 전송을 보장하고 있다.


## 4-way handshake(connection close)

1. 데이터를 전부 송신한 client가 fin을 송신한다.
2. server가 ack를 송신
3. server에서 남은 패킷을 송신(일정 시간을 대기한다.)
4. server가 fin 송신
5. client가 ack 송신을 마무리로 connection을 close하게 된다.


### tcp의 문제점

전송의 신뢰성을 보장하지만 매번 connection을 연결해서 시간 손실 발생할 수 있다.
또한 패킷을 조금만 실수해도 재전송을 해야 한다.

# UDP

- tcp보다 신뢰성이 떨어지지만 전송속도가 일반적으로 빠른 프로토콜이다.(순차전송x, 흐름제어x, 혼잡제어x)

- connectionless(3 way-handshake x)

- error detection

- 비교적 데이터 신뢰성이 중요하지 않을 때 사용(ex. 영상 스트리밍)


### User datagram

tcp에서는 세그먼트 단위로 동작하듯, udp에서는 user datagram 단위로 동작한다. 

데이터를 전송할때 쪼개는 것이 아니라 data 앞단에 udp Header를 붙여서 전송한다.

## UDP의 데이터 전송 방식

1. client가 패킷을 송신한다. (connection이 없어서 확인하지 않고 데이터를 바로 전송한다.)