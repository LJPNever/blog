---
title: Springboot 整合redis
date: 2018-11-01 21:19:58
tags:
---

1. ### Springboot 整合redis

   <!--more-->

   使用springboot的  Cache 进行缓存管理

   项目地址：https://github.com/LJPNever/RedisTest.git

   首先要导入pom.xml依赖

   ```xml
   <dependency>
   			<groupId>org.springframework.boot</groupId>
   			<artifactId>spring-boot-starter-data-redis</artifactId>
   			<version>1.5.0.RELEASE</version>
   		</dependency>
   ```

   以及使用JedisConnectionFactory建立连接工厂，所以导入对应的依赖

   ```xml
   	<dependency>
   			<groupId>com.fasterxml.jackson.core</groupId>
   			<artifactId>jackson-databind</artifactId>
   			<version>2.6.7</version>
   		</dependency>
   ```

   #### redis 的配置

   ```yml
   spring:
       redis:
         database: 0
         host: 127.0.0.1
         //密码默认为空，故没有添加
         port: 6379
         jedis:
           pool:
             max-active: 8
             max-wait: -1
             max-idle: 8
             min-idle: 0
   ```

   #### RedisCacheConfig的配置

   ```java
   @EnableCaching
   @Configuration
   @EnableAutoConfiguration
   public class RedisCacheConfig extends CachingConfigurerSupport {

       @Value("${spring.redis.host}")
       private String host;

       @Value("${spring.redis.port}")
       private int port;
   ```


       @Value("${spring.redis.database}")
       private int database;


       /**
        * 连接redis的工厂类
        * @return
        */
       @Bean
       public JedisConnectionFactory jedisConnectionFactory() {
           JedisConnectionFactory factory = new JedisConnectionFactory();
           factory.setHostName(host);
           factory.setPort(port);
    
           factory.setDatabase(database);
           return factory;
       }
    
       /**
        * 配置RedisTemplate
        * 设置添加序列化器
        * key 使用string序列化器
        * value 使用Json序列化器
        * 还有一种简答的设置方式，改变defaultSerializer对象的实现。
        * @return
        */
       @Bean
       public RedisTemplate<String, Object> redisTemplate() {
           //StringRedisTemplate的构造方法中默认设置了stringSerializer
           RedisTemplate<String, Object> template = new RedisTemplate<>();
           //set key serializer
           StringRedisSerializer stringRedisSerializer = new StringRedisSerializer();
           template.setKeySerializer(stringRedisSerializer);
           template.setHashKeySerializer(stringRedisSerializer);
    
           Jackson2JsonRedisSerializer jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer(Object.class);
           ObjectMapper objectMapper = new ObjectMapper();
           objectMapper.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
           objectMapper.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
    
           jackson2JsonRedisSerializer.setObjectMapper(objectMapper);
           //set value serializer
           template.setDefaultSerializer(jackson2JsonRedisSerializer);
    
           template.setConnectionFactory(jedisConnectionFactory());
           template.afterPropertiesSet();
           return template;
       }
   }
   ```

   使用@EnableCaching，则为开启springboot 提供的缓存机制，支持缓存注解

   #### 关于cache 与redis的结合

   **主要的缓存注解**  ：

   @Cacheable 触发缓存入口

   @CacheEvict 触发移除缓存

   @CacahePut 更新缓存

   @Caching 将多种缓存操作分组

   @CacheConfig 类级别的缓存注解，允许共享缓存名称

    

   1. @Cacheable 触发缓存入口: 当调用改注解的方法时，会在缓存中找寻对应的Key值，如果存在，则直接返回对应value,如果不存在，则访问DB,并把结果存入打key值对应的value中
   2. @CacheEvict 触发移除缓存：移除对应key值得全部内容或者部分内容，一般可在删除信息时对对应的缓存进行操作
   3. @CacahePut 更新缓存：把更新的信息更新到对应的缓存中，一般用于更新数据
   4. @CacheConfig：将缓存分类，它是类级别的注解方式，注解在类上，则类中所有方法的对应缓存都为在类别下
   5. @Caching这个注解将其他注解方式融合在一起了，我们可以根据需求来自定义注解，并将前面三个注解应用在一起

   #### 具体操作：

   下面为两个简单的例子，不局限为以下操作

   ```java
    @ResponseBody
       @RequestMapping("/get")
       @Cacheable(value="users", key="1")
       public List<User> get(){
           Map map=new HashMap();

           return userService.getAll();
       }
   //当每次调用该方法是，会先查询数据库，如果不存在在去查询DB，并把结果缓存
   ```

   ```java
   @ResponseBody
       @RequestMapping("/add")
       @CacheEvict(value="users", key="1")
       public Map add(@RequestBody User user){
           Map map=new HashMap();
           map.put("data",userService.add(user));
           return map;
       }
   //简单的更新操作，并把前面的缓存删除，避免缓存获取的数据缺失
   ```

   ​