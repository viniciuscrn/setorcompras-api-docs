# Aditivos

Base URL: `/api/aditivos`

**Autenticação:** Todas as rotas requerem `Bearer {token}` no header `Authorization`.

> Um aditivo representa uma alteração contratual de preço e/ou quantidade em um item específico de um contrato (`licitacoes_produtos`). Podem existir múltiplos aditivos para o mesmo item.

---

## GET /aditivos?licitacao_produto_id={id}

Lista todos os aditivos de um item contratual. O parâmetro `licitacao_produto_id` é **obrigatório**.

**Query string:**

| Parâmetro | Tipo | Obrigatório | Descrição |
|-----------|------|-------------|-----------|
| `licitacao_produto_id` | integer | Sim | ID do registro em `licitacoes_produtos` |

**Exemplo:** `GET /api/aditivos?licitacao_produto_id=3`

**Resposta de sucesso `200`:**

```json
{
  "data": [
    {
      "id": 1,
      "tipo": "quantidade",
      "quant": "2.00",
      "preco": null,
      "licitacao_produto_id": 3,
      "created_at": "2026-02-01T00:00:00.000000Z",
      "updated_at": "2026-02-01T00:00:00.000000Z",
      "licitacao_produto": {
        "id": 3,
        "produto": {
          "id": 1,
          "descricao": "Notebook i5 16GB SSD 512GB",
          "quant": "10.00",
          "preco": "4200.00"
        }
      }
    }
  ],
  "resumo": {
    "quant_original": 10,
    "quant_total_aditivada": 2,
    "percentual_aditivado": 20,
    "limite_recomendado": "25%",
    "limite_atingido": false,
    "preco_original": 4200,
    "preco_atual": 4200
  }
}
```

> O campo `resumo` consolida o estado atual do item: total aditivado de quantidade, percentual atingido e o preço vigente (último aditivo de preço, ou o original caso não haja).

---

## GET /aditivos/{id}

Retorna um aditivo específico.

**Resposta de sucesso `200`:**

```json
{
  "data": {
    "id": 1,
    "tipo": "ambos",
    "quant": "3.00",
    "preco": "4500.00",
    "licitacao_produto_id": 3,
    "licitacao_produto": {
      "id": 3,
      "produto": { ... }
    }
  }
}
```

---

## POST /aditivos

Cria um novo aditivo para um item contratual.

**Body (JSON):**

| Campo | Tipo | Obrigatório | Descrição |
|-------|------|-------------|-----------|
| `licitacao_produto_id` | integer | Sim | ID do item em `licitacoes_produtos` |
| `tipo` | string | Sim | Tipo do aditivo: `preco`, `quantidade` ou `ambos` |
| `quant` | numeric | Condicional | Obrigatório quando `tipo` for `quantidade` ou `ambos` |
| `preco` | numeric | Condicional | Obrigatório quando `tipo` for `preco` ou `ambos` |

**Resposta de sucesso `201` — sem aviso:**

```json
{
  "message": "Aditivo criado com sucesso.",
  "data": {
    "id": 2,
    "tipo": "quantidade",
    "quant": "2.00",
    "preco": null,
    "licitacao_produto_id": 3
  }
}
```

**Resposta de sucesso `201` — com aviso de limite de quantidade:**

```json
{
  "message": "Aditivo criado com sucesso.",
  "aviso": "A quantidade total aditivada (3) representa 30% da quantidade original (10), ultrapassando o limite recomendado de 25%.",
  "data": { ... }
}
```

> O campo `aviso` aparece apenas quando a soma acumulada de aditivos de quantidade ultrapassa 25% da quantidade original. O registro é salvo normalmente — é apenas um alerta informativo.

---

## PUT/PATCH /aditivos/{id}

Atualiza um aditivo existente. O `licitacao_produto_id` não pode ser alterado via update.

**Body (JSON):**

| Campo | Tipo | Obrigatório | Descrição |
|-------|------|-------------|-----------|
| `tipo` | string | Sim | `preco`, `quantidade` ou `ambos` |
| `quant` | numeric | Condicional | Obrigatório quando `tipo` for `quantidade` ou `ambos` |
| `preco` | numeric | Condicional | Obrigatório quando `tipo` for `preco` ou `ambos` |

**Resposta de sucesso `200`:**

```json
{
  "message": "Aditivo atualizado com sucesso.",
  "data": { ... }
}
```

---

## DELETE /aditivos/{id}

Remove um aditivo.

**Resposta de sucesso `200`:**

```json
{
  "message": "Aditivo removido com sucesso."
}
```

---

## Regras de negócio

**Tipos de aditivo:**

| `tipo` | `quant` | `preco` |
|--------|---------|---------|
| `quantidade` | Obrigatório | Ignorado |
| `preco` | Ignorado | Obrigatório |
| `ambos` | Obrigatório | Obrigatório |

**Limite de quantidade (25%):**

- A soma de todos os aditivos de quantidade (tipo `quantidade` ou `ambos`) de um mesmo item não deve ultrapassar 25% da quantidade original do produto.
- Caso ultrapasse, a API retorna o campo `aviso` na resposta, mas **o aditivo é salvo normalmente**.
- O campo `resumo.limite_atingido` no endpoint de listagem indica se o limite já foi ultrapassado.

**Preço:**

- Não há limite para aditivos de preço.
- O `resumo.preco_atual` sempre reflete o preço do aditivo mais recente. Caso não haja aditivo de preço, retorna o preço original do produto.
