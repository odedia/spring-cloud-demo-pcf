# spring-cloud-demo-pcf

Spring Cloud Meetup
-------------------

Generate todo-service
------------------------------

start.spring.io

Web
Actuator
Rest Repositories
JPA
H2
MySQL
Config Client
Eureka Discovery
Lombok
Cloud Stream
RabbitMQ

Add to pom.xml:

```	<build>
		<finalName>app</finalName>
```

Generate a greeting
------
```
@RestController
@Slf4j
class BeerGreeting {
	@Autowired
	Environment env;
	
	@GetMapping("/greeting")
	String greeting(){
		log.info("Returning greeting from service...");
		return "Hello from your TODO App! I am service instance " + env.getProperty("CF_INSTANCE_INDEX");
	}
}
```

Build locally and run:
`mvn clean package`
`mvn spring-boot:run`

Build and push:
`mvn clean package -DskipTests && cf push -p target/app.jar todo-service`


Externalize the configuration
------
```
@RestController
@RefreshScope
@Slf4j
class BeerGreeting {
	@Value("${greeting}")
	private String greeting;
	
	@GetMapping("/greeting")
	String greeting(){
		log.info("Returning greeting from service...");
		return greeting;
	}
}
```
