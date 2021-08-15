# Spring Boot Actuator: Production-ready Features
Spring Boot Actuator is a sub-project of the Spring Boot Framework. It includes a number of additional features that help us to monitor and manage the Spring Boot application. It contains the actuator endpoints (the place where the resources live). We can use HTTP and JMX endpoints to manage and monitor the Spring Boot application. If we want to get production-ready features in an application, we should use the Spring Boot actuator.

## Spring Boot Actuator Features
There are three main features of Spring Boot Actuator:
* Endpoints
* Metrics
* Audit


Endpoint: The actuator endpoints allows us to monitor and interact with the application. Spring Boot provides a number of built-in endpoints. We can also create our own endpoint. We can enable and disable each endpoint individually. Most of the application choose HTTP, where the Id of the endpoint, along with the prefix of /actuator, is mapped to a URL.

For example, the /health endpoint provides the basic health information of an application. The actuator, by default, mapped it to /actuator/health.

Metrics: Spring Boot Actuator provides dimensional metrics by integrating with the micrometer. The micrometer is integrated into Spring Boot. It is the instrumentation library powering the delivery of application metrics from Spring. It provides vendor-neutral interfaces for timers, gauges, counters, distribution summaries, and long task timers with a dimensional data model.

Audit: Spring Boot provides a flexible audit framework that publishes events to an AuditEventRepository. It automatically publishes the authentication events if spring-security is in execution.

## Enabling Production-ready Features
The recommended way to enable the features is to add a dependency on the spring-boot-starter-actuator ‘Starter’.
```
Definition of Actuator
An actuator is a manufacturing term that refers to a mechanical device for moving or controlling something. Actuators can generate a large amount of motion from a small change.
```
To add the actuator to a Maven based project, add the following ‘Starter’ dependency:
```
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
</dependencies>
```
## Enabling and Exposing Endpoints
Starting with Spring Boot 2, we have to enable and expose our endpoints. By default, all endpoints except /shutdown are enabled and only `/health` is exposed. All endpoints are found at `/actuator` even if we've configured a different root context for our application.

That means that once we've added the appropriate starters to our Maven configuration, we can access the `/health` at `http://localhost:8080/actuator/health`.

Let's go to `http://localhost:8080/actuator` and view a list of available endpoints because the actuator endpoints are HATEOS enabled. We should see /health and /info.
```
{
    "_links": {
        "self": {
            "href": "http://localhost:8080/actuator",
            "templated": false
        },
        "health": {
            "href": "http://localhost:8080/actuator/health",
            "templated": false
        },
        "health-path": {
            "href": "http://localhost:8080/actuator/health/{*path}",
            "templated": true
        }
    }
}
```
## Exposing All Endpoints
Now, let's expose all endpoints except `/shutdown` by modifying our application.properties file:
```
management.endpoints.web.exposure.include=*
```
By default, all endpoints except `/shutdown` are enabled. To configure the enablement of an endpoint, use its `management.endpoint.<id>.enabled` property. The following example enables the `shutdown` endpoint:
```
management.endpoint.shutdown.enabled=true
```

## Exposing Specific Endpoints
Some endpoints can expose sensitive data, so let's learn how to be more find-grained about which endpoints we expose.

The `management.endpoints.web.exposure.include` property can also take a comma-separated list of endpoints. So, let's only expose `/beans` and `/loggers`:
```
management.endpoints.web.exposure.include=beans, loggers
```
In addition to including certain endpoints with a property, we can also exclude endpoints. Let's expose all the endpoints except `/threaddump`:
```
management.endpoints.web.exposure.include=*
management.endpoints.web.exposure.exclude=threaddump
```
Both include and exclude properties take a list of endpoints. The exclude property takes precedence over include.

## Enabling Specific Endpoints
Next, let's learn how we can get more fine-grained about which endpoints we have enabled.

First, we need to turn off the default that enables all the endpoints:
```
management.endpoints.enabled-by-default=false
```
Next, let's enable and expose only the /health endpoint:
```
management.endpoint.health.enabled=true
management.endpoints.web.exposure.include=health
```
With this configuration, we can access only the `/health` endpoint.

## Enabling Shutdown
Because of its sensitive nature, the `/shutdown` endpoint is disabled by default.

Let's enable it now by adding a line to our application.properties file:
```
management.endpoint.shutdown.enabled=true
```
Now when we query the `/actuator` endpoint, we should see it listed. The `/shutdown` endpoint only accepts POST requests, so let's shut down our application gracefully:
```
curl -X POST http://localhost:8080/actuator/shutdown
```

**Moreover, if we want to configure a custom management base path, then we should use that base path as the discovery URL.**

For instance, if we set the `management.endpoints.web.base-path` to `/mgmt`, then we should send a request to the `/mgmt` endpoint to see the list of links.

Quite interestingly, when the management base path is set to /, the discovery endpoint is disabled to prevent the possibility of a clash with other mappings.
## Securing Endpoints
In a real-world application, we're most likely going to have security on our application. With that in mind, let's secure our actuator endpoints.

First, let's add security to our application by adding the security starter Maven dependency:
```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```
For the most basic security, that's all we have to do. Just by adding the security starter, we've automatically applied basic authentication to all exposed endpoints except /info and /health.

Now, let's customize our security to restrict the `/actuator` endpoints to an ADMIN role.

Let's start by excluding the default security configuration:
```
@SpringBootApplication(exclude = {
SecurityAutoConfiguration.class,
ManagementWebSecurityAutoConfiguration.class
})
```
Let's note the ManagementWebSecurityAutoConfiguration.class because this will let us apply our own security configuration to the `/actuator`.

Over in our configuration class, let's configure a couple of users and roles, so we have an ADMIN role to work with:
```
@Override
protected void configure(AuthenticationManagerBuilder auth) throws Exception {
    PasswordEncoder encoder = PasswordEncoderFactories.createDelegatingPasswordEncoder();
    auth
      .inMemoryAuthentication()
      .withUser("user")
      .password(encoder.encode("password"))
      .roles("USER")
      .and()
      .withUser("admin")
      .password(encoder.encode("admin"))
      .roles("USER", "ADMIN");
}
```
SpringBoot provides us with a convenient request matcher to use for our actuator endpoints.

Let's use it to lockdown our `/actuator` to only the ADMIN role:
```
@Override
protected void configure(HttpSecurity http) throws Exception {
    http.requestMatcher(EndpointRequest.toAnyEndpoint())
            .authorizeRequests((requests) -> requests.anyRequest()
                    .hasRole("ADMIN"));
    http.httpBasic();
}
```
Here, we learned how Spring Boot configures the actuator by default. After that, we customized which endpoints were enabled, disabled, and exposed in our application.properties file. Because Spring Boot configures the `/shutdown` endpoint differently by default, we learned how to enable it separately.

### Reference Documentation
For further reference, please consider the following sections:

* [Official Apache Maven documentation](https://maven.apache.org/guides/index.html)
* [Spring Boot Maven Plugin Reference Guide](https://docs.spring.io/spring-boot/docs/2.5.0/maven-plugin/reference/html/)
* [Create an OCI image](https://docs.spring.io/spring-boot/docs/2.5.0/maven-plugin/reference/html/#build-image)
* [Spring Boot Actuator](https://docs.spring.io/spring-boot/docs/2.5.0/reference/htmlsingle/#production-ready)

### Guides
The following guides illustrate how to use some features concretely:

* [Building a RESTful Web Service with Spring Boot Actuator](https://spring.io/guides/gs/actuator-service/)

