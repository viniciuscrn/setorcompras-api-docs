# Naturezas de despesa (limite de compra direta)

> Formatos comuns (paginação `?per_page=`/`?q=`, `{ data, message }`, números, erros): [padroes.md](padroes.md).

Base URL: `/api/{ente}/naturezas` — permissão `catalogo`.

Cada instituição cadastra os **objetos/naturezas de despesa** com o **limite anual de compra direta** (art. 75 da Lei 14.133 —
o limite vale por objeto no exercício, não por fornecedor). Toda solicitação direta escolhe uma natureza, e a API **bloqueia**
o item que faria o total do ano naquela natureza passar do limite.

| Método | Rota | Descrição |
|--------|------|-----------|
| GET | `/naturezas?ativo=1&q=` | Lista paginada em ordem alfabética; `ativo` filtra (`1`/`0`); `q` busca em nome e descrição |
| GET | `/naturezas/{id}` | Detalhe |
| POST | `/naturezas` | Cria (`201`) |
| PUT/PATCH | `/naturezas/{id}` | Atualiza |
| DELETE | `/naturezas/{id}` | Remove; `409` se já foi usada — nesse caso desative (`ativo: false`) |

## Body

| Campo | Tipo | Obrigatório | Regra |
|-------|------|-------------|-------|
| `nome` | string | Sim | ex.: "Material de expediente" |
| `descricao` | string | Não | |
| `limite_anual` | number | Sim | ≥ 0, em reais |
| `ativo` | boolean | Não | padrão `true`; natureza inativa não pode ser escolhida em solicitação nova |

```json
{
  "message": "Natureza de despesa criada com sucesso.",
  "data": { "id": 1, "nome": "Material de expediente", "descricao": null, "limite_anual": 62725.59, "ativo": true, "created_at": "...", "updated_at": "..." }
}
```

O consumo do ano por natureza está em `GET /solicitacoes/totais-diretos?ano=` ([solicitacoes.md](solicitacoes.md#get-solicitacoestotais-diretosano)).
