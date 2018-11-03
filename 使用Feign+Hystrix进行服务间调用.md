# 使用Feign+Hystrix进行服务间调用

## 0x01 Feign介绍
Feign是微服务体系中的消费端的http客户端，一般来与Hystrix熔断器、okHttp3结合使用。
Hystrix：
    1. Hystrix使用命令模式HystrixCommand(Command)包装依赖调用逻辑，每个命令在单独线程中/信号授权下执行。
    2. 可配置依赖调用超时时间,超时时间一般设为比99.5%平均时间略高即可.当调用超时时，直接返回或执行fallback逻辑。
    3. 为每个依赖提供一个小的线程池（或信号），如果线程池已满调用将被立即拒绝，默认不采用排队.加速失败判定时间。
    4. 依赖调用结果分:成功，失败（抛出异常），超时，线程拒绝，短路。 请求失败(异常，拒绝，超时，短路)时执行fallback(降级)逻辑。
    5. 提供熔断器组件,可以自动运行或手动调用,停止当前依赖一段时间(10秒)，熔断器默认错误率阈值为50%,超过将自动运行。
    6. 提供近实时依赖的统计和监控
在Spring Cloud的feign已经集成了Hystrix，

## 0x01 如何结合Hystrix

1. 导入依赖，注意如果是在springboot2.0以下的版本，需要指定version版本号。在feign已经依赖了hystrix了
```
<dependency>
    <groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-openfeign</artifactId>
	<version>1.4.6.RELEASE</version>
</dependency>
```

2. 在Application.java启动类增加注解@EnableFeignClients（另外，@EnableHystrixDashboard是开启Hystrix监控，后续补充）。
```java
@EnableMethodCache(basePackages = "hystrix.demo.api")
@EnableCreateCacheAnnotation
@SpringBootApplication
@EnableFeignClients(basePackages = "hystrix.demo.api")
@EnableHystrixDashboard
public class DemoApplication { }
```

3. 接下来，我们开始编写调用远程调用接口。这里我们选择的调用Forward服务。解释一下：在注释@FeignClient里的几个参数，*name*给客户端创建名字，当开启hystrix的enabled = true的情况下，会使用*forward-service*创建线程池。*url*是请求的上下文路径。*fallbackFactory*是hystrix提供的，当**请求是[200,300)、或者404、或者返回的数据反序列化失败时**时，会调用*HystrixClientFallbackFactory*里相应的fallback方法。此外，在FeignClient注解里还有一个与fallbackFactory相对应的属性*fallback*,这两者区别是*fallbackFactory*可以获取throwable，打印日志或者做一些异常处理逻辑。

代码如下：
```java
@FeignClient(name = "forward-service", url = "${forward-service.url}",
        fallbackFactory = ForwardApi.HystrixClientFallbackFactory.class)
public interface ForwardApi {

    /**
     * 调用Forward服务获取人员信息
     * @param empNo
     * @param auth
     * @param employeeShortId
     * @return
     */
    @GetMapping("user/queryUserCard?curEmployeeShortId=null&employeeShortId={employeeShortId}")
    ServiceData<List<UserDto>> fetchUser(@RequestHeader(value = "X-Emp-No") String empNo,
                                           @RequestHeader(value = "X-Auth-Value") String auth,
                                           @PathVariable("employeeShortId") String employeeShortId);

    @Component
    class HystrixClientFallbackFactory implements FallbackFactory<ForwardApi> {
        private static final Logger logger = LoggerFactory.getLogger(ForwardApi.class);
        @Override
        public ForwardApi create(Throwable throwable) {
            return new ForwardApi() {
                @Override
                public ServiceData<List<UserDto>> fetchUser(String empNo, String auth, String employeeShortId) {
                    logger.error("HystrixClient fallback's reason", throwable);
                    return ServiceData.errorReturn("0005","调用forward-service人员接口, 获取人员{"+empNo+"}失败: "+throwable.getLocalizedMessage());
                }
            };
        }
    }

//    @Component
//    class HystrixClientFallback implements ForwardApi {
//        @Override
//        public ServiceData fetchUser(String empNo, String auth, String employeeShortId) {
//            return ServiceData.errorReturn("0001","调用人员接口，获取人员{"+empNo+"}失败");
//        }
//    }
}
```

4. 当完成步骤3时，我们已经看通过@Autowired 注入ForwardApi 来调用方法了。这里，我们的接口返回的数据因为业务性可以在客户端，使用缓存来提升性能。这里采用阿里的JetCache来封装。
```java
@Service
public class ForwardService {

    @CreateCache(name = "forward:userDao:", cacheType = CacheType.LOCAL, expire = 7, timeUnit = TimeUnit.DAYS)
    private Cache<String, UserDto> userDtoCache;   

    public UserDto getUser(String empNo, String auth) {
        empNo = StringUtil.matchNumber(empNo);
        return userDtoCache.computeIfAbsent(empNo, key -> {
            ServiceData<List<UserDto>> responseData = forwardApi.fetchUser(key, auth, key);
            return Optional.ofNullable(responseData)
                    .map(ServiceData::getBo)
                    .flatMap(list -> list.stream().findFirst())
                    .orElseThrow(() -> new RpcException(responseData.getCode().getCode(), responseData.getCode().getMsg()));
        });
    }
```
