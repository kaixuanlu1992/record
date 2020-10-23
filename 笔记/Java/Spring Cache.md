### Spring Cache

#### 概述

**定位**

Spring Cache是一种缓存接口抽象，而不是具体的实现，需要使用实际的存储来存储缓存数据

**特点**

- 缓存应用于Java方法，从而根据缓存中可用的信息减少执行次数
- Spring提供该抽象的一些实现：`java.util.concurrent.ConcurrentMap`基于JDK的缓存，[Ehcache 2.x]，Gemfire缓存，[Caffeine]和符合JSR-107的缓存（例如Ehcache 3.x）
- 对于多线程和多进程环境，缓存抽象没有特殊处理，因为此类功能由缓存实现处理

#### 使用

- `@Cacheable`：缓存填充
- `@CacheEvict`：缓存删除
- `@CachePut`：更新缓存，而不会干扰方法的执行
- `@Caching`：组合要应用于一个方法的多个缓存操作
- `@CacheConfig`：在类级别共享一些与缓存相关的常见设置

**`@Cacheable`**

```
@Cacheable("books")
public Book findBook(ISBN isbn) {...}
```

- 指定缓存名称为cache,可以指定多个缓存

- 缓存本质上是键值存储，默认键值生成方法

  - 没有给出参数，返回`SimpleKey.EMPTY`
  - 仅给出一个参数，返回该实例
  - 给定多个参数，返回包含所有参数的`SimpleKey`

- 自定义key生成声明：

  - 通过`key`属性生成键值。可以使用[SpEL](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#expressions)选取感兴趣的参数（或其嵌套属性），执行操作，甚至调用任意方法，而无需编写任何代码或实现任何接口

  - 也可以自定义keyGenerator，并通过keyGenerator属性指定

    ```
    @Cacheable(cacheNames="books", key="#isbn")
    @Cacheable(cacheNames="books", key="#isbn.rawNumber")
    @Cacheable(cacheNames="books", key="T(someType).hash(#isbn)")
    ```

- 在多线程环境中，可以使用`sync`属性来指示基础缓存提供程序在计算值时锁定缓存条目
- 条件缓存：
  - 通过`condition`参数支持，使用`SpEL`表达式，表达式的值等于`true` 或`false`。为`true`，则将缓存该方法。不是，它的行为就像未缓存的方法、
  - 可以使用`unless`属性将某些值排除到缓存之外

**`@CachePut`**

作用：**在不影响方法执行**的情况下更新缓存，调用该方法，将其结果放入高速缓存中，拥有与@Cacheable一样的属性

**`@CacheEvict`**

- 用于删除缓存条目

- `void`方法可以与`@CacheEvict`-一起使用
- `allEntries`：为true删除缓存所有条目，此时会忽略key属性
- `beforeInvocation`：指定缓存删除执行的时间，在方法执行后（false）清楚缓存，此时方法异常或未运行，缓存不会被删除，在执行前（true）清楚缓存

**`@Caching`**

用于在同一个方法上嵌套多个@Cacheable，@CachePut`和`@CacheEvict注解，往往针对同一方法操作不同缓存

```
@Caching(evict = { @CacheEvict("primary"), @CacheEvict(cacheNames="secondary", key="#p0") })
```

**`@CacheConfig`**

类级别注解

指明类级别的公共配置，如缓存名称、KeyGenerator、CacheResolver、CacheManager

方法级别的自定义始终会覆盖上设置的自定义`@CacheConfig`

#### 配置

**启用缓存注释**

```
@EnableCaching
<cache:annotation-driven/>
```

**使用自定义注释**

使用`@Cacheable`，`@CachePut`，`@CacheEvict`，和`@CacheConfig`为 [元注释]

**配置缓存存储**

```
<bean id="cacheManager" class="org.springframework.cache.support.SimpleCacheManager">
    <property name="caches">
        <set>
            <bean class="org.springframework.cache.concurrent.ConcurrentMapCacheFactoryBean" p:name="default"/>
            <bean class="org.springframework.cache.concurrent.ConcurrentMapCacheFactoryBean" p:name="books"/>
        </set>
    </property>
</bean>
```



```
<bean id="cacheManager"
        class="org.springframework.cache.ehcache.EhCacheCacheManager" p:cache-manager-ref="ehcache"/>

<!-- EhCache library setup -->
<bean id="ehcache"
        class="org.springframework.cache.ehcache.EhCacheManagerFactoryBean" p:config-location="ehcache.xml"/>
```

