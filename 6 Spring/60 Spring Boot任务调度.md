[TOC]

### Spring Boot任务调度

### 异步任务

在Java应用中，绝大多数情况下都是通过同步的方式来实现交互处理的；但是在处理与第三方系统交互的时候，容易造成响应迟缓的情况，之前大部分都是使用多线程来完成此类任务，其实，在Spring 3.x之后，就已经内置了@Async来完美解决这个问题。

两个注解：

**@EnableAysnc、@Aysnc**

```java
@SpringBootApplication
@EnableAsync	// 开启异步注解功能
public class DemoApplication {

	public static void main(String[] args) {
		SpringApplication.run(DemoApplication.class, args);
	}
}
```

定义一个异步任务

```java
@Service
public class AsyncService {

    /**
     * 告诉Spring这是一个异步方法
     */
    @Async
    public void hello(){
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("处理数据中...");
    }
}
```

写一个Controller

```java
@RestController
public class AsyncController {

    @Autowired
    AsyncService asyncService;

    @GetMapping("/asyn")
    public String hello(){
        asyncService.hello();
        return "success";
    }
}
```

如果Service中不加@Async注解，那么请求Controller时会等待3秒才有结果，而注解@Async之后且用@EnableAsync开启异步注解功能之后请求Controller时会立即响应。







### 定时任务

项目开发中经常需要执行一些定时任务，比如需要在每天凌晨时候，分析一次前一天的日志信息。Spring为我们提供了异步执行任务调度的方式，提供TaskExecutor 、TaskScheduler 接口。

**两个注解：@EnableScheduling、@Scheduled**

#### cron表达式

用于构建定时任务表达式

| **字段** | **允许值**              | **允许的特殊字符** |
| -------- | ----------------------- | ------------------ |
| 秒       | 0-59                    | , -   * /          |
| 分       | 0-59                    | , -   * /          |
| 小时     | 0-23                    | , -   * /          |
| 日期     | 1-31                    | , -   * ? / L W C  |
| 月份     | 1-12                    | , -   * /          |
| 星期     | 0-7或SUN-SAT   0,7是SUN | , -   * ? / L C #  |

| **特殊字符** | **代表含义**               |
| ------------ | -------------------------- |
| ,            | 枚举                       |
| -            | 区间                       |
| *            | 任意                       |
| /            | 步长                       |
| ?            | 日/星期冲突匹配            |
| L            | 最后                       |
| W            | 工作日                     |
| C            | 和calendar联系后计算过的值 |
| #            | 星期，4#2，第2个星期四     |

测试

```java
@SpringBootApplication
@EnableScheduling	// 开启基于注解的定时任务功能
public class DemoApplication {

	public static void main(String[] args) {
		SpringApplication.run(DemoApplication.class, args);
	}

}
```

```java
@Service
public class ScheduledService {
	
    /**
     * second(秒), minute（分）, hour（时）, day of month（日）, month（月）, day of week（周几）.
     * 0 * * * * MON-FRI
     *  【0 0/5 14,18 * * ?】 每天14点整，和18点整，每隔5分钟执行一次
     *  【0 15 10 ? * 1-6】 每个月的周一至周六10:15分执行一次
     *  【0 0 2 ? * 6L】每个月的最后一个周六凌晨2点执行一次
     *  【0 0 2 LW * ?】每个月的最后一个工作日凌晨2点执行一次
     *  【0 0 2-4 ? * 1#1】每个月的第一个周一凌晨2点到4点期间，每个整点都执行一次；
     */
    // @Scheduled(cron = "0 * * * * MON-SAT") 
    // @Scheduled(cron = "0,1,2,3,4 * * * * MON-SAT")  枚举
    // @Scheduled(cron = "0-4 * * * * MON-SAT") 区间
    
    @Scheduled(cron = "0/4 * * * * MON-SAT")  // 每4秒执行一次
    public void hello(){
        System.out.println("hello ... ");
    }
}
```













