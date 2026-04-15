# Instituição

Base URL: `/api/instituicao`

**Autenticação:** Todas as rotas requerem `Bearer {token}` no header `Authorization`.

> Recurso singular — existe apenas um registro de instituição no sistema.

---

## GET /instituicao

Retorna os dados da instituição cadastrada.

**Resposta de sucesso `200`:**

```json
{
  "data": {
    "id": 1,
    "cnpj": "00.000.000/0001-00",
    "nome": "Prefeitura Municipal",
    "endereco": "Rua Exemplo, 123",
    "bairro": "Centro",
    "cidade": "Cidade",
    "uf": "SP",
    "tel": "(11) 1234-5678",
    "cel": "(11) 91234-5678",
    "email": "contato@prefeitura.gov.br",
    "site": "https://prefeitura.gov.br",
    "logo": "instituicoes/logos/exemplo.png",
    "created_at": "2026-01-01T00:00:00.000000Z",
    "updated_at": "2026-01-01T00:00:00.000000Z"
  }
}
```

**Resposta de erro `404`:**

```json
{
  "message": "Instituição não cadastrada.",
  "data": null
}
```

---

## POST /instituicao

Cria a instituição. Só pode ser criada uma vez.

**Body (multipart/form-data):**

| Campo | Tipo | Obrigatório | Descrição |
|-------|------|-------------|-----------|
| `cnpj` | string | Sim | CNPJ da instituição |
| `nome` | string | Sim | Nome da instituição |
| `endereco` | string | Sim | Endereço |
| `bairro` | string | Sim | Bairro |
| `cidade` | string | Sim | Cidade |
| `uf` | string | Sim | UF (ex: SP) |
| `tel` | string | Sim | Telefone |
| `cel` | string | Sim | Celular |
| `email` | string | Sim | E-mail |
| `site` | string | Sim | Site |
| `logo` | file | Sim | Imagem (jpg, jpeg, png, webp — máx. 2MB) |

**Resposta de sucesso `201`:**

```json
{
  "message": "Instituição criada com sucesso.",
  "data": { ... }
}
```

**Resposta de erro `409`:**

```json
{
  "message": "Instituição já cadastrada. Utilize PUT/PATCH para atualizar."
}
```

---

## PUT /instituicao / PATCH /instituicao

Atualiza os dados da instituição.

**Body (multipart/form-data):**

| Campo | Tipo | Obrigatório | Descrição |
|-------|------|-------------|-----------|
| `cnpj` | string | Sim | CNPJ da instituição |
| `nome` | string | Sim | Nome da instituição |
| `endereco` | string | Sim | Endereço |
| `bairro` | string | Sim | Bairro |
| `cidade` | string | Sim | Cidade |
| `uf` | string | Sim | UF |
| `tel` | string | Sim | Telefone |
| `cel` | string | Sim | Celular |
| `email` | string | Sim | E-mail |
| `site` | string | Sim | Site |
| `logo` | file | Não | Nova imagem (substitui a anterior) |

**Resposta de sucesso `200`:**

```json
{
  "message": "Instituição atualizada com sucesso.",
  "data": { ... }
}
```

**Resposta de erro `404`:**

```json
{
  "message": "Instituição não cadastrada. Utilize POST para criar."
}
```

---

## DELETE /instituicao

Remove a instituição cadastrada.

**Resposta de sucesso `200`:**

```json
{
  "message": "Instituição removida com sucesso."
}
```

**Resposta de erro `404`:**

```json
{
  "message": "Instituição não cadastrada."
}
```
