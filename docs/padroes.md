# Padrões da API

Vale para todas as rotas do ente (`/api/{ente}/...`). A área de admin (`/api/admin/...`, [admin.md](admin.md)) e o login/`/me`
([auth.md](auth.md)) têm formato próprio, descrito nas páginas deles.

## Listagens

Toda listagem de recurso é **paginada**:

| Parâmetro | Padrão | Regra |
|-----------|--------|-------|
| `per_page` | 15 | 1 a 100 (`422` fora disso) |
| `page` | 1 | |
| `q` | — | busca parcial (sem diferenciar maiúsculas) nas colunas indicadas em cada página |

Cada página documenta seus filtros próprios (ex.: `?ano=`, `?empresa_id=`). Os filtros e a busca são mantidos nos links.

```json
{
  "data": [ { "id": 1, "...": "..." } ],
  "links": { "first": ".../empresas?page=1&q=papel", "last": "...", "prev": null, "next": "...?page=2&q=papel" },
  "meta": { "current_page": 1, "from": 1, "last_page": 3, "per_page": 15, "to": 15, "total": 41, "path": "...", "links": [ ... ] }
}
```

Para preencher selects com cadastros pequenos (modalidades, naturezas, secretarias), use `?per_page=100`.

**Não paginadas** (listas filhas ou relatórios): itens do contrato, itens da solicitação, aditivos de um item,
`totais-diretos` e o preview do CSV.

## Gravações

| Ação | Status | Corpo |
|------|--------|-------|
| Criar | `201` | `{ "data": { ... }, "message": "... criado com sucesso." }` |
| Atualizar | `200` | `{ "data": { ... }, "message": "... atualizado com sucesso." }` |
| Excluir | `200` | `{ "message": "... removido com sucesso." }` |
| Consultar um | `200` | `{ "data": { ... } }` |

Algumas respostas trazem campos extras ao lado de `data`: `resumo` e `aviso` (aditivos), `saldo` (itens do contrato),
`valor_total` (itens da solicitação).

## Tipos

- **Números** (valores, quantidades, preços, limites) saem como **número** JSON (`27.5`), não como texto. Envie também como número.
- **Datas** de negócio: `YYYY-MM-DD` na entrada e na saída. `created_at`/`updated_at`: ISO 8601 (UTC).
- **CPF/CNPJ e telefones**: aceitos com ou sem máscara, devolvidos **só com dígitos**. CPF/CNPJ são validados pelos dígitos verificadores.
- **UF**: sigla de um dos 27 estados (`rn` vira `RN`).
- **Booleanos**: `true`/`false` (em multipart, `1`/`0`).

## Erros

Sempre `{ "message": "..." }` em português; validação traz também `errors` por campo.

| Status | Quando | `message` |
|--------|--------|-----------|
| `401` | sem token, token inválido ou expirado (12 h) | `Não autenticado. Faça login novamente.` |
| `403` | perfil sem permissão / senha provisória (`codigo: senha_provisoria`) | texto da regra |
| `404` | registro inexistente (ou de outro ente) | `Registro não encontrado.` |
| `404` | rota inexistente / instituição inexistente ou inativa | `Rota não encontrada.` / texto |
| `405` | método não aceito na rota | `Método não permitido para esta rota.` |
| `409` | exclusão de registro em uso ou que viola regra de integridade | texto da regra |
| `422` | validação ou regra de negócio | primeira mensagem + `errors` |
| `429` | muitas tentativas (login) | `Muitas tentativas. Aguarde um pouco e tente novamente.` |
| `500` | erro inesperado (sem detalhes em produção) | `Erro interno. Tente novamente ou contate o suporte.` |

```json
{
  "message": "É obrigatória a indicação de um valor para o campo razão social. (e mais 2 erros)",
  "errors": {
    "razaosocial": ["É obrigatória a indicação de um valor para o campo razão social."],
    "cpfcnpj": ["É obrigatória a indicação de um valor para o campo CPF/CNPJ."],
    "uf": ["O valor selecionado para o campo UF é inválido."]
  }
}
```
