## [ROTA](#rota)
### [Cadastro e Login do Usuário](#cadastro-usuario)

```mermaid
sequenceDiagram
    participant User as User
    participant Frontend as Frontend
    participant Backend as Backend

    User->>Frontend: Access Home Page
    Frontend->>Backend: POST /consult-travel
    Backend-->>Frontend: Return best routes
    Frontend-->>User: Display best routes
    User->>Frontend: Create an account
    Frontend->>Backend: POST /create-account
    Backend-->>Frontend: Account created
    User->>Frontend: Login
    Frontend->>Backend: POST /login
    Backend-->>Frontend: Login successful
    Frontend-->>User: Access Dashboard
```

## [INTERAÇÃO](#int)
### [Usuário e Chat-Bot](#user-bot)

```mermaid
sequenceDiagram
    Actor User as User
    Participant Frontend as Frontend
    Participant Backend as Backend

    User->>Frontend: Access Chat-bot
    Frontend->>Backend: GET /chat-bot
    Backend-->>Frontend: Return Chat-bot Interface
    Frontend-->>User: Display Chat-bot Interface
    User->>Frontend: Enter Question
    Frontend->>Backend: POST /submit-question
    Backend-->>Frontend: Return Answer
    Frontend-->>User: Display Answer
```

  <!--classDiagram
 class Jogador {
    <<class>>
    - Long id
    - String username
    - String tag
    - String playtime
    ...
  }
  class JogadorRepository {
    <<interface>>
    + List<Object[]> findWinPercentage()
  }
  class JogadorRequestDTO {
    <<record>>
    - Long id
    - String username
    - String tag
    ...
  }
  class JogadorController {
    + @GetMapping("/porcen-vitoria")
    + @GetMapping("/porcenVitAgent")
  }
  class JogadorResponeDTO {
    <<record>>
    - Long id
    - String username
    - String tag
  }

  JogadorController <|-- JogadorRepository
  Jogador o-- JogadorRepository
  Jogador o-- JogadorResponeDTO
  Jogador o-- JogadorRequestDTO
  Jogador o-- JogadorController

 -->