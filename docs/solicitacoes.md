# Solicitações de Compra

Base URL: `/api/solicitacoes`

**Autenticação:** Todas as rotas requerem `Bearer {token}` no header `Authorization`.

> Uma solicitação representa um pedido de compra, podendo ser de produtos **licitados** (`ehlicitada: true`) — vinculados a contratos de licitação — ou **não licitados** (`ehlicitada: false`) — compras diretas com a empresa.

---

## GET /solicitacoes

Lista todas as solicitações. Aceita filtros opcionais via query string.

| Parâmetro | Tipo | Descrição |
|-----------|------|-----------|
| `empresa_id` | integer | Filtra por empresa |
| `responsavel_id` | integer | Filtra por responsável |
| `ehlicitada` | boolean | `1` = licitadas, `0` = compras diretas |
| `ano` | integer | Filtra pelo ano da data da solicitação |

**Exemplo:** `GET /api/solicitacoes?ehlicitada=0&ano=2026`

**Resposta de sucesso `200`:**

```json
{
  "data": [
    {
      "id": 1,
      "guia": "1JX26",
      "protocolo": "202604041525148K3Z",
      "data": "2026-03-10",
      "descricao": "Aquisição de materiais de informática",
      "obs": null,
      "ehlicitada": false,
      "empresas_id": 1,
      "responsaveis_id": 2,
      "users_id": 1,
      "empresa": { "id": 1, "razaosocial": "Empresa Exemplo Ltda", ... },
      "responsavel": { "id": 2, "nome": "João Silva", "cargo": "Gestor" },
      "user": { "id": 1, "name": "Admin" }
    }
  ]
}
```

---

## GET /solicitacoes/{id}

Retorna uma solicitação completa com seus itens e valor total calculado.

**Resposta de sucesso `200`:**

```json
{
  "data": {
    "id": 1,
    "guia": "1JX26",
    "protocolo": "202604041525148K3Z",
    "data": "2026-03-10",
    "descricao": "Aquisição de materiais de informática",
    "obs": null,
    "ehlicitada": false,
    "empresas_id": 1,
    "responsaveis_id": 2,
    "users_id": 1,
    "empresa": { ... },
    "responsavel": { ... },
    "user": { ... },
    "produtos": [
      {
        "id": 1,
        "solicitacao_id": 1,
        "produto_id": 3,
        "quant": "2.00",
        "preco_unitario": "4200.00",
        "aditivo_id": null,
        "produto": { "id": 3, "descricao": "Notebook i5...", ... },
        "aditivo": null
      }
    ],
    "valor_total": 8400.00
  }
}
```

---

## POST /solicitacoes

Cria uma nova solicitação (sem itens — os produtos são adicionados separadamente).

> `guia` e `protocolo` são gerados automaticamente pelo servidor — **não devem ser enviados** no body.

**Body (JSON):**

| Campo | Tipo | Obrigatório | Descrição |
|-------|------|-------------|-----------|
| `data` | date | Sim | Data da solicitação (`YYYY-MM-DD`) |
| `descricao` | string | Sim | Descrição da solicitação |
| `obs` | string | Não | Observações |
| `ehlicitada` | boolean | Sim | `true` = produto licitado, `false` = compra direta |
| `empresas_id` | integer | Sim | ID da empresa fornecedora |
| `responsaveis_id` | integer | Sim | ID do responsável que solicitou |
| `users_id` | integer | Sim | ID do usuário que registrou |

**Resposta de sucesso `201`:**

```json
{
  "message": "Solicitação criada com sucesso.",
  "data": {
    "id": 1,
    "guia": "1JX26",
    "protocolo": "202604041525148K3Z",
    "data": "2026-04-04",
    ...
  }
}
```

---

## PUT/PATCH /solicitacoes/{id}

Atualiza uma solicitação existente.

> `guia` e `protocolo` são **somente leitura** e não podem ser alterados via update.

**Resposta de sucesso `200`:**

```json
{
  "message": "Solicitação atualizada com sucesso.",
  "data": { ... }
}
```

---

## DELETE /solicitacoes/{id}

Remove uma solicitação e seus itens (cascade delete).

**Resposta de sucesso `200`:**

```json
{
  "message": "Solicitação removida com sucesso."
}
```

---

## GET /solicitacoes/totais-diretos?ano={ano}

Retorna o total gasto em compras diretas (não licitadas) por empresa no ano informado. Inclui percentual do limite consumido quando o limite estiver configurado.

**Query string:**

| Parâmetro | Tipo | Obrigatório | Descrição |
|-----------|------|-------------|-----------|
| `ano` | integer | Sim | Ano de referência (ex: `2026`) |

**Exemplo:** `GET /api/solicitacoes/totais-diretos?ano=2026`

**Resposta de sucesso `200`:**

```json
{
  "ano": 2026,
  "data": [
    {
      "empresa_id": 1,
      "razaosocial": "Empresa Exemplo Ltda",
      "cpfcnpj": "00.000.000/0001-00",
      "ano": 2026,
      "total_gasto": 18500.00,
      "limite_compra_direta": 50000.00,
      "percentual_consumido": 37.00,
      "aviso": null
    },
    {
      "empresa_id": 2,
      "razaosocial": "Fornecedora ABC Ltda",
      "cpfcnpj": "11.111.111/0001-11",
      "ano": 2026,
      "total_gasto": 53200.00,
      "limite_compra_direta": 50000.00,
      "percentual_consumido": 106.40,
      "aviso": "Limite anual de compras diretas ultrapassado."
    },
    {
      "empresa_id": 3,
      "razaosocial": "Sem Limite Configurado",
      "cpfcnpj": "22.222.222/0001-22",
      "ano": 2026,
      "total_gasto": 7800.00,
      "limite_compra_direta": null,
      "percentual_consumido": null,
      "aviso": null
    }
  ]
}
```

> Apenas empresas com compras diretas no ano aparecem no resultado. O campo `aviso` é preenchido somente quando o limite está configurado e foi ultrapassado. Quando `limite_compra_direta` é `null`, `percentual_consumido` também é `null`.

---

# Itens da Solicitação

Base URL: `/api/solicitacoes/{id}/produtos`

---

## GET /solicitacoes/{id}/produtos

Lista todos os itens da solicitação com valor total por item e total geral.

**Resposta de sucesso `200`:**

```json
{
  "data": [
    {
      "id": 1,
      "solicitacao_id": 1,
      "produto_id": 3,
      "quant": "2.00",
      "preco_unitario": "4200.00",
      "aditivo_id": null,
      "produto": { ... },
      "aditivo": null,
      "valor_total_item": 8400.00
    }
  ],
  "valor_total": 8400.00
}
```

---

## POST /solicitacoes/{id}/produtos

Adiciona um item à solicitação. Se informar `aditivo_id` de um aditivo de preço, o `preco_unitario` é preenchido automaticamente.

**Body (JSON):**

| Campo | Tipo | Obrigatório | Descrição |
|-------|------|-------------|-----------|
| `produto_id` | integer | Sim | ID do produto |
| `quant` | numeric | Sim | Quantidade solicitada |
| `preco_unitario` | numeric | Condicional | Obrigatório quando não há aditivo de preço vinculado |
| `aditivo_id` | integer | Não | ID do aditivo vigente (preenche o preço automaticamente quando `tipo` for `preco` ou `ambos`) |

**Resposta de sucesso `201`:**

```json
{
  "message": "Item adicionado à solicitação com sucesso.",
  "data": {
    "id": 2,
    "solicitacao_id": 1,
    "produto_id": 5,
    "quant": "3.00",
    "preco_unitario": "850.00",
    "aditivo_id": null,
    "produto": { ... },
    "valor_total_item": 2550.00
  }
}
```

---

## PUT/PATCH /solicitacoes/{id}/produtos/{itemId}

Atualiza um item da solicitação.

**Body (JSON):**

| Campo | Tipo | Obrigatório | Descrição |
|-------|------|-------------|-----------|
| `produto_id` | integer | Sim | ID do produto |
| `quant` | numeric | Sim | Quantidade |
| `preco_unitario` | numeric | Sim | Preço unitário |
| `aditivo_id` | integer | Não | ID do aditivo |

**Resposta de sucesso `200`:**

```json
{
  "message": "Item atualizado com sucesso.",
  "data": { ... }
}
```

---

## DELETE /solicitacoes/{id}/produtos/{itemId}

Remove um item da solicitação.

**Resposta de sucesso `200`:**

```json
{
  "message": "Item removido da solicitação com sucesso."
}
```

**Erro — item não pertence à solicitação `403`:**

```json
{
  "message": "Este item não pertence à solicitação informada."
}
```

---

## Regras de negócio

**Campos gerados automaticamente:**

| Campo | Formato | Exemplo | Descrição |
|-------|---------|---------|-----------|
| `guia` | 5 caracteres alfanuméricos (A-Z, 0-9) | `1JX26` | Código único gerado no `POST`. Somente leitura após criação. |
| `protocolo` | `YYYYMMDDHHmmss` + 4 caracteres alfanuméricos | `202604041525148K3Z` | Gerado no `POST` com base no momento da criação. Somente leitura após criação. |

Ambos os campos são verificados contra duplicatas no banco antes de serem gravados. Em caso de colisão (extremamente improvável), um novo valor é gerado automaticamente.

**Compras diretas e limite anual:**

- Solicitações com `ehlicitada: false` são consideradas compras diretas.
- O limite anual por empresa é configurado no campo `limite_compra_direta` da empresa (via `PUT /api/empresas/{id}`).
- O endpoint `totais-diretos` consolida os gastos diretos por empresa no ano e sinaliza quando o limite foi ultrapassado.

**Preço dos itens:**

- O `preco_unitario` é sempre gravado como snapshot no momento da solicitação.
- Para produtos com aditivo de preço vigente, informar o `aditivo_id` preenche o preço automaticamente.
- Para compras diretas, o `preco_unitario` deve ser informado manualmente.
