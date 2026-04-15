# Produtos do Contrato Empresa-Licitação

Base URL: `/api/empresas-licitacoes/{empresaLicitacao}/produtos`

**Autenticação:** Todas as rotas requerem `Bearer {token}` no header `Authorization`.

> Todas as rotas são aninhadas sob um vínculo empresa-licitação (`empresaLicitacao`). O `{empresaLicitacao}` corresponde ao `id` de um registro em `/api/empresas-licitacoes`.

---

## GET /empresas-licitacoes/{id}/produtos

Lista todos os itens do contrato com disponibilidade calculada por item.

**Resposta de sucesso `200`:**

```json
{
  "data": [
    {
      "id": 1,
      "empresa_licitacao_id": 1,
      "produto_id": 1,
      "created_at": "2026-01-01T00:00:00.000000Z",
      "updated_at": "2026-01-01T00:00:00.000000Z",
      "produto": {
        "id": 1,
        "descricao": "Notebook i5 16GB SSD 512GB",
        "unidade": "UN",
        "quant": "10.00",
        "preco": "4200.00",
        "marca": "Dell",
        "numero_item": 1
      },
      "quantidade_contratada": 12.00,
      "preco_atual": 4500.00,
      "quantidade_utilizada": 4.00,
      "quantidade_disponivel": 8.00,
      "valor_disponivel": 36000.00
    }
  ]
}
```

> **Campos calculados por item:**
> - `quantidade_contratada` → `produto.quant` + soma dos aditivos de quantidade deste item
> - `preco_atual` → preço do aditivo de preço mais recente, ou `produto.preco` se não houver
> - `quantidade_utilizada` → soma de `quant` em solicitações licitadas desta empresa para este produto
> - `quantidade_disponivel` → `quantidade_contratada` − `quantidade_utilizada`
> - `valor_disponivel` → `quantidade_disponivel × preco_atual`

---

## POST /empresas-licitacoes/{id}/produtos

Vincula um produto já existente ao contrato.

**Body (JSON):**

| Campo | Tipo | Obrigatório | Descrição |
|-------|------|-------------|-----------|
| `produto_id` | integer | Sim | ID do produto (deve existir em `/api/produtos`) |

**Resposta de sucesso `201`:**

```json
{
  "message": "Produto vinculado ao contrato de licitação com sucesso.",
  "data": {
    "id": 5,
    "empresa_licitacao_id": 1,
    "produto_id": 3,
    "produto": { ... }
  }
}
```

**Erro — produto já vinculado `422`:**

```json
{
  "message": "Este produto já está vinculado a este contrato de licitação."
}
```

---

## DELETE /empresas-licitacoes/{id}/produtos/{licitacaoProdutoId}

Remove o vínculo de um produto do contrato pelo `id` do registro em `licitacoes_produtos`.

**Resposta de sucesso `200`:**

```json
{
  "message": "Produto desvinculado do contrato de licitação com sucesso."
}
```

**Erro — registro não pertence ao contrato `403`:**

```json
{
  "message": "Este produto não pertence ao contrato informado."
}
```

---

## POST /empresas-licitacoes/{id}/produtos/preview

Recebe um arquivo CSV e retorna os dados parseados para conferência. **Nenhum registro é gravado.**

**Body (form-data):**

| Campo | Tipo | Obrigatório | Descrição |
|-------|------|-------------|-----------|
| `arquivo` | file | Sim | Arquivo `.csv` com os produtos (máx. 2MB) |

**Formato esperado do CSV:**

```
descricao,unidade,quant,preco,marca,numero_item
Notebook i5 16GB SSD 512GB,UN,10,4200.0,Dell,1
Notebook i7 16GB SSD 1TB,UN,5,5600.0,Lenovo,2
```

| Coluna | Tipo | Obrigatório |
|--------|------|-------------|
| `descricao` | string | Sim |
| `unidade` | string | Sim |
| `quant` | numeric | Sim |
| `preco` | numeric | Sim |
| `marca` | string | Não |
| `numero_item` | integer | Não |

**Resposta de sucesso `200`:**

```json
{
  "total": 30,
  "tem_erro": false,
  "dados": [
    {
      "linha": 1,
      "descricao": "Notebook i5 16GB SSD 512GB",
      "unidade": "UN",
      "quant": 10,
      "preco": 4200.0,
      "marca": "Dell",
      "numero_item": 1,
      "erros": []
    },
    {
      "linha": 2,
      "descricao": null,
      "unidade": "UN",
      "quant": 5,
      "preco": 5600.0,
      "marca": "Lenovo",
      "numero_item": 2,
      "erros": ["The descricao field is required."]
    }
  ]
}
```

> Use o campo `tem_erro: true` para bloquear o botão de confirmação no front-end enquanto houver linhas com problema.

---

## POST /empresas-licitacoes/{id}/produtos/importar

Confirma a importação: cria cada produto na tabela `produtos` e o vincula ao contrato. Toda a operação ocorre em uma única transação — se qualquer item falhar, nenhum é salvo.

**Body (JSON):**

| Campo | Tipo | Obrigatório | Descrição |
|-------|------|-------------|-----------|
| `produtos` | array | Sim | Array de produtos a importar (mín. 1 item) |
| `produtos.*.descricao` | string | Sim | Descrição do produto |
| `produtos.*.unidade` | string | Sim | Unidade de medida |
| `produtos.*.quant` | numeric | Sim | Quantidade |
| `produtos.*.preco` | numeric | Sim | Preço unitário |
| `produtos.*.marca` | string | Não | Marca |
| `produtos.*.numero_item` | integer | Não | Número do item |

**Exemplo de body:**

```json
{
  "produtos": [
    {
      "descricao": "Notebook i5 16GB SSD 512GB",
      "unidade": "UN",
      "quant": 10,
      "preco": 4200.0,
      "marca": "Dell",
      "numero_item": 1
    },
    {
      "descricao": "Notebook i7 16GB SSD 1TB",
      "unidade": "UN",
      "quant": 5,
      "preco": 5600.0,
      "marca": "Lenovo",
      "numero_item": 2
    }
  ]
}
```

**Resposta de sucesso `201`:**

```json
{
  "message": "30 produto(s) importado(s) e vinculados com sucesso.",
  "data": [
    {
      "id": 1,
      "descricao": "Notebook i5 16GB SSD 512GB",
      "unidade": "UN",
      "quant": "10.00",
      "preco": "4200.00",
      "marca": "Dell",
      "numero_item": 1
    }
  ]
}
```

---

## Fluxo de importação via CSV

```
1. POST /produtos/preview   →  Envia o arquivo CSV (form-data)
                               Recebe o array "dados" para exibir na tela
                               Verifica "tem_erro" antes de habilitar o botão

2. (usuário revisa os dados na tela)

3. POST /produtos/importar  →  Envia o array "dados" como { "produtos": [...] }
                               Produtos criados e vinculados em transação única
```
