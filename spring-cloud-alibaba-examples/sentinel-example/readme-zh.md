# Sentinel Example

## 项目说明

本项目演示如何使用 Sentinel starter 完成 Spring Cloud 应用的限流管理。

[Sentinel](https://github.com/alibaba/Sentinel) 是阿里巴巴开源的分布式系统的流量防卫组件，Sentinel 把流量作为切入点，从流量控制，熔断降级，系统负载保护等多个维度保护服务的稳定性。


## 示例

### 如何接入
在启动示例进行演示之前，我们先了解一下如何接入 Sentinel。
**注意 本章节只是为了便于您理解接入方式，本示例代码中已经完成接入工作，您无需再进行修改。**

1. 首先，修改 pom.xml 文件，引入 Sentinel starter。

	    <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
        </dependency>
	
2. 接入限流埋点

	1. HTTP埋点
		Sentinel starter 默认为所有的 HTTP 服务提供了限流埋点，如果只想对 HTTP 服务进行限流，那么只需要引入依赖，无需修改代码。
		
	2. 自定义埋点
		如果需要对某个特定的方法进行限流或降级，可以通过 @SentinelResource 来完成限流的埋点，示例代码如下
	
			@SentinelResource("resource")
			public String hello() {
				return "Hello";
			}
		  
3. 配置限流规则
	
	Sentinel提供了两种配置限流规则的方式，代码配置 和 控制台配置，本示例使用的方式为通过控制台配置。

	1. 通过代码来实现限流规则的配置。一个简单的限流规则配置示例代码如下，更多限流规则配置详情请参考 [Sentinel 文档](https://github.com/alibaba/Sentinel/wiki/%E5%A6%82%E4%BD%95%E4%BD%BF%E7%94%A8#%E5%AE%9A%E4%B9%89%E8%A7%84%E5%88%99)。
	
			List<FlowRule> rules = new ArrayList<FlowRule>();
			FlowRule rule = new FlowRule();
			rule.setResource(str);
			// set limit qps to 10
			rule.setCount(10);
			rule.setGrade(RuleConstant.FLOW_GRADE_QPS);
			rule.setLimitApp("default");
			rules.add(rule);
			FlowRuleManager.loadRules(rules);
  
	2. 通过控制台进行限流规则配置请参考文章后面的图文说明。

### 启动 Sentinel 控制台

1. 首先需要获取 Sentinel 控制台，支持直接下载和源码构建两种方式。

	1. 直接下载：[下载 Sentinel 控制台](http://edas-public.oss-cn-hangzhou.aliyuncs.com/install_package/demo/sentinel-dashboard.jar) 
	2. 源码构建：进入 Sentinel [Github 项目页面](https://github.com/alibaba/Sentinel)，将代码 git clone 到本地自行编译打包，[参考此文档](https://github.com/alibaba/Sentinel/tree/master/sentinel-dashboard)。

2. 启动控制台，执行 Java 命令 `java -jar sentinel-dashboard.jar`完成 Sentinel 控制台的启动。
	控制台默认的监听端口为 8080。Sentinel 控制台使用 Spring Boot 编程模型开发，如果需要指定其他端口，请使用 Spring Boot 容器配置的标准方式，详情请参考 [Spring Boot 文档](https://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#boot-features-customizing-embedded-containers)。

### 应用启动

1. 增加配置，在应用的 /src/main/resources/application.properties 中添加基本配置信息
	
		spring.application.name=sentinel-example
		server.port=18083
		spring.cloud.sentinel.dashboard=localhost:8080
		
2. 启动应用，支持 IDE 直接启动和编译打包后启动。

	1. IDE直接启动：找到主类 `ServiceApplication`，执行 main 方法启动应用。
	2. 打包编译后启动：首先执行 `mvn clean package` 将工程编译打包，然后执行 `java -jar sentinel-example.jar`启动应用。

### 调用服务

使用 curl 分别调用两个 URL，可以看到访问成功。

<p align="center"><img src="https://cdn.yuque.com/lark/0/2018/png/54319/1532084640137-8f4bc16c-4336-4c1b-9ddd-4582b967717a.png" width="240" heigh='180' ></p>


### 配置限流规则并验证

1. 访问 http://localhost:8080 页面，可以在左侧看到 Sentinel-Example 应用已经注册到了控制台，单击 **流控规则** ，可以看到目前的流控规则为空。
**注意 如果您在控制台没有找到应用，请调用一下进行了 Sentinel 埋点的 URL 或方法，因为 Sentinel 使用了 lazy load 策略**

<p align="center"><img src="https://cdn.nlark.com/lark/0/2018/png/54319/1532315951819-9ffd959e-0547-4f61-8f06-91374cfe7f21.png" width="1000" heigh='400' ></p>


2. 配置 URL 限流规则：点击新增流控规则，资源名填写需要限流的 URL 相对路径，单机阈值选择需要限流的阈值，点击新增进行确认。(为了便于演示效果，这里将值设置成了 1)。

<p align="center"><img src="https://cdn.yuque.com/lark/0/2018/png/54319/1532078717483-62ab74cd-e5da-4241-a45d-66166b1bde99.png" width="480" heigh='180' ></p>


3. 配置自定义限流规则：点击新增流控规则，资源名填写 @SentinelResource 注解 value 字段的值，单机阈值选择需要限流的阈值，点击新增进行确认。(为了便于演示效果，这里将值设置成了 1)。


<p align="center"><img src="https://cdn.yuque.com/lark/0/2018/png/54319/1532080384317-2943ce0a-daaf-495d-8afc-79a0248a119a.png" width="480" heigh='180' ></p>


4. 访问 URL，当 QPS 超过 1 时,可以看到限流效果如下。

<p align="center"><img src="https://cdn.yuque.com/lark/0/2018/png/54319/1532080652178-be119c4a-2a08-4f67-be70-fe5ed9a248a3.png" width="480" heigh='180' ></p>

<p align="center"><img src="https://cdn.yuque.com/lark/0/2018/png/54319/1532080661437-b84ee161-6c2d-4df2-bdb7-7cf0d5be92fb.png" width="480" heigh='180' ></p>


## 自定义限流处理逻辑

1. URL 限流触发后默认处理逻辑是，直接返回 "Blocked by Sentinel (flow limiting)"。
	如果需要自定义处理逻辑，实现的方式如下

		public class CustomUrlBlockHandler implements UrlBlockHandler {
			@Override
			public void blocked(HttpServletRequest httpServletRequest,
			HttpServletResponse httpServletResponse) throws IOException {
				// todo add your logic
			}
		}
		
		WebCallbackManager.setUrlBlockHandler(new CustomUrlBlockHandler());



2. 自定义限流触发后，默认的处理逻辑是抛出异常。
	如果需要自定义处理逻辑，填写@SentinelResource注解的blockHandler和blockHandlerClass属性，指定后会去blockHandlerClass类里找对应的blockHandler静态方法。示例实现如下

		@SentinelResource(value = "resource", blockHandler = "", blockHandlerClass = ExceptionUtil.class)
    	public String hello() {
        	return "Hello";
    	}
    	
    	// ExceptionUtil.java
    	public class ExceptionUtil {
        	public static void handleException(BlockException ex) {
        		System.out.println("Oops: " + ex.getClass().getCanonicalName());
        	}
        }
    	

## Endpoint 信息查看

Spring Boot 应用支持通过 Endpoint 来暴露相关信息，Sentinel Starter 也支持这一点。

在使用之前需要在 maven 中添加 `spring-boot-starter-actuator`依赖，并在配置中允许 Endpoints 的访问。
* Spring Boot1.x 中添加配置 management.security.enabled=false
* Spring Boot2.x 中添加配置 management.endpoints.web.exposure.include=*

Spring Boot1.x 可以通过访问 http://127.0.0.1:18083/sentinel 来查看 Sentinel Endpoint 的信息。Spring Boot2.x 可以通过访问 http://127.0.0.1:18083/acutator/sentinel 来访问。

<p align="center"><img src="https://cdn.yuque.com/lark/0/2018/png/54319/1532084199224-1a41591d-7a06-4680-be8a-5de319ac635d.png" width="480" heigh='360' ></p>

## 查看实时监控
Sentinel 控制台支持实时监控查看，您可以通过 Sentinel 控制台查看各链路的请求的通过数和被限流数等信息。
其中 p_qps 为通过(pass) 流控的 QPS，b_qps 为被限流 (block) 的 QPS。

<p align="center"><img src="https://cdn.nlark.com/lark/0/2018/png/54319/1532313595369-8428cd7d-9eb7-4786-a149-acf0da4a2daf.png" width="480" heigh='180' ></p>

## DataSource支持

Sentinel内部提供了[动态规则的扩展实现DataSource](https://github.com/alibaba/Sentinel/wiki/%E5%8A%A8%E6%80%81%E8%A7%84%E5%88%99%E6%89%A9%E5%B1%95#datasource-%E6%89%A9%E5%B1%95)。

Sentinel starter整合了目前存在的4类DataSource。只需要在配置文件中进行相关配置，即可在Spring容器中自动注册DataSource。

比如要定义一个FileRefreshableDataSource，配置如下：

    spring.cloud.sentinel.datasource.type=file
    spring.cloud.sentinel.datasource.recommendRefreshMs=2000
    spring.cloud.sentinel.datasource.bufSize=2048
    spring.cloud.sentinel.datasource.charset=utf-8
    spring.cloud.sentinel.datasource.configParser=myParser
    spring.cloud.sentinel.datasource.file=/Users/you/rule.json
    
然后使用`@SentinelDataSource`注解修饰DataSource即可注入：
 
    @SentinelDataSource("spring.cloud.sentinel.datasource")
    private DataSource dataSource;
    
`@SentinelDataSource`注解的value属性可以不填。默认值就是spring.cloud.sentinel.datasource。

value属性代表配置前缀。示例中会去找spring.cloud.sentinel.datasource.xxx相关的配置。

spring.cloud.sentinel.datasource.type就是对应的DataSource类型。

spring.cloud.sentinel.datasource.recommendRefreshMs里的recommendRefreshMs对应相关DataSource的属性。

spring.cloud.sentinel.datasource.configParser代表ConfigParser在Spring容器里的name。如果没找到，会抛出异常。
    
type目前支持file, nacos, zk, apollo。

### 自定义DataSource

自定义DataSource只需要两部。

1. 定义DataSource
    
        public class CustomDataSource implements DataSource {
            private String fieldA;
            private String fieldB;
            ...
        }
    
2. 装配DataSource。有两种方式处理。

    * 直接构造DataSource
        
            @Bean
            public CustomDataSource customDataSource() {
                CustomDataSource customDataSource =
                                                new CustomDataSource();
                customDataSource.setFieldA("valueA");
                customDataSource.setFieldB("valueB");
                ...
                return customDataSource;
            }

    * 在classpath:/META-INF/sentinel-datasource.properties中管理DataSource信息
    
            custom = yourpackage.CustomDataSource
       
       在application.properties中定义DataSource
       
            spring.cloud.sentinel.datasource.type = custom
            spring.cloud.sentinel.datasource.fieldA = valueA
            spring.cloud.sentinel.datasource.fieldB = valueB
       
        注意：由于目前Sentinel的AbstractDataSource需要有个ConfigParser作为构造函数中的参数，并且它的子类的构造都是通过多个参数的构造函数构造的。
            所以目前所有的Sentinel starter中的DataSource都是基于FactoryBean并且通过设置属性构造的。如果有这方面的需求，需要再多加一个registerFactoryBean过程。
            
            SentinelDataSourceRegistry.registerFactoryBean("custeom", CustomDataSourceFactoryBean.class);
            
        如果自定义DataSource可以注入属性，那么没有必要使用SentinelDataSourceRegistry注册FactoryBean。
    

## More
Sentinel 是一款功能强大的中间件，从流量控制，熔断降级，系统负载保护等多个维度保护服务的稳定性。此 Demo 仅演示了 使用 Sentinel 作为限流工具的使用，更多 Sentinel 相关的信息，请参考 [Sentinel 项目](https://github.com/alibaba/Sentinel)。

如果您对 spring cloud sentinel starter 有任何建议或想法，欢迎在 issue 中或者通过其他社区渠道向我们提出。

