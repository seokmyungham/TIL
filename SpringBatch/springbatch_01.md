# 01. 스프링 배치

## 스프링 배치란? (Spring Batch)

많은 애플리케이션은 미션 크리티컬 환경에서 비즈니스 운영을 수행하기 위해 대량의 데이터 처리가 필요하다.

스프링 배치는 배치 프레임워크로서  
로깅/추적, 트랜잭션 관리, 작업 처리 통계, 작업 재시작, 리소스 관리 등  
레코드 처리에 필수적인 기능을 제공하여 시스템의 부하를 줄이고 효율적인 데이터 처리를 지원한다.


## Batch

### 배치(Batch)의 의미

배치란 "일련의 작업이나 프로세스를 일괄적으로 실행하는 방법이나 시스템" 을 말하며  
주로 대량의 작업을 처리하거나 반복적인 작업을 자동화할 때 사용된다.  

#

## 배치 시나리오

- 배치 프로세스를 주기적인 커밋
- 동시 다발적인 Job의 배치 처리, 대용량 병렬 처리
- 실패 후 수동 또는 스케줄링에 의한 재시작
- 의존관계가 있는 Step 여러 개를 순차적으로 처리
- 조건적 Flow 구성을 통한 체계적이고 유연한 배치 모델 구성
- 반복, 재시도, Skip 처리

---

## 스프링 배치 구성

### JOB, STEP, TASKLET

- JOB: 일, 일감
- STEP: 일의 항목 및 단계
- TASKLET: 작업 내용

Job 하나의 배치 처리 단위:  
  
Job이 구동되면 Step을 실행하고, Step이 구동되면 Tasklet을 실행하도록 설정하여  
Job > Step > Tasklet으로 이어지는 일련의 실행을 말한다.

#

## 스프링 배치 초기화 설정 클래스

<img src="/SpringBatch/img/image.png"  width="280" height="495">

### 1. BatchAutoConfiguration

- 스프링 배치가 초기화 될 때 자동으로 실행되는 설정 클래스이다.
- Job을 수행하는 JobLauncherApplicationRunner 빈을 생성한다.

### 2. SimpleBatchConfiguration

- JobBuilderFactory와 StepBuilderFactory를 생성한다.
> Java17 버전을 지원하는 스프링 부트 3.0, 스프링 배치 5.0부터는  
> JobBuilderFactory와 StepBuilderFactory는 Deprecated 되어 JobBuilder와 StepBuilder를 사용한다.
- 스프링 배치의 주요 구성 요소 생성 -> 프록시 객체로 생성한다.

### 3. BatchConfigurerConfiguration

- BasicBatchConfigurer
  - SimpleBatchConfiguration 에서 생성한 프록시 객체의 실제 대상 객체를 생성하는 생성 클래스이다.
  - 빈으로 의존성 주입을 받아서 주요 객체들을 참조해 사용할 수 있다.
- JpaBatchConfigurer
  - JPA 관련 객체를 생성하는 설정 클래스이다.
 
## Reference

- [스프링 배치 - Spring Boot 기반으로 개발하는 Spring Batch](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%EB%B0%B0%EC%B9%98/dashboard)
- [Spring Batch Introduction](https://docs.spring.io/spring-batch/docs/4.3.x/reference/html/spring-batch-intro.html)
- [What’s New in Spring Batch 5.0](https://docs.spring.io/spring-batch/docs/current/reference/html/whatsnew.html)



