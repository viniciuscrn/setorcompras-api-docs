# Contratos (empresas × licitações)

> Formatos comuns (paginação `?per_page=`/`?q=`, `{ data, message }`, números, erros): [padroes.md](padroes.md).

Base URL: `/api/{ente}/empresas-licitacoes` — permissão `licitacoes`.

Um contrato liga a **empresa vencedora** a uma **licitação**, com valor, vigência e itens ([licitacoes-produtos.md](licitacoes-produtos.md)).
A mesma empresa pode ter vários contratos (inclusive na mesma licitação, ex.: realinhamento); **o saldo é sempre por contrato**.

| Método | Rota | Descrição |
|--------|------|-----------|
| GET | `/empresas-licitacoes?licitacao_id=&empresa_id=&vigentes_em=YYYY-MM-DD&q=` | Lista paginada com saldo; `q` busca no nº do contrato, razão social e nº da licitação |
| GET | `/empresas-licitacoes/{id}` | Detalhe com saldo e itens |
| POST | `/empresas-licitacoes` | Cria (`201`) |
| PUT/PATCH | `/empresas-licitacoes/{id}` | Atualiza |
| DELETE | `/empresas-licitacoes/{id}` | Exclusão lógica; `409` se já tiver solicitações |

## Body

| Campo | Tipo | Obrigatório | Regra |
|-------|------|-------------|-------|
| `valor` | number | Sim | valor do contrato |
| `ehrealinhamento` | boolean | Sim | |
| `ehreforma` | boolean | Não | reforma de edifício ou equipamento: acréscimo por aditivo até **50 %** (padrão `false`) |
| `numerocontrato` | string | Não | |
| `iniciovigencia`, `fimvigencia` | date | Não | `YYYY-MM-DD`, os dois juntos; **sem datas, vale a vigência da licitação** |
| `empresa_id` | integer | Sim | |
| `licitacao_id` | integer | Sim | licitação não excluída |

Com solicitações já lançadas, não é possível trocar `empresa_id` nem `licitacao_id` (`422`).

## Resposta (GET)

```json
{
  "data": {
    "id": 1, "valor": 5000, "ehrealinhamento": false, "ehreforma": false, "numerocontrato": "12/2026",
    "iniciovigencia": null, "fimvigencia": null, "empresa_id": 1, "licitacao_id": 1,
    "empresa": { ... }, "licitacao": { ..., "modalidade": { ... } },
    "vigente_hoje": true,
    "saldo": {
      "valor_contrato": 5000.0,
      "valor_itens": 450.0,
      "valor_utilizado": 91.0,
      "valor_disponivel": 4909.0
    },
    "itens": [ { "id": 1, "produto": { ... }, "quant_contratada": 12, "quant_disponivel": 9, "...": "ver licitacoes-produtos.md" } ]
  }
}
```

- `valor_itens`: soma de quantidade original × preço original dos itens.
- `valor_utilizado`: soma dos itens das solicitações (não excluídas) que consomem este contrato.
- `valor_disponivel`: `valor_contrato − valor_utilizado`.
