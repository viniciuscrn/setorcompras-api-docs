# Secretarias

Base URL: `/api/secretarias`

**Autenticação:** Todas as rotas requerem `Bearer {token}` no header `Authorization`.

---

## GET /secretarias

Lista todas as secretarias com seus órgãos e responsáveis.

**Resposta de sucesso `200`:**

```json
{
  "data": [
    {
      "id": 1,
      "nome": "Secretaria de Saúde",
      "cnpj": "00.000.000/0001-00",
      "tel": "(11) 1234-5678",
      "created_at": "2026-01-01T00:00:00.000000Z",
      "updated_at": "2026-01-01T00:00:00.000000Z",
      "orgaos": [
        {
          "id": 1,
          "nome": "UBS Central",
          "endereco": "Rua Exemplo, 100",
          "secretaria_id": 1,
          "responsaveis": [
            {
              "id": 1,
              "nome": "João Silva",
              "cargo": "Diretor",
              "orgaos_id": 1
            }
          ]
        }
      ]
    }
  ]
}
```

---

## GET /secretarias/{id}

Retorna uma secretaria com seus órgãos e responsáveis.

**Resposta de sucesso `200`:**

```json
{
  "data": {
    "id": 1,
    "nome": "Secretaria de Saúde",
    "cnpj": "00.000.000/0001-00",
    "tel": "(11) 1234-5678",
    "created_at": "2026-01-01T00:00:00.000000Z",
    "updated_at": "2026-01-01T00:00:00.000000Z",
    "orgaos": [ ... ]
  }
}
```

---

## POST /secretarias

Cria uma nova secretaria.

**Body (JSON):**

| Campo | Tipo | Obrigatório | Descrição |
|-------|------|-------------|-----------|
| `nome` | string | Sim | Nome da secretaria |
| `cnpj` | string | Sim | CNPJ |
| `tel` | string | Sim | Telefone |

**Resposta de sucesso `201`:**

```json
{
  "message": "Secretaria criada com sucesso.",
  "data": {
    "id": 1,
    "nome": "Secretaria de Saúde",
    "cnpj": "00.000.000/0001-00",
    "tel": "(11) 1234-5678",
    "created_at": "2026-01-01T00:00:00.000000Z",
    "updated_at": "2026-01-01T00:00:00.000000Z"
  }
}
```

---

## PUT/PATCH /secretarias/{id}

Atualiza uma secretaria existente.

**Body (JSON):**

| Campo | Tipo | Obrigatório | Descrição |
|-------|------|-------------|-----------|
| `nome` | string | Sim | Nome da secretaria |
| `cnpj` | string | Sim | CNPJ |
| `tel` | string | Sim | Telefone |

**Resposta de sucesso `200`:**

```json
{
  "message": "Secretaria atualizada com sucesso.",
  "data": { ... }
}
```

---

## DELETE /secretarias/{id}

Remove uma secretaria.

**Resposta de sucesso `200`:**

```json
{
  "message": "Secretaria removida com sucesso."
}
```
