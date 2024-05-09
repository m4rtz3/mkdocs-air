
Esse microserviço é responsável por gerenciar as contas dos usuários do sistema que está sendo desenvolvido. Ele também pode ser utilizado como template para o desenvolvimento de outros microserviços que se utilizem de recuros semelhantes em seu funcionamento.

  1. [Endpoints](#endpoints)
  2. [Modularização](#modularizacao)
    * Interface
    * Resource
  3. [Persistência](#persitence)
  4. [Documentação](#documentation)
  5. [Integração](#integration)
  6. [Docker](#docker)

## Endpoints

??? note "Create Account"

    ``` url
    POST /accounts
    ```

    Request
    ``` json
    {
        "name": "Antonio do Estudo",
        "email": "acme@insper.edu.br",
        "password": "123@"
    }
    ```
    Response

    | code | body | 
    |--|--|
    | 201 | `{ "id": "45d16201-12a4-48bf-8c84-df768fdc4878", "name": "Antonio do Estudo", "email": "acme@insper.edu.br" }`{:.json} |
    | 401 | | 

??? info "Login :: find by email and password"

???+ info "Get Account"

    ```
    GET /accounts/{uuid}
    ```
    Response

    | code | body | 
    |--|--|
    | 200 | ```{"id": "45d16201-12a4-48bf-8c84-df768fdc4878", "name": "Antonio do Estudo", "email": "acme@insper.edu.br" }``` |
    | 401 | | 

## Modularização

Class Diagram

Exemplo para o microsserviço **Account**.

``` mermaid
classDiagram
  namespace Interface {
    class AccountController {
      <<interface>>
      create(AccountIn)
      read(String id): AccountOut
      update(String id, AccountIn)
      delete(String id)
      findByEmailAndPassword(AccountIn)
    }
    class AccountIn {
      <<record>>
      String name
      String email
      String password
    }
    class AccountOut {
      <<record>>
      String id
      String name
      String email
    }
  }
  namespace Resource {
    class AccountResource {
      <<REST API>>
      -accountService
    }
    class AccountService {
      <<service>>
      -accountRepository
      create(Account)
    }
    class AccountRepository {
      <<nterface>>
      findByEmailAndHash(String, String)
    }
    class AccountModel {
      <<entity>>
      String id
      String name
      String email
      String hash
    }
    class Account {
      <<dto>>
      String id
      String name
      String email
      String password
    }
  }
  AccountController <|-- AccountResource
  AccountResource o-- AccountService
  AccountService o-- AccountRepository
```

## POM dependecy

Note que esse microsserviço possui dependência da interface, o **Account**. Logo, se torna necessário explicitar essa dependência no `pom.xml` do microsserviço [Account](./account.md).

``` xml
<dependency>
  <groupId>insper.store</groupId>
  <artifactId>account</artifactId>
  <version>${project.version}</version>
</dependency>
```

Outras dependências relevantes para adicionar no `pom.xml` são o suporte ao registro no discovery.

``` xml
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```


Quando adicionado o acesso ao Discovery, é necessário definir no `application.yalm` o nome com o qual o serviço será invocado, assim bem como, o endereço de acesso do discovery ao qual o serviço irá conectar:

``` yaml
spring:
  application:
    name: store-account

eureka:
  client:
    register-with-eureka: true
    fetch-registry: true
    service-url:
      defaultZone: ${EUREKA_URI:http://localhost:8761/eureka/}
```

Já para disponibilizar o uso ao `OpenFeign`.

``` xml
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```
