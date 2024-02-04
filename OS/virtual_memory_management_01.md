# Virtual memory management

가상 메모리를 관리하는 목적은 페이지 폴트 발생률을 줄여서 문맥 교환과 커널의 개입을 최소화하는 데에 있다.  
TLB나 비트 벡터같은 `하드웨어 요소`를 이용하거나, `소프트웨어 요소`, `페이지 교체 전략`들을 활용하는 방법이 존재한다.  

---

## Hardware components

- Address translation device
- Bit Vectors

## Bit Vectors

<img src="OS/img/vm_management01.png" width=50%>

`비트 벡터`란 페이지 사용 상황에 대한 정보를 기록하는 비트들을 말한다.  
`Reference bit vector`와 `Update bit vector`를 이용해서 메모리에 적재된 각각의 페이지가  
최근에 참조 되었는지 그리고 프로세스에 의해 수정 되었는지를 표시하고 페이지 정보를 효율적으로 관리할 수 있다.

---

## Software components
