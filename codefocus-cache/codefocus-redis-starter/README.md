# codefocus-redis-starter 基于YML动态配置Redis

## 环境依赖：
    springboot 2.1.3.RELEASE 版本
    
### (step 1)使用教程：
```xml

<dependency>
    <groupId>club.codefocus.framework</groupId>
    <artifactId>codefocus-redis-starter</artifactId>
    <version>1.0-SNAPSHOT</version>
 </dependency>

```

### (step 2)YML配置详解

```yaml

spring:
  redis:
    port: 6379
    database: 0
    password:
    host: 127.0.0.1
    lettuce:
      pool:
        max-active: 8 #连接池最大连接数(使用负值表示没有限制) 默认为8
        max-idle: 8 # 连接池中的最大空闲连接 默认为8
        min-idle: 0 #连接池中的最小空闲连接 默认为 0
    timeout: 2000
    code-focus:
      global-limit-count: 200  #次数
      global-limit-period-time: 1 #毫秒  单位时间内的次数
      global-limit-open: true #是否开启服务限流 

```


##使用教程：

### (step 3)Bean实例详解：

    @Resource
    RedisLockHandler redisLockHandler;
    
        /**
         * 获取锁
         * @param key  锁的key
         * @param time 锁的过期时间
         * @return
         */
        boolean lock(String key, int time);
        /**
         * 持续获取锁
         * @param key  锁的key
         * @param time 锁的过期时间
         * @return
         */
        boolean getLockWhile(String key, int time) #不推荐使用
        
        #以上都会自动释放锁,也可以通过下面方式手动释放
        
        /**
         * 释放锁
         * @param key
         */
        public void unlock(String key)
    
    

    @Resource
    RedisStringHandler redisStringHandler;
    
        put(K key, V entity) ;
        
        put(K key, V entity, long timeout);#单位分钟
        
        put(K key, V entity, long timeout, TimeUnit unit);自定义
        
        V find(K key);#获取或者查找
        
        remove(K key);
        
        void increment(K key, long timeout, TimeUnit unit);
        
  
### (step 4)注解详解：

    /**
     * @Auther: jackl
     * @Date: 2019/10/25 10:26
     * @Description:分布式锁开关
     */
    @Target({ElementType.METHOD})
    @Retention(RetentionPolicy.RUNTIME)
    @Documented
    public @interface DistributedLock {
        /**
         * 分布式锁开关:true=开启;false=关闭
         */
        boolean open() default true;
    
        LockType lock() default LockType.IP;
    
        String field() default "";
    
        int expire() default 5000;
    
        int timeOut() default 3000;
    }
    
    使用列子：
        @DistributedLock(lock = LockType.UNIQUEID,field="a")
    
    
    /**
     * @Title: RequestLimit
     * @Description: 限制方法调用次数（可以注解在方法或类）
     * @ProjectName:
     * @author: jackl
     * @date: 2019/10/25 10:26
     * @version: V1.0.0
     */
    @Target({ElementType.METHOD, ElementType.TYPE})
    @Retention(RetentionPolicy.RUNTIME)
    @Documented
    public @interface RequestLimit {
    
        /**
         * 限制上限
         */
        int limit() default 5;
    
        /**
         * 单位时间内
         */
        int period() default 1;
    
        /**
         * 时间单位
         */
        TimeUnit unit() default TimeUnit.SECONDS;
    
        /**
         * 用户id
         * @return
         */
        String userId() default "";
    
        /**
         * 判断用户id具体的位置
         * @return
         */
        RequestLimitType userIdRequestLimitType() default RequestLimitType.REQUEST;
    
        /**
         * 客户唯一id
         * @return
         */
        String clientId() default "";
    
        /**
         * 客户唯一ID的类型
         * @return
         */
        RequestLimitType clientIdRequestLimitType() default RequestLimitType.REQUEST;
    
    }
    
    
    使用列子：
        @RequestLimit(limit = 1,period =1 ,unit = TimeUnit.SECONDS)
        
    
##Spring Cacheable 添加过期时间
        
    @Cacheable(value = "UserInfoList#30" ,key = "targetClass + methodName +#p0")
            UserInfoList:
            30:单位秒

