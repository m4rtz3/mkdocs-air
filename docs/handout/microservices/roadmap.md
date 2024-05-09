
## Microsserviço

A fim de implementar microsserviços em Spring Boot, aqui, é proposto uma abordagem de modularização de cada microsserviço, de forma que exista uma interface de comunicação Java a ser consumida por outros microsserviços, também em Java, e também um compromisso de implementação. Essa estratégia visa aumentar a produtividade do ambiente de desenvolvimento em Java, já que para o consumo da API por outros frameworks sempre será necessário reescrever as assinaturas de cada endpoint.

### Modularização

Crie dois projetos Mavens:

* um de interface, e;
* outro para o micro serviço.

A vantagem dessa abordagem é que a interface pode ser utilizada em outros projetos como uma biblioteca a ser consumida.

Exemplo de uso dessa abordagem no microsserviço **Account**:

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

#### Interface

Para compilar e instalar a interface do microsserviço, crie um `pom.xml` específico para essa interface e seus **dtos** (AccountIn e AccountOut).

``` shell title="Installing the microservice interface"
mvn clean install
```

#### Implementação

A implementação não precisa ser instalada como biblioteca do repositório Maven, pois é apenas para execução do microsserviço. Porém, o microsserviço deve ter explicíto a chamada da biblioteca de interface no seu `pom.xml`.

``` xml
<dependency>
  <groupId>insper.store</groupId>
  <artifactId>account</artifactId>
  <version>${project.version}</version>
</dependency>
```

O comando para empacotar o microsserviço é:

``` shell title="Packaging the microservice"
mvn clean package
```

Adicionalmente, para executar o microsserviço:

``` shell title="Packaging and running the microservice"
mvn clean package spring-boot:run
```

## Banco de dados

Muitos microsserviços podem persistir seus dados em banco de dados. Cada microsserviço é responsável pelo acesso e gravação de seus dados de forma autônoma.

Isso aumenta de forma significativa a complexidade do gerenciamento do microsserviço, pois se torna necessário manter o gerenciamento da base de dados tais como: alterações, versões e roteiros de retornos.

O [Flyway](https://flywaydb.org/){target="_blank"} é uma biblioteca que pode ser acoplado ao framework Spring Boot a fim de ajudar na tarefa de gerenciamento e criação do sistema de persitência dos dados do microsserviço.


Para fazer uso dessa biblioteca, altere o `pom.xml` adicionando a dependência da biblioteca **JPA** assim bem como a dependência da biblioteca **Flyway**.

``` xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>org.flywaydb</groupId>
    <artifactId>flyway-core</artifactId>
</dependency>
<dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
    <version>42.7.2</version>
</dependency>
```

Alterações no arquivo de propriedades também são necessárias, para definir o banco de dados e sua configuração JPA, assim bem como, a configuração do Flyway.

``` yaml title="Exemplo baseado no microsserviço Account"
spring:
  datasource:
    url: ${DATABASE_URL:jdbc:postgresql://localhost:5432/store}
    username: ${DATABASE_USERNAME:store}
    password: ${DATABASE_PASSWORD:store123321}
    driver-class-name: org.postgresql.Driver
  flyway:
    baseline-on-migrate: true
    schemas: account
  jpa:
    properties:
      hibernate:
        default_schema: account
```

A estrutura de organização e execução de scripts de banco de dados do Flyway é persistido na seguinte hierárquia de diretórios, onde cada arquivo é executado em ordem alfanumérica.

``` tree title="exemplo"
store.account
store.account-resource
    src
        main
            java
            resources
                db
                    migration
                        V2024.02.16.001__create_schema.sql
                        V2024.02.16.002__create_table_account.sql
                application.yaml
```

=== "V2024.02.16.001__create_schema.sql"

    ``` sql
    CREATE SCHEMA IF NOT EXISTS account;
    ```

=== "V2024.02.16.002__create_table_account.sql"

    ``` sql
    CREATE TABLE account
    (
        id_account character varying(36) NOT NULL,
        tx_name character varying(256) NOT NULL,
        tx_email character varying(256) NOT NULL,
        tx_hash character varying(256) NOT NULL,
        CONSTRAINT account_pkey PRIMARY KEY (id_account)
    );
    ```

## Conectando Microsserviços - OpenFeign

Nomeando o microsserviço dentro do sistema de discovery.

``` xml
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

``` java
@FeignClient(name = "store-account")
public interface AccountController {
  ...
}
```

## Docker

Para cada microsserviço Java Spring Cloud é aconselhável criar um arquivo `Dockerfile` no diretório raiz do projeto a fim de permitir a criação adequada da imagem do microserviço.

``` docker title="Typical Dockerfile for Java microservice"
FROM openjdk:23-slim
VOLUME /tmp
COPY target/*.jar app.jar
ENTRYPOINT ["java","-jar","/app.jar"]
```

## Docker Compose

O [Docker Compose](https://docs.docker.com/compose/){target='_blank'} permite criar um cluster com todos os microsserviços necesários para o funcionamento de um sistema em uma rede apartada (nat).

Para criar um docker compose basta criar um arquivo de configuração chamado `docker-compose.yaml` em uma pasta que possa acessar os demais microsserviços, como uma pasta **store.docker-platform**.

``` tree title="exemplo"
store.account
store.account-resource
store.docker-platform
    .env
    docker-compose.yaml
```

Dentro do arquivo, cada microsserviço é declarado e configurado, utilizando imagem que são criadas no momento de execução do docker engine ou imagens que estão disponíveis em algum diretório (eg.: DockerHub).

``` yaml title="docker-compose.yaml"
# docker compose up -d --build --force-recreate
version: '3.8'
name: store

services:

  db-store:
    container_name: store-db-store
    image: postgres:latest
    ports:
      - 5432:5432
    environment:
      - POSTGRES_USER=store
      - POSTGRES_PASSWORD=store
      - POSTGRES_DB=store
    volumes:
      - $VOLUME/postgres/store/data:/var/lib/postgresql/data
    restart: always
    networks:
      - private-network

  account:
    build:
      context: ../store.account-resource/
      dockerfile: Dockerfile
    image: store-account:latest
    environment:
      - spring.datasource.url=jdbc:postgresql://store-db-store:5432/store
      - spring.datasource.username=store
      - spring.datasource.password=store
    deploy:
      mode: replicated
      replicas: 1
    restart: always
    networks:
      - private-network
    depends_on:
      - db-store

networks:
  private-network:
    driver: bridge

```

Arquivo de configuração de ambiente.

``` shell title=".env"
VOLUME=./volume
CONFIG=./config
```


Na pasta do arquivo `docker-compose.yaml` execute o comando docker para criar as imagens e subir os containers:

``` shell title="Rise up a cluster"
docker compose up -d --build
```

``` shell title="Shutdown the cluster"
docker compose down
```

Referencia:

