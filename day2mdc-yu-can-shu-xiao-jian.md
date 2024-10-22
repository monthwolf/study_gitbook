# Day2-MDC与参数校检

## MDC定义与使用

MDC（Mapped Diagnostic Context）是 SLF4J 和 log4j 等日志框架提供的一种方案，它允许开发者将一些特定的数据（如用户ID、请求ID等）存储到当前线程的上下文中，使得这些数据可以在日志消息中使用。这对于跟踪多线程或高并发应用中的单个请求非常有用。

**在高并发环境中，由于多个请求可能同时处理，日志消息可能会交错在一起**。使用MDC，我们可以<mark style="background-color:orange;">为每个请求分配一个唯一的标识，并将该标识添加到每条日志消息中，从而方便地区分和跟踪每个请求的日志</mark>。

### 1. 配置日志格式

在 Log4j 或 Logback 的配置文件中，可以配置日志格式，以便输出 MDC 中的键值对。例如，在 Log4j2 的 `log4j2.xml` 配置文件中：

```xml
<PatternLayout pattern="%d{yyyy-MM-dd HH:mm:ss.SSS} [%t] %-5level %X{mdcKey} - %msg%n"/>
```

在 Logback 的 `logback.xml` 配置文件中：

```xml

    <!-- 其他内容... -->
    <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <!-- 格式化输出：%d 表示日期，%thread 表示线程名，%-5level：级别从左显示 5 个字符宽度 %errorMessage：日志消息，%n 是换行符-->
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%t] %-5level %X{mdcKey} - %msg%n</pattern>
        </encoder>
```

在上面的配置中，`%X{mdcKey}` 是一个占位符，用于输出 MDC 中键为 `mdcKey` 的值。

### 2. 在代码中使用 MDC

在代码中，可以在适当的位置（比如请求的开始处）设置 MDC 的值，在请求结束时清除这些值。

java

复制

```java
import org.slf4j.MDC;

public class MdcExample {

    public void processRequest() {
        try {
            // 假设这是请求的唯一标识
            String requestId = UUID.randomUUID().toString();
            MDC.put("requestId", requestId);
            
            // ... 你的业务逻辑代码 ...
            
            // 记录日志
            logger.info("Processing request");
        } finally {
            // 请求结束时清除 MDC
            MDC.clear();
        }
    }
}
```

### 注意事项

* MDC 的内容是保存在线程局部变量中的，因此它对子线程是不继承的。如果需要在子线程中使用父线程的 MDC 数据，需要手动复制。
* 使用 MDC 时，确保在所有可能的执行路径上都调用了 `MDC.clear()`，以避免内存泄漏。

## JSR380 参数校检

JSR 380 提供了一系列校验注解，可以用于各种数据校验场景。以下是一个全面的列表，包括了一些常用的校验注解及其简要说明：

### 核心注解

* `@NotNull`: 验证对象是否不为 null。
* `@Null`: 验证对象是否为 null。
* `@AssertTrue`: 验证 Boolean 对象是否为 true。
* `@AssertFalse`: 验证 Boolean 对象是否为 false。
* `@Min(value)`: 验证 Number 对象是否大于或等于指定的最小值。
* `@Max(value)`: 验证 Number 对象是否小于或等于指定的最大值。
* `@DecimalMin(value)`: 验证 Number 对象是否大于或等于指定的最小值（可以是小数）。
* `@DecimalMax(value)`: 验证 Number 对象是否小于或等于指定的最大值（可以是小数）。
* `@Size(min=, max=)`: 验证集合、数组或字符串的大小是否在指定的范围内。
* `@Digits(integer=, fraction=)`: 验证 Number 对象是否是一个整数或小数，并且整数部分和小数部分的位数分别不超过指定的值。
* `@Past`: 验证日期对象是否在当前时间之前。
* `@Future`: 验证日期对象是否在当前时间之后。
* `@Pattern(regex=, flags=)`: 验证字符串是否匹配指定的正则表达式。

### Bean Validation 2.0 新增注解

* `@Email`: 验证字符串是否是一个有效的电子邮件地址。
* `@NotEmpty`: 验证集合、数组或字符串是否不为 null 且不为空。
* `@NotBlank`: 验证字符串是否不为 null 且去除前后空格后长度不为 0。
* `@Positive`: 验证 Number 对象是否为正数。
* `@PositiveOrZero`: 验证 Number 对象是否为正数或零。
* `@Negative`: 验证 Number 对象是否为负数。
* `@NegativeOrZero`: 验证 Number 对象是否为负数或零。

### 分组

* `@GroupSequence`: 定义校验组序列。
* `@GroupSequenceProvider`: 动态定义校验组序列。

### 组合约束

* `@ReportAsSingleViolation`: 当一个字段上有多个约束时，将它们报告为一个单一的违规。

### Hibernate Validator 扩展

除了 JSR 380 标准注解，Hibernate Validator 还提供了一些额外的校验注解：

* `@Length`: 验证字符串的长度是否在指定的范围内。
* `@URL`: 验证字符串是否是一个有效的 URL。
* `@CreditCardNumber`: 验证字符串是否是一个有效的信用卡号码。
* `@ISBN`: 验证字符串是否是一个有效的 ISBN。
* `@SafeHtml`: 验证字符串是否是安全的 HTML。
* `@ScriptAssert`: 允许在注解中使用脚本表达式进行校验。 这些注解可以在实体类的字段、方法或构造器上使用，以实现各种数据校验需求。在使用时，需要确保你的项目中包含了相应的校验器实现，如 Hibernate Validator。

## 在Springboot中使用JSR380

在 Spring Boot 中使用 JSR 380 参数校验非常简单，因为 Spring Boot 自动配置了 Bean Validation。以下是如何在 Spring Boot 应用程序中使用 JSR 380 参数校验的步骤：

### 1. 添加依赖

确保你的 `pom.xml` 或 `build.gradle` 文件中包含以下依赖项： 对于 Maven:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

对于 Gradle:

```groovy
implementation 'org.springframework.boot:spring-boot-starter-validation'
```

### 2. 在实体类上添加注解

在你的实体类上使用 JSR 380 校验注解来定义校验规则：

```java
import javax.validation.constraints.*;
import java.math.BigDecimal;
import java.util.Date;
public class User {
    @NotNull
    @Size(min = 2, max = 30)
    private String username;
    @NotNull
    @Size(min = 5, max = 50)
    private String password;
    @Min(0)
    @Max(150)
    private int age;
    @Past
    private Date birthDate;
    @DecimalMin("0.0")
    @DecimalMax("1000000.00")
    private BigDecimal salary;
    @Email
    private String email;
    // Getters and setters...
}
```

### 3. 在控制器方法上使用校验

在控制器的方法参数上使用 `@Valid` 注解来触发校验：

```java
import org.springframework.validation.annotation.Validated;
import org.springframework.web.bind.annotation.*;
import javax.validation.Valid;
@RestController
@RequestMapping("/users")
@Validated
public class UserController {
    @PostMapping
    public ResponseEntity<String> createUser(@RequestBody @Valid User user) {
        // 如果用户对象不符合校验规则，将会抛出 MethodArgumentNotValidException
        // 业务逻辑处理，例如保存用户
        return ResponseEntity.ok("User created successfully");
    }
    @PutMapping("/{id}")
    public ResponseEntity<String> updateUser(@PathVariable("id") Long id, @RequestBody @Valid User user) {
        // 业务逻辑处理，例如更新用户
        return ResponseEntity.ok("User updated successfully");
    }
}
```

### 4. 处理校验异常

Spring Boot 会自动处理校验异常，你可以使用 `@ControllerAdvice` 和 `@ExceptionHandler` 来全局处理校验异常：

```java
import org.springframework.http.HttpStatus;
import org.springframework.web.bind.MethodArgumentNotValidException;
import org.springframework.web.bind.annotation.*;
@ControllerAdvice
public class GlobalExceptionHandler {
    @ExceptionHandler(MethodArgumentNotValidException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    @ResponseBody
    public ApiError handleValidationExceptions(MethodArgumentNotValidException ex) {
        ApiError apiError = new ApiError(HttpStatus.BAD_REQUEST);
        apiError.setMessage("Validation error");
        apiError.addValidationErrors(ex.getBindingResult().getFieldErrors());
        apiError.addValidationError(ex.getBindingResult().getGlobalErrors());
        return apiError;
    }
}
```

这里的 `ApiError` 是一个自定义的错误响应类，你可以根据需要自行定义。 通过以上步骤，你就可以在 Spring Boot 应用程序中全面使用 JSR 380 参数校验了。记得在出现校验错误时，Spring Boot 会返回一个 400 Bad Request 响应。

## MDC在项目中使用

在 `logback-weblog.xml` 配置文件中，要引用`ApiOperationLogAspect` 日志切面类 `doAround()` 方法中请求的跟踪标识`traceId`，需要这样配置`pattern`：

```xml
[TraceId: %X{traceId}] %d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n
```

完整配置如下:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration >
    <jmxConfigurator/>
    <include resource="org/springframework/boot/logging/logback/defaults.xml" />

    <!-- 应用名称 -->
    <property scope="context" name="appName" value="weblog" />
    <!-- 自定义日志输出路径，以及日志名称前缀 -->
<!--   日志文件存储地址linux格式: /app/weblog/logs/${appName}.%d{yyyy-MM-dd}
       测试时使用的windows格式： E:\\work\\java\\weblog\\logs\\${appName}.%d{yyyy-MM-dd}
-->
    <property name="LOG_FILE" value="E:\\work\\java\\weblog\\logs\\${appName}.%d{yyyy-MM-dd}"/>
    <property name="FILE_LOG_PATTERN" value="[TraceId: %X{traceId}] %d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n"/>
    <!--<property name="CONSOLE_LOG_PATTERN" value="${FILE_LOG_PATTERN}"/>-->

    <!-- 按照每天生成日志文件 -->
    <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!-- 日志文件输出的文件名 -->
            <FileNamePattern>${LOG_FILE}-%i.log</FileNamePattern>
            <!-- 日志文件保留天数 -->
            <MaxHistory>30</MaxHistory>
            <!-- 日志文件最大的大小 -->
            <TimeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>10MB</maxFileSize>
            </TimeBasedFileNamingAndTriggeringPolicy>
        </rollingPolicy>
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <!-- 格式化输出：%d 表示日期，%thread 表示线程名，%-5level：级别从左显示 5 个字符宽度 %errorMessage：日志消息，%n 是换行符-->
            <pattern>${FILE_LOG_PATTERN}</pattern>
        </encoder>
    </appender>

    <!-- dev 环境（仅输出到控制台） -->
    <springProfile name="dev">
        <include resource="org/springframework/boot/logging/logback/console-appender.xml" />
        <root level="info">
            <appender-ref ref="CONSOLE" />
        </root>
    </springProfile>

    <!-- prod 环境（仅输出到文件中） -->
    <springProfile name="prod">
        <include resource="org/springframework/boot/logging/logback/console-appender.xml" />
        <root level="INFO">
            <appender-ref ref="FILE" />
        </root>
    </springProfile>
</configuration>
```

将Web环境换为生产环境`prod`后，重新请求测试api，得到文件中日志格式已改变：

![image-20241022101028030](<.gitbook/assets/image 20241022101028030.png>)

## JSR380在项目中使用

### 添加参数校检

按照[添加依赖](day2mdc-yu-can-shu-xiao-jian.md#id-1.-tian-jia-yi-lai)的说明，在 `weblog-web` 模块中的 `pom.xml` 文件添加参数校验依赖：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

使用注解

```java
package cn.dogalist.weblog.web.model;

import lombok.Data;
import javax.validation.constraints.*;

@Data
public class User {
    @NotBlank(message = "用户名不能为空")
    private String username;
    @NotNull(message = "性别不能为空")
    private Integer sex;
    @NotNull(message = "年龄不能为空")
    @Min(value = 18,message = "年龄必须大于等于18")
    @Max(value = 100,message = "年龄必须小于等于100")
    private Integer age;
    @NotBlank(message = "邮箱不能为空")
    @Email(message = "邮箱格式不正确")
    private String email;
}
```

在`controller`层测试请求函数中添加捕获，将错误信息返回

```java
package cn.dogalist.weblog.web.controller;

import cn.dogalist.weblog.common.aspect.ApiOperationLog;
import cn.dogalist.weblog.web.model.User;
import lombok.extern.slf4j.Slf4j;
import org.springframework.http.ResponseEntity;
import org.springframework.validation.BindingResult;
import org.springframework.validation.FieldError;
import org.springframework.validation.annotation.Validated;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RestController;
import java.util.stream.Collectors;

@RestController
@Slf4j
public class TestController {
    @PostMapping("/test")
    @ApiOperationLog(description = "测试接口")
    public ResponseEntity<String> test(@RequestBody @Validated User user, BindingResult bindingResult) {
        // 是否存在校检错误
        if (bindingResult.hasErrors()) {
            String errorMsg = bindingResult.getFieldErrors()
                    .stream()
                    .map(FieldError::getDefaultMessage)
                    .collect(Collectors.joining(", "));
            return ResponseEntity.badRequest().body(errorMsg);
        }

        return ResponseEntity.ok("参数无误");
    }
}
```

测试入参正确和不正确的情况下的返回值：

* 入参正确

参数：

```json
{
    "username": "admin",
    "sex": 1,
    "age": 32,
    "email": "123124@qq.com"
}
```

结果：

![入参正确](<.gitbook/assets/image 20241022110432534.png>)

* 入参错误

参数：

```json
{
    "username": "",
    "sex": 22,
    "age": 120,
    "email": "123124qq.com"
}
```

结果：

![入参不正确](<.gitbook/assets/image 20241022110504415.png>)
