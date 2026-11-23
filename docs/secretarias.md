# Secretarias

> Formatos comuns (paginação `?per_page=`/`?q=`, `{ data, message }`, números, erros): [padroes.md](padroes.md).

Base URL: `/api/{ente}/secretarias`

> Todas as rotas desta página ficam sob o slug da instituição: `/api/{ente}/...` (ex.: `/api/agua-branca/...`). Veja [admin.md](admin.md).

**Autenticação:** Todas as rotas requerem `Bearer {token}` no header `Authorization`.

---

## GET /secretarias

Lista paginada (ordem alfabética) com órgãos e responsáveis. `?q=` busca no nome. Resposta com `data`, `links` e `meta` ([padroes.md](padroes.md)).

**Resposta de sucesso `200`:**

```json
{
  "data": [
    {
      "id": 1,
      "nome": "Secretaria de Saúde",
      "cnpj": "00000000000191",
      "tel": "1112345678",
      "created_at": "2026-01-01T00:00:00.000000Z",
      "updated_at": "2026-01-01T00:00:00.000000Z",
      "orgaos": [
        {
          "id": 1,
          "nome": "UBS Central",
          "endereco": "Rua Exemplo, 100",
          "secretaria_id": 1,
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
  ]
}
```

---

## GET /secretarias/{id}

Retorna uma secretaria com seus órgãos e responsáveis.

**Resposta de sucesso `200`:**

```json
{
  "data": {
    "id": 1,
    "nome": "Secretaria de Saúde",
    "cnpj": "00000000000191",
    "tel": "1112345678",
    "created_at": "2026-01-01T00:00:00.000000Z",
    "updated_at": "2026-01-01T00:00:00.000000Z",
    "orgaos": [ ... ]
  }
}
```

---

## POST /secretarias

Cria uma nova secretaria.

**Body (JSON):**

| Campo | Tipo | Obrigatório | Descrição |
|-------|------|-------------|-----------|
| `nome` | string | Sim | Nome da secretaria |
| `cnpj` | string | Não | CNPJ válido, com ou sem máscara (gravado só com dígitos) |
| `tel` | string | Não | Telefone (gravado só com dígitos) |

**Resposta de sucesso `201`:**

```json
{
  "message": "Secretaria criada com sucesso.",
  "data": {
    "id": 1,
    "nome": "Secretaria de Saúde",
    "cnpj": "00000000000191",
    "tel": "1112345678",
    "created_at": "2026-01-01T00:00:00.000000Z",
    "updated_at": "2026-01-01T00:00:00.000000Z"
  }
}
```

---

## PUT/PATCH /secretarias/{id}

Atualiza uma secretaria existente.

**Body (JSON):**

| Campo | Tipo | Obrigatório | Descrição |
|-------|------|-------------|-----------|
| `nome` | string | Sim | Nome da secretaria |
| `cnpj` | string | Não | CNPJ válido, com ou sem máscara (gravado só com dígitos) |
| `tel` | string | Não | Telefone (gravado só com dígitos) |

**Resposta de sucesso `200`:**

```json
{
  "message": "Secretaria atualizada com sucesso.",
  "data": { ... }
}
```

---

## DELETE /secretarias/{id}

Remove uma secretaria.

> Registro em uso por outros cadastros não é excluído: a API responde `409` com `message`.

**Resposta de sucesso `200`:**

```json
{
  "message": "Secretaria removida com sucesso."
}
```
