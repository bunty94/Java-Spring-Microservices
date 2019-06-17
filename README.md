# Java-Spring-Microservices

MicroService : REST, Small well chosen deployable units and cloud enabled.

Challenges in MicroService: 

Bounded Context : How to identify what a microservice should do.

Configuration Management: Each Microservice is on multiple instances in each environment.

 Ex: 10 Microservices with 5 environments(Dev,qa,qa2,stage,Prod) and 50 instances.
 
Dynamic Scaleup and ScaleDown: Scaling up and down based on traffic and Distribute load.

Visibility & Monitoring: Identify Bug Which service is causing it.Need to Have a Centralized log to find what happened for a specific request.
Montroing of which services are down and not enough disk space.


Pack of Cards : Microservices are like pack of cards.if one micorservices goes down then entire application goes down.Fault tolerance for micorservices are important.


Spring Cloud Solutions to Challenges: spring cloud is not a single project.there are multiple project that can be used to solving common problems.Spring Cloud Provides challenges mentioned above.
	
	Configuration Management: Spring Cloud Config Server.
	
	Dynamic Scaleup and ScaleDown:  
						Naming Server (Eureka)
						Ribbon (Client Side Load Balancing)
						Feign (Easier to call REST)
	
	Visibility and Monitoring : 
						Zipkin Distributed Tracing.
						Netflix API Gateway.
	Fault Tolerance: 
					Hystrix
	
	
Advantages of MicroService: Encourages to adapt new technology and process.Dynamic Scaling which can be different based on Demand of the season.


Faster Release Cycles: new Feature can be release easily.


Step1:

com.in28minutees.micorservices
limits-service
web,devtools,acutaor,config client


Step2:
spring.application.name=limits-service
@RestController
class LimitsConfigurationController 
getMapping("/limits")
method retrievalLimitsConfigurations()
returns LimitConfiguration.

LimitConfiguration with maximum and minimum

Step3:

limits-service.minimum=99
limits-service.maximum=999

@Component
@ConfigProperty("limits-service") -prefix
Class Configuration

private int minimum
private int maximum

generate setters and getters

@autowire Configuration in LimitConfiguration pass in getters of Configuration method.

Step4:

com.in28minutees.micorservices
spring-cloud-config-server
dep : devtools and config server.


spring.application.name=spring-cloud-config-server
server.port=8888

install git.

step6 & 7: Connection to SpringConfig Server to Git.

create a folder git-localconfig-repo
git  init
spring-cloud-config-server  -> build path-> linksource ->git-localconfig-repo
add limits-service.properties copy paste all limits properties.
commit to git add files

spring.cloud.config.server.git.uri=file:///{localgit_repo} location.(Connection between spring config and git local repo.)

enable @EnableConfigServer to SprinConfigserver

Step8 & 9:  Store all env config of service to git & limits to SpringConfigServer.
add files to git.

http://localhost:8888/limits-service/dev
http://localhost:8888/limits-service/qa


change limits-service.properties to bootstrap.properties remove limits related properties
add spring.cloud.config.uri=http://localhost:8888


step10: profiles for limits-service
bootstrap
spring.profiles.active=dev


step11: review 

step12:
setup currency-exchange-service
web,devtools,actuator,config client.

@RestController
public class CurrencyExchangeController {
	@GetMapping("/currency-exchange/from/{from}/to/{to}")
	public ExchangeValue retrieveExchangeValue(@PathVariable String from, @PathVariable String to) {
		return new ExchangeValue(1000L, from, to, BigDecimal.valueOf(65));
	}
	

@Entity
public class ExchangeValue {
	@Id
	@GeneratedValue
	private Long id;
	@Column(name="currency_from")
	private String from;
	@Column(name="currency_to")
	private String to;
	private int port;
	@Column(name="conversion_multiple")
	private BigDecimal conversionMultiple;
	
	protected ExchangeValue() {}
	
	public ExchangeValue(Long id, String from, String to, BigDecimal conVersionMultiple) {
		super();
		this.id = id;
		this.from = from;
		this.to = to;
		this.conversionMultiple = conVersionMultiple;
	}
	public Long getId() {
		return id;
	}
	public void setId(Long id) {
		this.id = id;
	}
	public String getFrom() {
		return from;
	}
	public void setFrom(String from) {
		this.from = from;
	}
	public String getTo() {
		return to;
	}
	public void setTo(String to) {
		this.to = to;
	}
	public int getPort() {
		return port;
	}
	public void setPort(int port) {
		this.port = port;
	}
	public BigDecimal getConVersionMultiple() {
		return conversionMultiple;
	}
	public void setConVersionMultiple(BigDecimal conVersionMultiple) {
		this.conversionMultiple = conVersionMultiple;
	}
	

}

	

Step15:Dynamic Port response

@Autowired
private Environment environment;

@GetMapping("/currency-exchange/from/{from}/to/{to}")
	public ExchangeValue retrieveExchangeValue(@PathVariable String from, @PathVariable String to) {
		ExchangeValue exchangeValue = new ExchangeValue(1000L, from, to, BigDecimal.valueOf(65));
		exchangeValue.setPort(Integer.parseInt(environment.getProperty("local.server.port")));
		return exchangeValue;
	}
	


Run Config : Arugments 
	-Dserver.port=8001
	
Step16 & 17: setup JPA and create Repo
<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-data-jpa</artifactId>
		</dependency>
		<dependency>
			<groupId>com.h2database</groupId>
			<artifactId>h2</artifactId>
		</dependency>

insert into exchange_value(id,currency_from,currency_to,conversion_multiple,port)
values(10001,'USD','INR',65,0);
insert into exchange_value(id,currency_from,currency_to,conversion_multiple,port)
values(10002,'EUR','INR',75,0);
insert into exchange_value(id,currency_from,currency_to,conversion_multiple,port)
values(10003,'AUS','INR',25,0);


spring.jpa.show-sql=true
spring.h2.console.enabled=true	

h2 console db url : http://localhost:8000/h2-console/
jdbc url: 
jdbc:h2:mem:testdb

		
Step 18 &19: create currency Conversion Service & creationg controller

currency-conversion-service
web,devtools,actuator,config client.

@RestController
public class CurrencyConversionController {
	@GetMapping("/currency-converter/from/{from}/to/{to}/quantity/{quantity}")
	public CurrencyConversionBean convertCurrency(@PathVariable String from, @PathVariable String to,
			@PathVariable BigDecimal quantity) {
		return new CurrencyConversionBean(1L, from, to, BigDecimal.ONE, quantity, quantity, 0);
	}
}


public class CurrencyConversionBean {
	private Long id;
	private String from;
	private String to;
	private BigDecimal conversionMultiple;
	private BigDecimal quantity;
	private BigDecimal totalCalculatedAmount;
	private int port;
...
 	
Step20:Invoke Currency Exchange service from calculation service.

	Map<String, String> uriVariable = new HashMap<String, String>();
		uriVariable.put("from", from);
		uriVariable.put("to", to);
		ResponseEntity<CurrencyConversionBean> responseEntity = new RestTemplate().getForEntity(
				"http://localhost:8000/currency-exchange/from/{from}/to/{to}", CurrencyConversionBean.class,
				uriVariable);

		CurrencyConversionBean response = responseEntity.getBody();

		return new CurrencyConversionBean(response.getId(), from, to, response.getConversionMultiple(), quantity,
				quantity.multiply(response.getConversionMultiple()), 0);
				
				
Step21: Using Feign Rest Client For Service Invocation.

Feign makes easy to invoke other micorservices using simple proxy. Integration with client side load balancing.

		<dependency>
		    <groupId>org.springframework.cloud</groupId>
		    <artifactId>spring-cloud-starter-openfeign</artifactId>
		</dependency>


Facing issue with Feign Dependency Jar.->maven->updateproject->check all and update.

@EnableFeignClients("com.in28minutees.micorservices.currencyconversionservice")
public class CurrencyConversionServiceApplication {


@FeignClient(name = "currency-exchange-service", url = "localhost:8000")
public interface CurrencyExchangeServiceProxy {
	@GetMapping("/currency-exchange/from/{from}/to/{to}")
	public CurrencyConversionBean retrieveExchangeValue(@PathVariable String from, @PathVariable String to);
}


@GetMapping("/currency-converter-feign/from/{from}/to/{to}/quantity/{quantity}")
	public CurrencyConversionBean convertCurrencyFeign(@PathVariable String from, @PathVariable String to,
			@PathVariable BigDecimal quantity) {

		CurrencyConversionBean response = proxy.retrieveExchangeValue(from, to);

		return new CurrencyConversionBean(response.getId(), from, to, response.getConversionMultiple(), quantity,
				quantity.multiply(response.getConversionMultiple()), 0);
	}


Step22
Ribbon  client side load balancing.

dependency:
		<dependency>
		    <groupId>org.springframework.cloud</groupId>
		    <artifactId>spring-cloud-starter-netflix-ribbon</artifactId>
		</dependency>


//@FeignClient(name = "currency-exchange-service", url = "localhost:8000")
@FeignClient(name = "currency-exchange-service")
@RibbonClient(name = "currency-exchange-service")
public interface CurrencyExchangeServiceProxy {



Step 23:	

@FeignClient(name = "currency-exchange-service")
@RibbonClient(name = "currency-exchange-service")
public interface CurrencyExchangeServiceProxy {

currency-exchange-service.ribbon.listOfServers=http://localhost:8000,http://localhost:8001


Step24 & 25:
with ribbon we need to add servers to the property file (Hard code it)

Name servers: all instances of all micorservices register with naming server.Also called service registration.

Whenever some service wants to talk to other service then it would talk to naming server. For all the instance of service running.Also called service Discovery.

netflix-eureka-naming-server

eureka server, config client actutaor devtools

@EnableEurekaServer
public class NetflixEurekaNamingServerApplication {

spring.application.name=netflix-eureka-naming-server
server.port=8761
eureka.client.register-with-eureka=false
eureka.client.fetch-registry=false

Error with Eureka Dependency.

Step26 & 27:
Connect Exchange and conversion services to Eureka naming server.
in conversion service

eureka.client.service-url.default-zone=http://localhost:8761

		<dependency>
		    <groupId>org.springframework.cloud</groupId>
		    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
		</dependency>

@EnableDiscoveryClient
public class CurrencyConversionServiceApplication {

eureka.client.service-url.default-zone=http://localhost:8761


Step28:Distributing calling using Eureka and Ribbon.

just comment listOfServers in conversion property and run multiple instance of exchange-service.

Conversion service called loading balancing and discovery is takend care by ribbon and eureka.

Step 29,30,31: review of Eureka and ribbon.

Introduction to API Gateway.

Authentication,Authorization and security.
Rate limits
Fault Tolerance.
Service Aggregation.

Setup zuul api gateway.
com.practice.micorservices
netflix-zuul-api-gateway-server
zuul,discovery client,actuator,devtools

@EnableZuulProxy
@EnableDiscoveryClient
public class NetflixZuulApiGatewayServerApplication {

spring.application.name=netflix-zuul-api-gateway-server
server.port=8765
eureka.client.service-url.default-zone=http://localhost:8761

Step 32: Imp Zuul logger Filter

Logging,Security,RateLimiting can be implemented using zuul.


@Component
public class ZuulLogginFilter extends  ZuulFilter{
	
	private Logger logger=LoggerFactory.getLogger(this.getClass());
	@Override
	public boolean shouldFilter() {
		return true;
	}

	@Override
	public Object run() throws ZuulException {
		HttpServletRequest request = RequestContext.getCurrentContext().getRequest();
		logger.info("request -> { request  uri -> {}",request,request.getRequestURI());
		return null;
	}

	@Override
	public String filterType() {
		return "pre";
	}

	@Override
	public int filterOrder() {
		return 1;
	}

}

Step33: executing request Zuul Api Gateway.

http://localhost:8000/currency-exchange/from/{from}/to/{to}

http://localhost:8765/currency-exchange-service/currency-exchange/from/{from}/to/{to}
http://localhost:8765/{app name}/{uri}

step34: set up zuul api gateway between micorservices invokcation.

calling currencyConversion service to exchange-service with api-proxy via api-gateway
@FeignClient(name = "netflix-zuul-api-gateway-server")
public interface CurrencyExchangeServiceProxy {
	@GetMapping("/currency-exchange-service/currency-exchange/from/{from}/to/{to}")
	public CurrencyConversionBean retrieveExchangeValue(@PathVariable String from, @PathVariable String to);
}

http://localhost:8100/currency-conversion-feign/from/EUR/to/INR/quantity/1000

calling currency-conversion-service via api-gateway

http://local:8765/{app.name}/{uri}

http://local:8765/currency-conversion-service/currency-conversion-feign/from/EUR/to/INR/quantity/1000

therefore every call to each micorservices go through api-gateway.

step35: adding same id for particular request across micorservices. distributed tracing.

assing id to request to trace.
below code to all micorservices 

		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-sleuth</artifactId>
		</dependency>

Run Applicaiton file.

@Bean
	public Sampler defaultSampler() {
		return Sampler.ALWAYS_SAMPLE;
	}



========15-06-2018==========

step 36 to 41: setup zipkin

download rabbit mq,erlang. check for erlang compatibilty with rabbit mq in rabbitmq site

https://zipkin.io/quickstart.sh

open quickstart.sh this will download zipkin.jar

open cmd 
check for java -version minimum required 1.8

cd F:\Nithin\Video Tutorials\Spring\Spring

java -jar zipkin.jar

http://localhost:9411

kill the server: cntrl + C

we need zipkin connect rabbitmq.

SET RABBIT_URI=amqp://localhost

java -jar zipkin.jar
	
connecting microservices to zipkin.

add to api-gateway,conversion,exchange service.
<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-sleuth-zipkin</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-bus-amqp</artifactId>
		</dependency>

		
run: zipkin server,eureka,exchange,conversion,zuul api-gateway.

check if all are running.

http://localhost:9411/zipkin/
http://localhost:8761/

traces of a microservice can be checked from zipkin dashboard.


step 42: understanding need for cloud bus.

limit services.
management.security.enabled=false


when a property is change property file. http://{host and port}/application/refresh or http://{host and port}/application/actuator/refresh need to refresh in order to reflect changes.
ex: http://localhost:8080/application/refresh or  http://localhost:8080/application/actuator/refresh

if there are 10 microservice with 3 instances then hitting every service refresh takes lot of time to do and not so practical.

step 43:
add to limits and spring-cloud-config

<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-bus-amqp</artifactId>
		</dependency>
		
http://localhost:8080/bus/refresh.
or 
spring-boot 2.0.0+
http://localhost:8080/actuator/bus-refresh this will update for all instances of microservice.

		
