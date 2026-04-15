# Usuários

Base URL: `/api/users`

**Autenticação:** Todas as rotas requerem `Bearer {token}` no header `Authorization`.

---

## GET /users

Lista todos os usuários paginados.

**Resposta de sucesso `200`:**

```json
{
  "current_page": 1,
  "data": [
    {
      "id": 1,
      "name": "Nome do Usuário",
      "email": "usuario@email.com",
      "role": "admin",
      "created_at": "2026-01-01T00:00:00.000000Z",
      "updated_at": "2026-01-01T00:00:00.000000Z"
    }
  ],
  "per_page": 10,
  "total": 1
}
```

---

## GET /users/{id}

Retorna um usuário específico.

**Resposta de sucesso `200`:**

```json
{
  "id": 1,
  "name": "Nome do Usuário",
  "email": "usuario@email.com",
  "role": "admin",
  "created_at": "2026-01-01T00:00:00.000000Z",
  "updated_at": "2026-01-01T00:00:00.000000Z"
}
```

---

## POST /users

Cria um novo usuário.

**Body (JSON):**

| Campo | Tipo | Obrigatório | Descrição |
|-------|------|-------------|-----------|
| `name` | string | Sim | Nome completo |
| `email` | string | Sim | E-mail único |
| `password` | string | Sim | Mínimo 6 caracteres |
| `password_confirmation` | string | Sim | Confirmação da senha |
| `role` | string | Sim | Perfil do usuário (deve existir na tabela `roles`) |

**Resposta de sucesso `201`:**

```json
{
  "id": 2,
  "name": "Novo Usuário",
  "email": "novo@email.com",
  "role": "editor",
  "created_at": "2026-01-01T00:00:00.000000Z",
  "updated_at": "2026-01-01T00:00:00.000000Z"
}
```

---

## PUT/PATCH /users/{id}

Atualiza dados e/ou perfil de um usuário. Todos os campos são opcionais.

**Body (JSON):**

| Campo | Tipo | Obrigatório | Descrição |
|-------|------|-------------|-----------|
| `name` | string | Não | Nome completo |
| `email` | string | Não | E-mail único |
| `password` | string | Não | Nova senha (mínimo 6 caracteres) |
| `password_confirmation` | string | Não | Confirmação da nova senha |
| `role` | string | Não | Novo perfil do usuário |

**Resposta de sucesso `200`:**

```json
{
  "id": 1,
  "name": "Nome Atualizado",
  "email": "usuario@email.com",
  "role": "admin",
  "created_at": "2026-01-01T00:00:00.000000Z",
  "updated_at": "2026-01-01T00:00:00.000000Z"
}
```

---

## DELETE /users/{id}

Remove um usuário. Não é possível remover a própria conta.

**Resposta de sucesso `204`:** Sem conteúdo.

**Resposta de erro `403`:**

```json
{
  "message": "Você não pode excluir sua própria conta."
}
```
