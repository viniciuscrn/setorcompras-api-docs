# Responsáveis

> Formatos comuns (paginação `?per_page=`/`?q=`, `{ data, message }`, números, erros): [padroes.md](padroes.md).

Base URL: `/api/{ente}/responsaveis`

> Todas as rotas desta página ficam sob o slug da instituição: `/api/{ente}/...` (ex.: `/api/agua-branca/...`). Veja [admin.md](admin.md).

**Autenticação:** Todas as rotas requerem `Bearer {token}` no header `Authorization`.

---

## GET /responsaveis

Lista paginada (ordem alfabética) com o órgão. `?q=` busca em nome e cargo. O detalhe traz órgão e secretaria.

**Resposta de sucesso `200`:**

```json
{
  "data": [
    {
      "id": 1,
      "nome": "João Silva",
      "cargo": "Diretor",
      "orgao_id": 1,
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
    "orgao_id": 1,
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
| `orgao_id` | integer | Sim | ID do órgão (deve existir) |

**Resposta de sucesso `201`:**

```json
{
  "message": "Responsável criado com sucesso.",
  "data": {
    "id": 1,
    "nome": "João Silva",
    "cargo": "Diretor",
    "orgao_id": 1,
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
| `orgao_id` | integer | Sim | ID do órgão |

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

> Registro em uso por outros cadastros não é excluído: a API responde `409` com `message`.

**Resposta de sucesso `200`:**

```json
{
  "message": "Responsável removido com sucesso."
}
```
