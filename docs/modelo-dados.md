# Modelo de Dados

## Diagrama de Entidades

```
┌─────────────────────────────┐
│           USUARIO           │
├─────────────────────────────┤
│ id         BIGSERIAL (PK)   │
│ nome       VARCHAR(100)     │
│ email      VARCHAR(100) UK  │
│ senha      VARCHAR          │
└──────────┬──────────────────┘
           │ 1
           │
    ┌──────┴────────┐
    │               │
    │ N             │ N
┌───▼──────────┐  ┌─▼────────────┐
│   ENDERECO   │  │   TELEFONE   │
├──────────────┤  ├──────────────┤
│ id  BIGSERIAL│  │ id  BIGSERIAL│
│ rua VARCHAR  │  │ ddd VARCHAR(3│
│ numero BIGINT│  │ numero VARCHAR│
│ complemento  │  │  (10)        │
│  VARCHAR(10) │  └──────────────┘
│ cidade       │
│  VARCHAR(150)│
│ estado       │
│  VARCHAR(2)  │
│ cep          │
│  VARCHAR(9)  │
└──────────────┘
```

## Entidades

### Usuario

Entidade principal. Implementa `UserDetails` do Spring Security para integração direta com o mecanismo de autenticação.

| Coluna | Tipo JPA | Restrições | Descrição |
|---|---|---|---|
| id | Long | PK, auto-generated | Identificador único |
| nome | String | max 100 | Nome completo |
| email | String | max 100, unique | Identificador de login |
| senha | String | — | Senha com hash BCrypt |
| enderecos | List\<Endereco\> | OneToMany, cascade ALL | Endereços associados |
| telefones | List\<Telefone\> | OneToMany, cascade ALL | Telefones associados |

**Cascade ALL** significa que ao salvar, atualizar ou deletar um `Usuario`, as operações se propagam automaticamente para os seus `Endereco` e `Telefone`.

**Implementação UserDetails:**

| Método | Retorno |
|---|---|
| `getUsername()` | e-mail do usuário |
| `getPassword()` | senha encoded |
| `getAuthorities()` | lista vazia (sem roles) |

---

### Endereco

Entidade de endereço associada ao usuário.

| Coluna | Tipo JPA | Restrições | Descrição |
|---|---|---|---|
| id | Long | PK, auto-generated | Identificador único |
| rua | String | — | Nome da rua |
| numero | Long | — | Número do imóvel |
| complemento | String | max 10 | Complemento (ap, sala etc.) |
| cidade | String | max 150 | Cidade |
| estado | String | max 2 | UF (ex: SP, RJ) |
| cep | String | max 9 | CEP (ex: 01234-567) |

---

### Telefone

Entidade de telefone associada ao usuário.

| Coluna | Tipo JPA | Restrições | Descrição |
|---|---|---|---|
| id | Long | PK, auto-generated | Identificador único |
| numero | String | max 10 | Número sem DDD |
| ddd | String | max 3 | Código de área (DDD) |

---

## Relacionamentos

```
Usuario  1──N  Endereco   (cascade ALL, orphanRemoval implícito)
Usuario  1──N  Telefone   (cascade ALL, orphanRemoval implícito)
```

Um usuário pode ter múltiplos endereços e múltiplos telefones. Os filhos são gerenciados pela entidade pai — não há repositório dedicado sendo chamado diretamente para persistir endereços ou telefones; basta salvar o `Usuario`.

---

## Configuração JPA

```properties
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.PostgreSQLDialect
```

- `ddl-auto=update` — Hibernate atualiza o schema automaticamente ao iniciar a aplicação. **Não usar em produção** (preferir Flyway ou Liquibase).
- `show-sql=true` — Exibe as queries SQL no console. Útil para desenvolvimento.

---

## Banco de Dados

| Item | Valor |
|---|---|
| SGBD | PostgreSQL 15+ |
| Host | localhost |
| Porta | 5432 |
| Database | db\_usuario |
| Schema | public (padrão) |
