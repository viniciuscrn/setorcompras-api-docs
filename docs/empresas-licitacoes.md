# Empresas Vencedoras de Licitações

Base URL: `/api/empresas-licitacoes`

**Autenticação:** Todas as rotas requerem `Bearer {token}` no header `Authorization`.

---

## GET /empresas-licitacoes

Lista todos os vínculos empresa-licitação.

Aceita filtros opcionais via query string:

| Parâmetro | Tipo | Descrição |
|-----------|------|-----------|
| `licitacao_id` | integer | Filtra pelas empresas vencedoras de uma licitação específica |
| `empresa_id` | integer | Filtra pelas licitações vencidas por uma empresa específica |

**Exemplo:** `GET /api/empresas-licitacoes?licitacao_id=1`

**Resposta de sucesso `200`:**

```json
{
  "data": [
    {
      "id": 1,
      "valor": "35000.00",
      "ehrealinhamento": false,
      "valor_total_produtos": 28400.00,
      "quantidade_total_produtos": 15.00,
      "valor_gasto_contrato": 12000.00,
      "valor_disponivel_contrato": 23000.00,
      "numerocontrato": "CT-001/2026",
      "iniciovigencia": "2026-02-10",
      "fimvigencia": "2027-02-10",
      "empresa_id": 1,
      "licitacao_id": 1,
      "created_at": "2026-01-01T00:00:00.000000Z",
      "updated_at": "2026-01-01T00:00:00.000000Z",
      "empresa": {
        "id": 1,
        "cpfcnpj": "00.000.000/0001-00",
        "razaosocial": "Empresa Exemplo Ltda"
      },
      "licitacao": {
        "id": 1,
        "numero": "001/2026",
        "objeto": "Aquisição de equipamentos de informática",
        "modalidade": {
          "id": 1,
          "nome": "Pregão Eletrônico"
        }
      }
    }
  ]
}
```

> **Campos calculados:**
> - `valor_total_produtos` → soma de `produto.preco × produto.quant` de todos os itens vinculados ao contrato
> - `quantidade_total_produtos` → soma de `produto.quant` de todos os itens
> - `valor_gasto_contrato` → total já pago em solicitações licitadas desta empresa (`ehlicitada = true`)
> - `valor_disponivel_contrato` → `valor` do contrato − `valor_gasto_contrato`

---

## GET /empresas-licitacoes/{id}

Retorna um vínculo empresa-licitação específico.

**Resposta de sucesso `200`:**

```json
{
  "data": {
    "id": 1,
    "valor": "35000.00",
    "ehrealinhamento": false,
    "numerocontrato": "CT-001/2026",
    "iniciovigencia": "2026-02-10",
    "fimvigencia": "2027-02-10",
    "empresa_id": 1,
    "licitacao_id": 1,
    "empresa": { ... },
    "licitacao": { ... }
  }
}
```

---

## POST /empresas-licitacoes

Registra uma empresa como vencedora de uma licitação.

**Body (JSON):**

| Campo | Tipo | Obrigatório | Descrição |
|-------|------|-------------|-----------|
| `valor` | numeric | Sim | Valor do contrato |
| `ehrealinhamento` | boolean | Sim | Indica se é um realinhamento de preço |
| `numerocontrato` | string | Não | Número do contrato gerado |
| `iniciovigencia` | date | Não | Início da vigência do contrato (`YYYY-MM-DD`) |
| `fimvigencia` | date | Não | Fim da vigência do contrato (`YYYY-MM-DD`). Deve ser igual ou posterior a `iniciovigencia` |
| `empresa_id` | integer | Sim | ID da empresa (deve existir) |
| `licitacao_id` | integer | Sim | ID da licitação (deve existir) |

**Resposta de sucesso `201`:**

```json
{
  "message": "Empresa vencedora vinculada à licitação com sucesso.",
  "data": {
    "id": 1,
    "valor": "35000.00",
    "ehrealinhamento": false,
    "numerocontrato": "CT-001/2026",
    "iniciovigencia": "2026-02-10",
    "fimvigencia": "2027-02-10",
    "empresa_id": 1,
    "licitacao_id": 1,
    "empresa": { ... },
    "licitacao": { ... }
  }
}
```

---

## PUT/PATCH /empresas-licitacoes/{id}

Atualiza um vínculo empresa-licitação existente.

**Body (JSON):**

| Campo | Tipo | Obrigatório | Descrição |
|-------|------|-------------|-----------|
| `valor` | numeric | Sim | Valor do contrato |
| `ehrealinhamento` | boolean | Sim | Indica se é um realinhamento de preço |
| `numerocontrato` | string | Não | Número do contrato |
| `iniciovigencia` | date | Não | Início da vigência (`YYYY-MM-DD`) |
| `fimvigencia` | date | Não | Fim da vigência (`YYYY-MM-DD`) |
| `empresa_id` | integer | Sim | ID da empresa |
| `licitacao_id` | integer | Sim | ID da licitação |

**Resposta de sucesso `200`:**

```json
{
  "message": "Vínculo empresa-licitação atualizado com sucesso.",
  "data": { ... }
}
```

---

## DELETE /empresas-licitacoes/{id}

Remove um vínculo empresa-licitação.

**Resposta de sucesso `200`:**

```json
{
  "message": "Vínculo empresa-licitação removido com sucesso."
}
```
