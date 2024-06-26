# 실행 계획

DBMS의 목적은 많은 데이터를 안전하게 저장 및 관리하고 사용자가 원하는 데이터를 빠르게 조회할 수 있도록 돕는 것이다.  
이를 달성하려면 옵티마이저가 사용자의 쿼리를 최적으로 처리될 수 있도록하는 `올바른 쿼리 실행계획`을 세울 수 있어야 한다.

옵티마이저가 항상 올바른 쿼리 실행계획을 세우는 것이 아니기 때문에  
DBMS는 관리자나 사용자가 옵티마이저가 수립한 실행 계획을 확인할 수 있는 `EXPLAIN` 명령어를 지원한다.

비용 기반 최적화에서 가장 중요한 것은 `통계 정보`다.  
통계 정보가 정확하지 않으면 옵티마이저는 엉뚱한 쿼리 실행계획을 수립하게 되고 비 효율적인 방향으로 쿼리를 실행하게 된다.  
> 실제로는 1억건의 레코드가 저장되어있지만 통계 정보가 갱신되지 않아 10건의 레코드가 저장되어 있는 것 처럼 되어있다면?

MySQL 5.5버전까지는 각 테이블의 통계 정보가 메모리에 저장되었지만, 5.6버전부터 테이블로 관리할 수 있도록 개선됐다.  
> `innodb_index_stats`, `innodb_table_stats`

또한 테이블을 생성할 때 `STATS_PERSISTENCE` 옵션을 설정해서 통계 정보를 메모리(휘발), 테이블 단위(비휘발)로 선택해서 결정하는 것이 가능하다.
> `STATS_PERSISTENCE`의 기본 값은 1, 테이블에 통계 정보를 저장한다.

MySQL 5.5 버전까지 메모리에 통계 정보를 저장함에 따라 서버가 재시작될 시 통계 정보가 초기화되거나  
사용자 혹은 관리자가 알지 못하는 순간에 통계 정보가 갱신되어 일관적이지 않은 쿼리 실행 계획이 수립되는 경우가 많았다.

- 테이블이 새로 오픈되는 경우
- 테이블의 레코드가 대량으로 변경되는 경우(테이블 전체 레코드의 1/16)
- ANALYZE TABLE 명령이 실행되는 경우
- SHOW TABLE STATUS 명령이나 SHOW INDEX FROM 명령이 실행되는 경우
- InnoDB 모니터가 활성화되는 경우
- innodb_stats_on_metadata 시스템 설정이 ON인 상태에서 SHOW TABLE STATUS 명령이 실행되는 경우
  
영구적인 통계 정보(테이블)가 도입되면서 이를 막을 수 있게 되었다. 추가로 `innodb_stats_auto_recalc` 옵션을 이용해서 자동으로 통계 정보를 갱신할지 선택할 수 있다.
OFF로 설정한다면 통계 정보가 자동적으로 갱신되는 것을 막고 원하는 시점의 영구적인 통계 정보를 이용하게 될 것이다.  
> 통계 정보를 자동으로 수집할지 여부도 테이블을 생성할 때 결정할 수 있다. `STATS_AUTO_RECALC` 옵션  
> 옵션을 1로 설정하면 위에 나열된 이벤트 발생 시마다 통계 정보를 업데이트한다. 기본 값은 0이다.
  
영구적인 통계 정보를 사용하면 MySQL 서버의 점검이나 사용량이 많지 않은 시간을 이용해 더 정확한 통계 정보를 수집할 수도 있다. 
더 정확한 통계 정보 수집에는 많은 시간이 소요되겠지만, 해당 통계 정보의 정확성에 의해 쿼리의 성능이 결정되기 때문에 충분히 시간을 투자할 가치가 있다.  

## 히스토그램

MySQL 5.7 버전까지의 통계 정보는 단순히 인덱스된 칼럼의 유니크 값의 개수 정도만 가지고 있었는데, 이는 옵티마이저가 최적의 실행 계획을 수립하기엔 많이 부족했다.  
그래서 옵티마이저는 실행 계획을 세우는 시점에 실제 인덱스의 일부 페이지를 랜덤으로 가져와 참조하는 방식을 사용했다.
  
그런데 8.0 버전으로 업그레이드되면서 칼럼의 데이터 분포도를 참조할 수 있는 히스토그램 정보를 활용할 수 있도록 변경됐다.
  
8.0 버전에서 히스토그램 정보는 칼럼 단위로 관리되는데, 히스토그램 수집 명령어를 수동으로 실행해야한다.  
수집된 히스토그램 정보는 시스템 딕셔너리에 함께 저장되고, MySQL 서버가 시작될 때 딕셔너리 히스토그램 정보를 information_schema DB의 column_statistics 테이블로 로드한다.
  
그래서 실제 히스토그램 정보를 column_statistics 테이블을 조회해서 확인할 수 있다.

8.0 버전에서는 2종류의 히스토그램 타입이 지원된다.

- `Singleton`: 칼럼값 개별로 레코드 건수를 관리한다. Value-Based 히스토그램 또는 도수 분포라고도 불린다.
- `Equi-Height`: 칼럼값 범위를 균등한 개수로 구분해서 관리한다. Height-Balanced 히스토그램이라고도 불린다.

히스토그램은 `버킷` 단위로 구분되어 레코드 건수나 칼럼값의 범위가 관리된다.
  
싱글톤 히스토그램은 각 버킷이 `칼럼의 값`과 `발생 빈도의 비율` 2가지 값을 가진다.  
반면 높이 균형 히스토그램에서는 `범위 시작 값`과 `마지막 값`, `발생 빈도율`, `각 버킷에 포함된 유니크한 값의 개수` 4가지 값을 가진다.

> 히스토그램을 생성할 때는 `샘플링 비율`이 정확도에 영향을 미친다. 정확도를 위해 테이블을 전부 스캔한다면 시스템에 부하를 주고 자원을 많이 소모하게 될 것이다.  
> MySQL 서버는 histogram_generation_max_mem_size 시스템 변수에 설정된 메모리 크기에 맞게 적절히 샘플링한다.  
> 8.0.19 미만 버전까지는 샘플링 비율과 시스템 변수 크기에 상관없이 풀 스캔으로 히스토그램을 생성했지만, 8.0.19 부터 스토리지 엔진 자체 샘플링 알고리즘을 구현하여 더이상 풀 테이블 스캔이 필요 없어졌다.

히스토그램 삭제는 딕셔너리 내용만 삭제하기 때문에 다른 쿼리 처리에 영향을 주지 않는다.  
당연하게도 히스토그램이 사라지면 쿼리 실행 계획이 달라질 수 있다.
또는 히스토그램을 삭제하지 않고 옵티마이저가 히스토그램을 사용하지 않도록 변경할 수도 있다.
특정 커넥션 또는 특정 쿼리에서만 히스토그램을 사용하지 않도록 설정하는 것도 가능하다.

### 용도

히스토그램이 도입되기 이전에는 테이블의 전체 레코드 건수와 인덱스된 칼럼이 가지는 유니크한 값의 개수 정도만을 통계정보로 가지고 있었다.  
예시로 테이블의 레코드가 1000건이고, 칼럼의 유니크 값 개수가 100개였다면 MySQL 서버는 동등 비교 검색을 하면 대략 10개의 레코드가 일치할 것이라 예측한다.
  
즉 데이터가 균등하게 분포되어 있을 것으로 예측한다. 하지만 데이터는 항상 균등한 분포도를 가지지 않는다.  
히스토그램은 특정 칼럼이 가지는 모든 값에 대한 분포도 정보를 가지지는 않지만, 각 버킷(범위)별로 레코드의 건수와 유니크한 값의 개수 정보를 가지기 때문에 훨씬 정확한 예측이 가능하다.

```sql
mysql> explain
    -> select *
    -> from employees
    -> where first_name='Zita'
    -> and birth_date between '1950-01-01' AND '1960-01-01';
+----+-------------+-----------+------------+------+---------------+--------------+---------+-------+------+----------+-------------+
| id | select_type | table     | partitions | type | possible_keys | key          | key_len | ref   | rows | filtered | Extra       |
+----+-------------+-----------+------------+------+---------------+--------------+---------+-------+------+----------+-------------+
|  1 | SIMPLE      | employees | NULL       | ref  | ix_firstname  | ix_firstname | 58      | const |  224 |    11.11 | Using where |
+----+-------------+-----------+------------+------+---------------+--------------+---------+-------+------+----------+-------------+
1 row in set, 1 warning (0.00 sec)

mysql> analyze table employees
    -> update histogram on first_name, birth_date;

mysql> explain
    -> select *
    -> from employees
    -> where first_name='Zita'
    -> and birth_date between '1950-01-01' AND '1960-01-01';
+----+-------------+-----------+------------+------+---------------+--------------+---------+-------+------+----------+-------------+
| id | select_type | table     | partitions | type | possible_keys | key          | key_len | ref   | rows | filtered | Extra       |
+----+-------------+-----------+------------+------+---------------+--------------+---------+-------+------+----------+-------------+
|  1 | SIMPLE      | employees | NULL       | ref  | ix_firstname  | ix_firstname | 58      | const |  224 |    60.85 | Using where |
+----+-------------+-----------+------------+------+---------------+--------------+---------+-------+------+----------+-------------+

mysql> select
    -> sum(case when birth_date between '1950-01-01' and '1960-01-01' then 1 else 0 end) / count(*) as ratio
    -> from employees where first_name='Zita';
+--------+
| ratio  |
+--------+
| 0.6384 |
+--------+
```

위 예시는 employees 테이블의 birth_date 칼럼에 대해 히스토그램이 없을 때와 히스토그램이 있을 때의 예측치가 얼마나 달라지는지 확인한 것이다.  
옵티마이저는 first_name='Zita' 조건에 일치하는 레코드가 224건이 있고, 그중에서 대략 11.11%가 1950년대 출생일 것으로 예측했다.  
하지만 히스토그램을 사용한 실행 계획에서는 대략 60.85%가 1950년대 출생인 것으로 예측했고, 실제로 데이터를 조회해보면 63.8%인 것을 확인할 수 있다.  
이처럼 단순 통계 정보만 이용한 경우와 히스토그램을 이용한 경우의 차이가 매우 큰 것을 알 수 있다.
  
## 히스토그램과 인덱스

MySQL 서버에서는 쿼리의 실행 계획을 수립할 때 사용 가능한 인덱스들로부터 조건절에 일치하는 레코드 건수를 대략 파악하고 최종적으로 가장 나은 실행 계획을 선택한다.
이 때 조건절에 일치하는 레코드 건수를 예측하기 위해 옵티마이저는 실제 인덱스의 B-Tree를 샘플링해서 살펴본다. 이 작업을 매뉴엘에서는 `인덱스 다이브(Index Dive)`라고 표현한다.

MySQL 8.0 서버에서 인덱스된 칼럼을 검색 조건으로 사용하는 경우, 그 칼럼의 히스토그램은 사용하지 않고 실제 인덱스 다이브를 통해 직접 수집한 정보를 활용한다.
이는 실제 검색 조건 대상 값에 대한 샘플링을 실행하는 것이므로 항상 히스토그램보다 정확한 결과를 기대할 수 있다.
  
그래서 MySQL 8.0 버전에서 히스토그램은 주로 인덱스되지 않은 칼럼에 대한 데이터 분포도를 참조하는 용도로 사용된다.  
하지만 인덱스 다이브 작업은 어느정도 비용이 필요하고, IN 절에 값이 많이 명시된 경우 실행 계획 수립만으로도 상당한 인덱스 다이브를 실행해서 비용이 그만큼 커질 수 있다.

## 코스트 모델

MySQL 서버가 쿼리를 처리하기 위해선 다음과 같은 다양한 작업들이 필요하다.

- 디스크로부터 데이터 페이지 읽기
- InnoDB 버퍼 풀로부터 데이터 페이지 읽기
- 인덱스 키 비교
- 레코드 평가
- 메모리 임시 테이블 작업
- 디스크 임시 테이블 작업

최적의 실행 계획을 찾기 위해서는 사용자 쿼리에 대해 이러한 다양한 작업이 얼마나 필요한지 예측해서 전체 작업 비용을 계산할 수 있어야 한다.  
이 때 필요한 단위 작업들의 비용을 `코스트 모델`이라고 한다.
  
MySQL 8.0 서버의 코스트 모델은 `server_cost`, `engine_cost` 2개 테이블에 저장돼 있는 설정값을 사용한다.  

- sever_cost
  - 인덱스를 찾고 레코드를 비교하고 임시 테이블 처리에 대한 비용을 관리한다.
- engine_cost
  - 레코드를 가진 데이터 페이지를 가져오는데 필요한 비용을 관리한다.
 
MySQL 8.0 버전의 코스트 모델에서 지원하는 단위 작업은 다음과 같이 8개다.

|                 | **cost_name**                | **default_value** | **설명**                         |
|:---------------:|------------------------------|-------------------|----------------------------------|
| **engine_cost** | io_block_read_cost           | 1.00              | 디스크 데이터 페이지 읽기        |
|                 | memory_block_read_cost       | 0.25              | 메모리 데이터 페이지 읽기        |
| **server_cost** | disk_temptable_create_cost   | 20.00             | 디스크 임시 테이블 생성          |
|                 | disk_temptable_row_cost      | 0.50              | 디스크 임시 테이블의 레코드 읽기 |
|                 | key_compare_cost             | 0.05              | 인덱스 키 비교                   |
|                 | memory_temptable_create_cost | 1.00              | 메모리 임시 테이블 생성          |
|                 | memory_temptable_row_cost    | 0.10              | 메모리 임시 테이블의 레코드 읽기 |
|                 | row_evaluate_cost            | 0.10              | 레코드 비교                      |

코스트 모델에서 중요한 것은 각 단위 작업에 설정되는 비용 값이 커지면 어떤 실행 계획들이 고비용으로 바뀌고 어떤 실행 계획들이
저비용으로 바뀌는지를 파악하는 것이다.

- key_compare_cost 비용을 높이면 옵티마이저가 가능한 정렬을 수행하지 않는 방향의 실행 계획을 선택할 가능성이 높아진다.
- row_evaluate_cost 비용을 높이면 풀 스캔을 실행하는 쿼리들의 비용이 높아지고 옵티마이저는 가능한 인덱스 레인지 스캔을 사용하는 실행 계획을 선택할 가능성이 높아진다.
- 등등..

## 실행 계획 분석

```sql
mysql> explain
    -> select *
    -> from employees e
    -> inner join salaries s on s.emp_no=e.emp_no
    -> where first_name='ABC';
+----+-------------+-------+------------+------+----------------------+--------------+---------+--------------------+------+----------+-------+
| id | select_type | table | partitions | type | possible_keys        | key          | key_len | ref                | rows | filtered | Extra |
+----+-------------+-------+------------+------+----------------------+--------------+---------+--------------------+------+----------+-------+
|  1 | SIMPLE      | e     | NULL       | ref  | PRIMARY,ix_firstname | ix_firstname | 58      | const              |    1 |   100.00 | NULL  |
|  1 | SIMPLE      | s     | NULL       | ref  | PRIMARY              | PRIMARY      | 4       | employees.e.emp_no |    9 |   100.00 | NULL  |
+----+-------------+-------+------------+------+----------------------+--------------+---------+--------------------+------+----------+-------+
2 rows in set, 1 warning (0.00 sec)
```

MySQL 8.0 부터는 EXPLAIN 명령 결과로 출력되는 실행 계획 포맷을 기존 테이블 포맷과 JSON, TREE 형태로 선택할 수 있다.  
아무런 옵션 없이 EXPLAIN 명령을 실행하면 쿼리 문장의 특성에 따라 표 형태로 1줄 이상의 결과가 표시된다.
  
표의 각 레코드는 쿼리 문장에서 사용된 테이블의 개수만큼 출력된다.  
실행 계획은 위에서 아래로 순서대로 표시된다.(UNION이나 상관 서브쿼리의 경우 순서대로 표시되지 않을 수 있음)
출력된 실행 계획에서 위쪽에 출력된 결과일수록 쿼리의 바깥 부분이거나 먼저 접근한 테이블이다.

### 분석 - id 칼럼

하나의 SELECT 문장은 다시 1개 이상의 하위 SELECT 문장을 포함할 수 있다.  
실행 계획에서 가장 왼쪽에 표시되는 id 칼럼은 단위 SELECT 쿼리별로 부여되는 식별자 값이다.

하나의 SELECT 문장 안에서 여러 개의 테이블을 조인하면 조인되는 테이블의 개수만큼 실행 계획 레코드가 출력되지만 같은 id 값이 부여된다.  

```sql
mysql> explain
    -> select
    -> ( (select count(*) from employees) + (select count(*) from departments) ) as total_count;
+----+-------------+-------------+------------+-------+---------------+-------------+---------+------+--------+----------+----------------+
| id | select_type | table       | partitions | type  | possible_keys | key         | key_len | ref  | rows   | filtered | Extra          |
+----+-------------+-------------+------------+-------+---------------+-------------+---------+------+--------+----------+----------------+
|  1 | PRIMARY     | NULL        | NULL       | NULL  | NULL          | NULL        | NULL    | NULL |   NULL |     NULL | No tables used |
|  3 | SUBQUERY    | departments | NULL       | index | NULL          | ux_deptname | 162     | NULL |      9 |   100.00 | Using index    |
|  2 | SUBQUERY    | employees   | NULL       | index | NULL          | ix_hiredate | 3       | NULL | 299969 |   100.00 | Using index    |
+----+-------------+-------------+------------+-------+---------------+-------------+---------+------+--------+----------+----------------+
3 rows in set, 1 warning (0.00 sec)
```
위 쿼리의 실행 계획의 경우 쿼리 문장이 3개의 단위 select 쿼리로 구성돼 있으므로 실행 계획의 각 레코드가 각기 다른 id값을 지닌다.  

```sql
mysql> explain
    -> select * from dept_emp de
    -> where de.emp_no= (select e.emp_no
    -> from employees e
    -> where e.first_name='Georgi'
    -> and e.last_name='Facello' limit 1);
+----+-------------+-------+------------+------+-------------------+-------------------+---------+-------+------+----------+-------------+
| id | select_type | table | partitions | type | possible_keys     | key               | key_len | ref   | rows | filtered | Extra       |
+----+-------------+-------+------------+------+-------------------+-------------------+---------+-------+------+----------+-------------+
|  1 | PRIMARY     | de    | NULL       | ref  | ix_empno_fromdate | ix_empno_fromdate | 4       | const |    1 |   100.00 | Using where |
|  2 | SUBQUERY    | e     | NULL       | ref  | ix_firstname      | ix_firstname      | 58      | const |  253 |    10.00 | Using where |
+----+-------------+-------+------------+------+-------------------+-------------------+---------+-------+------+----------+-------------+
2 rows in set, 1 warning (0.01 sec)
```
주의할 점은 실행 계획의 id 칼럼이 테이블의 접근 순서를 의미하지는 않는다는 것이다.
위 쿼리는 employees 테이블을 먼저 읽고 그 결과를 dept_emp 테이블과 조인함에도 불구하고 id가 순서와 일치하지 않는 것을 확인할 수 있다.

### 분석 - select_type 칼럼

select_type 칼럼에서는 각 단위 select 쿼리가 어떤 타입의 쿼리인지 표시되는 칼럼이다.  
  
다음과 같은 값들이 표시될 수 있다.
- `SIMPLE`
  - UNION이나 서브쿼리를 사용하지 않는 단순한 SELECT 쿼리의 경우 SIMPLE로 표시된다(조인도 마찬가지).
  - 쿼리 문장이 아무리 복잡하더라도 실행 계획에서 SIMPLE인 단위 SELECT 쿼리는 단 하나만 존재한다.
  - 일반적으로 가장 바깥 SELECT 쿼리가 SIMPLE로 표시된다.
- `PRIMARY`
  - UNION이나 서브쿼리를 가지는 SELECT 쿼리의 경우 가장 바깥쪽에 있는 단위 SELECT 쿼리는 PRIMARY로 표시된다.
  - SIMPLE과 마찬가지로 PRIMARY인 단위 SELECT 쿼리는 하나만 존재한다.
- `UNION`
  - UINON으로 결합하는 단위 SELECT 쿼리 가운데 `첫 번째를 제외한 두 번째 이후 단위 SELECT` 쿼리는 UNION으로 표시된다.
  - UNION의 첫 번째 단위 SELECT 쿼리는 UNION이 아니라 UNION되는 쿼리 결과들을 모아 저장하는 임시 테이블 DERIVED가 표시된다.
- `DEPENDENT_UNION`
  - DEPENDENT란 UNION이나 UNION ALL로 결합된 단위 쿼리가 외부 쿼리에 영향을 받는 것을 의미한다.
  - ```SQL
    mysql> explain
    -> select *
    -> from employees e1 where e1.emp_no in (
    -> select e2.emp_no from employees e2 where e2.first_name='Matt'
    -> union
    -> select e3.emp_no from employees e3 where e3.last_name='Matt'
    -> );
    +----+--------------------+------------+------------+--------+----------------------+---------+---------+------+--------+----------+-----------------+
    | id | select_type        | table      | partitions | type   | possible_keys        | key     | key_len | ref  | rows   | filtered | Extra           |
    +----+--------------------+------------+------------+--------+----------------------+---------+---------+------+--------+----------+-----------------+
    |  1 | PRIMARY            | e1         | NULL       | ALL    | NULL                 | NULL    | NULL    | NULL | 299969 |   100.00 | Using where     |
    |  2 | DEPENDENT SUBQUERY | e2         | NULL       | eq_ref | PRIMARY,ix_firstname | PRIMARY | 4       | func |      1 |     5.00 | Using where     |
    |  3 | DEPENDENT UNION    | e3         | NULL       | eq_ref | PRIMARY              | PRIMARY | 4       | func |      1 |    10.00 | Using where     |
    |  4 | UNION RESULT       | <union2,3> | NULL       | ALL    | NULL                 | NULL    | NULL    | NULL |   NULL |     NULL | Using temporary |
    +----+--------------------+------------+------------+--------+----------------------+---------+---------+------+--------+----------+-----------------+
    ```
  - 위 쿼리의 경우 옵티마이저가 외부의 employees 테이블을 먼저 읽은 다음 서브 쿼리를 실행하는데, 이 때 employees 테이블의 칼럼 값이 서브쿼리에 영향을 준다.
  - 이렇게 내부 쿼리가 외부의 값을 참조해서 처리될 때 DEPENDENT 키워드가 표시된다.
  - 내부적으로는 UNION에 사용된 SELECT 쿼리의 WHERE 조건에 e2.emp_no=e1.emp_no와 e3.emp_no=e1.emp_no라는 조건이 자동으로 추가되어 실행된다.
  - 외부에 정의된 employees 테이블의 emp_no 칼럼이 서브쿼리에 사용되기 때문에 DEPENDENT UNION이 표시되는 것이다.
- `UNION RESULT`
  - UNION RESULT는 UNION 결과를 담아두는 테이블을 의미한다.
  - 8.0 버전부터 UNION ALL의 경우 임시 테이블을 사용하지 않도록 기능이 개선됐지만, UNION(또는 UNION DISTINCT)은 여전히 임시 테이블에 결과를 버퍼링한다.
  - 이 임시 테이블을 가리키는 select_type이 UNION RESULT 이고, 실제 쿼리의 단위 쿼리가 아니기 때문에 별도의 id 값은 부여되지 않는다.
  - UNION 대신 UNION ALL을 사용할 경우 해당 실행 계획 레코드는 보이지 않게 된다.
- `SUBQUERY`
  - select_type의 `SUBQUERY는 FROM절 이외`에서 사용되는 서브쿼리만을 의미한다.
  - MySQL 서버의 실행 계획에서 `FROM 절에 사용된 서브쿼리는 DERIVED`로 표시되고, `그 밖의 위치에서 사용된 서브쿼리는 전부 SUBQUERY`라고 표시된다.
  - 서브쿼리는 사용하는 위치에 따라 각각 다른 이름을 지니고 있다.
    - `중첩된 쿼리(Nested Query)`: SELECT되는 칼럼에 사용된 서브쿼리를 네스티드 쿼리라고 한다.
    - `서브쿼리(Subquery)`: WHERE 절에 사용된 경우에는 일반적으로 그냥 서브쿼리라고 한다.
    - `파생 테이블(Derived Query)`: FROM 절에 사용된 서브쿼리를 MySQL에서는 파생 테이블이라고 하며, 일반적으로 RDBMS에서는 인라인 뷰, 또는 서브 셀렉트라고 부른다.
  - 또한 서브쿼리가 반환하는 값의 특성에 따라 다음과 같이 구분하기도 한다.
    - `스칼라 서브쿼리(Scalar Subquery)`: 하나의 값만(칼럼이 단 하나인 레코드 1건만) 반환하는 쿼리
    - `로우 서브쿼리(Row Subquery)`: 칼럼의 개수와 관계없이 하나의 레코드만 반환하는 쿼리
- `DEPENDENT SUBQUERY`
  - 서브쿼리가 바깥쪽 SELECT 쿼리에서 정의된 칼럼을 사용하는 경우 DEPENDENT SUBQUERY라고 표시된다.
  - ```sql
    mysql> explain
    -> select e.first_name,
    -> (select count(*)
    -> from dept_emp de, dept_manager dm
    -> where dm.dept_no=de.dept_no and de.emp_no=e.emp_no) as cnt
    -> from employees e
    -> where e.first_name='Matt';
    +----+--------------------+-------+------------+------+---------------------------+-------------------+---------+----------------------+------+----------+-------------+
    | id | select_type        | table | partitions | type | possible_keys             | key               | key_len | ref                  | rows | filtered | Extra       |
    +----+--------------------+-------+------------+------+---------------------------+-------------------+---------+----------------------+------+----------+-------------+
    |  1 | PRIMARY            | e     | NULL       | ref  | ix_firstname              | ix_firstname      | 58      | const                |  233 |   100.00 | Using index |
    |  2 | DEPENDENT SUBQUERY | de    | NULL       | ref  | PRIMARY,ix_empno_fromdate | ix_empno_fromdate | 4       | employees.e.emp_no   |    1 |   100.00 | Using index |
    |  2 | DEPENDENT SUBQUERY | dm    | NULL       | ref  | PRIMARY                   | PRIMARY           | 16      | employees.de.dept_no |    2 |   100.00 | Using index |
    +----+--------------------+-------+------------+------+---------------------------+-------------------+---------+----------------------+------+----------+-------------+
    ```
  - 또한 DEPENDENT UNION과 같이 DEPENDENT SUBQUERY 또한 외부 쿼리가 먼저 수행된 후 서브쿼리가 실행돼야 하므로 DEPENDENT 키워드가 없는 일반 서브쿼리보다는 처리속도가 느릴 때가 많다.
- `DERIVED`
  - DERIVED는 단위 SELECT 쿼리의 실행 결과로 메모리나 디스크에 임시 테이블을 생성하는 것을 의미한다.
  - 5.5 버전까지는 서브쿼리가 FROM절에 사용된 경우 항상 select_type이 DERIVED였지만, 5.6 버전부터 옵티마이저 옵션에 따라 FROM 절의 서브쿼리를 외부 쿼리와 통합하는 형태의 최적화가 수행되기도 한다.
  - 5.5 버전까지는 파생 테이블에 인덱스가 전혀 없으므로 다른 테이블과 조인할 때 성능상 불리할 때가 많았는데, 5.6 버전부터 옵티마이저 옵션에 따라 쿼리의 특성에 맞게 임시 테이블에도 인덱스를 만들 수 있게 최적화 됐다.
  - MySQL 서버는 버전이 업그레이드되면서 조인 쿼리에 대한 최적화는 많이 성숙된 상태이므로 파생 테이블에 대한 최적화가 부족한 MySQL 서버를 사용 중일 경우, 가능하다면 DERIVED 형태의 실행 계획을 조인으로 해결할 수 있게 쿼리를 바꿔주는 것이 좋다.
  - 8.0 버전부터는 FROM 절의 서브쿼리에 대한 최적화도 많이 개선되어 가능하다면 불필요한 서브쿼리는 조인으로 쿼리를 재작성해서 처리한다.
  - ```
    쿼리를 튜닝하기 위해 실행 계획을 확인할 때 가장 먼저 select_type 칼럼의 값이 DERIVED인 것이 있는지 확인해야 한다.
    서브쿼리를 조인으로 해결할 수 있는 경우라면 서브쿼리보다는 조인을 사용할 것을 강력히 권장한다.
    ```
- `DEPENDENT DERIVED`
  - 8.0 이전 버전에서는 FROM 절의 서브쿼리는 외부 칼럼을 사용할 수 없었는데, 8.0 버전부터 래터럴 조인(LATERAL JOIN) 기능이 추가되었다.
  - DEPENDENT DERIVED 키워드는 해당 테이블이 래터럴 조인으로 사용된 것을 의미한다.
- `UNCACHEABLE SUBQUERY`
  - 하나의 쿼리 문장에 서브 쿼리가 하나만 있더라도 실제로 그 서브쿼리가 한 번만 실행되는게 아님
  - 그런데 조건이 똑같은 서브쿼리가 실행될 때는 다시 실행하지 않고 이전의 실행 결과를 그대로 사용할 수 있게 서브쿼리의 결과를 내부적인 캐시 공간에 담아둔다.
  - select_type이 SUBQUERY인 경우와 UNCACHEABLE SUBQUERY는 이 캐시를 사용할 수 있느냐 없느냐의 차이가 있다.
  - 서브쿼리에 포함된 요소에 의해 캐시 자체가 불가능할 수가 있는데, 그럴 경우 select_type이 UNCACHEABLE SUBQUERY로 표시된다.
- `UNCACHEABLE UNION`
- `MATERIALIZED`
  - 주로 FROM 절이나 IN 형태의 쿼리에 사용된 서브쿼리의 최적화를 위해 사용된다.
  - ```sql
    mysql> explain
    -> select *
    -> from employees e
    -> where e.emp_no in (select emp_no from salaries where salary between 100 and 1000);
    +----+--------------+-------------+------------+--------+-------------------+-----------+---------+--------------------+------+----------+--------------------------+
    | id | select_type  | table       | partitions | type   | possible_keys     | key       | key_len | ref                | rows | filtered | Extra                    |
    +----+--------------+-------------+------------+--------+-------------------+-----------+---------+--------------------+------+----------+--------------------------+
    |  1 | SIMPLE       | <subquery2> | NULL       | ALL    | NULL              | NULL      | NULL    | NULL               | NULL |   100.00 | NULL                     |
    |  1 | SIMPLE       | e           | NULL       | eq_ref | PRIMARY           | PRIMARY   | 4       | <subquery2>.emp_no |    1 |   100.00 | NULL                     |
    |  2 | MATERIALIZED | salaries    | NULL       | range  | PRIMARY,ix_salary | ix_salary | 4       | NULL               |    1 |   100.00 | Using where; Using index |
    +----+--------------+-------------+------------+--------+-------------------+-----------+---------+--------------------+------+----------+--------------------------+
    3 rows in set, 1 warning (0.00 sec)
    ```

## Reference 

**위 글은 책 RealMySQL 8.0을 구입하여 읽고 정리한 내용입니다.**
- [도서 홈페이지 https://wikibook.co.kr/realmysql801/](https://wikibook.co.kr/realmysql801/)
