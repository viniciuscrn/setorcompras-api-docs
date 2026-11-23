# Aditivos

> Formatos comuns (paginação `?per_page=`/`?q=`, `{ data, message }`, números, erros): [padroes.md](padroes.md).

Base URL: `/api/{ente}/aditivos` — permissão `licitacoes`.

Um aditivo altera **um item do contrato**: novo preço, acréscimo ou supressão de quantidade (art. 125 da Lei 14.133).

| Método | Rota | Descrição |
|--------|------|-----------|
| GET | `/aditivos?licitacao_produto_id=` | Aditivos do item + resumo (filtro obrigatório) |
| GET | `/aditivos/{id}` | Detalhe (com `item.produto`) |
| POST | `/aditivos` | Registra (`201`) |
| PUT/PATCH | `/aditivos/{id}` | Altera (o item não muda) |
| DELETE | `/aditivos/{id}` | Remove |

## Body

| Campo | Tipo | Obrigatório | Regra |
|-------|------|-------------|-------|
| `licitacao_produto_id` | integer | Sim (só no POST) | item do contrato |
| `tipo` | string | Sim | `preco`, `quantidade` ou `ambos` |
| `quant` | number | Se `quantidade`/`ambos` | **positivo = acréscimo, negativo = supressão**; não pode ser 0 |
| `preco` | number | Se `preco`/`ambos` | novo preço unitário |
| `justificativa` | string | Acima do limite | ver abaixo |

O campo que o tipo não usa é gravado como `null` (ex.: `quant` num aditivo só de preço).

## Regras

- **Limite legal:** acréscimos e supressões acumulados do item, cada um, até **25 %** da quantidade original
  (`LIMITE_ADITIVO_PERCENTUAL`). Em contrato de **reforma** (`ehreforma: true`) os acréscimos vão até **50 %**
  (`LIMITE_ADITIVO_REFORMA_PERCENTUAL`); a supressão continua em 25 %. Acima disso o aditivo **só é aceito com `justificativa`** — sem ela, `422` em `justificativa`;
  com ela, a resposta traz `aviso`.
- **Supressão** não pode deixar a quantidade contratada abaixo do que já foi utilizado (`422` em `quant`).
- **Excluir** um acréscimo que já foi consumido é recusado (`422`).
- O preço das solicitações já lançadas não muda: o preço é gravado no item da solicitação no momento do lançamento.

## Resposta

```json
{
  "message": "Aditivo registrado com sucesso.",
  "aviso": "Os aditivos somam 30% de acréscimo e 0% de supressão sobre a quantidade original (100), acima do limite legal de 25%. Registrado com justificativa.",
  "data": { "id": 2, "tipo": "quantidade", "quant": 10, "preco": null, "justificativa": "Demanda emergencial", "licitacao_produto_id": 1 },
  "resumo": {
    "licitacao_produto_id": 1, "empresa_licitacao_id": 1, "produto_id": 3,
    "quant_original": 100, "quant_acrescida": 30, "quant_suprimida": 0, "quant_contratada": 130,
    "preco_original": 10, "preco_atual": 10, "quant_utilizada": 0, "quant_disponivel": 130, "valor_disponivel": 1300,
    "percentual_acrescido": 30, "percentual_suprimido": 0, "limite_acrescimo_percentual": 25, "limite_supressao_percentual": 25, "limite_ultrapassado": true
  }
}
```

`aviso` só aparece quando o limite foi ultrapassado. O `GET /aditivos?licitacao_produto_id=` (não paginado) devolve `data` (lista) e o mesmo `resumo`; o `GET /aditivos/{id}` também traz o `resumo`.
