# 에라토스테네스의 체
  
고대 그리스 수학자 에라토스테네스가 발견하였다.  
임의의 자연수 N에 대해 그 이하의 소수를 찾는 가장 간단하고 빠른 방법이다.
#
자연수 N을 입력받고, 1부터 N까지의 소수의 개수를 출력하는 프로그램을 작성

```java
import java.util.*;

public class Main {
    public int solution(int n) {
        int answer = 0; // 소수의 개수
        int[] ch = new int[n+1]; // 체의 역할을 하는 배열. n+1 길이로 할당한다.
		
        for(int i=2; i<=n; i++) { // 0과 1은 소수가 아니므로 2부터 반복문을 시작한다.
            if(ch[i]==0) { // i가 소수이면
                answer++; // 소수의 개수를 1개 증가시킨다.
        
                for(int j=i; j<=n; j+=i) { // i부터 n까지 i의 배수들을 모두 찾는다.
                    ch[j] = 1; // i의 배수는 이미 소수가 아니므로 체 배열값을 1로 변경한다.
                }
            }
        } // n까지 계속 반복한다.
    
        return answer;
    }
	
    public static void main(String[] args) {
        Main T = new Main();
        Scanner kb = new Scanner(System.in);
        int n = kb.nextInt();
        System.out.println(T.solution(n));
    }
}
```

위 프로그램에서는 소수의 개수를 구하기 위해 i를 n까지 반복시켰지만,  
소수를 출력만 하면 되는 프로그램에서는 굳이 그렇게 하지 않아도 된다.

#

자연수 N을 입력받아, 1부터 N까지 숫자 중 소수를 출력하는 프로그램을 작성

```java
import java.util.*;

public class Main {
    public void solution(int n) {
        boolean[] ch = new boolean[n+1]; // 체의 역할을 하는 배열을 n+1 길이로 할당한다.
		
        for(int i=2; i*i<=n; i++) { // 0과 1은 소수가 아니므로 2부터 반복문을 시작한다.
            if(!ch[i]) { // i이 소수이면
                for(int j=i*i; j<=n; j+=i) { // i*i부터 n까지 i의 배수들을 모두 찾는다
                    ch[j] = true; // i의 배수는 이미 소수가 아니므로 true를 할당한다.
                }
            }
        }
		
        for(int i=2; i<=n; i++) {
            if(!ch[i]) System.out.print(i + " ");
        }
    }
	
    public static void main(String[] args) {
        Main T = new Main();
        Scanner kb = new Scanner(System.in);
        int n = kb.nextInt();
        T.solution(n);
    }
}
```

이와 같은 프로그램을 짤 경우, 반복문에서 굳이 i를 n까지 반복시키지 않아도 된다.  
  
만약 n이 120이면 의미있는 반복은 7까지이다.  
8의 배수같은 경우, 2의 배수를 찾는 과정에서 모두 지워졌을 것이므로 찾을 필요가 없어진다.  
똑같이 9의 배수도 3의 배수를 찾는 과정에서 모두 지워진다.  
10의 배수도 지울 필요가 없다  
11 이상부터는 11^2 > 120 이기 때문에 역시 지울 필요가 없다.  
  
j 반복문에서도 j의 시작 값을 i\*i로 설정해서 최적화할 수 있다.  
i\*i 미만의 값들은 이미 이전 반복문에서 처리되었다.
