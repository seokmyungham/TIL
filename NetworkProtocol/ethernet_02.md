# Ethernet

## Multiple access protocols

<img sre="img/datalink04.png">  

`다중 접속 링크(Multiple Access Link)`는 여러 장치가 네트워크 채널을 공유하여 데이터를 송수신하는 방법을 의미한다.  

다중 접속 링크에는 두가지 타입이 존재한다.  

### Point-to-Point

- Point-to-Point Protocol(PPP)를 사용하여 다이얼업 액세스에 대한 연결
- 이더넷 스위치와 호스트의 직접적인 일대일 연결

### Broadcast

- 오래된 이더넷
- upstream HFC (유선 네트워크)
- 802.11 wireless LAN (Wi-Fi)

이러한 다중 접속 환경에서 여러 장치가 충돌을 피하고 효율적으로 네트워크 채널을 공유할지 결정하기 위해 
`Multiple access protocols`이란 분산 알고리즘이 존재한다.

#

## Random access protocols

랜덤 액세스 MAC 프로토콜은 데이터 충돌을 감지하고 충돌로부터 복구하기 위한 방법을 정의한다.  

- ALOHA
- slotted ALOHA
- CSMA, CSMA/CD, CSMA/CA

### Pure(unslotted) ALOHA

<img sre="img/datalink05.png">  

명칭 그대로 정해진 슬롯이 없다는 뜻으로 `no synchronization`하다는 의미이다.  
노드들은 프레임이 완성되자마자 즉시 링크에 데이터를 전송하고 collision이 발생한다.  

### Slotted ALOHA

<img sre="img/datalink06.png"> 

`Slotted ALOHA`는 `Pure ALOHA`의 개선된 버전으로, 시간을 슬롯으로 나누어 데이터 전송을 조정한다.  
  
각 시슬롯은 일정한 시간 간격으로 분할되며, 노드는 오직 슬롯의 시작에서만 데이터를 전송할 수 있다.  
이로 인해 `Pure ALOHA`보다 collision 가능성이 줄어든다.  
  
충돌이 발생할 경우, 일정한 시간 후에 다시 전송을 시도한다.

#

### CSMA (carrier sense multiple access)

`CSMA`는 데이터를 전송하기 전에 채널의 상태를 먼저 감지하고 전송 여부를 결정하는 방식이다.  

만약 채널이 사용 가능한 상태라면 노드는 데이터 프레임을 전송하고,  
채널이 사용중인 상태라면 노드는 전송을 연기하고 나중에 다시 시도한다.  

<img sre="img/datalink07.png">   
  
하지만 먼저 채널의 상태를 확인한다고 해도 
`두 노드의 거리`와 `전파 지연` 때문에 collision은 여전히 발생할 수 있다.

### CSMA/CD

`CSMA/CD`는 CSMA의 확장 버전으로, 충돌을 탐지하고 처리한다.  
충돌이 발생하면 이를 빠르게 감지하고 충돌 중인 전송을 중단시킨다.  
이후 백오프 알고리즘을 통해 다시 전송을 시도한다.

#

### Polling / Taking turns

마스터-슬레이브 관계처럼 중앙화 되어있는 컨트롤러가 중재하는 방식이다.  

### Token Passing / Taking turns

토큰을 노드들이 서로 돌려가면서, 토큰을 가지고 있는 단말은 데이터를 보낼 기회가 주어진다.  

Taking turns 프로토콜은 collision으로 부터 안전해서 블루투스 같은 네트워크에서 사용된다.
