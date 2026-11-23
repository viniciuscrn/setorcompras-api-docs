# Modalidades

> Formatos comuns (paginação `?per_page=`/`?q=`, `{ data, message }`, números, erros): [padroes.md](padroes.md).

Base URL: `/api/{ente}/modalidades`

> Todas as rotas desta página ficam sob o slug da instituição: `/api/{ente}/...` (ex.: `/api/agua-branca/...`). Veja [admin.md](admin.md).

**Autenticação:** Todas as rotas requerem `Bearer {token}` no header `Authorization`.

---

## GET /modalidades

Lista paginada (ordem alfabética). `?q=` busca no nome. O nome é único (`422` se repetido).

**Resposta de sucesso `200`:**

```json
{
  "data": [
    {
      "id": 1,
      "nome": "Pregão Eletrônico",
      "created_at": "2026-01-01T00:00:00.000000Z",
      "updated_at": "2026-01-01T00:00:00.000000Z"
    }
  ]
}
```

---

## GET /modalidades/{id}

Retorna uma modalidade específica.

**Resposta de sucesso `200`:**

```json
{
  "data": {
    "id": 1,
    "nome": "Pregão Eletrônico",
    "created_at": "2026-01-01T00:00:00.000000Z",
    "updated_at": "2026-01-01T00:00:00.000000Z"
  }
}
```

---

## POST /modalidades

Cria uma nova modalidade.

**Body (JSON):**

| Campo | Tipo | Obrigatório | Descrição |
|-------|------|-------------|-----------|
| `nome` | string | Sim | Nome da modalidade |

**Resposta de sucesso `201`:**

```json
{
  "message": "Modalidade criada com sucesso.",
  "data": {
    "id": 1,
    "nome": "Pregão Eletrônico",
    "created_at": "2026-01-01T00:00:00.000000Z",
    "updated_at": "2026-01-01T00:00:00.000000Z"
  }
}
```

---

## PUT/PATCH /modalidades/{id}

Atualiza uma modalidade existente.

**Body (JSON):**

| Campo | Tipo | Obrigatório | Descrição |
|-------|------|-------------|-----------|
| `nome` | string | Sim | Nome da modalidade |

**Resposta de sucesso `200`:**

```json
{
  "message": "Modalidade atualizada com sucesso.",
  "data": { ... }
}
```

---

## DELETE /modalidades/{id}

Remove uma modalidade.

> Registro em uso por outros cadastros não é excluído: a API responde `409` com `message`.

**Resposta de sucesso `200`:**

```json
{
  "message": "Modalidade removida com sucesso."
}
```
