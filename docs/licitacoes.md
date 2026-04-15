# Licitações

Base URL: `/api/licitacoes`

**Autenticação:** Todas as rotas requerem `Bearer {token}` no header `Authorization`.

---

## GET /licitacoes

Lista todas as licitações com sua modalidade.

**Resposta de sucesso `200`:**

```json
{
  "data": [
    {
      "id": 1,
      "numero": "001/2026",
      "objeto": "Aquisição de materiais de escritório",
      "datasessao": "2026-02-01",
      "vigenciainicio": "2026-02-10",
      "vigenciafim": "2027-02-10",
      "sequencial": null,
      "modalidade_id": 1,
      "created_at": "2026-01-01T00:00:00.000000Z",
      "updated_at": "2026-01-01T00:00:00.000000Z",
      "modalidade": {
        "id": 1,
        "nome": "Pregão Eletrônico"
      }
    }
  ]
}
```

---

## GET /licitacoes/{id}

Retorna uma licitação específica com sua modalidade.

**Resposta de sucesso `200`:**

```json
{
  "data": {
    "id": 1,
    "numero": "001/2026",
    "objeto": "Aquisição de materiais de escritório",
    "datasessao": "2026-02-01",
    "vigenciainicio": "2026-02-10",
    "vigenciafim": "2027-02-10",
    "sequencial": null,
    "modalidade_id": 1,
    "modalidade": {
      "id": 1,
      "nome": "Pregão Eletrônico"
    }
  }
}
```

---

## POST /licitacoes

Cria uma nova licitação.

**Body (JSON):**

| Campo | Tipo | Obrigatório | Descrição |
|-------|------|-------------|-----------|
| `numero` | string | Sim | Número da licitação |
| `objeto` | string | Sim | Descrição do objeto |
| `datasessao` | date | Sim | Data da sessão (formato: `YYYY-MM-DD`) |
| `vigenciainicio` | date | Sim | Início da vigência (formato: `YYYY-MM-DD`) |
| `vigenciafim` | date | Sim | Fim da vigência (formato: `YYYY-MM-DD`) |
| `sequencial` | string | Não | Número sequencial |
| `modalidade_id` | integer | Sim | ID da modalidade (deve existir) |

**Resposta de sucesso `201`:**

```json
{
  "message": "Licitação criada com sucesso.",
  "data": {
    "id": 1,
    "numero": "001/2026",
    "objeto": "Aquisição de materiais de escritório",
    "datasessao": "2026-02-01",
    "vigenciainicio": "2026-02-10",
    "vigenciafim": "2027-02-10",
    "sequencial": null,
    "modalidade_id": 1,
    "modalidade": { ... },
    "created_at": "2026-01-01T00:00:00.000000Z",
    "updated_at": "2026-01-01T00:00:00.000000Z"
  }
}
```

---

## PUT/PATCH /licitacoes/{id}

Atualiza uma licitação existente.

**Body (JSON):**

| Campo | Tipo | Obrigatório | Descrição |
|-------|------|-------------|-----------|
| `numero` | string | Sim | Número da licitação |
| `objeto` | string | Sim | Descrição do objeto |
| `datasessao` | date | Sim | Data da sessão (formato: `YYYY-MM-DD`) |
| `vigenciainicio` | date | Sim | Início da vigência (formato: `YYYY-MM-DD`) |
| `vigenciafim` | date | Sim | Fim da vigência (formato: `YYYY-MM-DD`) |
| `sequencial` | string | Não | Número sequencial |
| `modalidade_id` | integer | Sim | ID da modalidade |

**Resposta de sucesso `200`:**

```json
{
  "message": "Licitação atualizada com sucesso.",
  "data": { ... }
}
```

---

## DELETE /licitacoes/{id}

Remove uma licitação.

**Resposta de sucesso `200`:**

```json
{
  "message": "Licitação removida com sucesso."
}
```
