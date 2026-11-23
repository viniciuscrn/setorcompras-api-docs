# Instituição

Base URL: `/api/{ente}/instituicao` — permissão `instituicao` (ver: todos; alterar: admin).

> Formatos comuns (`{ data, message }`, erros): [padroes.md](padroes.md).

Recurso singular: **um registro por ente**, criado junto com o ente pela área de admin já com `nome`, `cnpj` e `uf`.
A própria instituição só **completa e atualiza** o cadastro — não há criação nem exclusão por esta API.
Enquanto faltar endereço, cidade, e-mail ou brasão, o login e o `/me` de quem pode alterar devolvem um alerta (`metodo: PUT`, `campos: [...]`).

| Método | Rota | Descrição |
|--------|------|-----------|
| GET | `/instituicao` | Dados da instituição |
| PUT/PATCH | `/instituicao` | Atualiza (JSON) |
| POST | `/instituicao` | Atualiza — use para enviar o brasão em `multipart/form-data` (o PHP não lê arquivos em PUT) |
| GET | `/instituicao/logo` | **Pública** (sem token): a imagem do brasão; `404` se não houver |

## GET /instituicao

```json
{
  "data": {
    "id": 1,
    "nome": "Prefeitura Municipal de Água Branca",
    "cnpj": "12345678000195",
    "endereco": "Praça Central, 1",
    "bairro": "Centro",
    "cidade": "Água Branca",
    "uf": "AL",
    "tel": "8233330000",
    "cel": "82999990000",
    "email": "contato@aguabranca.al.gov.br",
    "site": "https://aguabranca.al.gov.br",
    "logo_url": "https://api.exemplo.com/api/agua-branca/instituicao/logo",
    "created_at": "2026-01-01T00:00:00.000000Z",
    "updated_at": "2026-01-01T00:00:00.000000Z"
  }
}
```

`logo_url` é `null` sem brasão. O caminho interno do arquivo não é exposto.

## PUT / POST /instituicao

| Campo | Tipo | Obrigatório | Regra |
|-------|------|-------------|-------|
| `nome` | string | Sim | |
| `cnpj` | string | Não | CNPJ válido, com ou sem máscara (gravado só com dígitos) |
| `endereco`, `bairro`, `cidade` | string | Não | |
| `uf` | string | Não | sigla do estado |
| `tel`, `cel` | string | Não | gravados só com dígitos |
| `email` | string | Não | e-mail válido |
| `site` | string | Não | |
| `logo` | arquivo | Não | jpg, jpeg, png ou webp, até 2 MB; substitui o anterior |

Campos não enviados ficam como estão. Resposta: `{ "data": { ... }, "message": "Instituição atualizada com sucesso." }`.
