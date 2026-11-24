# Administração do sistema (instituições)

Base URL: `/api/admin`

Área da equipe que administra o SetorCompras. Usa o **banco central** e um login próprio (tabela `administradores`), separado dos usuários das instituições. Um token de admin não acessa rotas `/api/{ente}/...` e vice-versa (`401`).

Cada instituição é um **ente**: tem um `slug` (que vira o endereço `/api/{slug}/...`) e um banco de dados próprio, criado na hora do cadastro.

**Autenticação:** todas as rotas, exceto `/login`, exigem `Authorization: Bearer {token}` de um administrador.

---

## POST /login

| Campo | Tipo | Obrigatório |
|-------|------|-------------|
| `email` | string | Sim |
| `password` | string | Sim |
| `device_name` | string | Não |

**`200`:** `{ "token": "1|...", "user": { "id", "name", "email" } }` · **`422`:** credenciais incorretas · **`429`:** muitas tentativas.

## POST /logout · GET /me

Encerra o token atual · Dados do administrador logado.

---

## Instituição (formato)

Todas as rotas de instituição devolvem este objeto em `data`:

```json
{
  "id": 1, "slug": "agua-branca", "nome": "Prefeitura de Água Branca", "uf": "RN",
  "cnpj": "08123456000190", "codigo_ibge": "2400208", "ativo": true,
  "url_api": "https://setorcompras-api.viniciusm.com.br/api/agua-branca",
  "created_at": "2026-11-24T15:00:00.000000Z", "updated_at": "2026-11-24T15:00:00.000000Z"
}
```

`url_api` é a base das rotas da instituição (o que o front da instituição usa como base URL).
CNPJ e código IBGE saem só com dígitos. Formatos gerais (paginação, mensagens, erros): [padroes.md](padroes.md).

## GET /entes

Lista paginada no padrão da API (`{ data, links, meta }`), em ordem de nome.

| Parâmetro | Descrição |
|-----------|-----------|
| `q` | busca em nome, slug e CNPJ |
| `ativo` | `1` só ativas · `0` só desativadas |
| `per_page` · `page` | 1–100 (padrão 15) |

## POST /entes

Cria a instituição: registra o ente, **cria o banco**, roda as migrations, cria os perfis (`admin`, `gerente`, `licitacao`, `compras`), cria o registro da instituição e o usuário admin dela com **senha provisória**.
Leva alguns segundos — mostre um indicador de carregamento e não reenvie.

| Campo | Tipo | Obrigatório | Regras |
|-------|------|-------------|--------|
| `slug` | string | Sim | minúsculas, números e hífens (`agua-branca`), até 40; único; não pode ser `admin`, `api`, `login`, `logout`, `me`, `up`. **Não pode ser alterado depois.** |
| `nome` | string | Sim | até 191 |
| `uf` | string | Sim | sigla de estado válida (`rn` vira `RN`) |
| `cnpj` | string | Não | CNPJ válido, com ou sem máscara; gravado só com dígitos |
| `codigo_ibge` | string | Não | 7 dígitos |
| `admin.name` | string | Sim | admin da instituição |
| `admin.email` | string | Sim | |

**`201`:**

```json
{
  "data": { "id": 1, "slug": "agua-branca", "nome": "Prefeitura de Água Branca", "uf": "RN", "...": "..." },
  "message": "Instituição criada com sucesso. Repasse a senha provisória ao administrador da instituição.",
  "admin": { "email": "maria@aguabranca.gov.br", "senha_provisoria": "9cRx9lf5Y4gr" }
}
```

> A senha provisória só aparece nesta resposta. Se perder, use `resetar-admin`. No primeiro acesso o admin da instituição é obrigado a trocá-la (`PUT /api/{slug}/senha`).

**`422`** (em `errors`): slug fora do formato ("Use apenas letras minúsculas, números e hífens…"), reservado
("Este endereço é reservado pelo sistema…"), repetido ("Já existe uma instituição com este endereço."), UF/CNPJ/IBGE inválidos.

## GET /entes/{slug}

`{ data: instituição }` · `404` "Registro não encontrado.".

## PUT/PATCH /entes/{slug}

Edita `nome`, `uf`, `cnpj`, `codigo_ibge` — envie só o que mudar (mesmas regras do cadastro). `slug` e `admin` enviados são ignorados.
**`200`:** `{ data, message: "Instituição atualizada com sucesso." }`

## POST /entes/{slug}/desativar · POST /entes/{slug}/reativar

Desativar bloqueia todo o acesso da instituição (`/api/{slug}/...` passa a responder `404`) **sem apagar dados**. Reativar libera de novo.
**`200`:** `{ data, message }`.

> Não existe exclusão pela API. Apagar o banco de uma instituição só pelo terminal: `php artisan ente:excluir {slug}` (pede confirmação; faça backup antes).

## POST /entes/{slug}/resetar-admin

Gera nova senha provisória para um usuário com perfil `admin` da instituição e encerra todas as sessões (tokens) dele.

| Campo | Tipo | Obrigatório | Descrição |
|-------|------|-------------|-----------|
| `email` | string | Não | qual admin; sem ele, o primeiro admin cadastrado |

**`200`:** `{ "message": "...", "admin": { "email", "senha_provisoria" } }` · **`404`:** nenhum admin encontrado.

## GET /entes/{slug}/status

```json
{ "data": { "ente": "agua-branca", "ativo": true, "usuarios": 3, "migrations_pendentes": [] } }
```

## POST /entes/migrar

Aplica as migrations pendentes e sincroniza perfis/permissões (`MatrizPermissoes`) em todas as instituições — usar após um deploy com migrations novas ou mudança na matriz. **`200`:** `{ "data": [ { "ente": "agua-branca", "aplicadas": ["2026_..."] } ] }`.

---

## Comandos de terminal

| Comando | O que faz |
|---------|-----------|
| `php artisan migrate --seed` | Banco central + administrador inicial (`ADMIN_EMAIL` / `ADMIN_PASSWORD`) |
| `php artisan admin:criar {email}` | Novo administrador do sistema (senha gerada) |
| `php artisan ente:criar {slug} {nome} {uf} --admin-email=...` | Mesmo que `POST /entes` |
| `php artisan ente:migrar {slug?}` | Migrations de tenant + sincroniza perfis em um ou todos os entes |
| `php artisan ente:limpar-tokens` | Apaga tokens expirados (central e entes) — agendado diariamente às 03:00 |
| `php artisan ente:excluir {slug}` | **Apaga** o banco e o registro (irreversível) |
