# 쿼리 작성 및 최적화

## SELECT

### 서브쿼리

쿼리를 작성할 때 서브쿼리를 사용하면 단위 처리별로 쿼리를 독립적으로 작성할 수 있다.
조인처럼 여러 테이블을 섞어 두는 형태가 아니어서 쿼리의 가독성도 높아지며, 복잡한 쿼리도 손쉽게 작성할 수 있다.
8.0 버전부터는 서브쿼리 처리가 많이 개선됐다.
  
서브 쿼리는 여러 위치에서 사용될 수 있다. 대표적으로 SELECT 절과 FROM 절, WHERE 절이다.
사용되는 위치에 따라 쿼리의 성능 영향도와 MySQL 서버의 최적화 방법은 완전히 달라진다.

### SELECT 절 서브 쿼리

SELECT 절에 사용된 서브쿼리는 내부적으로 임시 테이블을 만들거나 쿼리를 비효율적으로 실행하게 만들지는 않는다.
그래서 서브쿼리가 적절히 인덱스를 사용할 수 있다면 크게 주의할 사항은 없다.
  
일반적으로 SELECT 절에 서브쿼리를 사용하면 그 서브쿼리는 항상 칼럼과 레코드가 하나인 결과를 반환해야 한다.
그 값이 NULL이든 아니든 레코드가 1건이 존재해야 한다는 것인데 MySQL에서는 이 체크 조건이 조금 느슨하다.

```SQL
mysql> SELECT emp_no, (SELECT dept_name FROM departments WHERE dept_name='Sales1')
    -> FROM dept_emp LIMIT 10;
+--------+--------------------------------------------------------------+
| emp_no | (SELECT dept_name FROM departments WHERE dept_name='Sales1') |
+--------+--------------------------------------------------------------+
| 110022 | NULL                                                         |
| 110085 | NULL                                                         |
| 110183 | NULL                                                         |
| 110303 | NULL                                                         |
| 110511 | NULL                                                         |
| 110725 | NULL                                                         |
| 111035 | NULL                                                         |
| 111400 | NULL                                                         |
| 111692 | NULL                                                         |
| 110114 | NULL                                                         |
+--------+--------------------------------------------------------------+

mysql> SELECT emp_no, (SELECT dept_name FROM departments)
    -> FROM dept_emp LIMIT 10;
ERROR 1242 (21000): Subquery returns more than 1 row

mysql> SELECT emp_no, (SELECT dept_no, dept_name FROM departments WHERE dept_name='Sales1')
    -> FROM dept_emp LIMIT 10;
ERROR 1241 (21000): Operand should contain 1 column(s)
```

- 첫 번째 쿼리에서 사용된 서브쿼리는 항상 결과가 0건이다. 하지만 첫 번째 쿼리는 에러를 발생시키지 않고 서브쿼리 결과가 NULL로 채워져서 반환된다.
- 두 번째 쿼리에서 서브쿼리가 2건 이상의 레코드르 반환하는 경우에는 에러가 나면서 쿼리가 종료된다.
- 세 번째 쿼리와 같이 SELECT 절에 사용된 서브쿼리가 2개 이상의 칼럼을 가져오려고 할 때도 에러가 발생한다.

즉 SELECT 절의 서브쿼리에는 `로우 서브쿼리`를 사용할 수 없고, 오로지 `스칼라 서브쿼리`만 사용할 수 있다.

> 서브쿼리는 만들어 내는 결과에 따라 스칼라 서브쿼리(Scalar subquery), 로우 서브쿼리(Row subquery)로 구분할 수 있다.
> 스칼라 서브쿼리는 레코드의 칼럼이 각각 하나인 결과를 만들어내는 서브쿼리며,
> 스칼라 서브쿼리보다 레코드 건수가 많거나 칼럼 수가 많은 결과를 만들어내는 서브쿼리를 로우 서브쿼리라고 한다.

가끔 조인으로 처리해도 되는 쿼리를 SELECT 서브 쿼리를 사용해서 작성할 때도 있는데, 서브쿼리로 실행될 때보다 조인으로 처리할 때가 조금 더 빠르다.
그래서 가능하면 조인으로 쿼리를 작성하는 것이 좋다.

### FROM 절 서브 쿼리

이전 버전의 MySQL 서버에서는 FROM 절에 서브쿼리가 사용되면 항상 임시 테이블을 생성하는 방식으로 처리했다.
하지만 5.7버전부터는 옵티마이저가 FROM 절의 서브쿼리를 외부 쿼리로 병합하는 최적화를 수행하도록 개선됐다.
  
모두 다 가능한 것은 아니고 대표적으로 서브쿼리에 아래 기능들이 사용되면 FROM 절의 서브쿼리는 외부 쿼리로 병합할 수 없다.

- 집합 함수 사용
- DISTINCT
- GROUP BY, HAVING
- LIMIT
- UNION
- SELECT 절에 서브쿼리가 사용된 경우
- 사용자 변수 사용

외부 쿼리와 병합하는 FROM 절의 서브쿼리가 ORDER BY 절을 가진 경우, 외부 쿼리가 GROUP BY나 DISTINCT 같은 기능을 사용하지 않는다면 서브쿼리의 정렬 조건을 외부쿼리로 같이 병합한다. 외부 쿼리에서 GROUP BY나 DISTINCT 같은 기능이 사용되고 있다면 서브쿼리의 정렬 작업은 무의미하기 때문에 서브쿼리의 ORDER BY 절은 무시된다.

### WHERE 절 서브 쿼리

WHERE 절 서브쿼리는 다양한 형태로 사용될 수 있다.

- 동등 또는 크다 작다 비교
- IN 비교
- NOT IN 비교

#### 동등 또는 크다 작다 비교

```SQL
SELECT * FROM dept_emp de
WHERE de.emp_no=(SELECT e.emp_no
                 FROM employees e
                 WHERE e.first_name='Georgi' AND e.last_name='Facello' LIMIT 1);
```

5.5 이전 버전까지는 서브 쿼리의 외부 조건으로 쿼리를 실행하고, 최종적으로 서브쿼리를 체크 조건으로 사용했다.
하지만 이러한 처리 방식은 풀 테이블 스캔이 필요한 경우가 많아 성능 저하가 심각했다.
  
5.5 버전 부터는 서브 쿼리를 먼저 실행한 후 상수로 변환한다. 그리고 상숫값으로 서브쿼리를 대체해서 나머지 쿼리 부분을 처리한다.
  
```sql
mysql> EXPLAIN
-> SELECT *
-> FROM dept_emp de WHERE (emp_no, from_date) = (
->     SELECT emp_no, from_date
->     FROM salaries
->     WHERE emp_no=100001 limit 1);
+----+-------------+----------+------+---------+--------+-------------+
| id | select_type | table    | type | key     | rows   | Extra       |
+----+-------------+----------+------+---------+--------+-------------+
|  1 | PRIMARY     | de       | ALL  | NULL    | 331143 | Using where |
|  2 | SUBQUERY    | salaries | ref  | PRIMARY |      4 | Using index |
+----+-------------+----------+------+---------+--------+-------------+
```

그런데 위와 같이 단일 값 비교가 아닌 튜플 비교 방식이 사용되면 서브쿼리가 먼저 처리되어 상수화되기는 하지만 외부 쿼리는
인덱스를 사용하지 못하고 풀 테이블 스캔을 실행하게된다.

#### IN 비교

```sql
SELECT *
FROM employees e
WHERE e.emp_no IN (SELECT de.emp_no FROM dept_emp de WHERE de.from_date='1999-01-01');
```

실제 조인은 아니지만 테이블의 레코드가 다른 테이블의 레코드를 이용한 표현식과 일치하는지를 체크하는 형태를 `세미 조인`이라고 한다.
즉 WHERE 절에 사용된 IN (subquery) 형태의 조건을 조인의 한 방식인 세미 조인이라고 보는 것이다.
  
5.5 버전까지는 세미 조인 최적화가 부족해서 대부분 풀 테이블 스캔을 실행했다. 하지만 5.6 버전부터 세미 조인의 최적화가 많이 개선되었다. MySQL의 세미 조인 최적화는 쿼리 특성이나 조인 관계에 맞게 5개의 최적화 전략을 선택적으로 사용한다.

- 테이블 풀 아웃(Table Pull-out)
- 퍼스트 매치(Firstmatch)
- 루스 스캔(Loosescan)
- 구체화(Materialization)
- 중복 제거(Duplicated Weed-out)

MySQL 8.0을 사용한다면 세미 조인 최적화에 익숙해져야 한다. 

#### NOT IN 비교

IN (subquery)와 비슷한 형태지만 이 경우를 `안티 세미 조인`이라고 부른다.





