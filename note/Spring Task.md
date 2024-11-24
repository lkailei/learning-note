

# Spring 任务调度

定时任务是在系统特定的时间段重复执行一段代码

定时任务实现有以下几中方式

1. Java自带的java.util.Timer类。它允许调度一个java.util.TimerTask任务，使用这种方式可以让程序按照某一个频度执行，但不能在指定时间运行，一般运用较少。
2. Spring3.0以后自带Spring Task。可以看成一个轻量级的Quartz
3. Quartz。是一个功能比较强大的调度器，可以让程序在执行时间执行，也可以按照某个频度执行，配置起来有点复杂

## Spring Task(spring 自带的定时任务处理器)

### spring task 特点

1. 默认单线程同步执行
   SpringTask 默认是单线程的，不同定时任务使用的都是同一个线程；当存在多个定时任务时，若当前任务执行时间过长则可能导致下一个任务无法执行。

2. 在实际开发中，不希望所有的任务都运行在一个线程中。可在配置类中使用ScheduledTaskRegistrar#setTaskScheduler(TaskScheduler taskScheduler)，SpringTask提供一个基于多线程的TaskScheduler接口的实现类（譬如ThreadPoolTaskScheduler类），Spring默认使用ConcurrentTaskScheduler。

3. 对异常的处理
   在SpringTask中，一旦某个任务在执行过程中抛出异常，则整个定时器生命周期就结束，以后永远不会再执行定时器任务。需要手动处理异常
4. 默认不适用分布式环境

**Spring Task** 并不是为分布式环境设计的，在分布式环境下，这种定时任务是不支持集群配置的，如果部署到多个节点上，各个节点之间并没有任何协调通讯机制，集群的节点之间是不会共享任务信息的，每个节点上的任务都会按时执行，导致任务的重复执行。我们可以使用支持分布式的定时任务调度框架，比如 **Quartz、XXL-Job、Elastic Job**。当然你可以借助 **zookeeper ，redis**  等实现分布式锁来处理各个节点的协调问题。或者把所有的定时任务抽成单独的服务单独部署。

spring-context里包下模块提供spring task 工具我们只需要在在启动类上开启即可。

### 使用

#### @EnableScheduling

使用任务调度需要在类上加入@EnableScheDuling注解标识进入任务调度

```java
 @Configuration
   @EnableScheduling
   public class AppConfig implements SchedulingConfigurer {
  
       @Override
       public void configureTasks(ScheduledTaskRegistrar taskRegistrar) {
           taskRegistrar.setScheduler(taskExecutor());
           taskRegistrar.addTriggerTask(
               new Runnable() {
                   public void run() {
                       myTask().work();
                   }
               },
               new CustomTrigger()
           );
       }
  
       @Bean(destroyMethod="shutdown")
       public Executor taskExecutor() {
           return Executors.newScheduledThreadPool(100);
       }
       
       @Bean
       public MyTask myTask() {
           return new MyTask();
       }
   }

 public class MyTask {
  
       @Scheduled(fixedRate=1000)
       public void work() {
           // task execution logic
       }
   }
```



####  @Scheduled

在执行方法中进行声明表示是一个需要调度的方法

```
@Service
public class TaskServiceImpl {

    @Scheduled(initialDelay = 1000,fixedRate = 2000)
    public Date showMessage(){
        Date date = new Date();
        System.out.println(date);
        return date;
    }
}
```

注解

```java
@Target({ElementType.METHOD, ElementType.ANNOTATION_TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Repeatable(Schedules.class)
public @interface Scheduled {
    String CRON_DISABLED = "-";

    String cron() default "";

    String zone() default "";

    long fixedDelay() default -1L;// 任务结束后等待几秒执行一次任务

    String fixedDelayString() default "";

    long fixedRate() default -1L; // 时间间隔形式

    String fixedRateString() default "";

    long initialDelay() default -1L; // // 项目启动后几秒后开始执行定时任务

    String initialDelayString() default "";
}
```

##### initalDelay

- **initialDelay** 初始化延迟时间，也就是第一次延迟执行的时间。这个参数对 `cron` 属性无效，只能配合 `fixedDelay` 或 `fixedRate` 使用。如 `@Scheduled(initialDelay=5000,fixedDelay = 1000)` 表示第一次延迟 `5000` 毫秒执行，下一次任务在上一次任务结束后 `1000` 毫秒后执行。

##### fixedDelay

- **fixedDelay**。它的间隔时间是根据上次的任务结束的时候开始计时的，只要盯紧上一次执行结束的时间即可，跟任务逻辑的执行时间无关，两个轮次的间隔距离是固定的

##### fixedRate

- fixedRate 时间间隔形式  比如 9.0-10.0点 间隔10分钟执行一次，但是 一次任务执行时间为20分钟，那么 本应该执行6次的的实际只能执行3次。那么真正的执行完就是 11点了

#####  corn表达式

​	进行拼接corn表示式即可。

#### 定时任务串行执行

```java
	@Scheduled(fixedDelay = 1000)
	public void task01() throws InterruptedException {
		System.out.println(Thread.currentThread().getName() + " | task01 " + new Date().toLocaleString());
		Thread.sleep(2000);
	}
 
	@Scheduled(fixedDelay = 2000)
	public void task02() {
		System.out.println(Thread.currentThread().getName() + " | task02 " + new Date().toLocaleString());
	}
```

- 若两个任务串行，则task01每3秒打印一次，而task02会受到task01影响每3秒打印一次
- 若两个任务并行，则task01每3秒打印一次，而task02则不受影响每2秒打印一次

上面输出currentThread都是pool-1-thread-1：进行调度两个task的。



在springboot默认的线程池中，是单一线程。所以默认情况下，所有Scheduled不能并发执行。`解决方法都是自定义一个线程池`

#### 动态创建定时任务并行执行

Springboot定时任务默认是由是单个线程串行调度所有任务，当定时任务很多的时候，为了提高任务执行效率，避免任务之间互相影响，可以采用并行方式执行定时任务。定义一个ScheduleConfig配置类并实现SchedulingConfigurer接口，重写configureTasks方法，配置任务调度线程池。 

使用注解的方式，无法实现动态的修改、添加或关闭定时任务，这个时候就需要使用编程的方式进行任务的更新操作了。可直接使用ThreadPoolTaskScheduler或者SchedulingConfigurer接口进行自定义定时任务创建



```java

@Configuration
@EnableScheduling
public class ScheduleConfig implements SchedulingConfigurer {
 
	@Override
	public void configureTasks(ScheduledTaskRegistrar taskRegistrar) {
		taskRegistrar.setTaskScheduler(taskScheduler());
	}
 
	@Bean
	public TaskScheduler taskScheduler() {
		ThreadPoolTaskScheduler taskScheduler = new ThreadPoolTaskScheduler();
		taskScheduler.setPoolSize(10);// 配置线程池大小，根据任务数量定制
		taskScheduler.setThreadNamePrefix("spring-task-scheduler-thread-");// 线程名称前缀
		taskScheduler.setAwaitTerminationSeconds(60);// 线程池关闭前最大等待时间，确保最后一定关闭
		taskScheduler.setWaitForTasksToCompleteOnShutdown(true);// 线程池关闭时等待所有任务完成
		taskScheduler.setRejectedExecutionHandler(new ThreadPoolExecutor.AbortPolicy());// 任务丢弃策略
		return taskScheduler;
	}
```

通过配置上面下面的可以在并行执行

```java

	@Scheduled(cron = "0/1 * * * * ?")
	public void task01() throws InterruptedException {
		System.out.println(Thread.currentThread().getName() + " | task01 " + new Date().toLocaleString());
		Thread.sleep(2000);
	}
 
	@Scheduled(cron = "0/2 * * * * ?")
	public void task02() {
		System.out.println(Thread.currentThread().getName() + " | task02 " + new Date().toLocaleString());
```

从上面的输出，可以简单的推理，每次调度上面的任务都是新开了一个线程来做的，所以如果在定时任务中写了死循环，可能会导致无限线程直至整个进程崩掉。

####  异步并行定时任务

上述并行任务的实现是通过增加任务调度线程池的线程来实现的，也可以结合spring的异步调用来实现，这时任务调度线程池的数量使用默认的单个也可，调用关系为：任务调度线程->任务执行线程->执行任务。首先使用spring提供的异步调用默认实现org.springframework.core.task.SimpleAsyncTaskExecutor，在配置类上添加注解@EnableAsync然后在执行任务的方法上添加注解@Async即可。

```java
	@Aysnc
	@Scheduled(cron = "0/1 * * * * ?")
	public void task01() throws InterruptedException {
		System.out.println(Thread.currentThread().getName() + " | task01 " + new Date().toLocaleString());
	}

	@Scheduled(cron = "0/2 * * * * ?")
	public void task02() throws InterruptedException {
		System.out.println(Thread.currentThread().getName() + " | task02 " + new Date().toLocaleString());
	}
```

运行上述测试代码，发现使用@Async注解的定时任务调度线程是`SimpleAsyncTaskExecutor`提供的，而没有使用@Async注解的定时任务使用的是自定义的任务调度线程池的线程:我们也可以使用自定义异步调用线程池

#### 自定义异步调用线程池

用自定义的线程池来取代默认线程管理方式，无疑是一个更加安全和灵活的方式，可以避免大量的线程阻塞带来的系统崩溃。

配置类依旧使用上面的ScheduleConfig，需要再实现接口：AsyncConfigurer，重写getAsyncExecutor()配置线程池。

```java
@Configuration
@EnableAsync
@EnableScheduling
public class ScheduleConfig implements SchedulingConfigurer , AsyncConfigurer {
 
	Logger logger = LoggerFactory.getLogger(this.getClass());
 
	@Override
	public void configureTasks(ScheduledTaskRegistrar taskRegistrar) {
		taskRegistrar.setTaskScheduler(taskScheduler());
	}
 
	@Bean
	public TaskScheduler taskScheduler() {
		ThreadPoolTaskScheduler taskScheduler = new ThreadPoolTaskScheduler();
		taskScheduler.setPoolSize(10);// 配置线程池大小，根据任务数量定制
		taskScheduler.setThreadNamePrefix("spring-task-scheduler-thread-");// 线程名称前缀
		taskScheduler.setAwaitTerminationSeconds(60);// 线程池关闭前最大等待时间，确保最后一定关闭
		taskScheduler.setWaitForTasksToCompleteOnShutdown(true);// 线程池关闭时等待所有任务完成
		taskScheduler.setRejectedExecutionHandler(new ThreadPoolExecutor.AbortPolicy());// 任务丢弃策略
		return taskScheduler;
	}
 
	@Override
	public Executor getAsyncExecutor() {
		ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
		executor.setCorePoolSize(20);// 配置核心线程数
		executor.setMaxPoolSize(50);// 配置最大线程数
		executor.setQueueCapacity(100);// 配置缓存队列大小
		executor.setKeepAliveSeconds(15);// 空闲线程存活时间
		executor.setThreadNamePrefix("spring-task-executor-thread-");
		// 线程池对拒绝任务的处理策略：这里采用了CallerRunsPolicy策略，当线程池没有处理能力的时候，该策略会直接在execute方法的调用线程中运行被拒绝的任务；如果执行程序已关闭，则会丢弃该任务
		executor.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy());// AbortPolicy()
		// 等待所有任务结束后再关闭线程池
		executor.setWaitForTasksToCompleteOnShutdown(true);
		// 设置线程池中任务的等待时间，如果超过这个时候还没有销毁就强制销毁，以确保应用最后能够被关闭，而不是被没有完成的任务阻塞
		executor.setAwaitTerminationSeconds(60);
		executor.initialize();
		return executor;
	}
 
	/**
	 * 处理异步方法的异常
	 */
	@Override
	public AsyncUncaughtExceptionHandler getAsyncUncaughtExceptionHandler() {
		return new SpringAsyncExceptionHandler();
	}
 
	class SpringAsyncExceptionHandler implements AsyncUncaughtExceptionHandler {
		@Override
		public void handleUncaughtException(Throwable arg0, Method arg1, Object... arg2) {
			logger.error("Exception occurs in async method", arg0);
		}
	}
}
```

执行测试代码

```java
	@Scheduled(cron = "0/2 * * * * ?")
	public void task02() {
		System.out.println(Thread.currentThread().getName() + " | task02 " + new Date().toLocaleString());
	}
 
	@Async
	@Scheduled(cron = "0/3 * * * * ?")
	public void task03() throws InterruptedException {
		System.out.println(Thread.currentThread().getName() + " | task03 " + new Date().toLocaleString());
	}
```

结论：标注@Async的task03由自定义异步调用线程调度，没有标注@Async的task02由自定义任务调度线程调度。

在上面的配置类中，自定义了异步调用线程池，当执行的异步调用任务的阻塞数量超过线程池缓冲能力，即会根据拒绝策略丢弃任务，避免系统崩溃，这里配置了CallerRunsPolicy()策略 -> 由调用线程处理丢弃任务，默认策略是AbortPolicy() -> 丢弃任务并抛出异常，简单说一下用自定义线程池的好处：

合理的分配线程池参数
可选择任务上限时的拒绝策略（可以按照自己的想法来处理"负载"的任务）
线程池命名，

### 基于内存的spring task 集成 动态加载spring task

```java
/**
 * 动态加载 Spring-Task 的抽象基础类，用于提供基本的属性和功能
 * @author van
 */
@Slf4j
public abstract class BaseSchedulerTaskManager<T> {
    /**
     * 记录所有在内存中的消息，用于实现定时任务的暂停功能，与 futureByTaskId 组合就可以得到执行中的/所有的/暂停的定时任务
     */
    protected Map<Integer, T> inMemoryTask = new ConcurrentHashMap<>(128);

    /**
     * 用于管控所有的消息子线程，key为 手动指定不重复Id
     */
    protected Map<Integer, ScheduledFuture<?>> futureByTaskId = new ConcurrentHashMap<>(128);

    /**
     * 用于项目启动时的初始化方法, 例如将所有的task装入定时容器内
     *
     * @param taskRegistrar taskRegistrar
     */
    protected abstract void taskConfigureInit(ScheduledTaskRegistrar taskRegistrar);

    /**
     * 发起一个新的Task
     *
     * @param t param
     */
    protected abstract void runTask(T t);

    /**
     * 取消指定任务
     *
     * @param id task id
     * @param mayInterruptIfRunning 设置当前正在进行的任务是否等待，还是强制关闭
     */
    public void cancelTaskById(Integer id, boolean mayInterruptIfRunning) {
        log.info("Canceling task : {}", id);
        ScheduledFuture<?> scheduledFuture = futureByTaskId.get(id);
        if (Objects.nonNull(scheduledFuture)) {
            scheduledFuture.cancel(mayInterruptIfRunning);
        }
        inMemoryTask.remove(id);
    }

    /**
     * 取消所有任务，因业务有不同情况处理，所以不通用，如有特殊处理，建议覆盖
     *
     * @param mayInterruptIfRunning 设置当前正在进行的任务是否等待，还是强制关闭
     * true为中断， false为等待任务完成
     */
    public void cancelTasks(boolean mayInterruptIfRunning) {
        log.info("Cancelling all item tasks");
        for (Map.Entry<Integer, ScheduledFuture<?>> integerScheduledFutureEntry : futureByTaskId.entrySet()) {
            integerScheduledFutureEntry.getValue().cancel(mayInterruptIfRunning);
            inMemoryTask.remove(integerScheduledFutureEntry.getKey());
        }
        inMemoryTask.clear();
    }

    /**
     * 暂停指定任务
     *
     * @param id id
     * @param mayInterruptIfRunning 设置当前正在进行的任务是否等待，还是强制关闭
     */
    public void stopTaskById(Integer id, boolean mayInterruptIfRunning) {
        log.info("Stop task : {}", id);
        ScheduledFuture<?> scheduledFuture = futureByTaskId.get(id);
        scheduledFuture.cancel(mayInterruptIfRunning);
    }

    /**
     * 恢复启动被暂停的消息任务
     *
     * @param id MessageId
     */
    public void startTaskById(Integer id) {
        log.info("Start task : {}", id);
        T task = inMemoryTask.get(id);
        if (Objects.isNull(task)) {
            throw new ApmCommonException("对应任务已不存在");
        }
        runTask(task);
    }
}
```



```java
@Slf4j
@Service
public class DynamicSchedulerConfigure implements SchedulingConfigurer {

    @Autowired
    private ApplicationContext applicationContext;

    public static ScheduledTaskRegistrar scheduledTaskRegistrar;

    /**
     * 初始化Task线程池
     *
     * @return threadPoll
     */
    public TaskScheduler poolScheduler() {
        ThreadPoolTaskScheduler scheduler = new ThreadPoolTaskScheduler();
        scheduler.setThreadNamePrefix("ThreadPoolTaskScheduler");
        scheduler.setPoolSize(16);
        scheduler.initialize();
        return scheduler;
    }

    /**
     * 在系统启动的时候会进行的任务初始化
     *
     * @param taskRegistrar taskRegistrar
     */
    @Override
    public void configureTasks(@NonNull ScheduledTaskRegistrar taskRegistrar) {
        if (scheduledTaskRegistrar == null) {
            scheduledTaskRegistrar = taskRegistrar;
        }
        if (taskRegistrar.getScheduler() == null) {
            taskRegistrar.setScheduler(poolScheduler());
        }
        // 获取 Spring 容器中所有 BaseSchedulerTaskManager 的实现类, 并调用其初始化方法
        applicationContext.getBeansOfType(BaseSchedulerTaskManager.class).values()
                .forEach(baseSchedulerTask -> baseSchedulerTask.taskConfigureInit(taskRegistrar));
    }

    /**
     * 取消所有任务
     *
     * @param mayInterruptIfRunning 设置当前正在进行的任务是否等待，还是强制关闭
     * true为中断， false为等待任务完成
     */
    public void cancelTasks(boolean mayInterruptIfRunning) {
        log.info("Cancelling all tasks");
        // 获取 Spring 容器中所有 BaseSchedulerTaskManager 的实现类, 并调用其取消内部所有数据的方法（因业务有不同情况处理，所以不通用）
        applicationContext.getBeansOfType(BaseSchedulerTaskManager.class).values()
                .forEach(baseSchedulerTask -> baseSchedulerTask.cancelTasks(mayInterruptIfRunning));
    }

    /**
     * 重新启动所有任务初始化
     */
    public void activateScheduler() {
        log.info("Re-Activating Scheduler");
        configureTasks(scheduledTaskRegistrar);
    }
```

```java
@Slf4j
@Service
public class UpdateDesignTaskManager extends BaseSchedulerTaskManager<ScheduleTask> {

    private final TaskManageService taskManageService;
    private final TaskProcessService taskProcessService;

    public UpdateDesignTaskManager(TaskManageService taskManageService,
                                   TaskProcessService taskProcessService) {
        this.taskManageService = taskManageService;
        this.taskProcessService = taskProcessService;
    }

    @Override
    protected void taskConfigureInit(ScheduledTaskRegistrar taskRegistrar) {
        List<ScheduleTask> allReportQuantitiesTask = taskManageService.getAllReportQuantitiesTask();
        allReportQuantitiesTask.forEach(this::runTask);
    }

    @Override
    public void runTask(ScheduleTask scheduleTask) {
        log.info("runTask : {}", scheduleTask.toString());
        inMemoryTask.put(scheduleTask.getId(), scheduleTask);
        // 每周对应周几的上午9点
        String weeklyCronStr = DateUtils.getWeeklyCronStr(scheduleTask.getExecuteCycle(), 9);
        CronTrigger cronTrigger = new CronTrigger(weeklyCronStr);
        futureByTaskId.put(scheduleTask.getId(), Objects.requireNonNull(scheduledTaskRegistrar.getScheduler()).schedule(() ->
                addChildScheduleTask(scheduleTask), cronTrigger));
    }

    /**
     * 根据原任务生成新任务
     * @param scheduleTask scheduleTask
     */
    private void addChildScheduleTask(ScheduleTask scheduleTask) {
        log.info("Info F003 addChildScheduleTask {}", scheduleTask.toString());
        taskProcessService.addUpdateTaskAndSendMessage(UpdateDesignTaskDTO.builder()
                .personId(scheduleTask.getCheckUser())
                .scheduleId(scheduleTask.getScheduleId())
                .weekday(scheduleTask.getExecuteCycle())
                .taskType(WbsTaskType.FREQUENCY_TASK.getType()).build(), scheduleTask.getName(), scheduleTask.getId());
    }
}
```

## Quartz

Quartz 是一个完全由 Java 编写的开源作业调度框架，为在 Java 应用程序中进行作业调度提供了简单却强大的机制。

Quartz 可以与[ J2EE](https://www.w3cschool.cn/java_interview_question/java_interview_question-wvr326ra.html) 与 J2SE 应用程序相结合也可以单独使用。

Quartz 允许程序开发人员根据时间的间隔来调度作业。

Quartz 实现了作业和触发器的多对多的关系，还能把多个作业与不同的触发器关联

### 核心概念

1. Job 

   表示一个工作，要执行的具体内容。此接口中只有一个方法，如下：

   ```
   void execute(JobExecutionContext context) 
   ```

2. **JobDetail** 表示一个具体的可执行的调度程序，Job 是这个可执行程调度程序所要执行的内容，另外 JobDetail 还包含了这个任务调度的方案和策略。

3. **Trigger** 代表一个调度参数的配置，什么时候去调。

4. **Scheduler** 代表一个调度容器，一个调度容器中可以注册多个 JobDetail 和 Trigger。当 Trigger 与 JobDetail 组合，就可以被 Scheduler 容器调度了。

### Spring boot 集成 quartz(内存中)

Quartz 存储方式有两种：MEMORY 和 JDBC。默认是内存形式维护任务信息，意味着服务重启了任务就从头再来。而 JDBC 形式就是能够把任务信息持久化到数据库，虽然服务重启了，依然还能接着来。

1. 引入pom.xml

   ```xml
   <!-- quartz -->
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-quartz</artifactId>
   </dependency>
   ```

2. 实现job接口定义任务

   ```java
   /**
    * 定义任务
    */
   public class DongAoJob extends QuartzJobBean {
   
       private static final Log logger = LogFactory.getLog(DongAoJob.class);
   
       @Override
       protected void executeInternal(JobExecutionContext context) throws JobExecutionException {
           logger.info("需要定时任务执行的逻辑");
       }
   }
   
   /**
    * 定义任务描述和具体的执行时间
    */
   @Configuration
   public class QuartzConfig {
       @Bean
       public JobDetail jobDetail() {
           //指定任务描述具体的实现类
           return JobBuilder.newJob(DongAoJob.class)
                   // 指定任务的名称
                   .withIdentity("dongAoJob")
                   // 任务描述
                   .withDescription("任务描述：用于输出冬奥欢迎语")
                   // 每次任务执行后进行存储
                   .storeDurably()
                   .build();
       }
       
       @Bean
       public Trigger trigger() {
           //创建触发器
           return TriggerBuilder.newTrigger()
                   // 绑定工作任务
                   .forJob(jobDetail())
                   // 每隔 5 秒执行一次 job
                   .withSchedule(SimpleScheduleBuilder.repeatSecondlyForever(5))
                   .build();
       }
   }
   
   @SpringBootApplication
   @EnableScheduling
   public class DemoJobApplication {
   
       public static void main(String[] args) {
           SpringApplication.run(DemoJobApplication.class, args);
       }
   }
   ```

   经过以上配置就可以在内存中进行执行任务了。

### Spring boot集成 quartz(持久化)

当前定时任务保存在内存中，每当项目重启的时候，定时任务就会清空并重写加载，原来的一些执行信息就丢失了，我们需要在保存下定时任务的执行信息，下次重启项目的时候我们的定时任务就可以根据上次执行的状态再执行下一次的定时任务。

1. 加入依赖pom.xml

```xml
<!-- quartz -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-quartz</artifactId>
</dependency>
```

2. yaml配置

   ```
   # 调度器实例名称（不配置则使用默认配置:quartzScheduler）
   org.quartz.scheduler.instanceName = Scheduler
   # 调度器实例编号自动生成
   org.quartz.scheduler.instanceId = AUTO
   
   #持久化方式配置   =org.quartz.simpl.RAMJobStore 即存储在内存中
   org.quartz.jobStore.class = org.quartz.impl.jdbcjobstore.JobStoreTX
   #持久化方式配置数据驱动，MySQL数据库
   org.quartz.jobStore.driverDelegateClass = org.quartz.impl.jdbcjobstore.StdJDBCDelegate
   #开启分布式部署
   org.quartz.jobStore.isClustered = true
   #分布式节点有效性检查时间间隔，单位：毫秒
   org.quartz.jobStore.clusterCheckinInterval = 10000
   # quartz相关数据表前缀名（默认QRTZ_）
   org.quartz.jobStore.tablePrefix = quartz_
   # JobDataMaps内容是否以key-value形式存储，默认true
   org.quartz.jobStore.useProperties = false
   
   #线程池实现类(不配置则使用默认配置)
   org.quartz.threadPool.class = org.quartz.simpl.SimpleThreadPool
   #执行最大并发线程数量
   org.quartz.threadPool.threadCount = 20
   #线程优先级
   org.quartz.threadPool.threadPriority = 5
   #配置是否启动自动加载数据库内的定时任务，默认true
   org.quartz.threadPool.threadsInheritContextClassLoaderOfInitializingThread = true
   ```

   

   ```xml
   spring.quartz.job-store-type=jdbc  # 设置quartz存储方式 memory或jdbc。
   spring.quartz.scheduler-name=epcmScheduler 
   spring.quartz.wait-for-jobs-to-complete-on-shutdown=true
   spring.quartz.jdbc.initialize-schema=never #设置每次启动是否重建表结构 
   ```

3. 初始话数据库 [sql官网](https://github.com/quartz-scheduler/quartz/blob/main/quartz/src/main/resources/org/quartz/impl/jdbcjobstore/tables_mysql_innodb.sql)

```sql
DROP TABLE IF EXISTS QRTZ_FIRED_TRIGGERS;
DROP TABLE IF EXISTS QRTZ_PAUSED_TRIGGER_GRPS;
DROP TABLE IF EXISTS QRTZ_SCHEDULER_STATE;
DROP TABLE IF EXISTS QRTZ_LOCKS;
DROP TABLE IF EXISTS QRTZ_SIMPLE_TRIGGERS;
DROP TABLE IF EXISTS QRTZ_SIMPROP_TRIGGERS;
DROP TABLE IF EXISTS QRTZ_CRON_TRIGGERS;
DROP TABLE IF EXISTS QRTZ_BLOB_TRIGGERS;
DROP TABLE IF EXISTS QRTZ_TRIGGERS;
DROP TABLE IF EXISTS QRTZ_JOB_DETAILS;
DROP TABLE IF EXISTS QRTZ_CALENDARS;

CREATE TABLE QRTZ_JOB_DETAILS(
SCHED_NAME VARCHAR(120) NOT NULL,
JOB_NAME VARCHAR(190) NOT NULL,
JOB_GROUP VARCHAR(190) NOT NULL,
DESCRIPTION VARCHAR(250) NULL,
JOB_CLASS_NAME VARCHAR(250) NOT NULL,
IS_DURABLE VARCHAR(1) NOT NULL,
IS_NONCONCURRENT VARCHAR(1) NOT NULL,
IS_UPDATE_DATA VARCHAR(1) NOT NULL,
REQUESTS_RECOVERY VARCHAR(1) NOT NULL,
JOB_DATA BLOB NULL,
PRIMARY KEY (SCHED_NAME,JOB_NAME,JOB_GROUP))
ENGINE=InnoDB;

CREATE TABLE QRTZ_TRIGGERS (
SCHED_NAME VARCHAR(120) NOT NULL,
TRIGGER_NAME VARCHAR(190) NOT NULL,
TRIGGER_GROUP VARCHAR(190) NOT NULL,
JOB_NAME VARCHAR(190) NOT NULL,
JOB_GROUP VARCHAR(190) NOT NULL,
DESCRIPTION VARCHAR(250) NULL,
NEXT_FIRE_TIME BIGINT(13) NULL,
PREV_FIRE_TIME BIGINT(13) NULL,
PRIORITY INTEGER NULL,
TRIGGER_STATE VARCHAR(16) NOT NULL,
TRIGGER_TYPE VARCHAR(8) NOT NULL,
START_TIME BIGINT(13) NOT NULL,
END_TIME BIGINT(13) NULL,
CALENDAR_NAME VARCHAR(190) NULL,
MISFIRE_INSTR SMALLINT(2) NULL,
JOB_DATA BLOB NULL,
PRIMARY KEY (SCHED_NAME,TRIGGER_NAME,TRIGGER_GROUP),
FOREIGN KEY (SCHED_NAME,JOB_NAME,JOB_GROUP)
REFERENCES QRTZ_JOB_DETAILS(SCHED_NAME,JOB_NAME,JOB_GROUP))
ENGINE=InnoDB;

CREATE TABLE QRTZ_SIMPLE_TRIGGERS (
SCHED_NAME VARCHAR(120) NOT NULL,
TRIGGER_NAME VARCHAR(190) NOT NULL,
TRIGGER_GROUP VARCHAR(190) NOT NULL,
REPEAT_COUNT BIGINT(7) NOT NULL,
REPEAT_INTERVAL BIGINT(12) NOT NULL,
TIMES_TRIGGERED BIGINT(10) NOT NULL,
PRIMARY KEY (SCHED_NAME,TRIGGER_NAME,TRIGGER_GROUP),
FOREIGN KEY (SCHED_NAME,TRIGGER_NAME,TRIGGER_GROUP)
REFERENCES QRTZ_TRIGGERS(SCHED_NAME,TRIGGER_NAME,TRIGGER_GROUP))
ENGINE=InnoDB;

CREATE TABLE QRTZ_CRON_TRIGGERS (
SCHED_NAME VARCHAR(120) NOT NULL,
TRIGGER_NAME VARCHAR(190) NOT NULL,
TRIGGER_GROUP VARCHAR(190) NOT NULL,
CRON_EXPRESSION VARCHAR(120) NOT NULL,
TIME_ZONE_ID VARCHAR(80),
PRIMARY KEY (SCHED_NAME,TRIGGER_NAME,TRIGGER_GROUP),
FOREIGN KEY (SCHED_NAME,TRIGGER_NAME,TRIGGER_GROUP)
REFERENCES QRTZ_TRIGGERS(SCHED_NAME,TRIGGER_NAME,TRIGGER_GROUP))
ENGINE=InnoDB;

CREATE TABLE QRTZ_SIMPROP_TRIGGERS
  (
    SCHED_NAME VARCHAR(120) NOT NULL,
    TRIGGER_NAME VARCHAR(190) NOT NULL,
    TRIGGER_GROUP VARCHAR(190) NOT NULL,
    STR_PROP_1 VARCHAR(512) NULL,
    STR_PROP_2 VARCHAR(512) NULL,
    STR_PROP_3 VARCHAR(512) NULL,
    INT_PROP_1 INT NULL,
    INT_PROP_2 INT NULL,
    LONG_PROP_1 BIGINT NULL,
    LONG_PROP_2 BIGINT NULL,
    DEC_PROP_1 NUMERIC(13,4) NULL,
    DEC_PROP_2 NUMERIC(13,4) NULL,
    BOOL_PROP_1 VARCHAR(1) NULL,
    BOOL_PROP_2 VARCHAR(1) NULL,
    PRIMARY KEY (SCHED_NAME,TRIGGER_NAME,TRIGGER_GROUP),
    FOREIGN KEY (SCHED_NAME,TRIGGER_NAME,TRIGGER_GROUP)
    REFERENCES QRTZ_TRIGGERS(SCHED_NAME,TRIGGER_NAME,TRIGGER_GROUP))
ENGINE=InnoDB;

CREATE TABLE QRTZ_BLOB_TRIGGERS (
SCHED_NAME VARCHAR(120) NOT NULL,
TRIGGER_NAME VARCHAR(190) NOT NULL,
TRIGGER_GROUP VARCHAR(190) NOT NULL,
BLOB_DATA BLOB NULL,
PRIMARY KEY (SCHED_NAME,TRIGGER_NAME,TRIGGER_GROUP),
INDEX (SCHED_NAME,TRIGGER_NAME, TRIGGER_GROUP),
FOREIGN KEY (SCHED_NAME,TRIGGER_NAME,TRIGGER_GROUP)
REFERENCES QRTZ_TRIGGERS(SCHED_NAME,TRIGGER_NAME,TRIGGER_GROUP))
ENGINE=InnoDB;

CREATE TABLE QRTZ_CALENDARS (
SCHED_NAME VARCHAR(120) NOT NULL,
CALENDAR_NAME VARCHAR(190) NOT NULL,
CALENDAR BLOB NOT NULL,
PRIMARY KEY (SCHED_NAME,CALENDAR_NAME))
ENGINE=InnoDB;

CREATE TABLE QRTZ_PAUSED_TRIGGER_GRPS (
SCHED_NAME VARCHAR(120) NOT NULL,
TRIGGER_GROUP VARCHAR(190) NOT NULL,
PRIMARY KEY (SCHED_NAME,TRIGGER_GROUP))
ENGINE=InnoDB;

CREATE TABLE QRTZ_FIRED_TRIGGERS (
SCHED_NAME VARCHAR(120) NOT NULL,
ENTRY_ID VARCHAR(95) NOT NULL,
TRIGGER_NAME VARCHAR(190) NOT NULL,
TRIGGER_GROUP VARCHAR(190) NOT NULL,
INSTANCE_NAME VARCHAR(190) NOT NULL,
FIRED_TIME BIGINT(13) NOT NULL,
SCHED_TIME BIGINT(13) NOT NULL,
PRIORITY INTEGER NOT NULL,
STATE VARCHAR(16) NOT NULL,
JOB_NAME VARCHAR(190) NULL,
JOB_GROUP VARCHAR(190) NULL,
IS_NONCONCURRENT VARCHAR(1) NULL,
REQUESTS_RECOVERY VARCHAR(1) NULL,
PRIMARY KEY (SCHED_NAME,ENTRY_ID))
ENGINE=InnoDB;

CREATE TABLE QRTZ_SCHEDULER_STATE (
SCHED_NAME VARCHAR(120) NOT NULL,
INSTANCE_NAME VARCHAR(190) NOT NULL,
LAST_CHECKIN_TIME BIGINT(13) NOT NULL,
CHECKIN_INTERVAL BIGINT(13) NOT NULL,
PRIMARY KEY (SCHED_NAME,INSTANCE_NAME))
ENGINE=InnoDB;

CREATE TABLE QRTZ_LOCKS (
SCHED_NAME VARCHAR(120) NOT NULL,
LOCK_NAME VARCHAR(40) NOT NULL,
PRIMARY KEY (SCHED_NAME,LOCK_NAME))
ENGINE=InnoDB;

CREATE INDEX IDX_QRTZ_J_REQ_RECOVERY ON QRTZ_JOB_DETAILS(SCHED_NAME,REQUESTS_RECOVERY);
CREATE INDEX IDX_QRTZ_J_GRP ON QRTZ_JOB_DETAILS(SCHED_NAME,JOB_GROUP);

CREATE INDEX IDX_QRTZ_T_J ON QRTZ_TRIGGERS(SCHED_NAME,JOB_NAME,JOB_GROUP);
CREATE INDEX IDX_QRTZ_T_JG ON QRTZ_TRIGGERS(SCHED_NAME,JOB_GROUP);
CREATE INDEX IDX_QRTZ_T_C ON QRTZ_TRIGGERS(SCHED_NAME,CALENDAR_NAME);
CREATE INDEX IDX_QRTZ_T_G ON QRTZ_TRIGGERS(SCHED_NAME,TRIGGER_GROUP);
CREATE INDEX IDX_QRTZ_T_STATE ON QRTZ_TRIGGERS(SCHED_NAME,TRIGGER_STATE);
CREATE INDEX IDX_QRTZ_T_N_STATE ON QRTZ_TRIGGERS(SCHED_NAME,TRIGGER_NAME,TRIGGER_GROUP,TRIGGER_STATE);
CREATE INDEX IDX_QRTZ_T_N_G_STATE ON QRTZ_TRIGGERS(SCHED_NAME,TRIGGER_GROUP,TRIGGER_STATE);
CREATE INDEX IDX_QRTZ_T_NEXT_FIRE_TIME ON QRTZ_TRIGGERS(SCHED_NAME,NEXT_FIRE_TIME);
CREATE INDEX IDX_QRTZ_T_NFT_ST ON QRTZ_TRIGGERS(SCHED_NAME,TRIGGER_STATE,NEXT_FIRE_TIME);
CREATE INDEX IDX_QRTZ_T_NFT_MISFIRE ON QRTZ_TRIGGERS(SCHED_NAME,MISFIRE_INSTR,NEXT_FIRE_TIME);
CREATE INDEX IDX_QRTZ_T_NFT_ST_MISFIRE ON QRTZ_TRIGGERS(SCHED_NAME,MISFIRE_INSTR,NEXT_FIRE_TIME,TRIGGER_STATE);
CREATE INDEX IDX_QRTZ_T_NFT_ST_MISFIRE_GRP ON QRTZ_TRIGGERS(SCHED_NAME,MISFIRE_INSTR,NEXT_FIRE_TIME,TRIGGER_GROUP,TRIGGER_STATE);

CREATE INDEX IDX_QRTZ_FT_TRIG_INST_NAME ON QRTZ_FIRED_TRIGGERS(SCHED_NAME,INSTANCE_NAME);
CREATE INDEX IDX_QRTZ_FT_INST_JOB_REQ_RCVRY ON QRTZ_FIRED_TRIGGERS(SCHED_NAME,INSTANCE_NAME,REQUESTS_RECOVERY);
CREATE INDEX IDX_QRTZ_FT_J_G ON QRTZ_FIRED_TRIGGERS(SCHED_NAME,JOB_NAME,JOB_GROUP);
CREATE INDEX IDX_QRTZ_FT_JG ON QRTZ_FIRED_TRIGGERS(SCHED_NAME,JOB_GROUP);
CREATE INDEX IDX_QRTZ_FT_T_G ON QRTZ_FIRED_TRIGGERS(SCHED_NAME,TRIGGER_NAME,TRIGGER_GROUP);
CREATE INDEX IDX_QRTZ_FT_TG ON QRTZ_FIRED_TRIGGERS(SCHED_NAME,TRIGGER_GROUP);

commit;
```

| 表名                     | 描述                                                         |
| ------------------------ | ------------------------------------------------------------ |
| QRTZ_CALENDARS           | 存储Quartz的Calendar信息                                     |
| QRTZ_TRIGGERS            | 存储已配置的Trigger信息                                      |
| QRTZ_BLOB_TRIGGERS       | Trigger 作为Blob类型存储                                     |
| QRTZ_SIMPLE_TRIGGERS     | 存储简单的Trigger,包括重复次数，间隔，已触发次数             |
| QRTZ_SIMPROP_TRIGGERS    | 存储CalendarInterValTrigger和DailyTimeIntercalTrigger两种类型触发器 |
| QRTZ_FIRED_TRIGGERS      | 存储已触发的状态信息，以及相关的job的执行信息                |
| QRTZ_JOB_DETAILS         | 存储每一个配置的job的详细信息                                |
| QRTZ_LOCKS               | 存储程序的悲观锁的信息                                       |
| QRTZ_SCHEDULER_STATE     | 存储少量的有关scheduler的状态信息和别的Sacheduler实例        |
| QRTZ_PAUSED_TRIGGER_GRPS | 存储已暂停的Trigger组的信                                    |

**使用**

```java
/**
* job配置
*/
@Component
public class PatrolInspectTaskExecutionJob extends QuartzJobBean {

    public static final String PATROL_INSPECT_TASKId = "prjTaskConfigId";
    private static final String JOB_GROUP = "PatrolInspectTaskExecutionJob";
    private static final String JOB_PREFIX = "job-patrol_inspect_task_execution_";

    /**
     * 任务模板id 参数传递
     * usingJobData() 将此参数进行传递后即可进行处理
     */
    @Getter
    @Setter
    private Integer prjTaskConfigId;

    @Autowired
    @Nullable
    private PatrolInspectService patrolInspectService;

    public String getJobGroup() {
        return JOB_GROUP;
    }

    @Override
    protected void executeInternal(JobExecutionContext context) throws JobExecutionException {
        JobKey jobKey = context.getJobDetail().getKey();
        log.info("fire jobKey: {}", jobKey);
        // 或者使用jobDetail存储的参数
        JobDataMap jobDataMap = jobExecutionContext.getJobDetail().getJobDataMap();
        // 获取jobDetail存储的参数
        long taskId = jobDataMap.getLongValue("prjTaskConfigId");
        try {
            patrolInspectService.fireTaskExecutionGeneration(prjTaskConfigId);
        } catch (Exception e) {
            log.error("execute job failed. {}", e.getMessage(), e);
        }
    }

    /**
     * 生成调度任务key
     *
     * @param prjTaskConfigId
     * @param s
     * @return
     */
    public static JobKey genJobKey(Integer prjTaskConfigId, String s) {
        return JobKey.jobKey(JOB_PREFIX + prjTaskConfigId + s ,JOB_GROUP);
    }

    /**
     * 生成调度任务详情
     *
     * @param config
     * @param s
     * @return
     */
    public static JobDetail genJobDetail(PatrolInspectConfig config, String s) {
        Integer taskTemplateId = config.getId();
        Preconditions.checkNotNull(taskTemplateId);
        JobKey jobKey = genJobKey(taskTemplateId, s);
        // 传入参数 usingJobData 任务执行时需要使用
        // 设置 描述 withDescription
        // 设置触发器 newJob(xx.class) <--> ofType(xx.class)
        // 设置唯一标识 withIdentity 使用给定的任务名称和默认的任务组名来创建JobKey,JobKey是定时任务JobDetail的标志
        return JobBuilder.newJob()
                .usingJobData(PATROL_INSPECT_TASKId, config.getId())
                .withIdentity(jobKey)
                .ofType(PatrolInspectTaskExecutionJob.class)
                .withDescription(config.getName())
                .build();
    }
}
```

任务开始

```java
/**
* 任务开始
**/
public void jobStartTest(){
 LocalTime parse1 = LocalTime.parse(time.getTime());
 // 进行任务调度处理
 int hour = parse1.getHour();
 int minute = parse1.getMinute();
 String cron = "0 " + minute + " " + hour + " " + time.getDate().getDayOfMonth() + " " + time.getDate().getMonthValue() + " ? " + time.getDate().getYear();
 // 生成quartz调度任务
 JobDetail jobDetail = PatrolInspectTaskExecutionJob.genJobDetail(patrolInspectConfig, "_" + time.getOrderDay() + "_" + time.getTime());
 JobKey jobKey = jobDetail.getKey();
 CronTrigger trigger = getCronTriggerOfJob(cron, jobKey, LocalDate.now(), null);
 try {
 log.info("Scheduler-patrol-inspect-job,{}", trigger.toString());
 quartzScheduler.scheduleJob(jobDetail, trigger);
 } catch (SchedulerException e) {
 log.error("generate quartz job failed. patrolInspectTask: {}", configId, e);
 }
}

// 获取cronTrigger
private CronTrigger getCronTriggerOfJob(String cron, JobKey jobKey, LocalDate startAt, LocalDate endAt) {
    Preconditions.checkArgument(StrUtil.isNotBlank(cron));
    Preconditions.checkNotNull(jobKey);
    CronScheduleBuilder cronScheduleBuilder = CronScheduleBuilder.cronSchedule(cron);
    TriggerBuilder<CronTrigger> triggerBuilder = TriggerBuilder.newTrigger()
        .withIdentity("trigger-" + jobKey.getName(), jobKey.getGroup())
        .withSchedule(cronScheduleBuilder);
    if (Objects.nonNull(startAt)) {
        triggerBuilder.startAt(DateUtils.toDate(startAt.atStartOfDay()));
    } else {
        triggerBuilder.startNow();
    }
    if (Objects.nonNull(endAt)) {
        triggerBuilder.endAt(DateUtils.toDate(endAt.atStartOfDay()));
    }
    return triggerBuilder.build();
}
/**
* 取消定时任务
**/
private void cancelScheduleTask(PatrolInspectConfig config) {
        // 取消定时任务
        List<PatrolInspectConfigPeriodicTime> times = periodicTimeDao.listByConfigId(config.getId());
        // 获取时间
        List<JobKey> jobKeys = new ArrayList<>();
        times.forEach(el -> {
            // 获取times
            List<String> time = Arrays.asList(el.getTime().split(","));
            time.forEach(ele -> {
                jobKeys.add(PatrolInspectTaskExecutionJob.genJobKey(config.getId(), "_" + el.getOrderDay() + "_" + ele));
            });
        });
        try {
            if (CollectionUtil.isEmpty(jobKeys)) return;
            log.info("取消定时任务，{}", jobKeys);
            quartzScheduler.deleteJobs(jobKeys);
        } catch (Exception e) {
            log.error("用户取消任务生效逻辑-删除任务调度失败，jobKey:{}", e.getMessage());
            throw new ApmCommonException("删除任务调度失败，请联系管理员");
        }
    }

/**
* 暂停任务
**/
public void pauseJob(String jobName,String jobGroup) throws SchedulerException {

    JobKey jobKey = JobKey.jobKey(jobName, jobGroup);
    JobDetail jobDetail = scheduler.getJobDetail(jobKey);
    if (jobDetail == null) {
        return;
    }
    scheduler.pauseJob(jobKey);
}

/**
* 恢复任务
**/
public void remuseJob(String jobName, String jobGroup) throws SchedulerException {
    JobKey jobKey = JobKey.jobKey(jobName, jobGroup);
    JobDetail jobDetail = scheduler.getJobDetail(jobKey);
    if (jobDetail == null) {
        return;
    }
    scheduler.resumeJob(jobKey);
}
```

### QuartzApi

Quartz API的关键接口是：

- Scheduler - 与调度程序交互的主要API。
- Job - 你想要调度器执行的任务组件需要实现的接口
- JobDetail - 用于定义作业的实例。
- Trigger（即触发器） - 定义执行给定作业的计划的组件。
- JobBuilder - 用于定义/构建 JobDetail 实例，用于定义作业的实例。
- TriggerBuilder - 用于定义/构建触发器实例。
- Scheduler 的生命期，从 SchedulerFactory 创建它时开始，到 Scheduler 调用shutdown() 方法时结束；Scheduler 被创建后，可以增加、删除和列举 Job 和 Trigger，以及执行其它与调度相关的操作（如暂停 Trigger）。但是，Scheduler 只有在调用 start() 方法后，才会真正地触发 trigger（即执行 job）

#### Job

 Job用于定义任务具体的逻辑。简单来说就是用来定义定时任务需要干的事情。比如我们想每天早上10点给某某人发一封邮件。Job干的事情就是发邮件的事情。每个任务对应一个Job(咱们一般会自定义一个Job，继承QuartzJobBean，通过实现里面的executeInternal方法实现具体的任务逻辑)。这样调度器(Scheduler)会根据触发器(Trigger)设置的时间叫这个任务(Job)起来干活了。

关于Job重点想说的是在Job类里面怎么获取到外部参数。首先我们肯定会在定义任务详情的时候给这个Job的JobDataMap属性里面设置一些参数。然后在Job执行的时候，我们通过JobDataMap jobDataMap = jobExecutionContext.getMergedJobDataMap();获取到JobDataMap(JobDataMap就相当于一个Map)，取出相应的参数了。

#### QuartzJobBean

Quartz Job接口的简单实现，将传入的JobDataMap和SchedulerContext应用为bean属性值。这是合适的，因为每次执行都会创建一个新的作业实例。JobDataMap条目将覆盖具有相同键的SchedulerContext条目。

例如，让我们假设JobDataMap包含一个值为“5”的键“myParam”：然后Job实现可以公开类型为int的bean属性“myParam”来接收这样的值，即方法“setMyParam（int）”。这也适用于业务对象等复杂类型。

请注意，将依赖项注入应用于Job实例的首选方式是通过JobFactory：即，将SpringBeanJobFactory指定为Quartz JobFactory（通常通过SchedulerFactoryBean.setJobFactory SchedulerFactoryBean的“JobFactory”属性}）。这允许在不依赖Spring基类的情况下实现依赖注入的Quartz作业。

**使用时我们只需要继承这个Bean对象即可**



#### JobDetail

JobDetail定义任务详情。包含执行任务的Job，任务的一些身份信息(可以帮助找到这个任务)， 可以给通过JobDetail给job设置name和group。给任务设置JobDataMap(把参数带到任务里面去)。

**重要属性:**

　　**name：任务的名称，必须属性**

　　**group：任务所在的组，也是必须，默认值是DEFAULT，必须的**

　　**jobclass:任务的实现类，必须的**

　　**jobdatamap:用于传递传参数**

JobDetail里面常用方法如下

```java
public interface JobDetail {

    /**
     * job的身份，通过他来找到对应的job
     */
    public JobKey getKey();

    /**
     * job描述
     */
    public String getDescription();

    /**
     * 执行job的具体类，定时任务的动作都在这个类里面完成
     */
    public Class<? extends Job> getJobClass();

    /**
     * 给job传递数据(把需要的参数带到job执行的类里面去)
     */
    public JobDataMap getJobDataMap();

    /**
     * 任务孤立的时候是否需要继续报错(孤立:没有触发器关联该任务)
     */
    public boolean isDurable();

    /**
     * 和@PersistJobDataAfterExecution注解一样
     * PersistJobDataAfterExecution注解是添加在Job类上的：表示 Quartz 将会在成功执行 execute()
     * 方法后（没有抛出异常）更新 JobDetail 的 JobDataMap，下一次执行相同的任务（JobDetail）
     * 将会得到更新后的值，而不是原始的值
     */
    public boolean isPersistJobDataAfterExecution();

    /**
     * 和DisallowConcurrentExecution注解的功能一样
     * DisallowConcurrentExecution注解添加到Job之后，Quartz 将不会同时执行多个 Job 实例，
     * 怕有数据更新的时候不知道取哪一个数据
     */
    public boolean isConcurrentExectionDisallowed();

    /**
     * 指示调度程序在遇到“恢复”或“故障转移”情况时是否应重新执行作业
     */
    public boolean requestsRecovery();


    /**
     * JobDetail是通过构建者模式来实现的
     */
    public JobBuilder getJobBuilder();
}
```

##### JobBuilder 构建jobDetail

```java
public class JobBuilder {
    private JobKey key;
    private String description;
    private Class<? extends Job> jobClass;
    private boolean durability;
    private boolean shouldRecover;
    private JobDataMap jobDataMap = new JobDataMap();

    protected JobBuilder() {
    }

    public static JobBuilder newJob() {
        return new JobBuilder();
    }
	// 创建用于定义JobDetail的JobBuilder，并设置要执行的作业的类名。
    public static JobBuilder newJob(Class<? extends Job> jobClass) {
        JobBuilder b = new JobBuilder();
        b.ofType(jobClass);
        return b;
    }
	
    // 生成由该JobBuilder定义的JobDetail实例。
    public JobDetail build() {
        JobDetailImpl job = new JobDetailImpl();
        job.setJobClass(this.jobClass);
        job.setDescription(this.description);
        if (this.key == null) {
            this.key = new JobKey(Key.createUniqueName((String)null), (String)null);
        }

        job.setKey(this.key);
        job.setDurability(this.durability);
        job.setRequestsRecovery(this.shouldRecover);
        if (!this.jobDataMap.isEmpty()) {
            job.setJobDataMap(this.jobDataMap);
        }

        return job;
    }

    // 使用具有给定名称和默认组的JobKey来标识JobDetail。如果在JobBuilder上没有设置任何“withIdentity”方法，则将生成一个随机的、唯一的JobKey。
    public JobBuilder withIdentity(String name) {
        this.key = new JobKey(name, (String)null);
        return this;
    }
	// 使用具有给定名称和组的JobKey来标识JobDetail。 如果在JobBuilder上没有设置任何“withIdentity”方法，则将生成一个随机的、唯一的JobKey。
    public JobBuilder withIdentity(String name, String group) {
        this.key = new JobKey(name, group);
        return this;
    }
	
    // 使用JobKey来标识JobDetail。如果在JobBuilder上没有设置任何“withIdentity”方法，则将生成一个随机的、唯一的JobKey。
    public JobBuilder withIdentity(JobKey jobKey) {
        this.key = jobKey;
        return this;
    }

    // 设置作业的给定 描述。
    public JobBuilder withDescription(String jobDescription) {
        this.description = jobDescription;
        return this;
    }
	
    // 设置当触发与此JobDetail关联的触发器时将被实例化和执行的类。
    public JobBuilder ofType(Class<? extends Job> jobClazz) {
        this.jobClass = jobClazz;
        return this;
    }

    public JobBuilder requestRecovery() {
        this.shouldRecover = true;
        return this;
    }

    public JobBuilder requestRecovery(boolean jobShouldRecover) {
        this.shouldRecover = jobShouldRecover;
        return this;
    }

    public JobBuilder storeDurably() {
        return this.storeDurably(true);
    }

    public JobBuilder storeDurably(boolean jobDurability) {
        this.durability = jobDurability;
        return this;
    }
	// 添加 key-value 参数到 JobDetail中的JobDataMap对象中.
    public JobBuilder usingJobData(String dataKey, String value) {
        this.jobDataMap.put(dataKey, value);
        return this;
    }

    public JobBuilder usingJobData(String dataKey, Integer value) {
        this.jobDataMap.put(dataKey, value);
        return this;
    }

    public JobBuilder usingJobData(String dataKey, Long value) {
        this.jobDataMap.put(dataKey, value);
        return this;
    }

    public JobBuilder usingJobData(String dataKey, Float value) {
        this.jobDataMap.put(dataKey, value);
        return this;
    }

    public JobBuilder usingJobData(String dataKey, Double value) {
        this.jobDataMap.put(dataKey, value);
        return this;
    }

    public JobBuilder usingJobData(String dataKey, Boolean value) {
        this.jobDataMap.put(dataKey, value);
        return this;
    }
	// 直接使用jobDataMap将数据放到JobBuilder中
    public JobBuilder usingJobData(JobDataMap newJobDataMap) {
        this.jobDataMap.putAll(newJobDataMap);
        return this;
    }

    public JobBuilder setJobData(JobDataMap newJobDataMap) {
        this.jobDataMap = newJobDataMap;
        return this;
    }
}
```

##### 实例

以下两个创建jobDetail方式相同

```java
public JobDetail genJobDetail(){
    return JobBuilder.newJob()
                     .usingJobData("key-param1","value1")
                     .withIdentity(JobKey.jobKey("key","groupName"))
                     .withDescription("描述")
                     .ofType("继承jobxxx.class") 
                     .build();
}

public JobDetail genJobDetail(){
    return JobBuilder.newJob(PatrolInspectTaskExecutionJob.class) // 设置触发器
                     .usingJobData("key-param1","value1") // 参数传递
                     .withIdentity(JobKey.jobKey("key","groupName")) // 使用给定的任务名称和默认的任务组名来创建JobKey,JobKey是定时任务JobDetail的标志
                     .withDescription("描述")
                     .build();
}
```



#### Trigger

Trigger 用于触发 Job 的执行。当你准备调度一个 job 时，你创建一个 Trigger 的实例，然后设置调度相关的属性。Trigger 也有一个相关联的 JobDataMap，用于给 Job 传递一些触发相关的参数。Quartz 自带了各种不同类型的 Trigger，最常用的主要是 SimpleTrigger 和 CronTrigger。 一个Trigger只能绑定一个JOb。但是一个job可以被多个Trigger绑定。



##### SimpleTrigger

SimpleTrigger 主要用于一次性执行的 Job（只在某个特定的时间点执行一次），或者 Job 在特定的时间点执行，重复执行 N 次，每次执行间隔T个时间单位. SimpleTrigger的属性包括：开始时间、结束时间、重复次数以及重复的间隔。这些属性的含义与你所期望的是一致的，只是关于结束时间有一些地方需要注意

重复次数，可以是0、正整数，以及常量SimpleTrigger.REPEAT_INDEFINITELY。重复的间隔，必须是0，或者long型的正数，表示毫秒。注意，如果重复间隔为0，trigger将会以重复次数并发执行(或者以scheduler可以处理的近似并发数)。

endTime属性的值会覆盖设置重复次数的属性值；比如，你可以创建一个trigger，在终止时间之前每隔10秒执行一次，你不需要去计算在开始时间和终止时间之间的重复次数，只需要设置终止时间并将重复次数设置为REPEAT_INDEFINITELY(当然，你也可以将重复次数设置为一个很大的值，并保证该值比trigger在终止时间之前实际触发的次数要大即可)。

```java
// 使用simpleTrigger规则, 任务每隔30S执行一次，执行100次
SimpleTrigger trigger = TriggerBuilder
    .newTrigger()
    .withIdentity(jobName, jobGroupName)
    .withSchedule(SimpleScheduleBuilder.repeatSecondlyForTotalCount(30, 100))
    .startNow()
    .build();

```

##### CronTrigger

CronTrigger 在基于日历的调度上非常有用，如“每个星期五的正午”，或者“每月的第十天的上午 10:15”等

CronTrigger通常比Simple Trigger更有用，如果您需要基于日历的概念而不是按照SimpleTrigger的精确指定间隔进行重新启动的作业启动计划。

使用CronTrigger，您可以指定号时间表，例如“每周五中午”或“每个工作日和上午9:30”，甚至“每周一至周五上午9:00至10点之间每5分钟”和1月份的星期五“。

即使如此，和SimpleTrigger一样，CronTrigger有一个startTime，它指定何时生效，以及一个（可选的）endTime，用于指定何时停止计划。

```java
    private CronTrigger getCronTriggerOfJob(String cron, JobKey jobKey, @Nullable LocalDate startAt,@Nullable LocalDate endAt) {
        Preconditions.checkArgument(StrUtil.isNotBlank(cron));
        Preconditions.checkNotNull(jobKey);
        CronScheduleBuilder cronScheduleBuilder = CronScheduleBuilder.cronSchedule(cron);
        TriggerBuilder<CronTrigger> triggerBuilder = TriggerBuilder.newTrigger()
                .withIdentity("trigger-" + jobKey.getName(), jobKey.getGroup())
                .withSchedule(cronScheduleBuilder);
        if (Objects.nonNull(startAt)) {
            triggerBuilder.startAt(DateUtils.toDate(startAt.atStartOfDay()));
        } else {
            triggerBuilder.startNow();
        }
        if (Objects.nonNull(endAt)) {
            triggerBuilder.endAt(DateUtils.toDate(endAt.atStartOfDay()));
        }
        return triggerBuilder.build();
    }

// 顶一个没一个周六 9点执行一次
CronTrigger trigger = TriggerBuilder.newTrigger()
                    .withIdentity(jobName, jobGroupName)// 触发器名,触发器组
                    //每个礼拜星期六的9:00执行
                    .withSchedule(CronScheduleBuilder.
                     weeklyOnDayAndHourAndMinute(DateBuilder.SATURDAY, 09, 00)) 
    .build();
```



#####  CalendarIntervalTrigger

类似于SimpleTrigger，指定从某一个时间开始，以一定的时间间隔执行的任务。 但是不同的是SimpleTrigger指定的时间间隔为**毫秒**，没办法指定每隔一个月执行一次（每月的时间间隔不是固定值），而CalendarIntervalTrigger支持的间隔单位有**秒，分钟，小时，天，月，年，星期**。

相较于SimpleTrigger有两个优势：1、更方便，比如每隔1小时执行，你不用自己去计算1小时等于多少毫秒。 2、支持不是固定长度的间隔，比如间隔为月和年。但劣势是精度只能到秒。

它适合的任务类似于：9:00 开始执行，并且以后每周 9:00 执行一次

它的属性有:

- interval 执行间隔
- intervalUnit 执行间隔的单位（秒，分钟，小时，天，月，年，星期）

```java
    private CalendarIntervalTrigger getFixedDelayDaysTrigger(int intervalDayNum, JobKey jobKey,
                                                             @Nullable LocalDate startAt,
                                                             @Nullable LocalDate endAt) {
        Preconditions.checkArgument(intervalDayNum >= 1);
        Preconditions.checkNotNull(jobKey);
        CalendarIntervalScheduleBuilder calendarIntervalScheduleBuilder = CalendarIntervalScheduleBuilder
                .calendarIntervalSchedule()
                .withIntervalInDays(intervalDayNum);
        TriggerBuilder<CalendarIntervalTrigger> triggerBuilder = TriggerBuilder.newTrigger()
                .withIdentity("trigger-" + jobKey.getName(), jobKey.getGroup())
                .withSchedule(calendarIntervalScheduleBuilder);
        if (Objects.nonNull(startAt)) {
            triggerBuilder.startAt(DateUtils.toDate(startAt.atStartOfDay()));
        } else {
            triggerBuilder.startAt(DateUtils.toDate(LocalDate.now().atStartOfDay()));
        }
        if (Objects.nonNull(endAt)) {
            triggerBuilder.endAt(DateUtils.toDate(endAt.atStartOfDay()));
        }
        return triggerBuilder.build();
    }


public CalendarIntervalTrigger test(){
    // 15:30开始，每隔一个星期执行一次的任务
    // 15:30
    Calendar c = Calendar.getInstance();
    c.setTime(new Date());
    c.setLenient(true);
    c.set(Calendar.HOUR_OF_DAY, 15);
    c.set(Calendar.MINUTE, 30);

    CalendarIntervalTrigger trigger = TriggerBuilder.newTrigger()
        .withIdentity(jobName, jobGroupName)// 触发器名,触发器组
        .startAt(c.getTime())
        .withSchedule(CalendarIntervalScheduleBuilder.calendarIntervalSchedule().withIntervalInWeeks(1)) //每隔星期执行
        .build();
    return trigger;
}
```



##### DailyTimeIntervalTrigger

指定每天的某个时间段内，以一定的时间间隔执行任务。并且它可以支持指定星期。

它适合的任务类似于：指定每天9:00 至 18:00 ，每隔70秒执行一次，并且只要周一至周五执行。

它的属性有:

- startTimeOfDay 每天开始时间
- endTimeOfDay 每天结束时间
- daysOfWeek 需要执行的星期
- interval 执行间隔
- intervalUnit 执行间隔的单位（秒，分钟，小时，天，月，年，星期）
- repeatCount 重复次数

```java

    /**
     * 每次任务的间隔时间
     */
    public DailyTimeIntervalScheduleBuilder withInterval(int timeInterval, DateBuilder.IntervalUnit unit);

    /**
     * 每次任务的间隔时间 -- 单位秒
     */
    public DailyTimeIntervalScheduleBuilder withIntervalInSeconds(int intervalInSeconds);

    /**
     * 每次任务的间隔时间 -- 单位分
     */
    public DailyTimeIntervalScheduleBuilder withIntervalInMinutes(int intervalInMinutes);

    /**
     * 每次任务的间隔时间 -- 单位小时
     */
    public DailyTimeIntervalScheduleBuilder withIntervalInHours(int intervalInHours) {
        withInterval(intervalInHours, DateBuilder.IntervalUnit.HOUR);
        return this;
    }

    /**
     * 每周的哪几天执行
     */
    public DailyTimeIntervalScheduleBuilder onDaysOfTheWeek(Set<Integer> onDaysOfWeek);

    /**
     * 每周的哪几天执行
     */
    public DailyTimeIntervalScheduleBuilder onDaysOfTheWeek(Integer ... onDaysOfWeek);

    /**
     * 周一到周五每天都执行
     */
    public DailyTimeIntervalScheduleBuilder onMondayThroughFriday();

    /**
     * 星期六和星期天每天执行
     */
    public DailyTimeIntervalScheduleBuilder onSaturdayAndSunday();

    /**
     * 每天执行
     */
    public DailyTimeIntervalScheduleBuilder onEveryDay();

    /**
     * 任务什么时候开始(时，分，秒)，比如可以设置8：30开始
     */
    public DailyTimeIntervalScheduleBuilder startingDailyAt(TimeOfDay timeOfDay);

    /**
     * 任务什么时候结束(时，分，秒)，比如可以设置15：30结束
     */
    public DailyTimeIntervalScheduleBuilder endingDailyAt(TimeOfDay timeOfDay);

    /**
     * 在任务开始时间的和任务间隔的基础上，执行多少次。算的任务的结束时间
     * 比如开始时间是8:30  时间间隔是30分钟，设置2次。那么结束时间就是9:00
     */
    public DailyTimeIntervalScheduleBuilder endingDailyAfterCount(int count);

    /**
     * 这个不是忽略已经错失的触发的意思，而是说忽略MisFire策略。它会在资源合适的时候，重新触发所有的MisFire任务，并且不会影响现有的调度时间。
     */
    public DailyTimeIntervalScheduleBuilder withMisfireHandlingInstructionIgnoreMisfires() {
        misfireInstruction = Trigger.MISFIRE_INSTRUCTION_IGNORE_MISFIRE_POLICY;
        return this;
    }

    /**
     * 不对MisFire的任务做任何处理
     */
    public DailyTimeIntervalScheduleBuilder withMisfireHandlingInstructionDoNothing() {
        misfireInstruction = DailyTimeIntervalTrigger.MISFIRE_INSTRUCTION_DO_NOTHING;
        return this;
    }

    /**
     * 针对MisFire的任务马上执行一次
     */
    public DailyTimeIntervalScheduleBuilder withMisfireHandlingInstructionFireAndProceed() {
        misfireInstruction = CalendarIntervalTrigger.MISFIRE_INSTRUCTION_FIRE_ONCE_NOW;
        return this;
    }

    /**
     * 任务重复多少次
     */
    public DailyTimeIntervalScheduleBuilder withRepeatCount(int repeatCount);
```



```java
DailyTimeIntervalScheduleBuilder.dailyTimeIntervalSchedule()
    .startingDailyAt(TimeOfDay.hourAndMinuteOfDay(9, 0)) //第天9：00开始
    .endingDailyAt(TimeOfDay.hourAndMinuteOfDay(16, 0)) //16：00 结束 
    .onDaysOfTheWeek(MONDAY,TUESDAY,WEDNESDAY,THURSDAY,FRIDAY) //周一至周五执行
    .withIntervalInHours(1) //每间隔1小时执行一次
    .withRepeatCount(100) //最多重复100次（实际执行100+1次）
    .build();
 
DailyTimeIntervalScheduleBuilder.dailyTimeIntervalSchedule()
    .startingDailyAt(TimeOfDay.hourAndMinuteOfDay(9, 0)) //第天9：00开始
    .endingDailyAfterCount(10) //每天执行10次，这个方法实际上根据 startTimeOfDay+interval*count 算出 endTimeOfDay
    .onDaysOfTheWeek(MONDAY,TUESDAY,WEDNESDAY,THURSDAY,FRIDAY) //周一至周五执行
    .withIntervalInHours(1) //每间隔1小时执行一次
    .build();
```

### Scheduler

 Scheduler调度器,是Quartz框架的心脏，用来管理Trigger和Job，并保证Job能在Trigger设置的时间被触发执行

```java
public interface Scheduler {
    String DEFAULT_GROUP = "DEFAULT";
    String DEFAULT_RECOVERY_GROUP = "RECOVERING_JOBS";
    String DEFAULT_FAIL_OVER_GROUP = "FAILED_OVER_JOBS";
    String FAILED_JOB_ORIGINAL_TRIGGER_NAME = "QRTZ_FAILED_JOB_ORIG_TRIGGER_NAME";
    String FAILED_JOB_ORIGINAL_TRIGGER_GROUP = "QRTZ_FAILED_JOB_ORIG_TRIGGER_GROUP";
    String FAILED_JOB_ORIGINAL_TRIGGER_FIRETIME_IN_MILLISECONDS = "QRTZ_FAILED_JOB_ORIG_TRIGGER_FIRETIME_IN_MILLISECONDS_AS_STRING";
    String FAILED_JOB_ORIGINAL_TRIGGER_SCHEDULED_FIRETIME_IN_MILLISECONDS = "QRTZ_FAILED_JOB_ORIG_TRIGGER_SCHEDULED_FIRETIME_IN_MILLISECONDS_AS_STRING";
   
/**
     * 方法获取的是正在执行的Job
     */
    List<JobExecutionContext> getCurrentlyExecutingJobs() throws SchedulerException;



    /**
     * 获取Scheduler上的监听器ListenerManager， 比如可以监听job,trigger添加移除的状态等
     */
    ListenerManager getListenerManager()  throws SchedulerException;

    /**
     * 把jobDetail添加到调度系统中，并且把任务和Trigger关联起来
     */
    Date scheduleJob(JobDetail jobDetail, Trigger trigger)
            throws SchedulerException;

    /**
     * 开始调度Trigger关联的job
     */
    Date scheduleJob(Trigger trigger) throws SchedulerException;

    /**
     * 开始调度job,同时设置多个
     */
    void scheduleJobs(Map<JobDetail, Set<? extends Trigger>> triggersAndJobs, boolean replace) throws SchedulerException;

    /**
     * 开始调度job,而且这个job可以关联一个或多个触发器Trigger
     */
    void scheduleJob(JobDetail jobDetail, Set<? extends Trigger> triggersForJob, boolean replace) throws SchedulerException;

    /**
     * 从触发器中移除Trigger(Trigger对应的任务会被移除掉)
     */
    boolean unscheduleJob(TriggerKey triggerKey)
            throws SchedulerException;

    /**
     * 从触发器中移除Trigger(Trigger对应的任务会被移除掉)
     */
    boolean unscheduleJobs(List<TriggerKey> triggerKeys)
            throws SchedulerException;

    /**
     * 移除triggerKey，添加newTrigger
     */
    Date rescheduleJob(TriggerKey triggerKey, Trigger newTrigger)
            throws SchedulerException;

    /**
     * 添加job到触发器中，当然这个时候任务是不会执行的，触发关联到了触发器Trigger上
     */
    void addJob(JobDetail jobDetail, boolean replace)
            throws SchedulerException;

    /**
     * 添加任务
     */
    void addJob(JobDetail jobDetail, boolean replace, boolean storeNonDurableWhileAwaitingScheduling)
            throws SchedulerException;

    /**
     * 删除任务
     */
    boolean deleteJob(JobKey jobKey)
            throws SchedulerException;

    /**
     * 删除任务
     */
    boolean deleteJobs(List<JobKey> jobKeys)
            throws SchedulerException;

    /**
     * 立即执行任务
     */
    void triggerJob(JobKey jobKey)
            throws SchedulerException;

    /**
     * 立即执行任务
     */
    void triggerJob(JobKey jobKey, JobDataMap data)
            throws SchedulerException;

    /**
     * 暂停任务
     */
    void pauseJob(JobKey jobKey)
            throws SchedulerException;

    /**
     * 暂停任务
     */
    void pauseJobs(GroupMatcher<JobKey> matcher) throws SchedulerException;

    /**
     * 暂停触发器对应的任务
     */
    void pauseTrigger(TriggerKey triggerKey)
            throws SchedulerException;

    /**
     * 暂停触发器对应的任务
     */
    void pauseTriggers(GroupMatcher<TriggerKey> matcher) throws SchedulerException;

    /**
     * 恢复任务
     */
    void resumeJob(JobKey jobKey)
            throws SchedulerException;

    /**
     * 恢复任务
     */
    void resumeJobs(GroupMatcher<JobKey> matcher) throws SchedulerException;

    /**
     * 恢复触发器对应的任务
     */
    void resumeTrigger(TriggerKey triggerKey)
            throws SchedulerException;

    /**
     * 恢复触发器对应的任务
     */
    void resumeTriggers(GroupMatcher<TriggerKey> matcher) throws SchedulerException;

    /**
     * 暂停所有的任务
     */
    void pauseAll() throws SchedulerException;

    /**
     * 恢复所有的任务
     */
    void resumeAll() throws SchedulerException;

    /**
     * 获取所有任务的jobGroup名字
     */
    List<String> getJobGroupNames() throws SchedulerException;

    /**
     * 获取jobKey
     */
    Set<JobKey> getJobKeys(GroupMatcher<JobKey> matcher) throws SchedulerException;

    /**
     * 获取任务对应的触发器Trigger
     *
     */
    List<? extends Trigger> getTriggersOfJob(JobKey jobKey)
            throws SchedulerException;

    /**
     * 获取所有触发器的Group name
     */
    List<String> getTriggerGroupNames() throws SchedulerException;

    /**
     * 获取TriggerKey
     */
    Set<TriggerKey> getTriggerKeys(GroupMatcher<TriggerKey> matcher) throws SchedulerException;

    /**
     * 获取所有暂停任务对应的触发器的Group Name
     */
    Set<String> getPausedTriggerGroups() throws SchedulerException;

    /**
     * 获取JobDetail
     *
     */
    JobDetail getJobDetail(JobKey jobKey)
            throws SchedulerException;

    /**
     * 获取触发器Trigger
     */
    Trigger getTrigger(TriggerKey triggerKey)
            throws SchedulerException;

    /**
     * 获取触发器的状态
     */
    Trigger.TriggerState getTriggerState(TriggerKey triggerKey)
            throws SchedulerException;

    /**
     * 恢复触发器的状态
     */
    void resetTriggerFromErrorState(TriggerKey triggerKey)
            throws SchedulerException;
    /**
     * 添加Calendar
     * 这里稍微解释下Calendar：Quartz的Calendar可以用于排除一些特定的日期不执行任务
     */
    void addCalendar(String calName, Calendar calendar, boolean replace, boolean updateTriggers)
            throws SchedulerException;

    /**
     * 删除Calendar
     */
    boolean deleteCalendar(String calName) throws SchedulerException;

    /**
     * 获取Calendar
     */
    Calendar getCalendar(String calName) throws SchedulerException;

    /**
     * 获取Calendar对应的名字
     */
    List<String> getCalendarNames() throws SchedulerException;

    /**
     * 中断某个任务
     */
    boolean interrupt(JobKey jobKey) throws UnableToInterruptJobException;

    /**
     * 中断任务
     * JobExecutionContext#getFireInstanceId()
     */
    boolean interrupt(String fireInstanceId) throws UnableToInterruptJobException;

    /**
     * 判断对应job是否存在
     */
    boolean checkExists(JobKey jobKey) throws SchedulerException;

    /**
     * 判断对应触发器是否存在
     */
    boolean checkExists(TriggerKey triggerKey) throws SchedulerException;
    
}

```

### SchedulerListener

SchedulerListeners非常类似于TriggerListeners和JobListeners，除了它们在Scheduler本身中接收到事件的通知 - 不一定与特定触发器（trigger）或job相关的事件。

与计划程序相关的事件包括：添加job/触发器，删除job/触发器，调度程序中的严重错误，关闭调度程序的通知等



SchedulerListeners注册到调度程序的ListenerManager。SchedulerListeners几乎可以实现任何实现org.quartz.SchedulerListener接口的对象。

添加SchedulerListener：

```
scheduler.getListenerManager().addSchedulerListener(mySchedListener);
```

删除SchedulerListener：

```
scheduler.getListenerManager().removeSchedulerListener(mySchedListener);
```

#### SchedulerListenerSupport

### JobListener

#### JobListenerSupport 

### TriggerListener

#### SimpleTriggerListener 

