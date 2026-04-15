# Modalidades

Base URL: `/api/modalidades`

**Autenticação:** Todas as rotas requerem `Bearer {token}` no header `Authorization`.

---

## GET /modalidades

Lista todas as modalidades.

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

**Resposta de sucesso `200`:**

```json
{
  "message": "Modalidade removida com sucesso."
}
```
