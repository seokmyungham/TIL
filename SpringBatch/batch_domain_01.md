# 스프링 배치 도메인 이해

## Job

Job은 하나의 명세서, 설계도라고 볼 수 있으며  
여러 Step을 포함하고 있는 일종의 컨테이너의 역할을 한다.  

```java
public interface Job {
    String getName();
    default boolean isRestartable() {return true;}
    void execute(JobExecution execution);

    @Nullable
    default JobParametersIncrementer getJobParametersIncrementer() {return null;}
    default JobParametersValidator getJobParametersValidator() {return new DefaultJobParametersValidator();}
}
```
  
배치 계층 구조에서 가장 상위에 있는 개념으로서 하나의 배치작업 자체를 의미하고,  
배치 Job을 구성하기 위한 최상위 인테페이스이며 스프링 배치가 기본 구현체를 제공한다.

#

Job의 기본 구현체로는 *SimpleJob*과 *FlowJob*이 존재한다.  

```java
public class SimpleJob extends AbstractJob {
    private final List<Step> steps;
    ... 생략

    public void addStep(Step step) {
        this.steps.add(step);
    }

    protected void doExecute(JobExecution execution) throws JobInterruptedException, JobRestartException, StartLimitExceededException {
        StepExecution stepExecution = null;
        Iterator var3 = this.steps.iterator();

        while(var3.hasNext()) {
            Step step = (Step)var3.next();
            stepExecution = this.handleStep(step, execution);
            if (stepExecution.getStatus() != BatchStatus.COMPLETED) {
                break;
            }
        }
    ... 생략
  }
}
```

기본 구현체들은 Job을 상속받은 추상클래스 AbstractJob을 상속받은 구조인데,  
이 중에서 SimpleJob의 경우 내부적으로 steps라고 불리는 List변수를 가지고 있고  
Job이 execute되면 순차적으로 Step이 실행되는 형태를 띄고있다.  

---

## Metadata DB Schema
<img src="img/meta-data-erd.png"  width="831" height="679">

스프링 배치 메타 데이터 테이블은 자동으로 생성되는 테이블 중에 SEQ 테이블 3개를 제외하고  
총 6개가 존재하는데 이 중 Job과 관련된 테이블은 네 가지가 존재한다.  

- BATCH_JOB_INSTANCE
- BATCH_JOB_EXECUTION
- BATCH_JOB_EXECUTION_PARAMS
- BATCH_JOB_EXECUTION_CONTEXT

#

## BATCH_JOB_INSTANCE

![](img/BATCH_JOB_INSTANCE.png)

BATCH_JOB_INSTANCE 테이블의 정보는 다음과 같다  
> JOB_INSTANCE_ID = 기본 키  
> VERSION = 업데이트 될 때마다 1씩 증가  
> JOB_NAME = Job을 구성할 때 부여하는 Job의 이름  
> JOB_KEY = job_name과 jobParameter를 직렬화하여 해싱한 값을 저장  

  
JobInstance는 Job이 실행될 때 생성되는 Job의 논리적 실행 단위 객체로서 고유하게 식별 가능한 작업 실행을 나타낸다.  
Job의 설정과 구성은 동일하지만 Job이 실행되는 시점에 처리하는 내용은 다르기 때문에 Job의 실행을 구분해줘야한다.  

```java
@Override
public void run(ApplicationArguments args) throws Exception {
    JobParameters jobParameters = new JobParametersBuilder()
            .addString("name", "user1")
            .toJobParameters();

    jobLauncher.run(job, jobParameters);
}
```

JobLauncher는 배치 Job을 실행시킬 때 Job + JobParameter라는 두 개의 인자를 필요로 하며, 
JobRepository는 DB로 부터 JobName과 JobParameter에 해당되는 JobInstance를 확인해서 현재 실행중인 Job이 처음 실행하는 Job인지 확인하는 과정을 거친다.  
  
이 때 만약 해당되는 JobInstance가 없을 경우에는 객체를 새로 만들어 반환하고  
실행시점에 해당되는 JobInstance가 존재한다면 기존 인스턴스를 반환한다음 더이상 Job이 실행되지않고 예외를 발생시킨다.

![](img/job_instance_error1.png)
![](img/job_instance_error2.png)  

이미 동일한 내용의 Job과 JobParameter로 이루어진 JobInstance가 이미 존재하므로(이전에 실행된 이력이 있으므로)  
다시 실행하고 싶으면 JobParameter의 값을 변경하라는 의미다.  
  
  
![](img/BATCH_JOB_PARAMETER.png)

JobParameter의 값은 BATCH_JOB_PARAMETER 테이블을 조회하면 쉽게 확인이 가능하다.  
  
위와같은 에러로 JOB_NAME(job)과 JOB_KEY(JobParameter 해시값)이 동일한 값은 중복해서 저장할 수 없고,  
Job과 JobInstance는 근본적으로 1대N 관계를 이루게 된다.

#

## BATCH_JOB_PARAMETER


