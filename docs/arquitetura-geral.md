# Arquitetura Geral

## Visão Geral

O projeto segue a arquitetura em camadas (Layered Architecture) com separação clara de responsabilidades dividida nos pacotes `controller`, `business` e `infrastructure`.

```
┌─────────────────────────────────────────────┐
│               Cliente (HTTP)                │
└─────────────────────┬───────────────────────┘
                      │ REST (JSON)
┌─────────────────────▼───────────────────────┐
│              Controller Layer               │
│  UsuarioController  │  AuthController       │
└─────────────────────┬───────────────────────┘
                      │
┌─────────────────────▼───────────────────────┐
│              Business Layer                 │
│              UsuarioService                 │
└─────────────────────┬───────────────────────┘
                      │
┌─────────────────────▼───────────────────────┐
│            Infrastructure Layer             │
│  Repository  │  Entity  │  Security         │
└─────────────────────┬───────────────────────┘
                      │ JPA/Hibernate
┌─────────────────────▼───────────────────────┐
│              PostgreSQL Database            │
└─────────────────────────────────────────────┘
```

## Camadas

### Controller Layer (`controller/`)

Responsável por receber requisições HTTP, validar a entrada e retornar respostas adequadas.

- **UsuarioController** — endpoints CRUD para usuários e login
- **AuthController** — endpoint auxiliar de autenticação
- **dtos/UsuarioDTO** — objeto de transferência para login (email + senha)

### Business Layer (`business/`)

Contém a lógica de negócio desacoplada da infraestrutura.

- **UsuarioService** — orquestra operações de criação, busca e remoção de usuários, aplica encoding de senha e valida regras como email duplicado

### Infrastructure Layer (`infrastructure/`)

Divide-se em três sub-pacotes:

| Sub-pacote | Responsabilidade |
|---|---|
| `entity/` | Mapeamento JPA para as tabelas do banco |
| `repository/` | Acesso a dados via Spring Data JPA |
| `security/` | Configuração do Spring Security + JWT |
| `exceptions/` | Exceções customizadas do domínio |

## Fluxo de Requisição

### Requisição Não Autenticada (ex: cadastro)

```
POST /usuario
      │
      ▼
JwtRequestFilter (sem token, passa adiante)
      │
      ▼
SecurityConfig (endpoint público, libera)
      │
      ▼
UsuarioController.salvarUsuario()
      │
      ▼
UsuarioService.salvarUsuario()
  ├── emailExiste() → ConflictException (409) se duplicado
  ├── passwordEncoder.encode(senha)
  └── usuarioRepository.save()
      │
      ▼
PostgreSQL (INSERT)
```

### Requisição Autenticada (ex: buscar usuário)

```
GET /usuario?email=
      │
      ▼
JwtRequestFilter
  ├── Extrai token do header Authorization
  ├── jwtUtil.validateToken()
  └── Seta autenticação no SecurityContextHolder
      │
      ▼
SecurityConfig (endpoint protegido, verifica auth)
      │
      ▼
UsuarioController.buscarUsuarioPorEmail()
      │
      ▼
UsuarioService.buscarUsuarioPorEmail()
  └── usuarioRepository.findByEmail() → ResourceNotFoundException (404) se não encontrado
      │
      ▼
PostgreSQL (SELECT)
```

## Padrões Utilizados

- **Repository Pattern** — abstração do acesso a dados via Spring Data JPA
- **DTO** — objeto UsuarioDTO para transporte de credenciais no login
- **Filter Chain** — JwtRequestFilter intercepta requisições antes dos controllers
- **Stateless Session** — sem estado de sessão no servidor; autenticação por token a cada requisição

## Stack Tecnológica

```
Spring Boot 4.0.6
  ├── Spring Framework 7
  ├── Spring Web MVC
  ├── Spring Data JPA → Hibernate → PostgreSQL
  └── Spring Security
        └── JwtRequestFilter (JJWT 0.13.0)

Java 26
Lombok 1.18.36
Maven 3.9+
```
