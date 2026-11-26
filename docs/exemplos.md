# Exemplos de chamadas — área de admin e área da instituição

Respostas reais da API (geradas a partir do código atual; só tokens e senhas foram abreviados).
Regras completas de cada rota: [admin.md](admin.md), [auth.md](auth.md) e as páginas de cada recurso. Formatos gerais: [padroes.md](padroes.md).

| Área | Base (homologação) | Quem entra |
|------|--------------------|------------|
| Admin do sistema | `https://setorcompras-api.viniciusm.com.br/api/admin` | equipe do SetorCompras |
| Instituição | `https://setorcompras-api.viniciusm.com.br/api/{slug}` (o `url_api` do ente) | usuários da prefeitura |

Os dois logins são independentes: **o token de uma área não vale na outra** (`401`). Guarde-os separados.

---

## Cliente HTTP mínimo (fetch)

Um único cliente serve às duas áreas — muda só a base e o token.

```js
const HOST = 'https://setorcompras-api.viniciusm.com.br';

export function criarCliente(base, obterToken, aoSairDaSessao) {
  return async function api(caminho, { method = 'GET', body, query } = {}) {
    const url = new URL(`${HOST}${base}${caminho}`);
    Object.entries(query ?? {}).forEach(([k, v]) => v !== undefined && v !== '' && url.searchParams.set(k, v));

    const ehArquivo = body instanceof FormData;
    const resposta = await fetch(url, {
      method,
      headers: {
        Accept: 'application/json', // sempre
        ...(ehArquivo || !body ? {} : { 'Content-Type': 'application/json' }),
        ...(obterToken() ? { Authorization: `Bearer ${obterToken()}` } : {}),
      },
      body: ehArquivo ? body : body && JSON.stringify(body),
    });

    const json = await resposta.json().catch(() => ({}));
    if (resposta.ok) return json;

    if (resposta.status === 401) aoSairDaSessao(); // token inválido/expirado (12 h) → login
    throw { status: resposta.status, ...json, retryAfter: resposta.headers.get('Retry-After') };
  };
}

// Área de admin
export const admin = criarCliente('/api/admin', () => localStorage.getItem('token_admin'), () => irPara('/admin/login'));

// Área da instituição (slug escolhido no login, vindo do subdomínio ou da configuração)
export const ente = (slug) =>
  criarCliente(`/api/${slug}`, () => localStorage.getItem(`token_${slug}`), () => irPara(`/${slug}/login`));
```

Tratamento dos erros (`catch (e)`):

| `e.status` | O que fazer |
|-----------|-------------|
| `422` | `e.errors` = `{ campo: [mensagens] }` → mostrar ao lado de cada campo; `e.message` serve de resumo |
| `403` com `e.codigo === 'senha_provisoria'` | levar para a tela de troca de senha |
| `403` | perfil sem permissão → mostrar `e.message` |
| `404` | `e.message` ("Registro não encontrado." / "Rota não encontrada.") — ou instituição desativada |
| `409` | registro em uso, não pode excluir → mostrar `e.message` |
| `429` | muitas tentativas → aguardar `e.retryAfter` segundos |
| `500` | "Erro interno. Tente novamente ou contate o suporte." |

---

# Parte 1 — Área de admin (cadastro de instituições)

### Login

```js
const { token, user } = await admin('/login', {
  method: 'POST',
  body: { email: 'equipe@setorcompras.com', password: '********', device_name: 'web-admin' },
});
localStorage.setItem('token_admin', token);
```

```json
{
  "token": "1|LdMo07bY...",
  "user": { "id": 1, "name": "Equipe SetorCompras", "email": "equipe@setorcompras.com" }
}
```

`GET /me` devolve o mesmo objeto `user` (sem `data`). `POST /logout` → `{ "message": "Logout realizado com sucesso" }`.

### Cadastrar instituição

```js
const r = await admin('/entes', {
  method: 'POST',
  body: {
    slug: 'agua-branca',
    nome: 'Prefeitura de Água Branca',
    uf: 'rn',
    cnpj: '08.170.862/0001-74',   // com ou sem máscara
    codigo_ibge: '2400208',
    admin: { name: 'Maria Souza', email: 'maria@aguabranca.rn.gov.br' },
  },
});
// Mostrar UMA vez, num modal: r.admin.email, r.admin.senha_provisoria e r.data.url_api
```

`201`:

```json
{
  "data": {
    "id": 1,
    "slug": "agua-branca",
    "nome": "Prefeitura de Água Branca",
    "uf": "RN",
    "cnpj": "08170862000174",
    "codigo_ibge": "2400208",
    "ativo": true,
    "url_api": "https://setorcompras-api.viniciusm.com.br/api/agua-branca",
    "created_at": "2026-11-26T15:36:17.000000Z",
    "updated_at": "2026-11-26T15:36:17.000000Z"
  },
  "message": "Instituição criada com sucesso. Repasse a senha provisória ao administrador da instituição.",
  "admin": { "email": "maria@aguabranca.rn.gov.br", "senha_provisoria": "aJ4mvZ4E...." }
}
```

> O cadastro cria um banco de dados: leva alguns segundos. Desabilite o botão e mostre carregamento.

`422` (slug com espaço e maiúscula, sem nome, UF inválida, admin incompleto):

```json
{
  "message": "Use apenas letras minúsculas, números e hífens (ex.: agua-branca). (e mais 4 erros)",
  "errors": {
    "slug": ["Use apenas letras minúsculas, números e hífens (ex.: agua-branca)."],
    "nome": ["É obrigatória a indicação de um valor para o campo nome."],
    "uf": ["O UF selecionado é inválido."],
    "admin.name": ["É obrigatória a indicação de um valor para o campo nome do administrador."],
    "admin.email": ["O campo e-mail do administrador não contém um endereço de e-mail válido."]
  }
}
```

Dica: gere o slug a partir do nome (minúsculas, sem acento, espaços → hífen) e deixe o usuário ajustar — ele **não muda depois**.

### Listar (busca, filtro e paginação)

```js
const lista = await admin('/entes', { query: { q: 'agua', ativo: 1, page: 1, per_page: 10 } });
// lista.data → linhas da tabela; lista.meta.total, lista.meta.last_page → paginação
```

```json
{
  "data": [
    { "id": 1, "slug": "agua-branca", "nome": "Prefeitura de Água Branca", "uf": "RN", "cnpj": "08170862000174",
      "codigo_ibge": "2400208", "ativo": true, "url_api": "https://setorcompras-api.viniciusm.com.br/api/agua-branca",
      "created_at": "2026-11-26T15:36:17.000000Z", "updated_at": "2026-11-26T15:36:17.000000Z" }
  ],
  "links": { "first": "...?page=1", "last": "...?page=1", "prev": null, "next": null },
  "meta": { "current_page": 1, "from": 1, "last_page": 1, "per_page": 10, "to": 1, "total": 1, "path": "...", "links": [ ... ] }
}
```

### Detalhe, edição, desativar/reativar

```js
await admin('/entes/agua-branca');                                                    // { data }
await admin('/entes/agua-branca', { method: 'PATCH', body: { nome: 'Município de Água Branca' } });
await admin('/entes/agua-branca/desativar', { method: 'POST' });
await admin('/entes/agua-branca/reativar', { method: 'POST' });
```

Edição e desativar/reativar respondem `{ data, message }`:

```json
{
  "data": { "id": 1, "slug": "agua-branca", "nome": "Município de Água Branca", "ativo": false, "...": "..." },
  "message": "Instituição desativada. O acesso foi bloqueado e os dados foram mantidos."
}
```

Desativada, a instituição responde `404` em todas as rotas `/api/agua-branca/...` até ser reativada.

### Resetar a senha do admin da instituição

```js
const r = await admin('/entes/agua-branca/resetar-admin', { method: 'POST', body: { email: 'maria@aguabranca.rn.gov.br' } });
```

```json
{
  "message": "Senha redefinida e sessões encerradas.",
  "admin": { "email": "maria@aguabranca.rn.gov.br", "senha_provisoria": "9cRbwmLi...." }
}
```

### Status

```js
await admin('/entes/agua-branca/status');
```

```json
{ "data": { "ente": "agua-branca", "ativo": true, "usuarios": 1, "migrations_pendentes": [] } }
```

---

# Parte 2 — Área da instituição (ex.: `agua-branca`)

```js
const api = ente('agua-branca');
```

### 1. Primeiro login (senha provisória)

```js
const r = await api('/login', {
  method: 'POST',
  body: { email: 'maria@aguabranca.rn.gov.br', password: senhaProvisoria, device_name: 'web' },
});
localStorage.setItem('token_agua-branca', r.token);
if (r.user.senha_provisoria) irPara('/agua-branca/trocar-senha');
```

```json
{
  "token": "1|Llbkyh1M...",
  "user": {
    "id": 1,
    "name": "Maria Souza",
    "email": "maria@aguabranca.rn.gov.br",
    "role": "admin",
    "permissoes": [
      "catalogo.gerenciar", "catalogo.ver", "estrutura.gerenciar", "estrutura.ver",
      "instituicao.gerenciar", "instituicao.ver", "licitacoes.gerenciar", "licitacoes.ver",
      "relatorios.ver", "solicitacoes.gerenciar", "solicitacoes.ver", "usuarios.gerenciar", "usuarios.ver"
    ],
    "senha_provisoria": true
  },
  "alertas": [
    { "tipo": "warning", "mensagem": "Sua senha é provisória. Defina uma nova senha para usar o sistema.",
      "link": "/api/agua-branca/senha", "metodo": "PUT" }
  ]
}
```

Enquanto a senha for provisória, só `/me`, `/logout` e `PUT /senha` funcionam. Qualquer outra rota:

```json
{ "message": "Defina uma nova senha antes de continuar.", "codigo": "senha_provisoria" }
```

(status `403`)

### 2. Trocar a senha

```js
const r = await api('/senha', {
  method: 'PUT',
  body: { senha_atual: senhaProvisoria, password: 'NovaSenha#2026', password_confirmation: 'NovaSenha#2026' },
});
// O token atual continua valendo; as outras sessões do usuário são encerradas.
```

```json
{
  "message": "Senha alterada com sucesso.",
  "user": { "id": 1, "name": "Maria Souza", "email": "maria@aguabranca.rn.gov.br", "role": "admin",
            "permissoes": [ "..." ], "senha_provisoria": false }
}
```

### 3. Quem sou eu / montar o menu

```js
const eu = await api('/me');
const pode = (p) => eu.permissoes.includes(p);   // ex.: pode('catalogo.gerenciar') mostra "Nova empresa"
```

```json
{
  "id": 1, "name": "Maria Souza", "email": "maria@aguabranca.rn.gov.br", "role": "admin",
  "permissoes": [ "..." ],
  "senha_provisoria": false,
  "alertas": [
    { "tipo": "warning",
      "mensagem": "Complete o cadastro da instituição (endereço, cidade, e-mail e brasão) para emitir os documentos.",
      "link": "/api/agua-branca/instituicao", "metodo": "PUT",
      "campos": ["endereco", "cidade", "email", "logo"] }
  ]
}
```

Mapa de permissões por perfil: [permissoes.md](permissoes.md).

### 4. Completar a instituição

```js
await api('/instituicao', {
  method: 'PUT',
  body: { nome: 'Prefeitura de Água Branca', cnpj: '08170862000174', endereco: 'Rua Principal, 100',
          bairro: 'Centro', cidade: 'Água Branca', uf: 'RN', email: 'compras@aguabranca.rn.gov.br' },
});

// Brasão: multipart com o arquivo em "logo" (POST, pois PUT com arquivo não funciona em PHP)
const fd = new FormData();
fd.append('nome', 'Prefeitura de Água Branca');
fd.append('logo', arquivo);
await api('/instituicao', { method: 'POST', body: fd });
```

```json
{
  "data": {
    "id": 1, "nome": "Prefeitura de Água Branca", "cnpj": "08170862000174",
    "endereco": "Rua Principal, 100", "bairro": "Centro", "cidade": "Água Branca", "uf": "RN",
    "tel": null, "cel": null, "email": "compras@aguabranca.rn.gov.br", "site": null,
    "logo_url": null,
    "created_at": "2026-11-26T15:36:18.000000Z", "updated_at": "2026-11-26T15:36:18.000000Z"
  },
  "message": "Instituição atualizada com sucesso."
}
```

### 5. Um cadastro típico: empresas

O mesmo padrão vale para secretarias, órgãos, responsáveis, modalidades, produtos, naturezas, licitações etc.

```js
// Criar
const { data: empresa } = await api('/empresas', {
  method: 'POST',
  body: { razaosocial: 'Papelaria Central Ltda', cpfcnpj: '11.222.333/0001-81', endereco: 'Av. Rio Branco, 500',
          bairro: 'Centro', cidade: 'Natal', uf: 'rn', tel: '(84) 3222-1000' },
});

// Listar com busca e paginação
const lista = await api('/empresas', { query: { q: 'papel', page: 1, per_page: 15 } });

// Detalhe, editar, excluir
await api(`/empresas/${empresa.id}`);
await api(`/empresas/${empresa.id}`, { method: 'PUT', body: { ...dadosDoFormulario } });
await api(`/empresas/${empresa.id}`, { method: 'DELETE' });   // { message } — ou 409 se estiver em uso
```

Criar → `201`:

```json
{
  "data": {
    "id": 1,
    "cpfcnpj": "11222333000181",
    "tipo_pessoa": "juridica",
    "razaosocial": "Papelaria Central Ltda",
    "endereco": "Av. Rio Branco, 500",
    "bairro": "Centro",
    "cidade": "Natal",
    "uf": "RN",
    "tel": "8432221000",
    "email": null,
    "created_at": "2026-11-26T15:36:28.000000Z",
    "updated_at": "2026-11-26T15:36:28.000000Z"
  },
  "message": "Empresa criada com sucesso."
}
```

CPF/CNPJ e telefone voltam **só com dígitos** — aplique a máscara na exibição.

Listar → `{ data: [...], links, meta }` (mesmo formato da listagem de instituições).

`422` (CPF/CNPJ inválido e campos faltando):

```json
{
  "message": "O CPF/CNPJ informado é inválido. (e mais 6 erros)",
  "errors": {
    "cpfcnpj": ["O CPF/CNPJ informado é inválido."],
    "razaosocial": ["É obrigatória a indicação de um valor para o campo razão social."],
    "endereco": ["É obrigatória a indicação de um valor para o campo endereço."],
    "bairro": ["É obrigatória a indicação de um valor para o campo bairro."],
    "cidade": ["É obrigatória a indicação de um valor para o campo cidade."],
    "uf": ["É obrigatória a indicação de um valor para o campo UF."],
    "tel": ["É obrigatória a indicação de um valor para o campo telefone."]
  }
}
```

`404` → `{ "message": "Registro não encontrado." }`

### 6. Sair

```js
await api('/logout', { method: 'POST' });   // { "message": "Logout realizado com sucesso" }
localStorage.removeItem('token_agua-branca');
```

---

## O mesmo pelo terminal (curl)

```bash
# Admin
curl -s -X POST https://setorcompras-api.viniciusm.com.br/api/admin/login \
  -H "Accept: application/json" -H "Content-Type: application/json" \
  -d '{"email":"equipe@setorcompras.com","password":"********"}'

curl -s https://setorcompras-api.viniciusm.com.br/api/admin/entes?q=agua \
  -H "Accept: application/json" -H "Authorization: Bearer TOKEN_ADMIN"

# Instituição
curl -s -X POST https://setorcompras-api.viniciusm.com.br/api/homologacao/login \
  -H "Accept: application/json" -H "Content-Type: application/json" \
  -d '{"email":"admin@exemplo.com","password":"********"}'

curl -s https://setorcompras-api.viniciusm.com.br/api/homologacao/empresas?per_page=5 \
  -H "Accept: application/json" -H "Authorization: Bearer TOKEN_DA_INSTITUICAO"
```
