---
title: Java学习 - Spring 依赖注入
pubDatetime: 2026-02-27T13:20:40Z
description: JavaWeb 学习笔记
featured: true
draft: false
tags:
  - Java
---

# Spring dependency Injection

## 案例

- `@Component`： 组件注解
- `@Service`：服务注解
- `@Repository`：仓储注解，提供对持久化类数据的操作的服务
- `@Controller/@RestController()`： 对外提供服务的注解

### 构造注入 & List、Map

```java
private final List<IAwardService> awardService;
private final Map<String, IAwardService> awardServiceMap;

public AwardController(List<IAwardService> awardServices, Map<String, IAwardService> awardServiceMap) {
    this.awardServices = awardServices;
    this.awardServiceMap = awardServiceMap;
}

public Response<String> distributeAward(@RequestParam String userId, @RequestParam String awardKey) {
    try {
        log.info("发放奖品服务 userId:{} awardKey:{}", userId, awardKey);
        awardServiceMap.get(awardKey);
        return Response.<String>builder()
                .code("0000")
                .info("调用成功")
                .data("发奖完成")
                .build();
    } catch (Exception e) {
        return Response.<String>builder()
                .code("0001")
                .info("调用失败")
                .build();
    }
}
```

- 场景：IAwardService 接口有多个实现类，可以通过@Resource、@Autowired 注解注入，也可以通过构造函数注入。在Spring官方文档中，官方推荐使用构造函数进行注入 `The Spring team generally advocates constructor injection, as it lets you implement application components as immutable objects and ensures that required dependencies are not null.`

- 用途：Map 注入是一个非常好的注入手段，我们可以把每个 IAwardService 实现类设定好 Bean 的名称为数据库中的奖品 awardKey。在发奖的时候，可以直接根据 awardKey 从 Map 中获取到对应的 Bean 对象，这样也就省去了 `if···else` 大量的判断操作。

### 空注入判断

```java
public class NullAwardService implements IAwardService {

    @Override
    public void doDistributeAward(String userId) {

    }

}

@Autowired(required = false)
private NullAwardService nullAwardService;
```

- 场景：NullAwardService 没有配置相关的 @Service 注册，或者在程序中手动实例化的这个Bean对象，根据不同诉求，在没有创建的时候，可以使用 `@Autowired(require = false)`

进行注入，这样可以避免空指针异常

- 用途：当我们使用支付、openai外部接口的测试阶段，需要关闭服务，不实例化这个对象，这时候就可以使用空注入来进行测试操作。

### 优先实例化

```java
@Slf4j
@Service("openai_model")
@Primary
@Order(1)
public class OpenAIModelAwardService implements IAwardService {

    @Override
   	public void doDistributeAward(String userId) {
        log.info("发奖服务")
    }
}


@Resource
private IAwardService awardService;
```

- 场景：当一个接口有多个实现类的时候，如果使用 @Resource 注入的时候是会报错 NoUniqueBeanDefinitionException 异常，这个时候使用 @Primary 就会标记为首选对象，注入的时候会注入这个对象，这里的 @Order（1）是对象加载的顺序。
- 用途：当我们为一组接口提供实现类，并需要提供默认的注入的时候，就可以使用 @Primary 注解来限定首选注入项

### 检测创建，避免重复

```java
@Bean("redisson01")
// 当 Spring 应用上下文中不存在某个特定类型的 Bean 时，才会创建和配置标注了 @ConditionalOnMissingBean 的 Bean 对象
@ConditionalOnMissingBean
public String redisson01() {
    return "模拟的 Redis 客户端 01";
}
@Bean("redisson02")
// 当 Spring 应用上下文中不存在某个特定类型的 Bean 时，才会创建和配置标注了 @ConditionalOnMissingBean 的 Bean 对象
@ConditionalOnMissingBean
public String redisson02() {
    return "模拟的 Redis 客户端 02";
}
```

- **场景**：`@Bean` 可以用于在方法，创建出对象。这有点类似于使用 Spring 的 FactoryBean 接口创建对象一样，这里可以直接使用方法创建。之后 `@ConditionalOnMissingBean` 注解的目的是为了避免重复创建，判断应用上下文中存在这个对象，则不会重复创建。
- **用途**：通常我们在做一些组件的时候，会加入这样一个注解，避免在业务工程中引入同类的组件的时候，会导致创建出相同对象而报错。

### 配置是否创建对象

```java
@Data
@ConfigurationProperties(prefix = "sdk.config", ignoreInvalidFields = true)
public class AutoConfigProperties {

    /** 状态：open = 开启、close 关闭 */
    private boolean enable;
    /** 转发地址 */
    private String apiHost;
    /** 可以申请 sk-*** */
    private String apiSecretKey;

}

@Bean
@ConditionalOnProperty(value = "sdk.config.enabled", havingValue = "true", matchIfMissing = false)
public String createTopic(@Qualifier("redisson01") String redisson, AutoConfigProperties properties) {
    log.info("redisson {} {} {}", redisson, properties.getApiHost(), properties.getApiSecretKey());
    return redisson;
}

sdk:
  config:
    enabled: false
    apiHost: https://open.bigmodel.cn/
    apiSecretKey: d570f7c5d289cdac2abdfdc562e39f3f.trqz1dH8ZK6ED7Pg
```

- **场景**：模拟创建 createTopic，入参的对象为注入的操作，@Qualifier 注解可以指定要注入哪个名字的对象。之后 `@ConditionalOnProperty` 注解可以通过配置的 enabled 值，来确定是否实例化对象。
- **用途**：这个场景是非常使用的，比如你做了一个组件，或者业务中要增加一些配置。启动或关闭某些服务，就可以使用了。而不需要把 pom 中引入的组件注释掉。

- @ConditionalOnProperty **条件装配注解** ： 只有满足条件的时候，才创建这个Bean
  - value = “sdk.config.enabled” 读取哪里的配置
  - havingValue = "true" 只有值为 true 时，才生效
  - matchIfMissing = false 如果配置不存在，就不创建这个Bean
- @Qualifier 这个是检测当前容器必须有当前的Bean对象，否则会报错
- AutoConfigProperties properties 这是通过注解 `@ConfigurationProperties` 绑定的配置类

> `@ConfigurationProperties` 用于将配置文件中的属性批量绑定到 Java 对象。
>
> `@ConditionalOnProperty` 是 Spring Boot 自动装配中的条件注解，用于根据配置决定是否加载某个 Bean。
>
> 常用于实现开关控制、Starter 自动配置。

### 自定义Condition，判断是否实例化对象

```java
public class BeanCreateCondition implements Condition {

    @Override
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        String active = System.getProperty("isOpenWhitelistedUsers");
        return null != active && active.equals("true");
    }

}

@Bean
@Conditional(BeanCreateCondition.class)
public List<String> whitelistedUsers() {
    return new ArrayList<String>() {{
        add("user001");
        add("user002");
        add("user003");
    }};
}

static {
    // BeanCreateCondition 会检测这个值，确定是否创建对象
    System.setProperty("isOpenWhitelistedUsers", "false");
}

@Autowired(required = false)
@Qualifier("whitelistedUsers")
private List<String> whitelistedUsers;
```

- **场景**：是一个案例中使用到了 `@ConditionalOnProperty` 注解，我们也可以自定义一个 Conditional 的实现类，之后把这个实现类配置到需要实例化的对象上面，通过 matches 匹配条件方法的实现，决定是否实例化。
- **用途**：这个场景的用途和 `@ConditionalOnProperty` 是一样的，只不过我们可以更好的自定义控制。

### 根据环境配置实例化对象

```java
@Slf4j
@Service
// 用于根据配置环境实例化 Bean 对象
@Profile({"prod", "test"})
@Lazy
public class AliPayAwardService implements IAwardService {

    public AliPayAwardService() {
        log.info("如一些支付场景，必须指定上线后才能实例化");
    }

    @Override
    public void doDistributeAward(String userId) {
        log.info("红包奖励 {}", userId);
    }

}

spring:
  config:
    name: xfg-dev-tech-spring-dependency-injection
  profiles:
    active: dev
```

- **场景**：`@Profile({"prod", "test"})` 注解可以配置你是在什么时候实例化这个对象，我们可以指定 application.yml 中配置的 `active: dev/prod/test` 来确定是在开发、测试还是上线才实例化这个对象。
- **用途**：一些只有到线上才能实例化对象的时候，就可以配置 `@Profile({"prod", "test"})` 注解，注意这个需要配合 `@Autowired(required = false)` 进行注入，否则会出现注入为空指针的异常

### 引入 Spring 配置

```java
@Slf4j
@SpringBootApplication
@Configurable
@PropertySource("classpath:properties/application.properties")
@ImportResource("classpath:spring/spring.xml")
@EnableScheduling
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class);
    }

}

<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                           http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="exampleBean" class="cn.bugstack.xfg.dev.tech.domain.SpringBeanTest"/>

</beans>

@Slf4j
public class SpringBeanTest {

    public SpringBeanTest() {
        log.info("我是通过 Spring 配置文件实例化的 Bean 对象");
    }

}
```

- **场景**：在 SpringBoot 工程中，可以通过 `@ImportResource`、`@PropertySource` 引入对应的配置文件，完成对象的初始化。
- **用途**：在实际的开发中，虽然使用 SpringBoot 工程，但为了兼容一些老的项目或者一些还没有升级到 SpringBoot Starter 的组件，则需要单独引入 Spring 配置文件来创建对象

Spring Boot 启动时：

```
AnnotationConfigApplicationContext
        ↓
加载配置类
        ↓
解析 @ImportResource
        ↓
XmlBeanDefinitionReader
        ↓
注册 BeanDefinition
        ↓
实例化 Bean
```

### 原型对象

```java
@Component
@Scope("prototype")
public class LogicChain {
}

@Resource
private ApplicationContext applicationContext;

@Test
public void test_prototype() {
    log.info("测试结果: {}", applicationContext.getBean(LogicChain.class).hashCode());
    log.info("测试结果: {}", applicationContext.getBean(LogicChain.class).hashCode());
}

        @小傅哥: 代码已经复制到剪贴板

```

- **场景**：`@Scope("prototype")` 可以设定对象类型为原型对象，每次获得的对象都是一个新的实例化对象。
- **用途**：对于动态，不同责任链创建，可以使用这个配置，确保每个对象都是自己的。

### 其他注解

- `@EnableScheduling`：允许启动任务的注解，放到 Application 上，确保任务启动执行。
- `@DependsOn({"openai_model", "openai_use_count", "user_credit_random"})` Bean 对象实例化中，依赖于哪些对象。
- `@Autowired private Environment env;` 环境配置注入，可以获取到 application.yml 中的配置数据 `env.getProperty("app.name"), env.getProperty("app.version")`
- `@Async` 异步方法注解，可以用于调用某个方法后，让下面的具体逻辑方法为异步执行，主方法直接返回结果。可以用于一些申请导出数据到文件的场景。

## 源码分析

在 Spring 框架中，依赖注入（DI）是通过一系列的步骤和组件来实现的。对于构造函数注入，特别是注入 `Map` 类型的依赖，Spring 需要处理以下几个关键步骤：

1. **Bean Definition 解析**：Spring 解析配置文件或注解，生成 BeanDefinition 对象。
2. **Bean 实例化**：Spring 根据 BeanDefinition 创建 Bean 实例。
3. **依赖注入**：Spring 将所需的依赖注入到 Bean 中。

### `AutowiredAnnotationBeanPostProcessor`

```java
public class AutowiredAnnotationBeanPostProcessor implements BeanPostProcessor {
    // 省略其他代码

    @Override
    public PropertyValues postProcessProperties(PropertyValues pvs, Object bean, String beanName) {
        // 查找注入的元数据，这是一个注入元数据的描述对象， 需要保存 需要注入的字段、需要调用的注入方法
        InjectionMetadata metadata = findAutowiringMetadata(beanName, bean.getClass(), pvs);
        try {
            // 执行注入 ： 遍历所有需要注入的字段，从容器中查找依赖Bean，通过反射设置字段值
            metadata.inject(bean, beanName, pvs);
        } catch (Throwable ex) {
            throw new BeanCreationException(beanName, "Injection of autowired dependencies failed", ex);
        }
        return pvs;
    }
}
```

`AutowiredAnnotationBeanPostProcessor` 是处理依赖注入的核心类之一。它会扫描 Bean 的构造函数、字段和方法上的 `@Autowired` 注解，并进行相应的依赖注入。

这个类是一个 Bean 后置处理器 （BeanPostProcessor）: 在Bean的生命周期过程中，对Bean进行增强处理，专门处理 @Autowired @Value @Inject

**执行时机：**

Bean生命周期的执行流程

```
1. 实例化（构造方法）
2. 属性填充
3. BeanPostProcessor.postProcessBeforeInitialization
4. 初始化
5. BeanPostProcessor.postProcessAfterInitialization
```

而 @Autowired 注入发生在属性填充的阶段对应的方法是 postProcessProperties();

### `ConstructorResolver`

```java
public class ConstructorResolver {
    // 省略其他代码

    public BeanWrapper autowireConstructor(
            final String beanName, final RootBeanDefinition mbd, Constructor<?>[] chosenCtors, final Object[] explicitArgs) {

        // 省略其他代码

        Constructor<?> constructorToUse = null;
        ArgumentsHolder argsHolderToUse = null;
        Object[] argsToUse = null;

        // 省略其他代码

        for (Constructor<?> candidate : candidates) {
            Class<?>[] paramTypes = candidate.getParameterTypes();
            if (argsToUse == null) {
                // 省略其他代码
                argsHolder = createArgumentArray(
                        beanName, mbd, resolvedValues, bw, paramTypes, paramNames, getUserDeclaredConstructor(candidate), autowiring);
            }
            // 省略其他代码
        }

        // 省略其他代码

        BeanWrapperImpl bw = new BeanWrapperImpl(beanInstance);
        initBeanWrapper(bw);

        return bw;
    }
}
```

`ConstructorResolver` 是负责解析和调用构造函数的类。它会根据 BeanDefinition 和构造函数的参数类型，选择合适的构造函数并进行实例化。

### `DefaultListableBeanFactory`

```java
public class DefaultListableBeanFactory extends AbstractAutowireCapableBeanFactory
        implements ConfigurableListableBeanFactory, BeanDefinitionRegistry {

    // 省略其他代码

    @Override
    protected Map<String, Object> findAutowireCandidates(String beanName, Class<?> requiredType, DependencyDescriptor descriptor) {
        String[] candidateNames = BeanFactoryUtils.beanNamesForTypeIncludingAncestors(this, requiredType);
        Map<String, Object> result = new LinkedHashMap<>(candidateNames.length);
        for (String candidate : candidateNames) {
            if (!isSelfReference(beanName, candidate) && isAutowireCandidate(candidate, descriptor)) {
                result.put(candidate, getBean(candidate));
            }
        }
        if (result.isEmpty() && !indicatesMultipleBeans(requiredType)) {
            DependencyDescriptor fallbackDescriptor = descriptor.forFallbackMatch();
            for (String candidate : candidateNames) {
                if (!isSelfReference(beanName, candidate) && isAutowireCandidate(candidate, fallbackDescriptor)) {
                    result.put(candidate, getBean(candidate));
                }
            }
        }
        return result;
    }
}
```

`DefaultListableBeanFactory` 是 Spring 中最常用的 BeanFactory 实现类。它负责管理 Bean 的定义和生命周期，并提供依赖查找和注入的功能。

### 具体流程

1. **解析 BeanDefinition**： Spring 解析配置文件或注解，生成 `AwardController` 的 `BeanDefinition` 对象。
2. **选择构造函数**： `AutowiredAnnotationBeanPostProcessor` 会扫描 `AwardController` 的构造函数，发现它有一个 `Map<String, IAwardService>` 类型的参数。
3. **查找依赖**： `ConstructorResolver` 会根据构造函数参数的类型，查找 Spring 容器中所有 `IAwardService` 类型的 Bean，并将它们放入一个 `Map` 中。这个 `Map` 的键是 Bean 的名称，值是对应的 `IAwardService` 实例。
4. **实例化 Bean**： `ConstructorResolver` 使用找到的依赖，调用 `AwardController` 的构造函数，创建 `AwardController` 实例。
5. **注入依赖**： `DefaultListableBeanFactory` 将创建好的 `Map<String, IAwardService>` 注入到 `AwardController` 的构造函数中。
