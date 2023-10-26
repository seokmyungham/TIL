# Network Layer

## DHCP  

`DHCP(Dynamic Host Configuration Protocol)`  
컴퓨터가 네트워크에 참여할 때 `IP 주소를 동적으로 할당`받을 수 있도록 해주는 프로토콜을 말한다.  

DHCP는 할당된 IP 주소뿐만 아니라  
`첫 번째 홉 라우터의 주소`, `DNS 서버의 이름 및 IP 주소`, `네트워크 마스크와` 같은 정보도 반환할 수 있다.  

#

### DHCP DORA

DORA는 DHCP를 사용하는 
`Discover`, `Offer`, `Request`, `Acknowledgement`로 이루어진 일련의 과정을 말한다.  

DHCP의 주요 목적은 호스트가 네트워크에 참가할 때 IP 주소를 동적으로 할당하는 것이다.  
DHCP 서버는 사용가능한 IP 주소 하나를 호스트에게 할당한다.  

<img src="dhcp01.png">  

과정은 아래와 같다

```
HOST (Discover): DHCP 서버가 존재하는지 브로드캐스트로 패킷을 던진다.
DHCP SERVER (Offer): DHCP는 서버가 존재한다는 것을 호스트에게 알리고 사용할 수 있는 IP 주소가 있다고 호스트에 알린다.
HOST (Request): 호스트는 DHCP 서버에 해당 IP 주소를 요청한다.
DHCP SERVER (Ack): DCHP 서버는 호스트에게 IP 주소를 할당한다.
```

보통 DHCP 클라이언트의 포트 번호는 68, 서버의 포트 번호는 67을 사용한다.  
src: 0.0.0.0, 68은 클라이언트가 ip주소가 존재하지 않고, DCHP의 클라이언트로서 패킷을 보냈다는 것을 의미한다.  
dest: 255.255.255.255는 브로드캐스트 요청을 의미한다.  
저기서 말하는 yiaddr은 your ip address이며, 아직 ip 주소가 없으므로 0.0.0.0이다.  

DCHP 서버는 discover 응답을 받고, 사용 가능한 ip 주소가 있음을 알리는 offer를 유니캐스트 혹은 브로드캐스트로 전송한다.  

클라이언트는 src를 비워둔 채, 해당 ip 주소를 요청하는 request를 유니캐스트 혹은 브로드캐스트로 전송하게 된다.  

서버는 최종적으로 ack 요청을 발신하게 되고, 클라이언트는 ip 주소를 할당받게 된다.
