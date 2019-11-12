# Hive与Hook

## Hive概述

[Hive](./README.md)

## Hook概述

Hook是一种在处理过程中拦截事件，消息或函数调用的机制。Hive Hook 可以将事件绑定在内部 Hive 的执行流程中，而无需重新编译 Hive。Hook 提供了扩展和集成外部组件的方式。根据不同的 Hook 类型，可以在不同的阶段运行。

- Pre-execution Hook 在执行引擎执行查询之前被调用。这个需要在 Hive 对查询计划进行过优化之后才可以使用。
- Post-execution hooks 在执行计划执行结束结果返回给用户之前被调用。
- Failure-execution hooks 在执行计划失败之后被调用。
- Pre-driver-run 和 post-driver-run 是在查询运行的时候运行的。
- Pre-semantic-analyzer and Post-semantic-analyzer Hook 在 Hive 对查询语句进行语义分析的时候调用。


## Hive查询的生命周期

1. Driver.run() 接收命令。
2. org.apache.hadoop.hive.ql.HiveDriverRunHook.preDriverRun() 读取 hive.exec.pre.hooks 中的配置，来看 pre 需要运行的 hook.
3. org.apache.hadoop.hive.ql.Driver.compile() 开始处理 query，并且生成 AST。
4. org.apache.hadoop.hive.ql.parse.AbstractSemanticAnalyzerHook 调用 preAnalyze() 方法。
5. Semantic analysis AST 上的语义分析。
6. org.apache.hadoop.hive.ql.parse.AbstractSemanticAnalyzerHook.postAnalyze() 在所有语义分析 Hook 方法执行完之后被调用。
7. 创建物理查询计划。
8. Driver.execute() 准备运行 job。
9. org.apache.hadoop.hive.ql.hooks.ExecuteWithHookContext.run() 运行所有 hook。
10. org.apache.hadoop.hive.ql.hooks.ExecDriver.execute() 运行所有任务的 query。
11. 每一个 job org.apache.hadoop.hive.ql.stats.ClientStatsPublisher.run() 都会被调用，来提供 job 的一些统计数据。间隔的默认值由 hive.exec.counters.pull.interval 来控制，默认是1000ms。
12. 完成所有 task。
13. 如果查询任务失败了，会调用 hive.exec.failure.hooks.
14. 运行执行后的 Hook ExecuteWithHookContext.run()
15. 运行 org.apache.hadoop.hive.ql.HiveDriverRunHook.postDriverRun()。注意这是在查询完成，结果返回之前执行的。
16. 返回结果。


## Hive Hook API

Hive 支持很多类型的 Hook。

1. PreExecute 和 PostExecute 扩展自 Hook 接口。
2. ExecuteWithHookContext 扩展 the Hook 接口，并且传递 HookContext 给 Hook. HookContext 封装了 Hook 所需要的所有信息。
3. HiveDriverRunHook 扩展 Hook 接口，运行常规的 Hive 逻辑在 Driver 上。
4. HiveSemanticAnalyzerHook 扩展 Hook 接口对查询进行语义分析. preAnalyze() 和 postAnalyze() 方法可以在语义分析的之前之后运行。
5. HiveSessionHook 扩展 Hook 接口提供 session 级别的 hooks. 当 Session 开始，hook 就会被调用。

Hive 的代码里有很多关于 Hook 的例子。

1. DriverTestHook 是非常简单的 HiveDriverRunHook，并且可以打印执行的命令在结果里。
2. PreExecutePrinter 和 PostExecutePrinter 是执行前后的 Hook，可以打印参数。
3. ATSHook 是运行时的 ExecuteWithHookContext，可以把运行时的 query 和 plan 发送到 YARN 的 timeline server。
4. EnforceReadOnlyTables 是 ExecuteWithHookContext，可以理解成检查是否有修改只读表的一个 Hook。
5. LineageLogger 是 ExecuteWithHookContext，可以记录 query 的血缘信息在日志文件中。
6. PostExecOrcFileDump 是一个运行后的 Hook，可以打印 ORC 文件的信息。
7. PostExecTezSummaryPrinter 是一个运行后的 Hook，可以打印 Tez 的 summary。
8. UpdateInputAccessTimeHook 是一个运行前的 Hook，可以更新所有的的在 query 运行前，所有的输入表的访问时间。


## 一个简单的 Hive Hook

现在准备写一个 Hive Hook 的例子，并且加载进去 Hive 里面。这个例子非常简单，用的是 pre-execution hook，意思就是在查询执行之前，打印 `“Hello from the hook !!"`。

**1.**创建工程的 pom 文件。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>hive-hook-example</groupId>
    <artifactId>Hive-hook-example</artifactId>
    <version>1.0</version>
</project>
```

**2.**在项目中引入 hive-exec 的 package。
```xml
<dependencies>
    <dependency>
        <groupId>org.apache.hive</groupId>
        <artifactId>hive-exec</artifactId>
        <version>1.1.0</version>
    </dependency>
</dependencies>
```

**3.**创建一个 Class 实现了 Hook 接口 org.apache.hadoop.hive.ql.hooks.ExecuteWithHookContext，这个接口只有一个方法。

```java
import org.apache.hadoop.hive.ql.hooks.ExecuteWithHookContext;
import org.apache.hadoop.hive.ql.hooks.HookContext;

public class HiveExampleHook implements ExecuteWithHookContext {
    public void run(HookContext hookContext) throws Exception {
        System.out.println("Hello from the hook !!");
    }
}
```

**4.**构建出的 jar 包放在 Hive 的 classpath，并且设置成 pre-execution hook。

打包hook
```
mvn package
```

设置为per-execution hook
```
add jar target/Hive-hook-example-1.0.jar;
set hive.exec.pre.hooks=HiveExampleHook;
```

这就是使用 Hive Hook 的全部，现在就可以看到每次 Hive 语句执行之前都会看到 `"Hello from hook !!"`。


## Metastore Hooks

Hive 也有 metastore 的定制的 hook 来处理关于 metastore 变化的事件。 Metastore initialization hooks 会在 metastore 初始化的时候被调用。比如说保持 HBase 同步到 Hive 的 metastore 中。如果想记录外部组件在 Hive 中创建了那些 tables/databases 之类的，那么这个 Hook 就很合适了。

```java
public void preCreateTable(Table table) throws MetaException;

public void rollbackCreateTable(Table table) throws MetaException;

public void commitCreateTable(Table table) throws MetaException;

public void preDropTable(Table table) throws MetaException;

public void rollbackDropTable(Table table) throws MetaException;

public void commitDropTable(Table table, boolean deleteData) throws MetaException;
```


## 注意

需要注意的是 Hook 的实例不能复用。
Hook 是在正常的 Hive 流程中被调用的，所以需要避免那些影响性能的操作。


## Resources

[https://github.com/apache/hive/tree/master/ql/src/java/org/apache/hadoop/hive/ql/hooks](https://github.com/apache/hive/tree/master/ql/src/java/org/apache/hadoop/hive/ql/hooks)
