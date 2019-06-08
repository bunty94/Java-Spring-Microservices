# Java-Spring-Microservices

MicroService : REST, Small well chosen deployable units and cloud enabled.

Challenges in MicroService: 

Bounded Context : How to identify what should a microservice should do.

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
						Feign (Easier REST Clients)
	
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


class LimitsConfigurationController 

method retrievalLimitsConfigurations()
returns LimitConfiguration.

LimitConfiguration with maximum and minimum

Step3:

limits-service.minimum=99
limits-service.maximum=999


@ConfigProperty("limits-service") -prefix
Class Configuration

private int minimum
private int maximum

generate setters and getters

@autowire in Configuration

 and LimitConfiguration pass in getters of Configuration method.

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

@RestController
public class CurrencyExchangeController {
	@GetMapping("/currency-exchange/from/{from}/to/{to}")
	public ExchangeValue retrieveExchangeValue(@PathVariable String from, @PathVariable String to) {
		return new ExchangeValue(1000L, from, to, BigDecimal.valueOf(65));
	}
	

public class ExchangeValue {
	private Long id;
	private String from;
	private String to;
	private BigDecimal ConversionMultiple;

	public ExchangeValue() {

	}

	public ExchangeValue(Long id, String from, String to, BigDecimal conversionMultiple) {
		super();
		this.id = id;
		this.from = from;
		this.to = to;
		ConversionMultiple = conversionMultiple;
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

	public BigDecimal getConversionMultiple() {
		return ConversionMultiple;
	}

	public void setConversionMultiple(BigDecimal conversionMultiple) {
		ConversionMultiple = conversionMultiple;
	}

}
	

Step15:Dynamic Port response

@GetMapping("/currency-exchange/from/{from}/to/{to}")
	public ExchangeValue retrieveExchangeValue(@PathVariable String from, @PathVariable String to) {
		ExchangeValue exchangeValue = new ExchangeValue(1000L, from, to, BigDecimal.valueOf(65));
		exchangeValue.setPort(Integer.parseInt(environment.getProperty("local.server.port")));
		return exchangeValue;
	}
	
public class ExchangeValue {
private int port;

Run Config : Arugments 
	-Dserver.port=8001
	
Step16: setup JPA 
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


@Entity
public class ExchangeValue {

	@Id
	private Long id;
	@Column(name = "currency_from")
	private String from;
	@Column(name = "currency_to")


spring.jpa.show-sql=true
spring.h2.console.enabled=true	

Step17: create Repo

public interface ExchangeValueRepository extends JpaRepository<ExchangeValue, Long> {
	ExchangeValue findByFromAndTo(String from, String to);
}


@Autowired
	private ExchangeValueRepository repository;
	
	
ExchangeValue exchangeValue = repository.findByFromAndTo(from, to);
		exchangeValue.setPort(Integer.parseInt(environment.getProperty("local.server.port")));
		return exchangeValue;	
		

Step 18 &19: create currency Conversion Service & creationg controller

currency-conversion-service

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

Facing issue with Feign Dependency Jar.

Feign makes easy to invoke other micorservices. Integration with client side load balancing.



		
