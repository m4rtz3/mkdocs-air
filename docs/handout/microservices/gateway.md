
O gateway tem como função ser o único ponto de entrada de todo o sistema, ele é responsável por redirecionar todas as requisições aos respectivos microsserviços. Assim bem como, de autorizar ou negar acesso ao sistema baseando-se no token de segurança passado pela requisição.

``` mermaid
flowchart LR
  subgraph Client
    direction LR
    Web
    Mobile
    Desktop
  end
  subgraph Microservices
    direction LR
    gateway["Gateway"]
    subgraph Essentials
      direction TB
      discovery["Discovery"]
      auth["Auth"]
      config["Configuration"]
    end
    subgraph Businesses
      direction TB
      ms1["Service 1"]
      ms2["Service 2"]
      ms3["Service 3"]
    end
  end
  Client --> lb["Load Balance"] --> gateway --> Businesses
  gateway --> auth
  gateway --> discovery
```


### Sequence Diagram

``` mermaid
sequenceDiagram
  autonumber
  actor User
  User->>Gateway: route(ServerHttpRequest)
  Gateway->>+AuthenticationFilter: filter(ServerWebExchange, GatewayFilterChain)
  AuthenticationFilter->>RouteValidator: isSecured.test(ServerHttpRequest)
  RouteValidator-->>AuthenticationFilter: True | False
  critical notSecured
    AuthenticationFilter->>Gateway: follow the flux
  end
  AuthenticationFilter->>AuthenticationFilter: isAuthMissing(ServerHttpRequest)
  critical isAuthMissing
    AuthenticationFilter->>User: unauthorized message
  end
  AuthenticationFilter->>AuthenticationFilter: validateAuthorizationHeader()
  critical isInvalidAuthorizationHeader
    AuthenticationFilter->>User: unauthorized message
  end
  AuthenticationFilter->>Auth: solve(Token)
  critical isInvalidToken
    Auth->>User: unauthorized message
  end
  Auth->>AuthenticationFilter: returns token claims
  AuthenticationFilter->>AuthenticationFilter: updateRequestHeader(ServerHttpRequest)
  AuthenticationFilter->>Gateway: follow the flux
```




``` xml title="pom.xml"
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-loadbalancer</artifactId>
</dependency>
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-webflux</artifactId>
</dependency>
<!-- https://mvnrepository.com/artifact/com.github.ben-manes.caffeine/caffeine -->
<dependency>
  <groupId>com.github.ben-manes.caffeine</groupId>
  <artifactId>caffeine</artifactId>
  <version>3.1.8</version>
</dependency>
<dependency>
  <groupId>insper.store</groupId>
  <artifactId>auth</artifactId>
  <version>0.0.1-SNAPSHOT</version>
</dependency>
```

``` yaml title="application.yaml"
spring:
  application:
    name: store-gateway
  cloud:
    discovery:
      locator:
        enabled: true
    gateway:
      routes:
            
        - id: auth
          uri: lb://store-auth
          predicates:
            - Path=/auth/**

        # - id: product
        #   uri: lb://store-product
        #   predicates:
        #     - Path=/product/**

      default-filters:
        - DedupeResponseHeader=Access-Control-Allow-Credentials Access-Control-Allow-Origin
      globalcors:
        corsConfigurations:
          '[/**]':
            allowedOrigins: "http://localhost"
            allowedHeaders: "*"
            allowedMethods:
            - GET
            - POST
      
api:
  endpoints:
    open: >
      POST /auth/register/,
      POST /auth/login/
```


``` java title="GatewayConfiguration.java"
package insper.store.gateway;

import org.springframework.cloud.client.loadbalancer.LoadBalanced;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.reactive.function.client.WebClient;

@Configuration
public class GatewayConfiguration {

    @Bean
    @LoadBalanced
    public WebClient.Builder webClient() {
        return WebClient.builder();
    }
    
}
```

``` java title="RouterValidator.java"
package insper.store.gateway.security;

import java.util.List;
import java.util.function.Predicate;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.http.server.reactive.ServerHttpRequest;
import org.springframework.stereotype.Component;

@Component
public class RouterValidator {

        @Value("${api.endpoints.open}}") 
        private List<String> openApiEndpoints;

        public Predicate<ServerHttpRequest> isSecured =
                request -> openApiEndpoints
                        .stream()
                        .noneMatch(uri -> {
                                String[] parts = uri.replaceAll("[^a-zA-Z0-9// ]", "").split(" ");
                                return request.getMethod().toString().equalsIgnoreCase(parts[0])
                                    && request.getURI().getPath().equals(parts[1]);
                        });

}
```

``` java title="AuthenticationFilter.java"
package insper.store.gateway.security;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.cloud.gateway.filter.GatewayFilterChain;
import org.springframework.cloud.gateway.filter.GlobalFilter;
import org.springframework.http.HttpHeaders;
import org.springframework.http.HttpStatus;
import org.springframework.http.MediaType;
import org.springframework.http.server.reactive.ServerHttpRequest;
import org.springframework.stereotype.Component;
import org.springframework.web.reactive.function.client.WebClient;
import org.springframework.web.server.ResponseStatusException;
import org.springframework.web.server.ServerWebExchange;

import reactor.core.publisher.Mono;
import store.auth.IdIn;
import store.auth.IdOut;

@Component
public class AuthenticationFilter implements GlobalFilter {

    private static final String HEADER_AUTHORIZATION = "Authorization";
    private static final String HEADER_BEARER = "Bearer";

    @Autowired
    private RouterValidator routerValidator;

    @Autowired
    private WebClient.Builder webClient;

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        ServerHttpRequest request = exchange.getRequest();
        if (!routerValidator.isSecured.test(request)) {
            return chain.filter(exchange);
        }
        if (!isAuthMissing(request)) {
            final String[] parts = this.getAuthHeader(request).split(" ");
            if (parts.length != 2 || !parts[0].equals(HEADER_BEARER)) {
                throw new ResponseStatusException(HttpStatus.FORBIDDEN, "Authorization header format must be Bearer {token}");
            }
            final String token = parts[1];
            return webClient
                .defaultHeader(HttpHeaders.CONTENT_TYPE, MediaType.APPLICATION_JSON_VALUE)
                .build()
                .post()
                .uri("http://store-auth/auth/token/")
                .bodyValue(new IdIn(token))
                .retrieve()
                .toEntity(IdOut.class)
                .flatMap(response -> {
                    if (response != null && response.getBody() != null) {
                        this.updateRequest(exchange, response.getBody().id());
                        return chain.filter(exchange);
                    } else {
                        throw new ResponseStatusException(HttpStatus.UNAUTHORIZED, "Invalid token");
                    }
                });
        }
        throw new ResponseStatusException(HttpStatus.UNAUTHORIZED, "Missing authorization header");
    }

    private String getAuthHeader(ServerHttpRequest request) {
        return request.getHeaders().getOrEmpty(HEADER_AUTHORIZATION).get(0);
    }

    private boolean isAuthMissing(ServerHttpRequest request) {
        return !request.getHeaders().containsKey("Authorization");
    }    

    private void updateRequest(ServerWebExchange exchange, String id) {
        exchange.getRequest().mutate()
                .header("id-user", id)
                .build();
    }

}
```