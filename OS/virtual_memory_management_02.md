# Virtual memory managament

## Replacement Strategies

- Fixed allocation
  - MIN algorithm
  - Random algorithm
  - FIFO algorithm
  - LRU(Least Recently Used)
  - LFU(Least Frequently Used)
  - NUR(Not Used Recently)
  - Clock algorithm
  - Second chance algorithm
- Variable allocation
  - WS(Working Set) algorithm
  - PFF(Page Fault Frequency)
  - VMIN(Variable MIN)
 
---

## Min algorithm (OPT)

페이지 교체 전략 중 Minimize page fault frequency 알고리즘은 최적의 알고리즘으로써  
페이지 폴트가 발생할 가능성을 최소화하기 위해 설계된 알고리즘이다.  

이 알고리즘은 페이지 교체를 결정할 때 현재 상태에서 앞으로 참조될 가능성이 가장 낮은 페이지를 교체하는 방식으로 동작한다.  
페이지 참조 문자열을 미리 알고 있어야 하기 때문에 실현 불가능한 기법이지만 다른 교체 전략의 성능 평가 도구로 활용되는 데에 가치가 있다.
