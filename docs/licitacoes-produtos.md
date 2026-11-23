# Itens do contrato

> Formatos comuns (paginação `?per_page=`/`?q=`, `{ data, message }`, números, erros): [padroes.md](padroes.md).

Base URL: `/api/{ente}/empresas-licitacoes/{contrato}/produtos` — permissão `licitacoes`.

Cada item é um produto do catálogo com **quantidade, preço unitário, marca e nº do item** registrados no contrato.
O mesmo produto não se repete no mesmo contrato.

| Método | Rota | Descrição |
|--------|------|-----------|
| GET | `/produtos` | Itens com saldo + resumo do contrato |
| POST | `/produtos` | Inclui um item (`201`) |
| PUT/PATCH | `/produtos/{item}` | Altera quantidade, preço, marca, nº |
| DELETE | `/produtos/{item}` | Remove; `409` se já tiver solicitações ou aditivos |
| POST | `/produtos/preview` | Confere um CSV, sem gravar |
| POST | `/produtos/importar` | Grava os itens conferidos (tudo ou nada) |

Item de outro contrato na URL → `404`.

## GET — itens com saldo

```json
{
  "data": [
    {
      "id": 1, "empresa_licitacao_id": 1, "produto_id": 3,
      "produto": { "id": 3, "descricao": "Papel A4", "unidade": "CX" },
      "numero_item": 1, "marca": "Chamex",
      "quant_original": 10, "preco_original": 25,
      "quant_acrescida": 2, "quant_suprimida": 0, "quant_contratada": 12,
      "preco_atual": 27,
      "quant_utilizada": 3, "quant_disponivel": 9, "valor_disponivel": 243
    }
  ],
  "saldo": { "valor_contrato": 5000, "valor_itens": 450, "valor_utilizado": 91, "valor_disponivel": 4909 }
}
```

- `quant_contratada` = original + acréscimos − supressões (aditivos).
- `preco_atual` = preço do último aditivo de preço, ou o original.
- `quant_utilizada` = soma das quantidades nas solicitações **não excluídas** que apontam para este item.

## POST — incluir item

| Campo | Tipo | Obrigatório | Regra |
|-------|------|-------------|-------|
| `produto_id` | integer | Sim, sem `descricao` | produto do catálogo |
| `descricao` + `unidade` | string | Sim, sem `produto_id` | reaproveita o produto igual do catálogo ou cria um novo |
| `quant` | number | Sim | > 0 |
| `preco` | number | Sim | ≥ 0 (até 4 casas) |
| `marca` | string | Não | |
| `numero_item` | integer | Não | ≥ 1 |

Produto já no contrato → `422` em `produto_id`. A resposta traz o item no formato do GET.

## PUT — alterar item

Body: `quant`, `preco` (obrigatórios), `marca`, `numero_item`. O produto não muda (remova e inclua outro).
A quantidade não pode deixar o item abaixo do que já foi utilizado (`422`).

## Importação via CSV

Fluxo sempre **preview → importar**.

**Preview** — `multipart/form-data` com `arquivo` (`.csv`, máx. 2 MB), separador vírgula, primeira linha com o cabeçalho:

```
descricao,unidade,quant,preco,marca,numero_item
Papel A4,CX,10,25.00,Chamex,1
Caneta azul,UN,100,2.00,,2
```

```json
{
  "total": 2,
  "tem_erro": true,
  "data": [
    { "linha": 1, "descricao": "Papel A4", "unidade": "CX", "quant": 10, "preco": 25, "marca": "Chamex", "numero_item": 1, "erros": [] },
    { "linha": 2, "descricao": "papel a4", "unidade": "cx", "quant": 3, "preco": 1, "marca": null, "numero_item": null, "erros": ["Produto repetido no arquivo (linha 1)."] }
  ]
}
```

Erros por linha: campos inválidos, `"Produto repetido no arquivo (linha N)."`, `"Produto já está neste contrato."`.

**Importar** — JSON com as linhas aprovadas:

```json
{ "produtos": [ { "descricao": "Papel A4", "unidade": "CX", "quant": 10, "preco": 25, "marca": "Chamex", "numero_item": 1 } ] }
```

Cada produto é reaproveitado do catálogo (ou criado). Se algum já estiver no contrato, nada é gravado (`422` em `produtos.{i}.descricao`).
