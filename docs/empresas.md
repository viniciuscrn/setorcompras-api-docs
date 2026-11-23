# Empresas (fornecedores)

> Formatos comuns (paginação `?per_page=`/`?q=`, `{ data, message }`, números, erros): [padroes.md](padroes.md).

Base URL: `/api/{ente}/empresas` — permissão `catalogo` ([permissoes.md](permissoes.md)).

> Todas as rotas ficam sob o slug da instituição (`/api/{ente}/...`) e exigem `Authorization: Bearer {token}`.

| Método | Rota | Descrição |
|--------|------|-----------|
| GET | `/empresas?q=` | Lista paginada (ordem alfabética); `q` busca em razão social, CPF/CNPJ e cidade |
| GET | `/empresas/{id}` | Detalhe |
| POST | `/empresas` | Cria (`201`) |
| PUT/PATCH | `/empresas/{id}` | Atualiza |
| DELETE | `/empresas/{id}` | Remove; `409` se tiver contratos ou solicitações |

## Body (POST / PUT)

| Campo | Tipo | Obrigatório | Regra |
|-------|------|-------------|-------|
| `cpfcnpj` | string | Sim | CPF ou CNPJ **válido** (dígitos verificadores), com ou sem máscara; único na instituição |
| `razaosocial` | string | Sim | máx. 191 |
| `endereco`, `bairro`, `cidade` | string | Sim | máx. 191 |
| `uf` | string | Sim | sigla do estado (`RN`) |
| `tel` | string | Sim | máx. 20 |
| `email` | string | Não | e-mail válido |

CPF/CNPJ e telefone são **gravados e devolvidos só com dígitos** — a máscara fica por conta do front.

```json
{
  "message": "Empresa criada com sucesso.",
  "data": {
    "id": 1, "cpfcnpj": "11222333000181", "tipo_pessoa": "juridica", "razaosocial": "Fornecedora Ltda",
    "endereco": "Rua A, 10", "bairro": "Centro", "cidade": "Natal", "uf": "RN",
    "tel": "84999990000", "email": "contato@fornecedora.com",
    "created_at": "...", "updated_at": "..."
  }
}
```

**Erros `422`:** `"O CPF/CNPJ informado é inválido."`, `"Já existe uma empresa com este CPF/CNPJ."`.

> O limite de compra direta **não é mais por empresa**: ele fica na natureza de despesa ([naturezas.md](naturezas.md)).
