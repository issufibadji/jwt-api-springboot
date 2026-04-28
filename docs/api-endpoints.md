# API Endpoints

**Base URL:** `http://localhost:8081`

**Content-Type:** `application/json`

---

## Autenticação

Endpoints protegidos exigem o header:

```
Authorization: Bearer <jwt-token>
```

---

## Usuários

### POST /usuario — Cadastrar Usuário

Cria um novo usuário. Endpoint público.

**Request Body:**

```json
{
  "nome": "João Silva",
  "email": "joao@email.com",
  "senha": "senha123",
  "enderecos": [
    {
      "rua": "Rua das Flores",
      "numero": 123,
      "complemento": "Ap 4",
      "cidade": "São Paulo",
      "estado": "SP",
      "cep": "01234-567"
    }
  ],
  "telefones": [
    {
      "ddd": "11",
      "numero": "987654321"
    }
  ]
}
```

**Respostas:**

| Status | Descrição |
|---|---|
| 200 OK | Usuário criado — retorna o objeto `Usuario` |
| 409 Conflict | E-mail já cadastrado |

---

### POST /usuario/login — Login

Autentica o usuário e retorna o token JWT. Endpoint público.

**Request Body:**

```json
{
  "email": "joao@email.com",
  "senha": "senha123"
}
```

**Resposta 200 OK:**

```
eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJqb2FvQGVtYWlsLmNvbSIsImlhdC...
```

> A resposta é o token JWT em formato texto puro (String).

**Respostas:**

| Status | Descrição |
|---|---|
| 200 OK | Token JWT gerado |
| 403 Forbidden | Credenciais inválidas |

---

### GET /usuario?email={email} — Buscar Usuário

Retorna os dados completos de um usuário pelo e-mail. Requer autenticação.

**Query Params:**

| Parâmetro | Tipo | Obrigatório |
|---|---|---|
| email | String | Sim |

**Exemplo:**

```
GET /usuario?email=joao@email.com
Authorization: Bearer <token>
```

**Resposta 200 OK:**

```json
{
  "id": 1,
  "nome": "João Silva",
  "email": "joao@email.com",
  "senha": "$2a$10$...",
  "enderecos": [...],
  "telefones": [...]
}
```

**Respostas:**

| Status | Descrição |
|---|---|
| 200 OK | Dados do usuário |
| 404 Not Found | Usuário não encontrado |
| 403 Forbidden | Token ausente ou inválido |

---

### DELETE /usuario/{email} — Deletar Usuário

Remove um usuário pelo e-mail. Requer autenticação.

**Path Params:**

| Parâmetro | Tipo | Obrigatório |
|---|---|---|
| email | String | Sim |

**Exemplo:**

```
DELETE /usuario/joao@email.com
Authorization: Bearer <token>
```

**Respostas:**

| Status | Descrição |
|---|---|
| 200 OK | Usuário deletado |
| 403 Forbidden | Token ausente ou inválido |

---

## Auth

### GET /auth?email={email} — Detalhes de Autenticação

Retorna os dados do usuário para fins de autenticação. Endpoint público.

**Query Params:**

| Parâmetro | Tipo | Obrigatório |
|---|---|---|
| email | String | Sim |

**Exemplo:**

```
GET /auth?email=joao@email.com
```

**Resposta 200 OK:** objeto `Usuario` completo.

---

## Códigos de Erro

| Status | Exceção | Situação |
|---|---|---|
| 409 Conflict | ConflictException | E-mail já cadastrado |
| 404 Not Found | ResourceNotFoundException | Usuário não encontrado |
| 403 Forbidden | — | Token ausente, inválido ou expirado |
