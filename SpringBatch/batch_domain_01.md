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

기본 구현체들은 Job을 상속받은 추상클래스 AbstractJob을 상속받은 구조인데  
이 중에서 SimpleJob의 경우 내부적으로 steps라고 불리는 List변수를 가지고 있고  
간단하게 보면 Job이 execute되면 순차적으로 Step이 실행되는 형태를 띄고있다.  

---

## Metadata
