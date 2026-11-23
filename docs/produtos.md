# Produtos (catálogo)

> Formatos comuns (paginação `?per_page=`/`?q=`, `{ data, message }`, números, erros): [padroes.md](padroes.md).

Base URL: `/api/{ente}/produtos` — permissão `catalogo`.

O produto é só o **catálogo**: descrição e unidade. Quantidade, preço, marca e nº do item ficam no **item do contrato**
([licitacoes-produtos.md](licitacoes-produtos.md)); o preço de compra direta fica no item da solicitação.

| Método | Rota | Descrição |
|--------|------|-----------|
| GET | `/produtos?q=` | Lista paginada em ordem alfabética; `q` busca na descrição |
| GET | `/produtos/{id}` | Detalhe |
| POST | `/produtos` | Cria — ou devolve o existente (ver abaixo) |
| PUT/PATCH | `/produtos/{id}` | Atualiza |
| DELETE | `/produtos/{id}` | Remove; `409` se estiver em algum contrato ou solicitação |

## Body

| Campo | Tipo | Obrigatório | Regra |
|-------|------|-------------|-------|
| `descricao` | string | Sim | espaços repetidos são removidos |
| `unidade` | string | Sim | máx. 30; gravada em maiúsculas (`un` → `UN`) |

**Sem duplicidade:** se já houver produto com a mesma descrição (ignorando maiúsculas e espaços extras) e unidade,
o `POST` devolve esse produto com `200` e `"message": "Produto já existia no catálogo."`. Produto novo → `201`.

```json
{ "message": "Produto criado com sucesso.", "data": { "id": 3, "descricao": "Papel A4 75g", "unidade": "CX", "created_at": "...", "updated_at": "..." } }
```
