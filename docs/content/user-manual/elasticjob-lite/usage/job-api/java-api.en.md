+++
title = "Use Java API"
weight = 2
chapter = true
+++

## Job configuration

ElasticJob-Lite uses the builder mode to create job configuration objects.
The code example is as follows:

```java
    JobConfiguration jobConfig = JobConfiguration.newBuilder("myJob", 3).cron("0/5 * * * * ?").shardingItemParameters("0=Beijing,1=Shanghai,2=Guangzhou").build();
```

## Job start

ElasticJob-Lite scheduler is divided into two types: timed scheduling and one-time scheduling.
Each scheduler needs three parameters: registry configuration, job object (or job type), and job configuration when it starts.

### Timed scheduling

```java
public class JobDemo {
    
    public static void main(String[] args) {
        // Class-based Scheduling Jobs
        new ScheduleJobBootstrap(createRegistryCenter(), new MyJob(), createJobConfiguration()).schedule();
        // Type-based Scheduling Jobs
        new ScheduleJobBootstrap(createRegistryCenter(), "MY_TYPE", createJobConfiguration()).schedule();
    }
    
    private static CoordinatorRegistryCenter createRegistryCenter() {
        CoordinatorRegistryCenter regCenter = new ZookeeperRegistryCenter(new ZookeeperConfiguration("zk_host:2181", "elastic-job-demo"));
        regCenter.init();
        return regCenter;
    }
    
    private static JobConfiguration createJobConfiguration() {
        // Create job configuration
        ...
    }
}
```

### One-Off scheduling

```java
public class JobDemo {
    
    public static void main(String[] args) {
        OneOffJobBootstrap jobBootstrap = new OneOffJobBootstrap(createRegistryCenter(), new MyJob(), createJobConfiguration());
        // One-time scheduling can be called multiple times
        jobBootstrap.execute();
        jobBootstrap.execute();
        jobBootstrap.execute();
    }
    
    private static CoordinatorRegistryCenter createRegistryCenter() {
        CoordinatorRegistryCenter regCenter = new ZookeeperRegistryCenter(new ZookeeperConfiguration("zk_host:2181", "elastic-job-demo"));
        regCenter.init();
        return regCenter;
    }
    
    private static JobConfiguration createJobConfiguration() {
        // Create job configuration
        ...
    }
}
```

## Job Dump

Using ElasticJob may meet some distributed problem which is not easy to observe.

Because of developer can not debug in production environment, ElasticJob provide `dump` command to export job runtime information for debugging.

Please refer to [Operation Manual](/en/user-manual/elasticjob-lite/operation/dump) for more details.

The example below is how to configure spring namespace for open listener port to dump.

```java
public class JobMain {
    
    public static void main(final String[] args) {
        SnapshotService snapshotService = new SnapshotService(regCenter, 9888).listen();
    }
    
    private static CoordinatorRegistryCenter createRegistryCenter() {
        // create registry center
    }
}
```

## Configuration error handler strategy

In the process of using ElasticJob-Lite, when the job is abnormal, the following error handling strategies can be used.

| *Error handler strategy name*            | *Description*                                                 |  *Built-in*  | *Default*| *Extra config*   |
| ---------------------------------------- | ------------------------------------------------------------- |  -------     |  --------|  --------------  |
| Log Strategy                             | Log error and do not interrupt job                            |   Yes        |     Yes  |                  |
| Throw Strategy                           | Throw system exception and interrupt job                      |   Yes        |          |                  |
| Ignore Strategy                          | Ignore exception and do not interrupt job                     |   Yes        |          |                  |
| Email Notification Strategy              | Send email message notification and do not interrupt job      |              |          |    Yes           |
| Wechat Enterprise Notification Strategy  | Send wechat message notification and do not interrupt job     |              |          |    Yes           |
| Dingtalk Notification Strategy           | Send dingtalk message notification and do not interrupt job   |              |          |    Yes           |

### Log Strategy
```java
public class JobDemo {
    
    public static void main(String[] args) {
        //  Scheduling Jobs
        new ScheduleJobBootstrap(createRegistryCenter(), new MyJob(), createScheduleJobConfiguration()).schedule();
        // One-time Scheduling Jobs
        new OneOffJobBootstrap(createRegistryCenter(), new MyJob(), createOneOffJobConfiguration()).execute();
    }
    
    private static JobConfiguration createScheduleJobConfiguration() {
        // Create scheduling job configuration, and the use of log strategy
        return JobConfiguration.newBuilder("myScheduleJob", 3).cron("0/5 * * * * ?").jobErrorHandlerType("LOG").build();
    }

    private static JobConfiguration createOneOffJobConfiguration() {
        // Create one-time job configuration, and the use of log strategy
        return JobConfiguration.newBuilder("myOneOffJob", 3).jobErrorHandlerType("LOG").build();
    }

    private static CoordinatorRegistryCenter createRegistryCenter() {
        // create registry center
        ...
    }
}
```

### Throw Strategy
```java
public class JobDemo {
    
    public static void main(String[] args) {
        //  Scheduling Jobs
        new ScheduleJobBootstrap(createRegistryCenter(), new MyJob(), createScheduleJobConfiguration()).schedule();
        // One-time Scheduling Jobs
        new OneOffJobBootstrap(createRegistryCenter(), new MyJob(), createOneOffJobConfiguration()).execute();
    }
    
    private static JobConfiguration createScheduleJobConfiguration() {
        // Create scheduling job configuration, and the use of throw strategy.
        return JobConfiguration.newBuilder("myScheduleJob", 3).cron("0/5 * * * * ?").jobErrorHandlerType("THROW").build();
    }

    private static JobConfiguration createOneOffJobConfiguration() {
        // Create one-time job configuration, and the use of throw strategy
        return JobConfiguration.newBuilder("myOneOffJob", 3).jobErrorHandlerType("THROW").build();
    }

    private static CoordinatorRegistryCenter createRegistryCenter() {
        // create registry center
        ...
    }
}
```


### Ignore Strategy
```java
public class JobDemo {
    
    public static void main(String[] args) {
        //  Scheduling Jobs
        new ScheduleJobBootstrap(createRegistryCenter(), new MyJob(), createScheduleJobConfiguration()).schedule();
        // One-time Scheduling Jobs
        new OneOffJobBootstrap(createRegistryCenter(), new MyJob(), createOneOffJobConfiguration()).execute();
    }
    
    private static JobConfiguration createScheduleJobConfiguration() {
        // Create scheduling job configuration, and the use of ignore strategy.
        return JobConfiguration.newBuilder("myScheduleJob", 3).cron("0/5 * * * * ?").jobErrorHandlerType("IGNORE").build();
    }

    private static JobConfiguration createOneOffJobConfiguration() {
        // Create one-time job configuration, and the use of ignore strategy.
        return JobConfiguration.newBuilder("myOneOffJob", 3).jobErrorHandlerType("IGNORE").build();
    }

    private static CoordinatorRegistryCenter createRegistryCenter() {
        // create registry center.
        ...
    }
}
```

### Email Notification Strategy

Please refer to [here](/en/user-manual/elasticjob-lite/configuration/built-in-strategy/error-handler/#email-notification-strategy) for more details.

Maven POM:
```xml
<dependency>
    <groupId>org.apache.shardingsphere.elasticjob</groupId>
    <artifactId>elasticjob-error-handler-email</artifactId>
    <version>${latest.release.version}</version>
</dependency>
```
```java
public class JobDemo {
    
    public static void main(String[] args) {
        //  Scheduling Jobs
        new ScheduleJobBootstrap(createRegistryCenter(), new MyJob(), createScheduleJobConfiguration()).schedule();
        // One-time Scheduling Jobs
        new OneOffJobBootstrap(createRegistryCenter(), new MyJob(), createOneOffJobConfiguration()).execute();
    }
    
    private static JobConfiguration createScheduleJobConfiguration() {
        // Create scheduling job configuration, and the use of email notification strategy.
        JobConfiguration jobConfig = JobConfiguration.newBuilder("myScheduleJob", 3).cron("0/5 * * * * ?").jobErrorHandlerType("EMAIL").build();
        setEmailConfiguration(jobConfig);
        return jobConfig;

    }

    private static JobConfiguration createOneOffJobConfiguration() {
        // Create one-time job configuration, and the use of email notification strategy.
        JobConfiguration jobConfig = JobConfiguration.newBuilder("myOneOffJob", 3).jobErrorHandlerType("EMAIL").build();
        setEmailConfiguration(jobConfig);
        return jobConfig;
    }

    private static void setEmailConfiguration(final JobConfiguration jobConfig) {
        // Set the mail configuration.
        jobConfig.getExtraConfigurations().add(new EmailConfiguration(
                "host", 465, "username", "password", true, "Test elasticJob error message", "from@xxx.com", "to1@xxx.com,to2xxx.com", "cc@xxx.com", "bcc@xxx.com", false));
    }

    private static CoordinatorRegistryCenter createRegistryCenter() {
        // create registry center.
        ...
    }
}
```

### Wechat Enterprise Notification Strategy

Please refer to [here](/en/user-manual/elasticjob-lite/configuration/built-in-strategy/error-handler/#wechat-enterprise-notification-strategy) for more details.

Maven POM:
```xml
<dependency>
    <groupId>org.apache.shardingsphere.elasticjob</groupId>
    <artifactId>elasticjob-error-handler-wechat</artifactId>
    <version>${latest.release.version}</version>
</dependency>
```
```java
public class JobDemo {
    
    public static void main(String[] args) {
        //  Scheduling Jobs.
        new ScheduleJobBootstrap(createRegistryCenter(), new MyJob(), createScheduleJobConfiguration()).schedule();
        // One-time Scheduling Jobs.
        new OneOffJobBootstrap(createRegistryCenter(), new MyJob(), createOneOffJobConfiguration()).execute();
    }
    
    private static JobConfiguration createScheduleJobConfiguration() {
        // Create scheduling job configuration, and the use of wechat enterprise notification strategy.
        JobConfiguration jobConfig = JobConfiguration.newBuilder("myScheduleJob", 3).cron("0/5 * * * * ?").jobErrorHandlerType("WECHAT").build();
        setWechatConfiguration(jobConfig);
        return jobConfig;

    }

    private static JobConfiguration createOneOffJobConfiguration() {
        // Create one-time job configuration, and the use of wechat enterprise notification strategy.
        JobConfiguration jobConfig = JobConfiguration.newBuilder("myOneOffJob", 3).jobErrorHandlerType("WECHAT").build();
        setWechatConfiguration(jobConfig);
        return jobConfig;
    }

    private static void setWechatConfiguration(final JobConfiguration jobConfig) {
        // Set the configuration for the enterprise wechat.
        jobConfig.getExtraConfigurations().add(new WechatConfiguration("webhook", 3000, 5000));
    }

    private static CoordinatorRegistryCenter createRegistryCenter() {
        // create registry center.
        ...
    }
}
```

### Dingtalk Notification Strategy

Please refer to [here](/en/user-manual/elasticjob-lite/configuration/built-in-strategy/error-handler/#dingtalk-notification-strategy) for more details.

Maven POM:
```xml
<dependency>
    <groupId>org.apache.shardingsphere.elasticjob</groupId>
    <artifactId>elasticjob-error-handler-dingtalk</artifactId>
    <version>${latest.release.version}</version>
</dependency>
```
```java
public class JobDemo {
    
    public static void main(String[] args) {
        //  Scheduling Jobs.
        new ScheduleJobBootstrap(createRegistryCenter(), new MyJob(), createScheduleJobConfiguration()).schedule();
        // One-time Scheduling Jobs.
        new OneOffJobBootstrap(createRegistryCenter(), new MyJob(), createOneOffJobConfiguration()).execute();
    }
    
    private static JobConfiguration createScheduleJobConfiguration() {
        // Create scheduling job configuration, and the use of wechat enterprise notification strategy.
        JobConfiguration jobConfig = JobConfiguration.newBuilder("myScheduleJob", 3).cron("0/5 * * * * ?").jobErrorHandlerType("DINGTALK").build();
        setWechatConfiguration(jobConfig);
        return jobConfig;

    }

    private static JobConfiguration createOneOffJobConfiguration() {
        // Create one-time job configuration, and the use of wechat enterprise notification strategy.
        JobConfiguration jobConfig = JobConfiguration.newBuilder("myOneOffJob", 3).jobErrorHandlerType("DINGTALK").build();
        setWechatConfiguration(jobConfig);
        return jobConfig;
    }

    private static void setDingtalkConfiguration(final JobConfiguration jobConfig) {
        // Set the configuration of the dingtalk.
        jobConfig.getExtraConfigurations().add(new DingtalkConfiguration("webhook", 
                "keyword", "secret", 3000, 5000));
    }

    private static CoordinatorRegistryCenter createRegistryCenter() {
        // create registry center.
        ...
    }
}
```