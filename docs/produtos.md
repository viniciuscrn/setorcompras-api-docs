# Produtos

Base URL: `/api/produtos`

**Autenticação:** Todas as rotas requerem `Bearer {token}` no header `Authorization`.

---

## GET /produtos

Lista todos os produtos cadastrados.

**Resposta de sucesso `200`:**

```json
{
  "data": [
    {
      "id": 1,
      "descricao": "Notebook i5 16GB SSD 512GB",
      "unidade": "UN",
      "quant": "10.00",
      "preco": "4200.00",
      "marca": "Dell",
      "numero_item": 1,
      "created_at": "2026-01-01T00:00:00.000000Z",
      "updated_at": "2026-01-01T00:00:00.000000Z"
    }
  ]
}
```

---

## GET /produtos/{id}

Retorna um produto específico.

**Resposta de sucesso `200`:**

```json
{
  "data": {
    "id": 1,
    "descricao": "Notebook i5 16GB SSD 512GB",
    "unidade": "UN",
    "quant": "10.00",
    "preco": "4200.00",
    "marca": "Dell",
    "numero_item": 1,
    "created_at": "2026-01-01T00:00:00.000000Z",
    "updated_at": "2026-01-01T00:00:00.000000Z"
  }
}
```

---

## POST /produtos

Cadastra um novo produto.

**Body (JSON):**

| Campo | Tipo | Obrigatório | Descrição |
|-------|------|-------------|-----------|
| `descricao` | string | Sim | Descrição do produto |
| `unidade` | string | Sim | Unidade de medida (ex: `UN`, `KG`, `CX`) |
| `quant` | numeric | Sim | Quantidade |
| `preco` | numeric | Sim | Preço unitário |
| `marca` | string | Não | Marca do produto |
| `numero_item` | integer | Não | Número do item no processo licitatório |

**Resposta de sucesso `201`:**

```json
{
  "message": "Produto criado com sucesso.",
  "data": {
    "id": 1,
    "descricao": "Notebook i5 16GB SSD 512GB",
    "unidade": "UN",
    "quant": "10.00",
    "preco": "4200.00",
    "marca": "Dell",
    "numero_item": 1,
    "created_at": "2026-01-01T00:00:00.000000Z",
    "updated_at": "2026-01-01T00:00:00.000000Z"
  }
}
```

---

## PUT/PATCH /produtos/{id}

Atualiza um produto existente.

**Body (JSON):**

| Campo | Tipo | Obrigatório | Descrição |
|-------|------|-------------|-----------|
| `descricao` | string | Sim | Descrição do produto |
| `unidade` | string | Sim | Unidade de medida |
| `quant` | numeric | Sim | Quantidade |
| `preco` | numeric | Sim | Preço unitário |
| `marca` | string | Não | Marca |
| `numero_item` | integer | Não | Número do item |

**Resposta de sucesso `200`:**

```json
{
  "message": "Produto atualizado com sucesso.",
  "data": { ... }
}
```

---

## DELETE /produtos/{id}

Remove um produto.

**Resposta de sucesso `200`:**

```json
{
  "message": "Produto removido com sucesso."
}
```
