## Spring Cloud Config

Spring Cloud Config provides server-side and client-side support for externalized configuration in a distributed system. With the Config Server, you have a central place to manage external properties for applications across all environments.

Key components of Spring Cloud Config include:

- **Config Server**: A standalone server that provides a REST API for providing configuration properties to clients. The server is embeddable in a Spring Boot application, by using the `@EnableConfigServer` annotation. The properties can be stored in various types of repositories (Git, SVN, filesystem, etc.).

- **Config Client**: A library for Spring Boot applications. It fetches the configuration properties from the Config Server and bootstrap them into the application's context. It's included in the classpath by adding the `spring-cloud-starter-config` dependency.

- **Refresh Scope**: Spring Cloud Config includes a `RefreshScope` capability which allows properties to be reloaded without restarting the application. You can expose a `/refresh` endpoint in your application that, when invoked, will cause the application to re-fetch properties from the Config Server.

[Spring Cloud Config Server](https://docs.spring.io/spring-cloud-config/reference/server.html)