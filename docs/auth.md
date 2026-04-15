# Auth

Base URL: `/api`

---

## POST /login

Autentica o usuário e retorna o token de acesso.

**Autenticação:** Não requerida

**Body (JSON):**

| Campo | Tipo | Obrigatório | Descrição |
|-------|------|-------------|-----------|
| `email` | string | Sim | E-mail do usuário |
| `password` | string | Sim | Senha do usuário |
| `device_name` | string | Não | Nome do dispositivo (ex: "Web", "iPhone") |

**Resposta de sucesso `200`:**

```json
{
  "token": "1|abc123...",
  "user": {
    "id": 1,
    "name": "Nome do Usuário",
    "email": "usuario@email.com",
    "role": "admin"
  }
}
```

**Resposta de erro `422`:**

```json
{
  "message": "As credenciais fornecidas estão incorretas.",
  "errors": {
    "email": ["As credenciais fornecidas estão incorretas."]
  }
}
```

---

## POST /logout

Invalida o token atual do usuário autenticado.

**Autenticação:** Requerida — `Bearer {token}`

**Resposta de sucesso `200`:**

```json
{
  "message": "Logout realizado com sucesso"
}
```

---

## GET /me

Retorna os dados do usuário autenticado com base no token.

**Autenticação:** Requerida — `Bearer {token}`

**Resposta de sucesso `200`:**

```json
{
  "id": 1,
  "name": "Nome do Usuário",
  "email": "usuario@email.com",
  "created_at": "2026-01-01T00:00:00.000000Z",
  "updated_at": "2026-01-01T00:00:00.000000Z"
}
```
