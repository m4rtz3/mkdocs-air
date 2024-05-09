
A fim do sistema possuir um controle de acesso, é conveniente a criação de um microsserviço Auth, que será responsável pelo cadastro de usuários do sistema.

  1. [Endpoints](#endpoints)
  2. [Modularização](#modularizacao)
    * Interface
    * Resource
  3. [Documentação](#documentacao)
  4. [Integração](#integracao)
  5. [Token](#token)
  6. [Docker](#docker)

## Endpoints

??? note "Register"

    ``` url
    POST /auth/register
    ```

    ### Request
    ``` json
    {
        "name": "Antonio do Estudo",
        "email": "acme@insper.edu.br",
        "password": "123@321"
    }
    ```
    
    ### Response

    | code | body | 
    |--|--|
    | 201 |  |

    ### Sequence Diagram

    ``` mermaid
    sequenceDiagram
    autonumber
    actor User
    User->>+Auth: register(RegisterIn)
    Auth->>+Account: create(AccountIn)
    Account->>-Auth: returns the new account (AccountOut)
    Auth->>-User: returns 201
    ```

??? note "Autenticação :: Login"

    ``` 
    POST /auth/login
    ```

    ### Request

    ``` json
    {
        "email": "acme@insper.edu.br",
        "password": "123@321"
    }
    ```

    ### Response

    | code | body | 
    |--|--|
    | 201 | `{ "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiI0NWQxNjIwMS0xMmE0LTQ4YmYtOGM4NC1kZjc2OGZkYzQ4NzgiLCJuYW1lIjoiQW50b25pbyBkbyBFc3R1ZG8iLCJpYXQiOjE1MTYyMzkwMjIsInJvbGUiOiJyZWd1bGFyIn0.8eiTZjXGUFrseBP5J91UdDctw-Flp7HP-PAp1eO8f1M" }`{:.json} |
    | 403 | | 

    ### Sequence Diagram

    ``` mermaid
    sequenceDiagram
    autonumber
    actor User
    User->>+Auth: authenticate(CredentiaIn)
    Auth->>+Account: login(LoginIn)
    critical validated
        Account->>-Auth: returns the account
    option denied
        Auth-->>User: unauthorized message
    end  
    Auth->>Auth: generates a token
    Auth->>-User: returns LoginOut
    User->>User: stores the token to use for the next requests
    ```

## Modularização

Exemplo para o microsserviço **Auth**.

``` mermaid
classDiagram
  namespace Interface {
    class AuthController {
      <<interface>>
      register(RegisterIn)
      authenticate(CredentialIn): LoginOut
      solve(SolveIn): SolveOut
    }
    class RegisterIn {
      <<record>>
      String name
      String email
      String password
    }
    class CredentialIn {
      <<record>>
      String email
      String password
    }
    class LoginOut {
      <<Record>>
      String token
    }
    class SolveIn {
      <<Record>>
      String token
    }
    class SolveOut {
      <<Record>>
      String id
      String name
      String role
    }
  }
  namespace Resource {
    class AuthResource {
      <<REST API>>
      -authService
    }
    class AuthService {
      <<service>>
      JwtService jwtService
      register(RegisterIn)
      authenticate(CredentialIn)
    }
    class JwtService {
      <<service>>
      String secretKey
      String issuer
      long duration
      SecretKey key
      JwtParser parser
      init()
      create(String id, String name, String role): String
      getToken(String token): Token
      getRole(String token): String
    }
    class Token {
      <<record>>
      String id
      String name
      String role
    }
  }
  AuthController <|-- AuthResource
  AuthResource o-- AuthService
  AuthService o-- JwtService
```

Exemplo de uma implementação da interface **AuthController**.

``` java title="AuthController.java"
package store.auth;

import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;

@FeignClient("store-auth")
public interface AuthController {

    @PostMapping("/auth/register")
    ResponseEntity<?> create (
        @RequestBody(required = true) RegisterIn in
    );

    @PostMapping("/auth/login")
    ResponseEntity<LoginOut> authenticate (
        @RequestBody(required = true) Credential in
    );
}
```

Repare que há a publicação da interface como sendo um serviço a ser registrado no Discovery.

## Documentação

Para fazer a documentação dos APIs, de forma automatizada, é aconselhável a utilização da biblioteca `SpringDoc OpenAPI`.

``` xml title="pom.xml"
<!-- https://mvnrepository.com/artifact/org.springdoc/springdoc-openapi-ui -->
<dependency>
  <groupId>org.springdoc</groupId>
  <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
  <version>[2.3.0,)</version>
</dependency>
```

## Integração

A integração entre os microsserviços é feita via [OpenFeign](https://spring.io/projects/spring-cloud-openfeign). Esse framework precisa saber, quando a aplicação sobe, em quais pacotes irá procurar os serviços. Para isso, se torna necessário anotar a classe `AuthApplication` com a lista de pacotes, assim bem como, anotar que esse microsserviço irá trabalhar com a sistema de descoberta de microsserviços habitado.

``` java title="AuthApplication.java"
package store.auth;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.openfeign.EnableFeignClients;

@EnableFeignClients(basePackages = {
	"insper.store.account"
})
@EnableDiscoveryClient
@SpringBootApplication
public class AuthApplication {

	public static void main(String[] args) {
		SpringApplication.run(AuthApplication.class, args);
	}

}
```

Necessário também atualizar o `pom.xml` para que o microsserviço possa enxergar o outro microsserviço.

Note que esse microsserviço possui dependência de outro, o [Account](./account.md), além da dependência da interface do próprio microsserviço. Logo, se torna necessário explicitar essa dependência no `pom.xml` do microsserviço [Auth](./auth.md).

``` xml title="pom.xml"
<dependency>
  <groupId>insper.store</groupId>
  <artifactId>auth</artifactId>
  <version>${project.version}</version>
</dependency>
<dependency>
  <groupId>insper.store</groupId>
  <artifactId>account</artifactId>
  <version>${project.version}</version>
</dependency>

<!-- https://mvnrepository.com/artifact/io.jsonwebtoken/jjwt-api -->
<dependency>
  <groupId>io.jsonwebtoken</groupId>
  <artifactId>jjwt-api</artifactId>
  <version>0.12.3</version>
</dependency>
<dependency>
  <groupId>io.jsonwebtoken</groupId>
  <artifactId>jjwt-impl</artifactId>
  <version>0.12.3</version>
  <scope>runtime</scope>
</dependency>
<dependency>
  <groupId>io.jsonwebtoken</groupId>
  <artifactId>jjwt-jackson</artifactId> <!-- or jjwt-gson if Gson is preferred -->
  <version>0.12.3</version>
  <scope>runtime</scope>
</dependency>
```

Aproveitando esse ponto, vale a pena já incluir também no `pom.xml`.

## Token

Para gerar o token de acesso, no caso [JWT](../../platform/security/jwt.md), um serviço foi criado, `JwtService.java`.

Para gerar o JWT, alguns atributos são adicionados no `application.yaml`.

``` yaml title="application.yaml"
store:
  jwt:
    issuer: "In5pEr"
    secretKey: ""
    duration: 31536000000 # 365 days in miliseconds
```

``` java title="JwtService.java"
package store.auth;

import java.util.Date;

import javax.crypto.SecretKey;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Service;

import io.jsonwebtoken.Claims;
import io.jsonwebtoken.ExpiredJwtException;
import io.jsonwebtoken.JwtParser;
import io.jsonwebtoken.Jwts;
import io.jsonwebtoken.io.Decoders;
import io.jsonwebtoken.security.Keys;
import jakarta.annotation.PostConstruct;

@Service
public class JwtService {
    
    @Value("${store.jwt.secret-key}")
    private String secretKey;

    @Value("${store.jwt.issuer}")
    private String issuer;

    @Value("${store.jwt.duration}")
    private long duration = 1l;

    private SecretKey key;
    private JwtParser parser;

    @PostConstruct
    public void init() {
        this.key = Keys.hmacShaKeyFor(Decoders.BASE64.decode(secretKey));
        this.parser = Jwts.parser().verifyWith(key).build();
    }

    public String create(String id, String name, String role) {
        String jwt = Jwts.builder()
            .header()
            .and()
            .id(id)
            .issuer(issuer)
            .subject(name)
            .signWith(key)
            .claim("role", role)
            .notBefore(new Date())
            .expiration(new Date(new Date().getTime() + duration))
            .compact();
        return jwt;
    }

    public String getToken(String token) {
        final Claims claims = resolveClaims(token);
        return Token.builder
            .id(claims.getId())
            .role(claims.get("role", String.class))
            .build();
    }

    private Claims resolveClaims(String token) {
        if (token == null) throw new io.jsonwebtoken.MalformedJwtException("token is null");
        return validateClaims(parser.parseSignedClaims(token).getPayload());
    }

    private Claims validateClaims(Claims claims) throws ExpiredJwtException {
        if (claims.getExpiration().before(new Date())) throw new ExpiredJwtException(null, claims, issuer);
        if (claims.getNotBefore().after(new Date())) throw new ExpiredJwtException(null, claims, issuer);
        return claims;
    }

}
```

## Docker

Adicione no `docker-compose.yaml` o registro desse novo microsserviço:

``` yaml title="docker-compose.yaml"
  auth:
    build:
      context: ../store.auth-resource/
      dockerfile: Dockerfile
    container_name: store-auth
    image: store-auth:latest
    # ports:
    #   - 8080:8080
    environment:
      - eureka.client.service-url.defaultZone=http://store-discovery:8761/eureka/
    deploy:
      mode: replicated
      replicas: 1
    restart: always
    networks:
      - private-network
    depends_on:
      - discovery
      - account
```

???+ tip "NICE TO HAVE"

    O projeto da disciplina pode ter um microsserviço de registro que valide email ou SMS para criar a conta.