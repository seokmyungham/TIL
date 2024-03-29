# Transport Layer

`전송 계층`은 신뢰성 있는 전송을 하기 위해 여러 매커니즘을 사용한다.  
그 중 주요한 것으로는 `오류 제어(Error Control)`, `흐름 제어(Flow Control)`, `혼잡 제어(Congestion Control)`가 존재한다.  

#

## 오류 제어(Error Control)

`오류 제어`는 데이터 통신 과정에서 발생하는 오류를 감지하고 복구하는 매커니즘을 의미한다.  

<img src="img/transport03.png">
  
`훼손된 패킷을 감지하고 폐기`,  
`손실되거나 제거된 패킷 추적과 재전송`,  
`중복 수신 패킷 확인 및 폐기` 모두 오류 제어 프로세스에 해당된다.  

`오류 제어`를 위해 `ACK`, `Retransmission`을 주로 이용하는데,  
  
데이터를 보낸 후 수신 측에서 정상적으로 수신했다는 의미의 `확인 신호(ACK)`와  
`패킷 로스`, `ACK 로스`, `잘못된 데이터를 감지했을 경우` 해당 데이터를 다시 전송하는 `재전송(Retransmission)` 과정으로 이루어진다.  

#

## 흐름 제어(Flow Control)

`흐름 제어`는 송신 측과 수신 측의 데이터 처리 속도 차이로 인해 발생할 수 있는 문제를 제어하는 매커니즘이다.  

<img src="img/transport11.png">

패킷에 `sequence number`를 부여해서, `패킷의 순서를 확인`할 수 있도록 하고 `어떤 패킷이 손실되었는지 특정`할 수 있도록 한다.  
    
또한 `흐름 제어`를 구현하기 위한 한 가지 방법으로 `버퍼`를 이용한다.  
  
송신 측과 수신 측은 각각 하나 혹은 그 이상의 `버퍼를 사용`하여  
`버퍼가 가득 차거나`, `버퍼의 빈 공간`을 반대 사용자에게 알려주어 `패킷의 폐기` 및 `재전송을 줄이고 통신 성능을 향상`시킨다.

#

## 혼잡 제어(Congestion Control)

`혼잡 제어`는 네트워크 내에서 발생할 수 있는 혼잡을 방지하고, 네트워크 자원의 효율적인 사용을 위한 매커니즘이다.  
  
네트워크 내의 패킷 수가 너무 많아져서 네트워크가 처리하기 어려워지는 혼잡 상태를 감지하고  
송신자에게 전송 속도, `전송하는 패킷의 양을 줄이도록 신호를 보내어(CWND)` 네트워크 혼잡을 완화한다.
