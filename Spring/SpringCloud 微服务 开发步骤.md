<b>参考：http://blog.didispace.com/springcloud1/</b>
<b>创建一个基础的Spring Boot工程,每个项目都需加上SpringCloud的依赖</b>
<b>使用的是Brixton版本</b>

``` 
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.3.5.RELEASE</version>
    <relativePath/> <!-- lookup parent from repository -->
</parent>
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-test</artifactId>
	<scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-eureka-server</artifactId>
    </dependency>
</dependencies>
<dependencyManagement>
    <dependencies>
        <dependency>
	    <groupId>org.springframework.cloud</groupId>
	    <artifactId>spring-cloud-dependencies</artifactId>
	    <version>Brixton.RELEASE</version>
	    <type>pom</type>
	    <scope>import</scope>
	</dependency>
    </dependencies>
</dependencyManagement>
```
#### 一.服务注册中心
    a)加入依赖
```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-eureka-server</artifactId>
</dependency>
```
    b)在主程序中加入注解@EnableEurekaServer
        启动一个服务注册中心
        
        
    c)默认情况下，注册中心会将自己作为客户端需要禁用,配置application.properties
```
server.port=1111
eureka.client.register-with-eureka=false
eureka.client.fetch-registry=false
eureka.client.serviceUrl.defaultZone=http://localhost:${server.port}/eureka/
```
       
#### 二.服务提供方
    a)加入依赖
```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-eureka</artifactId>
</dependency>
```
    b)实现处理接口，通过DiscoveryClient获取信息
```
@RestController
public class ComputeController {
    private final Logger logger = Logger.getLogger(getClass());
    @Autowired
    private DiscoveryClient client;
    @RequestMapping(value = "/add" ,method = RequestMethod.GET)
    public Integer add(@RequestParam Integer a, @RequestParam Integer b) {
        ServiceInstance instance = client.getLocalServiceInstance();
        Integer r = a + b;
        logger.info("/add, host:" + instance.getHost() + ", service_id:" + instance.getServiceId() + ", result:" + r);
        return r;
    }
}
```
    c)在主类中加入@EnableDiscoveryClient注解
        激活Eureka中的DiscoveryClient实现
    
    d)配置application.properties
```
#微服务的名称
spring.application.name=compute-service
server.port=2222
#指定服务注册中心得位置
eureka.client.serviceUrl.defaultZone=http://localhost:1111/eureka/
```

#### 三.服务消费者

##### a)Ribbon
        1）加入依赖
```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-ribbon</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-eureka</artifactId>
</dependency>
```
        2）在应用主类中加入@EnableDiscoveryClient注解
        添加发现服务的能力，创建RestTemplate实例，添加@LoadBalanced注解开启均衡负载能力
```
@SpringBootApplication
@EnableDiscoveryClient
public class RibbonApplication {
	@Bean
	@LoadBalanced
	RestTemplate restTemplate() {
		return new RestTemplate();
	}
	public static void main(String[] args) {
		SpringApplication.run(RibbonApplication.class, args);
	}
}
```
        3)创建ConsumerController来消费，直接通过RestTemplate来调用服务
```
@RestController
public class ConsumerController {
    @Autowired
    RestTemplate restTemplate;
    @RequestMapping(value = "/add", method = RequestMethod.GET)
    public String add() {
        return restTemplate.getForEntity("http://COMPUTE-SERVICE/add?a=10&b=20", String.class).getBody();
    }
}
```
        4)配置application.properties
```
spring.application.name=ribbon-consumer
server.port=3333
eureka.client.serviceUrl.defaultZone=http://localhost:1111/eureka/
```
##### b)Feign
        1)加入依赖
```
<dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-feign</artifactId>
</dependency>
<dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-eureka</artifactId>
</dependency>
```
        2)在应用主类加入@EnableFeignClients注解
        开启Fegin功能
```
@SpringBootApplication
@EnableDiscoveryClient
@EnableFeignClients
public class FeignApplication {
	public static void main(String[] args) {
		SpringApplication.run(FeignApplication.class, args);
	}
}
```
        3)定义想要调用的服务的接口
```
//通过FeignClient("compute-service")绑定接口对应的service服务
@FeignClient("compute-service")
public interface ComputeClient {
    //是这里的value在起作用，它映射到服务提供方的方法
    @RequestMapping(method = RequestMethod.GET, value = "/add")
    Integer add(@RequestParam(value = "a") Integer a, @RequestParam(value = "b") Integer b);
}
```
        4)在web层调用接口
```
@RestController
public class ConsumerController {
    @Autowired
    ComputeClient computeClient;
    @RequestMapping(value = "/add", method = RequestMethod.GET)
    public Integer add() {
        return computeClient.add(10, 20);
    }
}
```
        5)配置application.properties
```
spring.application.name=feign-consumer
server.port=3333
eureka.client.serviceUrl.defaultZone=http://localhost:1111/eureka/

```

#### 四.断路器
##### a)Ribbon引入Hystrix
        1)添加依赖
```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-hystrix</artifactId>
</dependency>
```
        2)在主类中加入@EnableCircuitBreaker注解
        开启断路器功能
```
@SpringBootApplication
@EnableDiscoveryClient
@EnableCircuitBreaker
public class RibbonApplication {
	@Bean
	@LoadBalanced
	RestTemplate restTemplate() {
		return new RestTemplate();
	}
	public static void main(String[] args) {
		SpringApplication.run(RibbonApplication.class, args);
	}
}
```
        3)改造原来的服务消费方式，新增ComputeService类，在使用的函数上增加@HystrixCommand注解指定回调方法
```
@Service
public class ComputeService {
    @Autowired
    RestTemplate restTemplate;
    @HystrixCommand(fallbackMethod = "addServiceFallback")
    public String addService() {
        return restTemplate.getForEntity("http://COMPUTE-SERVICE/add?a=10&b=20", String.class).getBody();
    }
    public String addServiceFallback() {
        return "error";
    }
}
```
        4)将提rest接口的Controller改为调用ComputeService的addService
```
@RestController
public class ConsumerController {
    @Autowired
    private ComputeService computeService;
    @RequestMapping(value = "/add", method = RequestMethod.GET)
    public String add() {
        return computeService.addService();
    }
}
```

##### b)Feign使用Hystrix
        1)使用@FeignClient注解中的fallback属性指定回调类
```
@FeignClient(value = "compute-service", fallback = ComputeClientHystrix.class)
public interface ComputeClient {
    @RequestMapping(method = RequestMethod.GET, value = "/add")
    Integer add(@RequestParam(value = "a") Integer a, @RequestParam(value = "b") Integer b);
}
```
        2)创建回调类ComputeClientHystrix，实现含有@FeignClient注解的接口
```
@Component
public class ComputeClientHystrix implements ComputeClient {
    @Override
    public Integer add(@RequestParam(value = "a") Integer a, @RequestParam(value = "b") Integer b) {
        return -9999;
    }
}
```

#### 五.分布式配置中心
##### a)构建Config Server
        1)加入依赖
```
<dependency>
		<groupId>org.springframework.cloud</groupId>
		<artifactId>spring-cloud-config-server</artifactId>
</dependency>
```
        2)在应用主类中加入@EnableConfigServer注解
        开启Config Server
```
@EnableConfigServer
@SpringBootApplication
public class Application {
	public static void main(String[] args) {
		new SpringApplicationBuilder(Application.class).web(true).run(args);
	}
}
```
        3)配置application.properties，添加服务和git信息
```
spring.application.name=config-server
server.port=7001
# git管理配置
#git仓库的位置
spring.cloud.config.server.git.uri=http://git.oschina.net/didispace/SpringBoot-Learning/
#配置仓库路径下的相对搜索路径，可配置多个
spring.cloud.config.server.git.searchPaths=Chapter9-1-4/config-repo
#访问git仓库的用户名
spring.cloud.config.server.git.username=username
#访问git仓库的用户密码
spring.cloud.config.server.git.password=password
```
##### b)微服务端映射配置
        1）加入依赖
```
<dependency>
		<groupId>org.springframework.cloud</groupId>
		<artifactId>spring-cloud-starter-config</artifactId>
</dependency>
```
        2)创建bootstrap.properties配置，指定config server
```
#{application}-{profile}.properties对应的配置文件
spring.application.name=didispace
spring.cloud.config.profile=dev
#对应当前配置的git分支
spring.cloud.config.label=master
#配置中心的地址
spring.cloud.config.uri=http://localhost:7001/
server.port=7002
```
        3)创建一个Rest Api来返回配置中心的from属性
```
@RefreshScope
@RestController
class TestController {
    //通过@Value("${from}")绑定配置服务中配置的from属性
    @Value("${from}")
    private String from;
    @RequestMapping("/from")
    public String from() {
        return this.from;
    }
}
```

#### 六.分布式配置中心(高可用)
##### a)config-server配置
        1)新增依赖
```
<dependency>
		<groupId>org.springframework.cloud</groupId>
		<artifactId>spring-cloud-starter-eureka</artifactId>
</dependency>
```
        2)配置application.properties
```
spring.application.name=config-server
server.port=7001
# 配置服务注册中心
eureka.client.serviceUrl.defaultZone=http://localhost:1111/eureka/
# git仓库配置
spring.cloud.config.server.git.uri=http://git.oschina.net/didispace/SpringCloud-Learning/
spring.cloud.config.server.git.searchPaths=Chapter1-1-8/config-repo
spring.cloud.config.server.git.username=username
spring.cloud.config.server.git.password=password
```
        3)新增@EnableDiscoveryClient注解，将config-server注册到服务注册中心
```
@EnableDiscoveryClient
@EnableConfigServer
@SpringBootApplication
public class Application {
	public static void main(String[] args) {
		new SpringApplicationBuilder(Application.class).web(true).run(args);
	}
}
```

##### b)config-client配置
        1)新增依赖
```
<dependency>
		<groupId>org.springframework.cloud</groupId>
		<artifactId>spring-cloud-starter-eureka</artifactId>
</dependency>
```
        2)配置bootstrap.properties
```
spring.application.name=didispace
server.port=7002
eureka.client.serviceUrl.defaultZone=http://localhost:1111/eureka/
#开启通过服务访问Config Server的功能
spring.cloud.config.discovery.enabled=true
#指定Config Server注册的服务名
spring.cloud.config.discovery.serviceId=config-server
spring.cloud.config.profile=dev
```

        3)在应用主类新增@EnableDiscoveryClient注解
        用来发现config-server服务
        
##### c)配置刷新
        1)新增依赖
```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

#### 七.服务网关
##### a)使用Zuul
        1)添加依赖
```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-zuul</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-eureka</artifactId>
</dependency>
```
        2)在应用主类加入@EnableZuulProxy注解
        开启Zuul
```
@EnableZuulProxy
//整合了@SpringBootApplication、@EnableDiscoveryClient、@EnableCircuitBreaker
@SpringCloudApplication
public class Application {
	public static void main(String[] args) {
		new SpringApplicationBuilder(Application.class).web(true).run(args);
	}
}
```
        3)配置application.properties
```
spring.application.name=api-gateway
server.port=5555
```

##### c)服务路由
        1)通过url直接映射(不太友好)
```
# routes to url
zuul.routes.api-a-url.path=/api-a-url/**
zuul.routes.api-a-url.url=http://localhost:2222/
```
        2)通过serviceId
```
zuul.routes.api-a.path=/api-a/**
zuul.routes.api-a.serviceId=service-A
zuul.routes.api-b.path=/api-b/**
zuul.routes.api-b.serviceId=service-B
eureka.client.serviceUrl.defaultZone=http://localhost:1111/eureka/
```

##### d)服务过滤
        1)创建Zuul过滤器
```
public class AccessFilter extends ZuulFilter  {
    private static Logger log = LoggerFactory.getLogger(AccessFilter.class);
    @Override
    public String filterType() {
        return "pre";
    }
    @Override
    public int filterOrder() {
        return 0;
    }
    @Override
    public boolean shouldFilter() {
        return true;
    }
    @Override
    public Object run() {
        RequestContext ctx = RequestContext.getCurrentContext();
        HttpServletRequest request = ctx.getRequest();
        log.info(String.format("%s request to %s", request.getMethod(), request.getRequestURL().toString()));
        Object accessToken = request.getParameter("accessToken");
        if(accessToken == null) {
            log.warn("access token is empty");
            //令zuul过滤该请求，不对其进行路由
            ctx.setSendZuulResponse(false);
            //设置了其返回的错误码
            ctx.setResponseStatusCode(401);
            return null;
        }
        log.info("access token ok");
        return null;
    }
}
```
    2)实例化自定义过滤器
```
@EnableZuulProxy
@SpringCloudApplication
public class Application {
	public static void main(String[] args) {
		new SpringApplicationBuilder(Application.class).web(true).run(args);
	}
	@Bean
	public AccessFilter accessFilter() {
		return new AccessFilter();
	}
}
```

##### 八.高可用服务注册中心
        a)创建application-peer1.properties，作为peer1服务中心的配置，并将serviceUrl指向peer2
```
spring.application.name=eureka-server
server.port=1111
eureka.instance.hostname=peer1
eureka.client.serviceUrl.defaultZone=http://peer2:1112/eureka/
```
        b)创建application-peer2.properties，作为peer2服务中心的配置，并将serviceUrl指向peer1
```
spring.application.name=eureka-server
server.port=1112
eureka.instance.hostname=peer2
eureka.client.serviceUrl.defaultZone=http://peer1:1111/eureka/
```
        c)对hosts文件进行转换
```
127.0.0.1 peer1
127.0.0.1 peer2
```
        d)将项目打包(mvn install),通过spring.profiles.active属性启动peer1和peer2
```
java -jar eureka-server-1.0.0.jar --spring.profiles.active=peer1
java -jar eureka-server-1.0.0.jar --spring.profiles.active=peer2
```
        e)服务注册与发现
        修改application.properties
```
spring.application.name=compute-service
server.port=2222
#将服务指向了peer1和peer2
eureka.client.serviceUrl.defaultZone=http://peer1:1111/eureka/,http://peer2:1112/eureka/

```

#### 九.消息总线
##### a)RabbitMQ
        1)扩展config-client-eureka，加入依赖
```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-bus-amqp</artifactId>
</dependency>
```
        2)在配置文件中增加关于RabbitMQ的连接和用户信息
        需要先安装RabbitMQ，然后创建一个用户，为用户添加角色和权限
```
spring.rabbitmq.host=localhost
spring.rabbitmq.port=5672
spring.rabbitmq.username=springcloud
spring.rabbitmq.password=123456
```
        3)指定刷新范围
        /bus/refresh接口还提供了destination参数，用来定位具体要刷新的应用程序。比如，我们可以请求/bus/refresh?destination=customers:9000，destination参数除了可以定位具体的实例之外，还可以用来定位具体的服务。定位服务的原理是通过使用Spring的PathMatecher（路径匹配）来实现，比如：/bus/refresh?destination=customers:**，该请求会触发customers服务的所有实例进行刷新。
        
        4)架构优化
        在之前的架构中，服务的配置更新需要通过向具体服务中的某个实例发送请求，再触发对整个服务集群的配置更新。
        
            1.在Config Server中也引入Spring Cloud Bus，将配置服务端也加入到消息总线中来。
            
            2./bus/refresh请求不在发送到具体服务实例上，而是发送给Config Server，并通过destination参数来指定需要更新配置的服务或实例。
            

##### b)Kafka
        1)整合Spring Cloud Bus,替换依赖
```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-bus-kafka</artifactId>
</dependency>
```
        2)Kafka的配置
```

属性名	说明	默认值
spring.cloud.stream.kafka.binder.brokers	           Kafka的服务端列表	localhost
spring.cloud.stream.kafka.binder.defaultBrokerPort	    Kafka服务端的默认端口，当brokers属性中没有配置端口信息时，就会使用这个默认端口	9092
spring.cloud.stream.kafka.binder.zkNodes	           Kafka服务端连接的ZooKeeper节点列表	localhost
spring.cloud.stream.kafka.binder.defaultZkPort	       ZooKeeper节点的默认端口，当zkNodes属性中没有配置端口信息时，就会使用这个默认端口
```


        

        
        

    
