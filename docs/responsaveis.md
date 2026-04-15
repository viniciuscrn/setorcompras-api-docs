# Responsáveis

Base URL: `/api/responsaveis`

**Autenticação:** Todas as rotas requerem `Bearer {token}` no header `Authorization`.

---

## GET /responsaveis

Lista todos os responsáveis com seu órgão vinculado.

**Resposta de sucesso `200`:**

```json
{
  "data": [
    {
      "id": 1,
      "nome": "João Silva",
      "cargo": "Diretor",
      "orgaos_id": 1,
      "created_at": "2026-01-01T00:00:00.000000Z",
      "updated_at": "2026-01-01T00:00:00.000000Z",
      "orgao": {
        "id": 1,
        "nome": "UBS Central",
        "endereco": "Rua Exemplo, 100",
        "secretaria_id": 1
      }
    }
  ]
}
```

---

## GET /responsaveis/{id}

Retorna um responsável com seu órgão vinculado.

**Resposta de sucesso `200`:**

```json
{
  "data": {
    "id": 1,
    "nome": "João Silva",
    "cargo": "Diretor",
    "orgaos_id": 1,
    "orgao": { ... }
  }
}
```

---

## POST /responsaveis

Cria um novo responsável.

**Body (JSON):**

| Campo | Tipo | Obrigatório | Descrição |
|-------|------|-------------|-----------|
| `nome` | string | Sim | Nome do responsável |
| `cargo` | string | Sim | Cargo |
| `orgaos_id` | integer | Sim | ID do órgão (deve existir) |

**Resposta de sucesso `201`:**

```json
{
  "message": "Responsável criado com sucesso.",
  "data": {
    "id": 1,
    "nome": "João Silva",
    "cargo": "Diretor",
    "orgaos_id": 1,
    "orgao": { ... },
    "created_at": "2026-01-01T00:00:00.000000Z",
    "updated_at": "2026-01-01T00:00:00.000000Z"
  }
}
```

---

## PUT/PATCH /responsaveis/{id}

Atualiza um responsável existente.

**Body (JSON):**

| Campo | Tipo | Obrigatório | Descrição |
|-------|------|-------------|-----------|
| `nome` | string | Sim | Nome do responsável |
| `cargo` | string | Sim | Cargo |
| `orgaos_id` | integer | Sim | ID do órgão |

**Resposta de sucesso `200`:**

```json
{
  "message": "Responsável atualizado com sucesso.",
  "data": { ... }
}
```

---

## DELETE /responsaveis/{id}

Remove um responsável.

**Resposta de sucesso `200`:**

```json
{
  "message": "Responsável removido com sucesso."
}
```
