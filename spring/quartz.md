### Spring Quartz
Spring Quartz는 복잡한 스케줄링 요구사항을 지원하며, 한 번 설정하면 특정 시간에 반복적으로 작업을 실행할 수 있다.

### 왜 Quartz 스케줄러를 사용하는가?
* 복잡한 스케줄링 : Quartz는 복잡한 스케줄링 요구사항을 쉽게 구현할 수 있도록 한다.
* 지속성과 클러스터링 지원 : Quartz는 지속성을 지원하며, 클러스터 환경에서의 스케줄링을 위한 기능을 제공한다.
* 유연한 트리거 옵션 : 다양한 유형의 트리거를 제공하여, 정확한 타이밍으로 작업을 실행할 수 있다.

### 주요 속성
* Scheduler
  * Job과 Trigger를 연결하고 Job을 실행시키는 역할
* JobDetail
  * 실행할 작업의 정의
  * 어떤 작업을 할 것인지에 대한 모든 정보가 포함
* Trigger
  * 언제 Job을 실행할지 결정하는 역할
  * 하나의 Job에 여러 개의 Trigger를 설정 가능
  * cronExpression: 실행 시간 정의
  * misfireInstruction: 트리거가 실행되지 못한 경우 어떻게 처리할지 정의

**즉,  JobDetail이 무엇을 할지 정의한다면, Trigger는 그 작업이 언제 실행될지를 정의**

### 설정 예시

**빈 등록 방식** : 여러 개를 등록해야 하는 경우 코드 중복이 발생한다.

```java
@Configuration
public class QuartzSchedulerConfig {

    @Bean
    public JobDetail memberSaveJobDetail() {
        return JobBuilder.newJob(MemberSaveQuartzJob.class)
                .withIdentity("memberSaveQuartzJob")
                .storeDurably()
                .build();
    }

    @Bean
    public JobDetail memberUpdateJobDetail() {
        return JobBuilder.newJob(MemberUpdateQuartzJob.class)
                .withIdentity("memberUpdateQuartzJob")
                .storeDurably()
                .build();
    }

    @Bean
    public Trigger memberSaveTrigger(JobDetail jobDetail) {
        return TriggerBuilder.newTrigger()
                .forJob(jobDetail)
                .withIdentity("memberSaveTrigger")
                .withSchedule(createCronSchedule())
                .build();
    }

    @Bean
    public Trigger memberUpdateTrigger(JobDetail jobDetail) {
        return TriggerBuilder.newTrigger()
                .forJob(jobDetail)
                .withIdentity("memberUpdateTrigger")
                .withSchedule(createCronSchedule())
                .build();
    }

    private CronScheduleBuilder createCronSchedule() {
        return CronScheduleBuilder.cronSchedule("*/5 * * * * ?");
    }

}
```

**한 번에 여러 개 등록** : DB에 저장된 JobDetail, Trigger 속성 값을 가져와 등록한다.

```java
@Configuration
@RequiredArgsConstructor
public class QuartzSchedulerConfig {

    private final Scheduler scheduler;
    private final ScheduleJobRepository scheduleJobRepository;

    @PostConstruct
    public void initializeJobSchedules() throws SchedulerException {
        List<ScheduleJob> retrievedScheduleJobs = scheduleJobRepository.findAll();

        Map<JobDetail, Set<? extends Trigger>> scheduleJobs = retrievedScheduleJobs.stream()
                .collect(Collectors.toMap(
                        this::createJobDetail,
                        jobSchedule -> Collections.singleton(createTrigger(jobSchedule))
                ));

        scheduler.scheduleJobs(scheduleJobs, true);
    }

    private JobDetail createJobDetail(ScheduleJob scheduleJob) {
        Class<? extends Job> jobClass = getJobClass(scheduleJob.getJobClass());
        return JobBuilder.newJob(jobClass)
                .withIdentity(scheduleJob.getJobName())
                .build();
    }

    private Class<? extends Job> getJobClass(String jobClassName) {
        try {
            return (Class<? extends Job>) Class.forName(jobClassName);
        } catch (ClassNotFoundException e) {
            throw new RuntimeException("Job class not found: " + jobClassName, e);
        }
    }

    private Trigger createTrigger(ScheduleJob scheduleJob) {
        return TriggerBuilder.newTrigger()
                .withIdentity(scheduleJob.getTriggerName())
                .withSchedule(CronScheduleBuilder.cronSchedule(scheduleJob.getCronExpression()))
                .forJob(scheduleJob.getJobName())
                .build();
    }

}
```
