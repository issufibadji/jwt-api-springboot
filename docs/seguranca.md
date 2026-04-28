# Segurança e JWT

## Visão Geral

A segurança da aplicação é implementada com **Spring Security** em modo stateless, utilizando **JWT (JSON Web Token)** para autenticação. Não há sessão no servidor — cada requisição deve carregar o token no cabeçalho HTTP.

## Fluxo de Autenticação

```
1. Cliente envia credenciais
   POST /usuario/login { "email": "...", "senha": "..." }
         │
         ▼
2. UsuarioController valida credenciais via AuthenticationManager
         │
         ▼
3. UserDetailsServiceImpl carrega o usuário do banco pelo e-mail
         │
         ▼
4. Spring Security verifica a senha (BCrypt)
         │
         ▼
5. JwtUtil.generateToken(email) → token HS256 válido por 1 hora
         │
         ▼
6. Token retornado ao cliente
         │
7. Cliente usa token em todas as próximas requisições
   Header: Authorization: Bearer <token>
         │
         ▼
8. JwtRequestFilter valida o token e seta a autenticação
```

## Componentes de Segurança

### JwtUtil

Serviço responsável por toda a manipulação de tokens JWT.

| Método | Descrição |
|---|---|
| `generateToken(username)` | Gera token HS256 com expiração de 1 hora |
| `extractUsername(token)` | Extrai o e-mail do subject do token |
| `extractClaims(token)` | Retorna todos os claims do token |
| `isTokenExpired(token)` | Verifica se o token está expirado |
| `validateToken(token, username)` | Valida token contra o usuário esperado |

**Configuração do token:**

```
Algoritmo:  HS256 (HMAC-SHA256)
Expiração:  3600000 ms (1 hora)
Subject:    e-mail do usuário
```

> A chave secreta deve ser externalizada para variável de ambiente em produção.

### JwtRequestFilter

Filtro que intercepta todas as requisições HTTP antes dos controllers.

**Lógica de processamento:**

```
Requisição recebida
      │
      ▼
Header "Authorization" presente e começa com "Bearer "?
      │
   Sim│                              Não│
      ▼                                ▼
Extrai token                     Passa adiante (sem auth)
      │
      ▼
jwtUtil.extractUsername(token)
      │
      ▼
SecurityContextHolder já tem autenticação?
      │
   Não│
      ▼
userDetailsService.loadUserByUsername(email)
      │
      ▼
jwtUtil.validateToken(token, username)?
      │
   Sim│
      ▼
Cria UsernamePasswordAuthenticationToken
e seta no SecurityContextHolder
      │
      ▼
Passa para o próximo filtro/controller
```

### SecurityConfig

Configura o comportamento global do Spring Security.

**Endpoints públicos (sem autenticação):**

| Método | Endpoint | Motivo |
|---|---|---|
| POST | `/usuario` | Cadastro de novos usuários |
| POST | `/usuario/login` | Login para obter o token |
| GET | `/auth` | Endpoint auxiliar de verificação |

**Todos os demais endpoints exigem JWT válido.**

**Configurações:**

- `SessionCreationPolicy.STATELESS` — sem sessão no servidor
- CSRF desabilitado — API REST sem estado não precisa de proteção CSRF
- `BCryptPasswordEncoder` — hash de senhas com fator de custo padrão (10)

### UserDetailsServiceImpl

Implementação de `UserDetailsService` para integração com Spring Security.

- Carrega o usuário do banco via `UsuarioRepository.findByEmail()`
- Retorna um objeto `UserDetails` com e-mail e senha encoded
- A entidade `Usuario` implementa `UserDetails` diretamente, retornando lista de autoridades vazia (sem roles configuradas)

## Boas Práticas a Adotar em Produção

| Item | Situação Atual | Recomendação |
|---|---|---|
| Chave JWT | Hardcoded no código | Externalizar para `JWT_SECRET` em variável de ambiente |
| Expiração | 1 hora | Considerar refresh token para sessões longas |
| Roles | Sem controle de roles | Adicionar `GrantedAuthority` para controle de acesso fino |
| HTTPS | Não configurado | Obrigatório em produção (Spring Boot SSL ou reverse proxy) |
| Logs de segurança | Ausentes | Adicionar logging de tentativas de login inválidas |
