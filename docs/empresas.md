# Empresas

Base URL: `/api/empresas`

**Autenticação:** Todas as rotas requerem `Bearer {token}` no header `Authorization`.

---

## GET /empresas

Lista todas as empresas.

**Resposta de sucesso `200`:**

```json
{
  "data": [
    {
      "id": 1,
      "cpfcnpj": "00.000.000/0001-00",
      "razaosocial": "Empresa Exemplo Ltda",
      "endereco": "Rua Exemplo, 123",
      "bairro": "Centro",
      "cidade": "Cidade",
      "uf": "SP",
      "tel": "(11) 1234-5678",
      "email": "contato@empresa.com.br",
      "created_at": "2026-01-01T00:00:00.000000Z",
      "updated_at": "2026-01-01T00:00:00.000000Z"
    }
  ]
}
```

---

## GET /empresas/{id}

Retorna uma empresa específica.

**Resposta de sucesso `200`:**

```json
{
  "data": {
    "id": 1,
    "cpfcnpj": "00.000.000/0001-00",
    "razaosocial": "Empresa Exemplo Ltda",
    "endereco": "Rua Exemplo, 123",
    "bairro": "Centro",
    "cidade": "Cidade",
    "uf": "SP",
    "tel": "(11) 1234-5678",
    "email": "contato@empresa.com.br",
    "created_at": "2026-01-01T00:00:00.000000Z",
    "updated_at": "2026-01-01T00:00:00.000000Z"
  }
}
```

---

## POST /empresas

Cria uma nova empresa.

**Body (JSON):**

| Campo | Tipo | Obrigatório | Descrição |
|-------|------|-------------|-----------|
| `cpfcnpj` | string | Sim | CPF ou CNPJ |
| `razaosocial` | string | Sim | Razão social |
| `endereco` | string | Sim | Endereço |
| `bairro` | string | Sim | Bairro |
| `cidade` | string | Sim | Cidade |
| `uf` | string | Sim | UF (ex: SP) |
| `tel` | string | Sim | Telefone |
| `email` | string | Não | E-mail |

**Resposta de sucesso `201`:**

```json
{
  "message": "Empresa criada com sucesso.",
  "data": { ... }
}
```

---

## PUT/PATCH /empresas/{id}

Atualiza uma empresa existente.

**Body (JSON):**

| Campo | Tipo | Obrigatório | Descrição |
|-------|------|-------------|-----------|
| `cpfcnpj` | string | Sim | CPF ou CNPJ |
| `razaosocial` | string | Sim | Razão social |
| `endereco` | string | Sim | Endereço |
| `bairro` | string | Sim | Bairro |
| `cidade` | string | Sim | Cidade |
| `uf` | string | Sim | UF |
| `tel` | string | Sim | Telefone |
| `email` | string | Não | E-mail |

**Resposta de sucesso `200`:**

```json
{
  "message": "Empresa atualizada com sucesso.",
  "data": { ... }
}
```

---

## DELETE /empresas/{id}

Remove uma empresa.

**Resposta de sucesso `200`:**

```json
{
  "message": "Empresa removida com sucesso."
}
```
