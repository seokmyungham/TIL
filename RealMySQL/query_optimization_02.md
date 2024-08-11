# 쿼리 작성 및 최적화

## SELECT

웹 서비스와 같이 일반적인 OLTP 환경의 데이터베이스에서는 INSERT나 UPDATE 같은 작업은 거의 레코드 단위로 발생하므로
성능상 문제가 되는 경우는 별로 없다.
  
하지만 SELECT는 여러 개의 테이블로부터 데이터를 조합해서 빠르게 가져와야 하기 때문에 여러 개의 테이블을 어떻게 읽을 것인가가
많이 중요하다. 그러려면 SELECT 쿼리 각 부분에 사용되는 기능이 성능적으로 어떻게 작용하는지 알아야한다.
  
![image](https://github.com/user-attachments/assets/a6755727-cd8f-43a4-962e-0ddadbf6883b)


위는 SQL 쿼리가 실행되는 순서다. 각 요소가 없는 경우는 가능하지만, 이 순서가 바뀌어서 실행되는 쿼리는 거의 없다.
또한 ORDER BY나 GROUP BY 절이 있더라도 인덱스를 이용해 처리할 때는 처리 단계가 생략된다.

![image](https://github.com/user-attachments/assets/324d8ae8-c265-4e9f-9c0e-9e2bb066b211)

ORDER BY가 사용된 쿼리에서 예외적인 순서로 실행되는 경우가 있다. JOIN이 동반된 쿼리일 경우 첫 번째 테이블만 읽어서
정렬을 수행한 뒤에 나머지 테이블을 읽을 수 있는데 주로 GROUP BY 절 없이 ORDER BY만 사용된 쿼리에서 사용될 수 있는 순서다.

#

### WHERE 절과 GROUP BY 절, ORDER BY 절의 인덱스 사용

WHERE 절이나 ORDER BY 또는 GROUP BY가 인덱스를 사용하려면 기본적으로 인덱스된 칼럼의 값 자체를 변환하지 않고
그대로 사용할 수 있어야 한다. 인덱스는 칼럼의 값을 아무런 변환 없이 B-TREE에 저장하기 때문에 WHERE, GROUP BY, ORDER BY 에서도
원본값을 검색하거나 정렬할 때만 B-TREE에 정렬된 인덱스를 이용할 수 있다.

예를 들면 아래와 같은 쿼리는 인덱스를 이용하지 못한다.

```sql
SELECT * FROM salaries WHERE salry*10 > 150000;
```

추가로 WHERE 절에 사용되는 비교 조건에서 연산자 양쪽의 두 비교 대상 데이터 타입이 일치해야 한다.

### WHERE 절의 인덱스 사용

WHERE 조건이 인덱스를 사용하는 방법은 `작업 범위 결정 조건`과 `체크 조건` 두 가지 방식으로 구분할 수 있다.

![image](https://github.com/user-attachments/assets/a37b013b-b132-4cf5-9907-49c03150c8c0)

WHERE 조건절의 순서는 실제 인덱스의 사용 여부와 무관하다. WHERE 조건절에 나열된 순서가 인덱스와 다르더라도
MySQL 서버 옵티마이저는 인덱스를 사용할 수 있는 조건들을 뽑아서 최적화를 수행할 수 있다.
  
COL_1과 COL_2는 동등 비교 조건이며, COL_3 은 범위 비교 조건이므로 COL_4의 조건은 단지 체크 조건으로 사용된다.
인덱스 순서상 COL_4의 직전 칼럼인 COL_3이 동등 비교 조건이 아니라 범위 비교 조건으로 사용됐기 때문이다.
  
8.0 버전부터는 다음 예제와 같이 인덱스를 구성하는 칼럼별로 정순과 역순 정렬을 혼합해서 생성할 수 있게 개선됐다.

```
ALTER TABLE ... ADD INDEX ix_col1234 (col_1 ASC, col_2 DESC, col_3 ASC, col_4 ASC);
```

AND 연산자와 달리 OR 연산자는 처리 방법이 완전히 달라진다.

```sql
SELECT *
FROM employees
WHERE first_name='Kebin' OR last_name='Poly';
```
위 쿼리에서 `first_name='Kebin'` 조건은 인덱스를 이용할 수 있지만 `last_name='Poly'`는 인덱스를 사용할 수 없다.
이 두 조건이 AND 연산자로 연결됐다면 first_name 인덱스를 이용하겠지만 OR 연산자로 연결됐기 때문에 결국 `풀 테이블 스캔`을
선택할 수 밖에 없다. (풀 테이블 스캔) + (인덱스 레인지 스캔)의 작업량 보다는 (풀 테이블 스캔) 한 번이 더 빠르기 때문이다.
  
이 경우 first_name과 last_name 칼럼에 각각 인덱스가 있다면 `index_merge` 접근 방법으로 실행할 수 있긴하다. 물론 풀 테이블 스캔보다는
빠르지만 여전히 제대로 된 인덱스 하나를 레인지 스캔하는 것 보다 느리다. WHERE 절에서 각 조건이 AND로 연결되면 읽어와야 할 레코드의 건수를 줄이는
역할을 하지만 각 조건이 OR로 연결되면 읽어서 비교해야 할 레코드가 더 늘어나기 때문에 OR 연산자가 있다면 주의해야 한다.

### GROUP BY 절의 인덱스 사용

GROUP BY 절에 명시된 칼럼의 순서가 인덱스를 구성하는 칼럼의 순서와 같으면 GROUP BY 절은 일단 인덱스를 이용할 수 있다.

- GROUP BY 절에 명시된 칼럼이 인덱스 칼럼의 순서와 위치가 같아야 한다.
- 인덱스를 구성하는 칼럼 중에서 뒤쪽에 있는 칼럼은 GROUP BY 절에 명시되지 않아도 인덱스를 사용할 수 있지만 인덱스의 앞족에 있는 칼럼이 GROUP BY 절에 명시되지 않으면 인덱스를 사용할 수 없다.
- WHERE 조건절과는 달리 GROUP BY 절에 명시된 칼럼이 하나라도 없으면 GROUP BY 절은 전혀 인덱스를 이용하지 못한다.

![image](https://github.com/user-attachments/assets/5e197ec4-793c-4554-a423-fe46bcd0aed7)

```SQL
GROUP BY COL_2, COL_1 
GROUP BY COL_1, COL_3, COL_2 
GROUP BY COL_1, COL_3 
GROUP BY COL_1, COL_2, COL_3, COL_4, COL_5
```

위 쿼리들은 모두 인덱스를 이용하지 못하는 경우들이다.

- 첫 번째와 두 번째 예제는 GROUP BY 칼럼이 인덱스를 구성하는 칼럼의 순서와 일치하지 않기 때문에 사용하지 못하 는 것이다.
- 세 번째 예제는 GROUP BY 절에 COL_3가 명시됐지만 COL_2가 그 앞에 명시되지 않았기 때문이다.
- 네 번째 예제에서는 GROUP BY 절의 마지막에 있는 COL_5가 인덱스에는 없어서 인덱스를 사용하지 못하는 것이다.

```SQL
GROUP BY COL_1 
GROUP BY COL_1, COL_2
GROUP BY COL_1, COL_2, COL_3 
GROUP BY COL_1, COL_2, COL_3, COL_4
```

```SQL
WHERE COL_1='상수' GROUP BY COL_2, COL_3 
WHERE COL_1='상수' AND COL_2='상수' GROUP BY COL_3, COL_4
WHERE COL_1='상수' AND COL_2='상수' AND COL_3='상수' GROUP BY COL_4
```

위 쿼리들은 인덱스를 사용할 수 있는 패턴이다.
WHERE 조건절에 COL_1이나 COL_2가 동등 비교 조건으로 사용된다면 GROUP BY 절에 COL_1이나 COL_2가 빠져도 인덱스를 이용한
GROUP BY가 가능할 때도 있다.

### ORDER BY 절의 인덱스 사용

MySQL에서 GROUP BY와 ORDER BY는 처리 방법이 상당히 흡사하다. 그래서 ORDER BY 절의 인덱스 적용 여부는 GROUP BY 요건과 거의 흡사하다.
그런데 ORDER BY는 추가로 정렬되는 각 칼럼의 오름차순 및 내림차순 옵션이 인덱스와 같거나 정반대인 경우에만 사용할 수 있다.

![image](https://github.com/user-attachments/assets/4c3d8072-a8ee-4c41-a145-9efc30e9f06b)

인덱스의 모든 칼럼이 ORDER BY 절에 사용돼야 하는 것은 아니지만, ORDER BY 절의 칼럼들이 인덱스에 정의된 칼럼의 왼쪽부터 일치해야 한다.

```SQL
ORDER BY COL_2, COL_3 
ORDER BY COL_1, COL_3, COL_2 
ORDER BY COL_1, COL_2 DESC, COL_3 
ORDER BY COL_1, COL_3
ORDER BY COL_1, COL_2, COL_3, COL_4, COL_5
```

위 쿼리들은 모두 인덱스를 이용할 수 없다. 정렬 순서가 생략되면 오름차순으로 해석한다.

- 첫 번째 예제는 인덱스의 제일 앞쪽 칼럼인 COL_1이 ORDER BY 절에 명시되지 않았기 때문에 인덱스를 사용할 수 없다.
- 두 번째 예제는 인덱스와 ORDER BY 절의 칼럼 순서가 일치하지 않기 때문에 인덱스를 사용할 수 없다.
- 세 번째 예제는 ORDER BY 절의 다른 칼럼은 모두 오름차순인데, 두 번째 칼럼인 COL_2의 정렬 순서가 내림차순이라 서 인덱스를 사용할 수 없다. 인덱스가 "(COL_1 ASC, COL_2 DESC, COL_3 ASC, COL_4 ASC)"와 같이 정의됐다 면 이 정렬은 인덱스를 사용할 수 있게 된다.
- 네 번째 예제는 인덱스에는 COL_ 1과 COL_3 사이에 COL_2 칼럼이 있지만 ORDER BY 절에는 COL.2 칼럼이 명시되지 않았기 때문에 인덱스를 사용할 수 없다.
- 다섯 번째 예제는 인덱스에 존재하지 않는 COL_5가 ORDER BY 절에 명시됐기 때문에 인덱스를 사용하지 못한다.

### WHERE 조건과 ORDER BY(또는 GROUP BY) 절의 인덱스 사용

일반적으로 우리가 사용하는 쿼리는 WHERE 절을 가지고 있으며, 선택적으로 ORDER BY, GROUP BY 절을 포함할 것이다.
쿼리에 WHERE 절만 또는 GROUP BY, ORDER BY 절만 포함돼 있다면 사용된 절 하나에만 초점을 맞춰서 인덱스를 사용할 수 있게 튜닝하면 된다.
  
하지만 애플리케이션에서 사용되는 쿼리는 그렇게 단순하지 않다. SQL 문장이 WHERE 절과 ORDER BY절을 가지고 있다고 가정했을 때 WHERE 조건은
A 인덱스를 사용하고 ORDER BY는 B 인덱스를 사용하도록 쿼리가 실행될 수는 없다.

- WHERE 절과 ORDER BY 절이 동시에 같은 인덱스를 이용
    - WHERE 절의 비교 조건에서 사용하는 칼럼과 ORDER BY 절의 `정렬 대상 칼럼이 모두 하나의 인덱스에 연속해서 포함`돼 있을 때 이 방식으로 인덱스를 사용할 수 있다. 이 방법 은 나머지 2가지 방식보다 훨씬 빠른 성능을 보이기 때문에 가능하다면 이 방식으로 처리할 수 있게 쿼리를 튜닝하 거나 인덱스를 생성하는 것이 좋다.
- WHERE 절만 인덱스를 이용
    - ORDER BY 절은 인덱스를 이용한 정렬이 불가능하며, 인덱스를 통해 검색된 결과 레코드를 별도의 정렬 처리 과정(Using Filesort)을 거쳐 정렬을 수행한다. 주로 이 방법은 WHERE 절의 조건에 일치하는 `레코드의 건수가 많지 않을 때` 효율적인 방식이다.
- ORDER BY 절만 인덱스를 이용
    - ORDER BY 절은 인덱스를 이용해 처리하지만 WHERE 절은 인덱스를 이용하지 못한다. 이 방식은 ORDER BY 절의 순서대로 인덱스를 읽으면서 레코드 한 건씩 WHERE 절의 조건에 일치하는지 비교하고, 일치하지 않을 때는 버리는 형태로 처리한다. 주로 `아주 많은 레코드를 조회해서 정렬해야 할 때`는 이런 형태로 튜닝 하기도 한다.
 
또한 WHERE 절에서 동등 비교 조건으로 비교된 칼럼과 ORDER BY 절에 명시된 칼럼이 순서대로 빠짐없이 인덱스 칼럼의 왼쪽부터 일치해야 한다.
8.0 버전에 새롭게 추가된 인덱스 스킵 스캔 최적화는 인덱스에 나열된 칼럼 순서상 선행되는 칼럼의 조건이 없다고 하더라도 인덱스의 후행 칼럼을 이용할 수 있게 해주긴 한다.

```SQL
SELECT * FROM tb_test WHERE COL_1 > 10 ORDER BY COL_1, COL_2, COL_3;
SELECT * FROM tb_test WHERE COL_1 > 10 ORDER BY COL_2, COL_3;
```

첫 번째 예제 쿼리에서 COL_1 > 10 조건을 만족하는 COL_1 값은 여러 개일 수 있다. 
하지만 ORDER BY 절에 COL_1부터 COL_3까지 순서대로 명시됐기 때문에 인덱스를 사용해 WHERE 조건절과 ORDER BY 절을 처리할 수 있다.
  
하지만 두 번째 쿼리에서는 WHERE 절에서 COL_1 동등 조건이 아니라 범위 조건으로 검색됐는데 ORDER BY 절에 COL_1이 명시되지 않았기 때문에
정렬할 때는 인덱스를 이용할 수 없게된다.

### GROUP BY 절과 ORDER BY 절의 인덱스 사용

GROUP BY와 ORDER BY 절이 동시에 사용된 쿼리에서 두 절이 모두 하나의 인덱스를 사용해서 처리되려면 
GROUP BY 절에 명시된 칼럼과 ORDER BY에 명시된 칼럼의 순서와 내용이 모두 같아야 한다.
  
GROUP BY와 ORDER BY가 같이 사용된 쿼리에서는 둘 중 하나라도 인덱스를 이용할 수 없을 때는 둘 다 인덱스를 사용하지 못한다.
GROUP BY는 인덱스를 이용할 수 있지만 ORDER BY가 인덱스를 이용할 수 없을 때 GROUP BY, ORDER BY절이 모두 인덱스를 이용하지 못한다는 것이다.

```SQL
GROUP BY COL_1, COL_2 ORDER BY COL_2
GROUP BY COL_1, COL_2 ORDER BY COL_1, COL_3
```

## Reference 

**위 글은 책 RealMySQL 8.0 2권을 구입하여 읽고 정리한 내용입니다.**
- [도서 홈페이지 https://wikibook.co.kr/realmysql802/](https://wikibook.co.kr/realmysql802/)
