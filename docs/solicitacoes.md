# Solicitações de compra

> Formatos comuns (paginação `?per_page=`/`?q=`, `{ data, message }`, números, erros): [padroes.md](padroes.md).

Base URL: `/api/{ente}/solicitacoes` — permissão `solicitacoes` (o relatório `totais-diretos` usa `relatorios.ver`).

> Todas as rotas ficam sob o slug da instituição (`/api/{ente}/...`) e exigem `Authorization: Bearer {token}`.

Há dois tipos:

| | Licitada (`ehlicitada: true`) | Direta (`ehlicitada: false`) |
|---|---|---|
| Liga-se a | um **contrato** (`empresa_licitacao_id`) da mesma empresa, **vigente na data** | uma **natureza de despesa** ativa (`natureza_id`) |
| Itens | itens do contrato; quantidade ≤ saldo do item | produtos do catálogo com preço informado |
| Preço | sempre o **preço atual** do item do contrato | informado no item |
| Limite | saldo de cada item do contrato | **limite anual** da natureza (bloqueia) |

Excluir uma solicitação (exclusão lógica) devolve o saldo ao contrato e o limite à natureza.

---

## GET /solicitacoes

Lista paginada. Filtros: `empresa_id`, `responsavel_id`, `empresa_licitacao_id`, `natureza_id`, `ehlicitada` (`1`/`0`), `ano`; `q` busca em guia, protocolo, descrição e razão social.

```json
{
  "data": [
    {
      "id": 1, "guia": "1JX26", "protocolo": "202604041525148K3Z", "data": "2026-03-10",
      "descricao": "Material de escritório", "obs": null, "ehlicitada": true,
      "empresa_id": 1, "empresa_licitacao_id": 1, "natureza_id": null, "responsavel_id": 2, "user_id": 1,
      "empresa": { ... }, "contrato": { ..., "licitacao": { ... } }, "natureza": null,
      "responsavel": { ... }, "user": { "id": 1, "name": "Admin" }
    }
  ]
}
```

## GET /solicitacoes/{id}

Igual ao item da lista, mais `produtos` (com `produto` e `item_do_contrato`) e `valor_total`.

## POST /solicitacoes

| Campo | Tipo | Obrigatório | Regra |
|-------|------|-------------|-------|
| `data` | date | Sim | `YYYY-MM-DD` |
| `descricao` | string | Sim | |
| `obs` | string | Não | |
| `ehlicitada` | boolean | Sim | |
| `empresa_id` | integer | Sim | |
| `empresa_licitacao_id` | integer | Se licitada | contrato **da mesma empresa** e **vigente na data**; ignorado na direta |
| `natureza_id` | integer | Se direta | natureza **ativa**; ignorado na licitada |
| `responsavel_id` | integer | Sim | |

`guia`, `protocolo` e `user_id` (o usuário do token) são definidos pela API — se enviados, são ignorados.

Erros `422`: `empresa_licitacao_id` ("Informe o contrato…", "O contrato informado é de outra empresa."),
`data` ("A data da solicitação está fora da vigência do contrato."), `natureza_id` ("Informe a natureza…", "…desativada.").

## PUT/PATCH /solicitacoes/{id}

Mesmo body. Com itens já lançados **não é possível mudar tipo, empresa, contrato, natureza nem o ano da data** (`422` em `ehlicitada`) —
remova os itens antes.

## DELETE /solicitacoes/{id}

Exclusão lógica. `{ "message": "Solicitação removida com sucesso." }`

---

## GET /solicitacoes/totais-diretos?ano={ano}

Compras diretas do ano **por natureza de despesa**, contra o limite anual de cada uma, com o detalhe por empresa.
Aparecem as naturezas ativas e as inativas que tiveram gasto no ano. Solicitações excluídas não contam.

```json
{
  "ano": 2026,
  "data": [
    {
      "natureza_id": 1, "nome": "Material de expediente", "ativo": true, "ano": 2026,
      "limite_anual": 1000.0, "total_gasto": 850.0, "disponivel": 150.0, "percentual_consumido": 85.0,
      "aviso": "Limite anual de compra direta próximo do fim.",
      "empresas": [ { "empresa_id": 1, "razaosocial": "Fornecedora Ltda", "cpfcnpj": "11222333000181", "total": 850.0 } ]
    }
  ]
}
```

`aviso`: a partir de 80 % → "Limite anual de compra direta próximo do fim."; 100 % → "Limite anual de compra direta atingido.".
`percentual_consumido` é `null` quando o limite é 0.

---

# Itens da solicitação

Base URL: `/api/{ente}/solicitacoes/{id}/produtos`. Item de outra solicitação na URL → `404`.

## GET

```json
{
  "data": [
    {
      "id": 1, "solicitacao_id": 1, "produto_id": 3, "licitacao_produto_id": 1,
      "quant": 3, "preco_unitario": 27,
      "produto": { ... }, "item_do_contrato": { ... },
      "valor_total": 81.0
    }
  ],
  "valor_total": 91.0
}
```

## POST / PUT

**Solicitação licitada:**

| Campo | Tipo | Obrigatório | Regra |
|-------|------|-------------|-------|
| `licitacao_produto_id` | integer | Sim | item **do contrato da solicitação** |
| `quant` | number | Sim | > 0 e ≤ `quant_disponivel` do item (na edição, a quantidade do próprio item não conta) |

O `produto_id` vem do item do contrato e o `preco_unitario` é o **preço atual** do item — qualquer valor enviado é ignorado.

**Solicitação direta:**

| Campo | Tipo | Obrigatório | Regra |
|-------|------|-------------|-------|
| `produto_id` | integer | Sim | produto do catálogo |
| `quant` | number | Sim | > 0 |
| `preco_unitario` | number | Sim | ≥ 0 |

O total do ano na natureza (todas as empresas), somado a este item, não pode passar do `limite_anual` (`422` em `quant`,
com o valor ainda disponível na mensagem).

O preço fica gravado no item: aditivos posteriores não mudam itens já lançados.

## DELETE

`{ "message": "Item removido da solicitação com sucesso." }`

---

## Campos gerados

| Campo | Formato | Exemplo |
|-------|---------|---------|
| `guia` | 5 caracteres A-Z/0-9, único | `1JX26` |
| `protocolo` | `YYYYMMDDHHmmss` + 4 caracteres A-Z/0-9, único | `202604041525148K3Z` |
