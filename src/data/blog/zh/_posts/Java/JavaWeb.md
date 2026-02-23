---
title: Java学习 - JavaWeb
tags: []
copyright: true
date: 2025-07-03 22:20:40
permalink:
categories: Java
description:
ogImage: http://img.wiretender.top/img/20250703231202262.webp
mathjax: true
katex: true
---

# JavaWeb

## 事务进阶-propagation

- 事务传播行为：指的就是当一个事务办法被另外一个实物办法调用时，这个事务方法应该如何进行事务控制
- `REQUIRED`: 默认值- 需要事务，有则加入，无则创建新事务
- `REQUIRES_NEW`: 需要新事务，无论有无，总是创建新事务

```java
@Transactional(propagation = Propagation.REQUIRES_NEW) 
// 这个方法在新的事务中创建并提交
@Override
public void insertLog(EmpLog empLog) {
    empLogMapper.insert(empLog);
}
```

![image-20250628212500961](http://img.wiretender.top/img/20250628212501367.webp)

## 文件上传

### 简介

- 将我们的文件上传到服务器，供其他用户浏览或下载的过程
- 比如发微博和发朋友圈

### 前端代码示例

![image-20250628212737057](http://img.wiretender.top/img/20250628212737292.webp)

### 后端代码

```java
/**
 * @author Wiretender
 * @version 1.0
 */
package com.itheima.controller;

import com.itheima.pojo.Result;
import lombok.extern.slf4j.Slf4j;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.multipart.MultipartFile;

import java.io.File;
import java.io.IOException;
import java.util.UUID;

@Slf4j
@RestController
public class UploadController {

    @PostMapping("/upload")
    public Result upload(String name, Integer age, MultipartFile file) throws IOException {
        log.info("接收参数： {} {} {}", name, age, file);
        String originalFilename = file.getOriginalFilename();
        String extension = originalFilename.substring(originalFilename.lastIndexOf("."));
        String newFileName = UUID.randomUUID().toString() + extension;
        file.transferTo(new File("E:/BaiduNetdiskDownload/images/" + originalFilename));
        return Result.success();
    }
}

```

## 全局异常处理器

我们该怎么样定义全局异常处理器？

- 定义全局异常处理器非常简单，就是定义一个类，在类上加上一个注解 ==@RestControllerAdvice==，加上这个注解就代表我们定义了一个全局异常处理器。
- 在全局异常处理器当中，需要定义一个方法来捕获异常，在这个方法上需要加上注解@ExceptionHandler。通过@ExceptionHandler 注解当中的 value 属性来指定我们要捕获的是哪一类型的异常。

```Java
@RestControllerAdvice
public class GlobalExceptionHandler {
    
    //处理异常
    @ExceptionHandler
    public Result ex(Exception e){//方法形参中指定能够处理的异常类型
        e.printStackTrace();//打印堆栈中的异常信息
        //捕获到异常之后，响应一个标准的Result
        return Result.error("对不起,操作失败,请联系管理员");
    }
    
}
```

> @RestControllerAdvice = @ControllerAdvice + @ResponseBody
>
> 处理异常的方法返回值会转换为 json 后再响应给前端

## 实现职位统计的函数（==SQL 语句重点==）

**1). 定义封装结果对象 JobOption**

在 `com.itheima.pojo` 包中定义实体类 `JobOption`

```Java
package com.itheima.pojo;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

import java.util.List;

@Data
@NoArgsConstructor
@AllArgsConstructor
public class JobOption {
    private List jobList;
    private List dataList;
}
```

**1). 定义 ReportController，并添加方法。**

```Java
@Slf4j
@RequestMapping("/report")
@RestController
public class ReportController {

    @Autowired
    private ReportService reportService;

    /**
     * 统计各个职位的员工人数
     */
    @GetMapping("/empJobData")
    public Result getEmpJobData(){
        log.info("统计各个职位的员工人数");
        JobOption jobOption = reportService.getEmpJobData();
        return Result.success(jobOption);
    }
}
```

**2). 定义 ReportService 接口，并添加接口方法。**

```Java
public interface ReportService {
    /**
     * 统计各个职位的员工人数
     * @return
     */
    JobOption getEmpJobData();
}
```

**3). 定义 ReportServiceImpl 实现类，并实现方法**

```Java
@Service
public class ReportServiceImpl implements ReportService {

    @Autowired
    private EmpMapper empMapper;
        
    @Override
    public JobOption getEmpJobData() {
        List<Map<String,Object>> list = empMapper.countEmpJobData();
        List<Object> jobList = list.stream().map(dataMap -> dataMap.get("pos")).toList();
        List<Object> dataList = list.stream().map(dataMap -> dataMap.get("total")).toList();
        return new JobOption(jobList, dataList);
    }
}
```

**4). 定义 EmpMapper 接口**

统计的是员工的信息，所以需要操作的是员工表。 所以代码我们就写在 `EmpMapper` 接口中即可。

```Java
/**
 * 统计各个职位的员工人数
 */
@MapKey("pos")
List<Map<String,Object>> countEmpJobData();
```

如果查询的记录往 Map 中封装，可以通过@MapKey 注解指定返回的 map 中的唯一标识是那个字段。【也可以不指定】

**5). 定义 EmpMapper.xml**

```XML
<!-- 统计各个职位的员工人数 -->
<select id="countEmpJobData" resultType="java.util.Map">
    select
    (case job when 1 then '班主任' 
                     when 2 then '讲师' 
                     when 3 then '学工主管' 
                     when 4 then '教研主管' 
                     when 5 then '咨询师' 
                     else '其他' end)  pos,
    count(*) total
    from emp group by job
    order by total
</select>
```

**case 流程控制函数：**

- 语法一：case when cond1 then res1 [ when cond2 then res2 ] else res end ;
- 含义：如果 cond1 成立， 取 res1。  如果 cond2 成立，取 res2。 如果前面的条件都不成立，则取 res。

- 语法二（仅适用于等值匹配）：case expr when val1 then res1 [ when val2 then res2 ] else res end ;
- 含义：如果 expr 的值为 val1 ， 取 res1。  如果  expr 的值为 val2 ，取 res2。 如果前面的条件都不成立，则取 res。

### 重点剖析代码

```java
List<Map<String,Object>> list = empMapper.countEmpJobData();
List<Object> jobList =
    list.stream().map(dataMap -> dataMap.get("pos")).toList();
List<Object> dataList =
    list.stream().map(dataMap -> dataMap.get("total")).toList();
return new JobOption(jobList, dataList);
```

#### 按行来解释（配合自己的理解）：

1. **第一行**  
   ```java
   List<Map<String,Object>> list = empMapper.countEmpJobData();
   ```
   - 这句就是调用 empMapper 的接口，去数据库查各个职位的员工人数。返回的是一个 List，这个 List 的“每一项”其实是一个 Map。
   - 为什么是 Map？因为我们用的 Mybatis 查询，每查出一行（比如“班主任 5 人”），就会用一个 Map 封装它，这个 Map 里 key 就是每个字段名（比方说“pos”和“total”），value 就是字段的值，非常灵活实用。

2. **对于 Map 的理解**  
    - 这里的 Map，一般是 Map <String, Object> 类型，也就是说，key 一定是 String，例如“pos”和“total”。
    - value 是 Object，理论上可以装任何类型，但我们这里，实际用到的通常是 String（比如“班主任”）和 Integer（比如“5”）。
    - @MapKey("pos") 主要是在结果用 Map 返回时，为 Map 的 key 指定数据库的某个字段。本文这里用的是 List <Map>，不用指定 key，直接这样用即可。

3. **直观感受：最终的 list 的内容长这样——**
   ```java
   [
       {"pos": "班主任", "total": 5},
       {"pos": "讲师",   "total": 7},
       {"pos": "学工主管", "total": 2}
   ]
   ```
   - 每一个 Map，代表一类职位及其人数。
   - “pos” 对应职位名称（String），而 “total” 对应该职位统计人数（Integer）。

4. **提取所有的职位名**  
   ```java
   List<Object> jobList =
       list.stream().map(dataMap -> dataMap.get("pos")).toList();
   ```
   - 这一行用 Java 的 stream 流，遍历整个 list，把每个 Map 里的 “pos” 拿出来（也就是所有职位的名字），最后组成一个 List。
   - 结果就是：["班主任", "讲师", "学工主管"]

5. **提取所有的人数数据**  
   ```java
   List<Object> dataList =
       list.stream().map(dataMap -> dataMap.get("total")).toList();
   ```
   - 这行做的事情和上面类似，只不过提取的是每个 Map 里的 “total” 字段（也就是人数），最后也是组成一个 List。
   - 结果就是：[5, 7, 2]

6. **最后，进行封装，方便后续传递使用**  
   ```java
   return new JobOption(jobList, dataList);
   ```
   - 用 JobOption 这个对象，把职位名列表和数据列表都打包起来，后续 Controller 返回的时候会直接返回这个对象，结构简单明了，方便前端做展示。

#### 总结一下
- **list 就像是一叠一叠的小表，每个 Map 记录了一个职位和人数，用 List 把它们串联起来。**
- **再进一步处理，把所有的职位名称和人数各自提取成 List，专门让前端展示的时候用。**
- 这样一来，数据传递和处理非常灵活，结构清晰，前后端都很友好。

## 过滤器（Filter）

- 概念：Fileter 是 Javaweb 的三大组件之一（Servlet， Filter，Listener ）
- 可以把对后端的所有请求拦截下来，然后实现一些特殊的功能
- 过滤器一般完成一些通用的操作，比如：登录校验，统一编码处理，敏感字符处理

### 快速入门

![image-20250702153712116](http://img.wiretender.top/img/20250702153712543.webp)

### 实现令牌校验过滤器

```java
/**
 * @author Wiretender
 * @version 1.0
 */
package com.itheima.filter;

import com.itheima.util.JwtUtils;
import jakarta.servlet.*;
import jakarta.servlet.annotation.WebFilter;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import lombok.extern.slf4j.Slf4j;

import java.io.IOException;

@WebFilter(urlPatterns = "/*")
@Slf4j
public class TokenFilter implements Filter {

    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {

        HttpServletRequest request = (HttpServletRequest) servletRequest;
        HttpServletResponse response = (HttpServletResponse) servletResponse;
        // 获取请求路径
        String requestURL = request.getRequestURI();
        // 判断是否是登录请求，如果路径中包含 /login， 说明是登录请求，放行
        if (requestURL.contains("/login")) {
            log.info("登录请求，放行");
            filterChain.doFilter(request, response);
            return;
        }
        // 获取请求头的token
        String token = request.getHeader("token");
        // 判断token是否存在，如果不存在，说明用户没有登录返回错误信息
        if (token == null || token.isEmpty()) {
            log.info("令牌为空，响应401");
            response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
            return ;
        }

        // 如果token存在，校验令牌，失败返回错误信息
        try {
            JwtUtils.parseJWT(token);
        } catch (Exception e) {
            log.info("令牌为空，响应401");
            response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
            return ;
        }
        // 校验通过放行
        log.info("令牌合法，放行");
        filterChain.doFilter(request, response);
        return;
    }
}

```

## 拦截器（Interceptor）

![image-20250702162521009](http://img.wiretender.top/img/20250702162521361.webp)

### 快速入门

![image-20250702162601308](http://img.wiretender.top/img/20250702162601620.webp)

```java
/**
 * @author Wiretender
 * @version 1.0
 */
package com.itheima.interceptor;

import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import lombok.extern.slf4j.Slf4j;
import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;

@Slf4j
public class DemoInterceptor implements HandlerInterceptor {
    // 在目标资源运行之前运行，放行返回True 不放行返回false
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        log.info("preHandle...");
        return true;
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        HandlerInterceptor.super.postHandle(request, response, handler, modelAndView);
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        HandlerInterceptor.super.afterCompletion(request, response, handler, ex);
    }
}

```

```java
/**
 * @author Wiretender
 * @version 1.0
 */
package com.itheima.config;

import com.itheima.interceptor.DemoInterceptor;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.InterceptorRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

@Configuration
public class WebConfig implements WebMvcConfigurer {


    @Autowired
    private DemoInterceptor demoInterceptor;

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(demoInterceptor).addPathPatterns("/**"); // 拦截所有请求
    }
}

```

### 拦截路径

![image-20250702170222707](http://img.wiretender.top/img/20250702170223019.webp)

## AOP 切面编程

- 连接点：JoinPoint， 可以被 AOP 控制的方法
- 通知：Advice，指那些重复的逻辑
- 切入点：PointCut，切入点，匹配连接点的条件，通知仅会在切入点方法执行时被应用
- 切面：Aspect，描述通知与切入点的对应关系
- 目标对象：Target， 通知所应用的对象

![image-20250702203423180](http://img.wiretender.top/img/20250702203423814.webp)

### 通知类型：

![image-20250702204049593](http://img.wiretender.top/img/20250702204050029.webp)

### 通知顺序

![image-20250702205550221](http://img.wiretender.top/img/20250702205550700.webp)

### 切入点表达式

![image-20250702205811612](http://img.wiretender.top/img/20250702205812011.webp)

#### execution

```java
execution(访问修饰符？ 返回值 包名.类名.?方法名(方法参数) throw 异常？)
```

- 其中？代表可以省略的部分

![image-20250702210006780](http://img.wiretender.top/img/20250702210007157.webp)

### annotation注解来进行匹配

```java
package com.itheima.anno;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Log {
}

```

- 定义一个注解来进行标记作用，这里只要定义两个元注解即可

- 然后应用到相应的接口上，我们实现的一个切面类如下

```java
package com.itheima.aop;

import com.itheima.mapper.OperateLogMapper;
import com.itheima.pojo.OperateLog;
import lombok.extern.slf4j.Slf4j;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Pointcut;
import org.aspectj.lang.reflect.MethodSignature;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

import java.lang.reflect.Method;
import java.time.LocalDateTime;
import java.util.Arrays;
import java.util.stream.Collectors;

@Aspect
@Component
@Slf4j
public class OperateLogAspect {

    // 注入 Mapper
    @Autowired
    private OperateLogMapper operateLogMapper;

    // 定义切点：controller包下的所有方法 或 使用了@LogOperation注解的方法
    @Pointcut("@annotation(com.itheima.anno.Log)")
    public void logPointCut() {}

    // 环绕通知
    @Around("logPointCut()")
    public Object doAround(ProceedingJoinPoint joinPoint) throws Throwable {

        long startTime = System.currentTimeMillis();

        // 获取方法签名
        MethodSignature signature = (MethodSignature) joinPoint.getSignature();
        Method method = signature.getMethod();

        // 获取目标类和方法名
        String className = joinPoint.getTarget().getClass().getName();
        String methodName = method.getName();

        // 获取参数名和参数值
        Object[] args = joinPoint.getArgs();
        String params = Arrays.stream(args)
                .map(Object::toString)
                .collect(Collectors.joining(", "));

        Object result = null;
        try {
            result = joinPoint.proceed(); // 执行原方法
        } catch (Exception e) {
            throw e;
        } finally {
            long costTime = System.currentTimeMillis() - startTime;

            // 构建操作日志对象
            OperateLog operateLog = new OperateLog();
            operateLog.setOperateEmpId(getCurrentUserId()); // 从 ThreadLocal 或 SecurityContext 获取当前用户ID
            operateLog.setOperateTime(LocalDateTime.now());
            operateLog.setClassName(className);
            operateLog.setMethodName(methodName);
            operateLog.setMethodParams(params);
            operateLog.setReturnValue(result != null ? result.toString() : "null");
            operateLog.setCostTime(costTime);

            // 插入日志
            log.info("记录操作日志：{}", operateLog);
            try {
                operateLogMapper.insert(operateLog);
            } catch (Exception ex) {
                log.error("插入操作日志失败", ex);
            }
        }

        return result;
    }

    // 示例方法：获取当前登录用户的ID（根据你的项目实际情况修改）
    private Integer getCurrentUserId() {
        // 假设你是通过 ThreadLocal 存储的当前登录用户
        // 或者 Spring Security 获取
        // UserDetails userDetails = (UserDetails) SecurityContextHolder.getContext().getAuthentication().getPrincipal();
        // return userDetails.getId();

        // 示例默认值
        return 1; // TODO: 替换为真实用户ID
    }
}
```

### 完善获取ID方法

#### ThreadLocal

![image-20250702231451577](http://img.wiretender.top/img/20250702231452145.webp)

![image-20250702233609833](http://img.wiretender.top/img/20250702233610358.webp)

## 总结

![image-20250703231117796](http://img.wiretender.top/img/20250703231118258.webp)

![image-20250703231201855](http://img.wiretender.top/img/20250703231202262.webp)
