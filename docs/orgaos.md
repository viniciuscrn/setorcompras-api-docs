# Órgãos

> Formatos comuns (paginação `?per_page=`/`?q=`, `{ data, message }`, números, erros): [padroes.md](padroes.md).

Base URL: `/api/{ente}/orgaos`

> Todas as rotas desta página ficam sob o slug da instituição: `/api/{ente}/...` (ex.: `/api/agua-branca/...`). Veja [admin.md](admin.md).

**Autenticação:** Todas as rotas requerem `Bearer {token}` no header `Authorization`.

---

## GET /orgaos

Lista paginada (ordem alfabética) com a secretaria. `?q=` busca no nome do órgão e da secretaria. O detalhe (`GET /orgaos/{id}`) traz também os responsáveis.

**Resposta de sucesso `200`:**

```json
{
  "data": [
    {
      "id": 1,
      "nome": "UBS Central",
      "endereco": "Rua Exemplo, 100",
      "secretaria_id": 1,
      "created_at": "2026-01-01T00:00:00.000000Z",
      "updated_at": "2026-01-01T00:00:00.000000Z",
      "secretaria": {
        "id": 1,
        "nome": "Secretaria de Saúde",
        "cnpj": "00000000000191",
        "tel": "1112345678"
      },
      "responsaveis": [
        {
          "id": 1,
          "nome": "João Silva",
          "cargo": "Diretor",
          "orgao_id": 1
        }
      ]
    }
  ]
}
```

---

## GET /orgaos/{id}

Retorna um órgão com sua secretaria e responsáveis.

**Resposta de sucesso `200`:**

```json
{
  "data": {
    "id": 1,
    "nome": "UBS Central",
    "endereco": "Rua Exemplo, 100",
    "secretaria_id": 1,
    "secretaria": { ... },
    "responsaveis": [ ... ]
  }
}
```

---

## POST /orgaos

Cria um novo órgão.

**Body (JSON):**

| Campo | Tipo | Obrigatório | Descrição |
|-------|------|-------------|-----------|
| `nome` | string | Sim | Nome do órgão |
| `endereco` | string | Sim | Endereço |
| `secretaria_id` | integer | Sim | ID da secretaria (deve existir) |

**Resposta de sucesso `201`:**

```json
{
  "message": "Órgão criado com sucesso.",
  "data": {
    "id": 1,
    "nome": "UBS Central",
    "endereco": "Rua Exemplo, 100",
    "secretaria_id": 1,
    "secretaria": { ... },
    "created_at": "2026-01-01T00:00:00.000000Z",
    "updated_at": "2026-01-01T00:00:00.000000Z"
  }
}
```

---

## PUT/PATCH /orgaos/{id}

Atualiza um órgão existente.

**Body (JSON):**

| Campo | Tipo | Obrigatório | Descrição |
|-------|------|-------------|-----------|
| `nome` | string | Sim | Nome do órgão |
| `endereco` | string | Sim | Endereço |
| `secretaria_id` | integer | Sim | ID da secretaria |

**Resposta de sucesso `200`:**

```json
{
  "message": "Órgão atualizado com sucesso.",
  "data": { ... }
}
```

---

## DELETE /orgaos/{id}

Remove um órgão.

> Registro em uso por outros cadastros não é excluído: a API responde `409` com `message`.

**Resposta de sucesso `200`:**

```json
{
  "message": "Órgão removido com sucesso."
}
```
