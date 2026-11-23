# Usuários

Base URL: `/api/{ente}/users`

> Formatos comuns (paginação `?per_page=`/`?q=`, `{ data, message }`, números, erros): [padroes.md](padroes.md).

**Permissões:** listar/ver exige `usuarios.ver` (admin, gerente); criar, editar e excluir exigem `usuarios.gerenciar` (só admin). Ver [permissoes.md](permissoes.md).

> **Senha provisória:** toda senha definida por um admin (ao criar o usuário ou ao trocar a senha de outra pessoa) é provisória — o usuário só acessa `/me`, `/logout` e `PUT /senha` até definir a própria senha (ver [auth.md](auth.md)). Trocar a senha de outra pessoa também encerra as sessões dela.

| Método | Rota | Descrição |
|--------|------|-----------|
| GET | `/users?role=&q=` | Lista paginada (ordem alfabética); `role` filtra pelo perfil; `q` busca em nome e e-mail |
| GET | `/users/{id}` | Detalhe |
| POST | `/users` | Cria (`201`) |
| PUT/PATCH | `/users/{id}` | Atualiza só os campos enviados |
| DELETE | `/users/{id}` | Remove e encerra as sessões dele |

## Usuário

```json
{
  "data": {
    "id": 2,
    "name": "Maria Souza",
    "email": "maria@prefeitura.gov.br",
    "role": "compras",
    "senha_provisoria": true,
    "created_at": "2026-01-01T00:00:00.000000Z",
    "updated_at": "2026-01-01T00:00:00.000000Z"
  },
  "message": "Usuário criado com sucesso."
}
```

## Body

| Campo | Tipo | POST | PUT/PATCH | Regra |
|-------|------|------|-----------|-------|
| `name` | string | Sim | Não | máx. 255 |
| `email` | string | Sim | Não | e-mail único na instituição |
| `password` + `password_confirmation` | string | Sim | Não | mín. 8, letras e números; provisória se for de outra pessoa |
| `role` | string | Sim | Não | `admin`, `gerente`, `licitacao` ou `compras` |

## Regras

- Ninguém altera o próprio perfil (`422` em `role`) nem exclui a própria conta (`403`).
- A instituição nunca fica sem `admin`: rebaixar ou excluir o último admin dá `422`.
- Exclusão: `200` com `{ "message": "Usuário removido com sucesso." }`.
