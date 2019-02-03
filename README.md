# Spring Cloud Demo on PCF - Walkthrough

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

Add to application.properties:

`greeting=Greetings from your TODO app!`


Test with `mvn spring-boot:run`

What's wrong with this implementation?
- Internal configuration
- Impossible to dynamically change config
- Every property change requires a rebuild

A Config Server can help here.

Create at `start.spring.io`

Add to main class:
`@EnableConfigServer`

Add to `application.properties`:
```
spring.application.name=config-server
spring.cloud.config.server.git.uri=https://github.com/odedia/spring-cloud-meetup-config-repo.git
server.port=8888
```
In todo-service, rename `application.properties` to `bootstrap.properties` and add the following:

```
spring.application.name=todo-service
spring.cloud.config.uri=http://localhost:8888
```

This all works for local work, but what about PCF?
Just create a service and provide the properties from above:

```
cf create-service p-config-server standard config-server -c '{"count": 2, "git": { "uri": "https://github.com/odedia/spring-cloud-meetup-config-repo.git" } }'
```

```mvn clean package && cf push -p target/app.jar todo-service```

