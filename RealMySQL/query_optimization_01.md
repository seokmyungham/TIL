# 쿼리 작성 및 최적화

SQL은 어떤 데이터를 요청하기 위한 언어일 뿐, 어떻게 데이터를 읽을지 표현하는 언어가 아니다.
그래서 쿼리가 빠르게 수행되게 하려면 데이터베이스 서버에서 쿼리가 어떻게 요청을 처리하는지 예측할 수 있어야 한다.

## MySQL 연산자와 내장 함수

### 리터럴 표기법 문자열

#### 숫자

```sql
SELECT * FROM tab_test WHERE number_column='10001';
SELECT * FROM tab_test WHERE string_column=10001;
```

위 쿼리와 같이 두 비교 대상이 문자열과 숫자 타입으로 다를 때는 자동으로 타입 변환이 발생한다.
MySQL은 숫자 타입과 문자열 타입 간의 비교에서 숫자 타입을 우선시하므로 문자열 값을 숫자 값으로 변환한 후 비교를 수행한다.
  
따라서 첫 번째 쿼리는 주어진 상숫값을 숫자로 변환하는데 성능과 관련된 문제가 발생하지 않는다.  
  
반면 두 번째 쿼리는 주어진 상숫값이 숫자 값인데, 비교되는 칼럼은 문자열 칼럼이다. 이때 MySQL은 문자열 칼럼을 숫자로 변환해서 비교한다.
즉, `string_column` 칼럼의 모든 문자열 값을 숫자로 변환해서 비교를 수행해야 하므로 `string_column`에 인덱스가 있더라도 이를 활용하지 못한다.
`string_column`에 알파벳과 같은 문자가 포함된 경우에는 숫자 값으로 변환할 수 없으므로 쿼리 자체가 실패할 수도 있다.
  
원천적으로 이러한 문제점을 제거하려면 숫자 값은 숫자 타입의 칼럼에만 저장해야 한다.  
주로 코드나 타입과 같은 값을 저장하는 칼럼에서 이 같은 현상이 자주 발생하므로 주의해야 한다.

#### 날짜

MySQL 에서는 정해진 형태의 날짜 포맷으로 표기하면 MySQL 서버가 자동으로 `DATE`나 `DATETIME` 값으로 변환하기 때문에 복잡하게
`STR_TO_DATE()` 같은 함수를 사용하지 않아도 된다.

```sql
SELECT * FROM dept_emp WHERE from_date='2011-04-29';
# SELECT * FROM dept_emp WHERE from_date=STR_TO_DATE('2011-04-29', '%Y-%m-%d');
```

위 두 쿼리의 차이점은 없다.  
첫 번째 쿼리와 같이 비교한다고 해서 from_date 칼럼의 값을 문자열로 변환하지 않기 때문에 from_date 칼럼으로 생성된 인덱스를 이용하는 데 문제가 없다.


### MySQL 연산자

#### 동등 비교

MySQL에서는 동등 비교를 위해 `<=>` 연산자도 제공한다.  
`<=>` 연산자는 `=` 연산자와 같으며, 부가적으로 `NULL` 값에 대한 비교까지 수행한다. 이 연산자를 `NULL-Safe` 비교 연산자라고 한다.

```sql
mysql> SELECT 1 = 1, NULL = NULL, 1 = NULL;
+-------+-------------+----------+
| 1 = 1 | NULL = NULL | 1 = NULL |
+-------+-------------+----------+
|     1 |        NULL |     NULL |
+-------+-------------+----------+

mysql> SELECT 1 <=> 1, NULL <=> NULL, 1 <=> NULL;
+---------+---------------+------------+
| 1 <=> 1 | NULL <=> NULL | 1 <=> NULL |
+---------+---------------+------------+
|       1 |             1 |          0 |
+---------+---------------+------------+
```

### REGEXP 연산자 (RLIKE)

문자열 값이 어떤 패턴을 만족하는지 확인하는 연산자이다.  
`REGEXP` 연산자를 사용하려면 다음 예제와 같이 `REGEXP` 연산자의 좌측에 비교 대상 문자열 값 또는 문자열 칼럼을, 우측에 검증하고자 하는 정규 표현식을 사용하면 된다.

```sql
mysql> SELECT 'abc' REGEXP '^[x-z]';
+-----------------------+
| 'abc' REGEXP '^[x-z]' |
+-----------------------+
|                     0 |
+-----------------------+
```

`REGEXP` 연산자를 문자열 칼럼에 비교할 때 `REGEXP` 조건의 비교는 인덱스 레인지 스캔을 사용할 수 없다. 
따라서 WHERE 조건절에 `REGEXP` 연산자를 사용한 조건을 단독으로 사용하는 것은 성능상 좋지 않다.
가능하다면 데이터 조회 범위를 줄일 수 있는 조건과 함께 `REGEXP` 연산자를 사용하길 권장한다.

### LIKE 연산자

REGEXP 연산자 보다는 훨씬 단순한 문자열 패턴 비교 연산자이지만 DBMS에서는 LIKE 연산자를 더 많이 사용한다.
LIKE 연산자는 인덱스를 사용할 수 있고, 정규 표현식을 검사하는 것이 아니라 어떤 상수 문자열이 있는지 없는지 정도를 판단하는 연산자다.

```sql
mysql> SELECT 'a%' LIKE 'a%';
+----------------+
| 'a%' LIKE 'a%' |
+----------------+
|              1 |
+----------------+

mysql> SELECT 'a%' LIKE 'a/%' ESCAPE '/';
+----------------------------+
| 'a%' LIKE 'a/%' ESCAPE '/' |
+----------------------------+
|                          1 |
+----------------------------+
```

와일드카드 문자인 '%'나 '_' 문자 자체를 비교한다면 ESCAPE 절을 LIKE 조건 뒤에 추가해 이스케이프 문자를 설정할 수 있다.
  
LIKE 연산자는 와일드카드 문자인 (%, _)가 검색어의 뒤쪽에 있다면 인덱스 레인지 스캔으로 사용할 수 있지만 와일드카드가
검색어의 앞족에 있다면 인덱스 레인지 스캔을 사용할 수 없으므로 주의해서 사용해야 한다.

```sql
mysql> EXPLAIN
    -> SELECT COUNT(*)
    -> FROM employees
    -> WHERE first_name LIKE 'Christ%';
+----+-----------+-------+--------------+------+--------------------------+
| id | table     | type  | key          | rows | Extra                    |
+----+-----------+-------+--------------+------+--------------------------+
|  1 | employees | range | ix_firstname | 1157 | Using where; Using index |
+----+-----------+-------+--------------+------+--------------------------+
```
`Christ`로 시작하는 이름을 검색하려면 다음과 같이 인덱스 레인지 스캔을 이용해 검색할 수 있다.

```sql
mysql> EXPLAIN
    -> SELECT COUNT(*)
    -> FROM employees
    -> WHERE first_name LIKE '%rist';
+----+-----------+-------+--------------+--------+--------------------------+
| id | table     | type  | key          | rows   | Extra                    |
+----+-----------+-------+--------------+--------+--------------------------+
|  1 | employees | index | ix_firstname | 299477 | Using where; Using index |
+----+-----------+-------+--------------+--------+--------------------------+
```
하지만 `rist`로 끝나는 이름을 검색할 때는 와일드카드가 검색어 앞쪽에 있게 되는데, 이 경우 인덱스 레인지 스캔을 사용하지 못하고
인덱스를 처음부터 끝까지 읽는 인덱스 풀 스캔 방식으로 쿼리가 처리된다.

### BETWEEN 연산자

## Reference 

**위 글은 책 RealMySQL 8.0 2권을 구입하여 읽고 정리한 내용입니다.**
- [도서 홈페이지 https://wikibook.co.kr/realmysql802/](https://wikibook.co.kr/realmysql802/)

