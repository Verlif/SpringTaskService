# TaskService

定时任务与多线程任务管理服务。  
简化定时任务方式，并集中管理定时任务，便于动态增加、关闭定时任务与多线程任务。  
有允许名单与屏蔽名单配置，不修改代码即可对定时任务进行控制。

## 使用

1. 添加Jitpack仓库源

> maven
> ```xml
> <repositories>
>    <repository>
>        <id>jitpack.io</id>
>        <url>https://jitpack.io</url>
>    </repository>
> </repositories>
> ```

2. 添加依赖

> maven
> ```xml
>    <dependencies>
>        <dependency>
>            <groupId>com.github.Verlif</groupId>
>            <artifactId>task-spring-boot-starter</artifactId>
>            <version>2.6.1-beta0.1</version>
>        </dependency>
>    </dependencies>
> ```

3. 启用任务管理服务

在任意配置类上使用`@EnableTaskService`注解启用任务管理服务

4. 制定定时任务

本项目中的定时任务由`TaskService`统一维护，开发者可以通过以下注解标记定时任务：
- `TaskTip`

需要注意的是，对于不同类型的任务，所需的注解参数是不同的。
- `CRON`需要`cron()`
- `REPEAT_DELAY`和`REPEAT_RATE`需要`interval()`，可选`delay()`和`unit()`
- `DELAY`需要`delay()`，可选`unit()`

使用注解后还需要实现`Runnable`接口才可以注册添加到`TaskService`。
且`TaskService`会在启动时从Bean池中主动加载添加了`@TaskTip`注解的任务。

## 举例

以下方式实现的定时任务需要通过`TaskService.insert(Runnable runnable)`方法手动添加到任务表中。

```java
@TaskTip(type = TaskType.DELAY, value="name", interval = 5000)
public class DemoSchedule implements Runnable {

    @Override
    public void run() {
        // TODO: 任务内容
    }
}
```

------

以下方式实现的定时任务会自动添加到任务表中。
- 增加了`@component`注解
```java
@Component
@TaskTip(type = TaskType.DELAY, value="name", interval = 5000)
public class DemoSchedule implements Runnable {

    @Override
    public void run() {
        // TODO: 任务内容
    }
}
```

------

注意，任务表中不允许出现同名任务，否则会添加不进去。可以在日志中发现未能添加的任务。

## 使用举例

```java
@Autowired
private TaskService taskService;
private Runnable task = new DemoRunnable();

    // 使用注解名称或默认名称添加可重复任务
    taskService.insert(task);
    // 使用动态名称添加可重复任务
    taskService.insert("name", task);
    // 取消名为[name]的任务
    taskService.cancel("name");
    // 2000毫秒后执行任务
    taskService.delay(task, 2000);
```

## 配置
在`application.yml`中通过以下配置限制任务表。被限制的任务是无法自动或手动添加到任务表中的。
```yaml
station:
  # 可重复任务配置
  task:
    # 允许的可重复任务列表。格式为yml的列表格式
    allowed:
      - name1
      - name2
    # 屏蔽的可重复任务列表
    blocked:
      - name3
      - name4
```
