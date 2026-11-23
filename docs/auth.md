# Auth

Base URL: `/api/{ente}`

> Todas as rotas desta página ficam sob o slug da instituição: `/api/{ente}/...` (ex.: `/api/agua-branca/...`). Veja [admin.md](admin.md).

---

## POST /login

Autentica o usuário e retorna o token de acesso.

**Autenticação:** Não requerida

**Body (JSON):**

| Campo | Tipo | Obrigatório | Descrição |
|-------|------|-------------|-----------|
| `email` | string | Sim | E-mail do usuário |
| `password` | string | Sim | Senha do usuário |
| `device_name` | string | Não | Nome do dispositivo (ex: "Web", "iPhone") |

**Resposta de sucesso `200`:**

```json
{
  "token": "1|abc123...",
  "user": {
    "id": 1,
    "name": "Nome do Usuário",
    "email": "usuario@email.com",
    "role": "admin",
    "permissoes": ["catalogo.gerenciar", "catalogo.ver", "estrutura.gerenciar", "..."],
    "senha_provisoria": false
  },
  "alertas": []
}
```

- `permissoes`: use para mostrar/esconder telas e botões (lista completa em [permissoes.md](permissoes.md)).
- `senha_provisoria: true` → leve o usuário direto para a troca de senha (`PUT /senha`); as demais rotas respondem `403` com `"codigo": "senha_provisoria"` até lá.
- `alertas`: pendências (senha provisória; cadastro da instituição incompleto — este só para quem pode corrigir). Cada item tem `tipo`, `mensagem`, `link`, `metodo` e, às vezes, `campos`.
- O token expira em 12 h (`SANCTUM_EXPIRATION`); depois disso as rotas respondem `401` e é preciso logar de novo.

**Resposta de erro `429`:** mais de 5 tentativas por minuto para o mesmo e-mail/IP (ou 20 por IP). Aguarde o tempo do header `Retry-After`.

**Resposta de erro `422`:**

```json
{
  "message": "As credenciais fornecidas estão incorretas.",
  "errors": {
    "email": ["As credenciais fornecidas estão incorretas."]
  }
}
```

---

## POST /logout

Invalida o token atual do usuário autenticado.

**Autenticação:** Requerida — `Bearer {token}`

**Resposta de sucesso `200`:**

```json
{
  "message": "Logout realizado com sucesso"
}
```

---

## GET /me

Retorna o usuário autenticado (mesmo formato de `user` no login) e os alertas. Funciona mesmo com senha provisória.

**Autenticação:** Requerida — `Bearer {token}`

**Resposta de sucesso `200`:**

```json
{
  "id": 1,
  "name": "Nome do Usuário",
  "email": "usuario@email.com",
  "role": "compras",
  "permissoes": ["catalogo.ver", "estrutura.ver", "instituicao.ver", "licitacoes.ver", "relatorios.ver", "solicitacoes.gerenciar", "solicitacoes.ver"],
  "senha_provisoria": false,
  "alertas": []
}
```

---

## PUT /senha

Troca a senha do próprio usuário. Obrigatório quando `senha_provisoria` é `true`. Encerra as **outras** sessões do usuário; o token atual continua valendo.

**Autenticação:** Requerida — `Bearer {token}` (liberado mesmo com senha provisória)

| Campo | Tipo | Obrigatório | Descrição |
|-------|------|-------------|-----------|
| `senha_atual` | string | Sim | Senha atual (ou a provisória) |
| `password` | string | Sim | Nova senha: mín. 8 caracteres, letras e números, diferente da atual |
| `password_confirmation` | string | Sim | Confirmação |

**`200`:** `{ "message": "Senha alterada com sucesso.", "user": { ...mesmo formato do login... } }` · **`422`:** senha atual incorreta ou nova senha inválida.

---

## Erros comuns em todas as rotas autenticadas

| Status | Quando |
|--------|--------|
| `401` | Sem token, token inválido, expirado ou de outra instituição |
| `403` + `"codigo": "senha_provisoria"` | Usuário precisa trocar a senha (`PUT /senha`) |
| `403` | Perfil sem permissão para a ação (ver [permissoes.md](permissoes.md)) |
| `404` | Instituição (slug) inexistente ou desativada |
