# Ordem de Implementação dos Endpoints — Frontend

Este documento define a ordem recomendada para o frontend implementar a integração com a API, respeitando as dependências entre os recursos.

> **Multi-instituição:** cada instituição (ente) tem banco próprio e é identificada pelo slug na URL — as rotas das etapas 1 a 9 ficam em `/api/{ente}/...` (ex.: `/api/agua-branca/login`). O token emitido em um ente só vale nele.
>
> **Duas áreas independentes:** a **área de admin** (`/api/admin/...`, Etapa 0), onde a equipe do SetorCompras cadastra as instituições, e a **área da instituição** (`/api/{ente}/...`, etapas 1 a 9), usada pela prefeitura. Cada uma tem o seu login; o token de uma não vale na outra (`401`). Podem ser dois módulos do mesmo front ou dois apps.

---

## Visão Geral das Dependências

```
Admin (área de administração — /api/admin)
 └── Instituições (cria o ente + admin da instituição com senha provisória)

Auth da instituição (/api/{ente})
 └── Instituição (configuração inicial)
 └── Cadastros Base (independentes entre si)
      ├── Secretarias
      ├── Órgãos
      ├── Responsáveis
      ├── Empresas
      ├── Modalidades
      ├── Produtos (catálogo)
      ├── Naturezas de despesa
      └── Usuários
           └── Licitações (requer Modalidade)
                └── Empresas x Licitações (requer Empresa + Licitação)
                     └── Itens do contrato (requer EmpresaLicitação + Produto)
                          └── Aditivos (requer item do contrato)
Solicitações (requer Empresa + Responsável + Contrato vigente OU Natureza)
 └── Itens da solicitação (licitada: item do contrato; direta: produto + preço)
```

---

## Etapa 0 — Área de administração (cadastro de instituições)

**Prioridade:** Primeira a implementar — sem uma instituição cadastrada não há onde entrar nas etapas seguintes.
Base: `/api/admin`. Login próprio (administrador do sistema), separado dos usuários das instituições.

| Endpoint | Método | Descrição |
| -------- | ------ | --------- |
| `/api/admin/login` | POST | Login do administrador do sistema, retorna token Bearer |
| `/api/admin/logout` | POST | Logout e revogação do token |
| `/api/admin/me` | GET | Dados do administrador logado |
| `/api/admin/entes` | GET | Lista paginada (`q`, `ativo`, `per_page`) |
| `/api/admin/entes` | POST | Cria a instituição (banco, perfis, admin da instituição) — **mostrar a senha provisória uma única vez** |
| `/api/admin/entes/{slug}` | GET | Detalhe |
| `/api/admin/entes/{slug}` | PUT/PATCH | Edita nome, UF, CNPJ, código IBGE (o slug é fixo) |
| `/api/admin/entes/{slug}/desativar` | POST | Bloqueia o acesso da instituição (sem apagar dados) |
| `/api/admin/entes/{slug}/reativar` | POST | Libera o acesso de novo |
| `/api/admin/entes/{slug}/resetar-admin` | POST | Nova senha provisória para o admin da instituição |
| `/api/admin/entes/{slug}/status` | GET | Usuários e migrations pendentes |

> Telas sugeridas: login · lista de instituições (busca, filtro ativas/desativadas) · cadastro (o POST leva alguns segundos:
> mostrar carregamento e não reenviar) · modal com a senha provisória e o `url_api` para repassar à instituição ·
> edição · ações de desativar/reativar e resetar admin.
>
> `POST /api/admin/entes/migrar` é operacional (usado após deploy); não precisa de tela.

Consulte: [docs/admin.md](admin.md)

---

## Etapa 1 — Autenticação

**Prioridade:** Obrigatório antes de qualquer outra tela.

| Endpoint      | Método | Descrição                              |
| ------------- | ------ | -------------------------------------- |
| `/api/{ente}/login`  | POST   | Login do usuário, retorna token Bearer |
| `/api/{ente}/logout` | POST   | Logout e revogação do token            |
| `/api/{ente}/me`     | GET    | Dados do usuário autenticado (perfil, permissões, alertas) |
| `/api/{ente}/senha`  | PUT    | Troca da própria senha (obrigatória se `senha_provisoria`) |

> Todos os demais endpoints exigem o header `Authorization: Bearer {token}`. Paginação, formato das respostas, tipos e erros: [padroes.md](padroes.md).
>
> Depois do login: se `user.senha_provisoria` for `true`, a primeira tela é a troca de senha. Use `user.permissoes` para montar menus e botões — cada perfil vê só o que pode usar ([permissoes.md](permissoes.md)).

Consulte: [docs/auth.md](auth.md)

---

## Etapa 2 — Configuração Inicial da Instituição

**Prioridade:** Fazer logo após o login. A instituição já nasce com nome, CNPJ e UF (criada pelo admin); a API retorna um alerta enquanto endereço, cidade, e-mail ou brasão não forem preenchidos.

| Endpoint           | Método    | Descrição                  |
| ------------------ | --------- | -------------------------- |
| `/api/{ente}/instituicao` | GET       | Busca dados da instituição |
| `/api/{ente}/instituicao` | POST      | Atualiza com brasão (multipart) |
| `/api/{ente}/instituicao` | PUT/PATCH | Atualiza os dados          |

Consulte: [docs/instituicao.md](instituicao.md)

---

## Etapa 3 — Cadastros Base (sem dependências)

Estes módulos são independentes entre si e podem ser implementados em paralelo. Devem ser feitos antes dos módulos de Licitação e Solicitação.

### 3.1 Secretarias

| Endpoint                | Método    | Descrição         |
| ----------------------- | --------- | ----------------- |
| `/api/{ente}/secretarias`      | GET       | Listagem paginada |
| `/api/{ente}/secretarias`      | POST      | Criar             |
| `/api/{ente}/secretarias/{id}` | GET       | Detalhe           |
| `/api/{ente}/secretarias/{id}` | PUT/PATCH | Atualizar         |
| `/api/{ente}/secretarias/{id}` | DELETE    | Excluir           |

Consulte: [docs/secretarias.md](secretarias.md)

### 3.2 Órgãos

| Endpoint           | Método    | Descrição         |
| ------------------ | --------- | ----------------- |
| `/api/{ente}/orgaos`      | GET       | Listagem paginada |
| `/api/{ente}/orgaos`      | POST      | Criar             |
| `/api/{ente}/orgaos/{id}` | GET       | Detalhe           |
| `/api/{ente}/orgaos/{id}` | PUT/PATCH | Atualizar         |
| `/api/{ente}/orgaos/{id}` | DELETE    | Excluir           |

Consulte: [docs/orgaos.md](orgaos.md)

### 3.3 Responsáveis

| Endpoint                 | Método    | Descrição         |
| ------------------------ | --------- | ----------------- |
| `/api/{ente}/responsaveis`      | GET       | Listagem paginada |
| `/api/{ente}/responsaveis`      | POST      | Criar             |
| `/api/{ente}/responsaveis/{id}` | GET       | Detalhe           |
| `/api/{ente}/responsaveis/{id}` | PUT/PATCH | Atualizar         |
| `/api/{ente}/responsaveis/{id}` | DELETE    | Excluir           |

Consulte: [docs/responsaveis.md](responsaveis.md)

### 3.4 Empresas

| Endpoint             | Método    | Descrição         |
| -------------------- | --------- | ----------------- |
| `/api/{ente}/empresas`      | GET       | Listagem paginada |
| `/api/{ente}/empresas`      | POST      | Criar             |
| `/api/{ente}/empresas/{id}` | GET       | Detalhe           |
| `/api/{ente}/empresas/{id}` | PUT/PATCH | Atualizar         |
| `/api/{ente}/empresas/{id}` | DELETE    | Excluir           |

Consulte: [docs/empresas.md](empresas.md)

### 3.5 Modalidades

| Endpoint                | Método    | Descrição         |
| ----------------------- | --------- | ----------------- |
| `/api/{ente}/modalidades`      | GET       | Listagem paginada |
| `/api/{ente}/modalidades`      | POST      | Criar             |
| `/api/{ente}/modalidades/{id}` | GET       | Detalhe           |
| `/api/{ente}/modalidades/{id}` | PUT/PATCH | Atualizar         |
| `/api/{ente}/modalidades/{id}` | DELETE    | Excluir           |

Consulte: [docs/modalidades.md](modalidades.md)

### 3.6 Produtos

| Endpoint             | Método    | Descrição         |
| -------------------- | --------- | ----------------- |
| `/api/{ente}/produtos`      | GET       | Listagem paginada |
| `/api/{ente}/produtos`      | POST      | Criar             |
| `/api/{ente}/produtos/{id}` | GET       | Detalhe           |
| `/api/{ente}/produtos/{id}` | PUT/PATCH | Atualizar         |
| `/api/{ente}/produtos/{id}` | DELETE    | Excluir           |

Consulte: [docs/produtos.md](produtos.md)

### 3.6b Naturezas de despesa

Limite anual de compra direta por objeto/natureza. Necessárias antes da primeira solicitação direta.

| Endpoint             | Método    | Descrição         |
| -------------------- | --------- | ----------------- |
| `/api/{ente}/naturezas`      | GET       | Listagem (`?ativo=1`) |
| `/api/{ente}/naturezas`      | POST      | Criar             |
| `/api/{ente}/naturezas/{id}` | GET       | Detalhe           |
| `/api/{ente}/naturezas/{id}` | PUT/PATCH | Atualizar         |
| `/api/{ente}/naturezas/{id}` | DELETE    | Excluir (`409` se usada — desative) |

Consulte: [docs/naturezas.md](naturezas.md)

### 3.7 Usuários

| Endpoint          | Método    | Descrição         |
| ----------------- | --------- | ----------------- |
| `/api/{ente}/users`      | GET       | Listagem paginada |
| `/api/{ente}/users`      | POST      | Criar             |
| `/api/{ente}/users/{id}` | GET       | Detalhe           |
| `/api/{ente}/users/{id}` | PUT/PATCH | Atualizar         |
| `/api/{ente}/users/{id}` | DELETE    | Excluir           |

Consulte: [docs/users.md](users.md)

---

## Etapa 4 — Licitações

**Depende de:** Modalidades (Etapa 3.5)

| Endpoint               | Método    | Descrição                            |
| ---------------------- | --------- | ------------------------------------ |
| `/api/{ente}/licitacoes`      | GET       | Listagem (retorna `{ data: [...] }`) |
| `/api/{ente}/licitacoes`      | POST      | Criar (requer `modalidade_id`)       |
| `/api/{ente}/licitacoes/{id}` | GET       | Detalhe                              |
| `/api/{ente}/licitacoes/{id}` | PUT/PATCH | Atualizar                            |
| `/api/{ente}/licitacoes/{id}` | DELETE    | Excluir                              |

Consulte: [docs/licitacoes.md](licitacoes.md)

---

## Etapa 5 — Empresas x Licitações (Vencedores/Contratos)

**Depende de:** Empresas (3.4) + Licitações (Etapa 4)

| Endpoint                        | Método    | Descrição                                            |
| ------------------------------- | --------- | ---------------------------------------------------- |
| `/api/{ente}/empresas-licitacoes`      | GET       | Listagem (filtros: `?licitacao_id=`, `?empresa_id=`) |
| `/api/{ente}/empresas-licitacoes`      | POST      | Vincular empresa vencedora à licitação               |
| `/api/{ente}/empresas-licitacoes/{id}` | GET       | Detalhe                                              |
| `/api/{ente}/empresas-licitacoes/{id}` | PUT/PATCH | Atualizar                                            |
| `/api/{ente}/empresas-licitacoes/{id}` | DELETE    | Excluir                                              |

Consulte: [docs/empresas-licitacoes.md](empresas-licitacoes.md)

---

## Etapa 6 — Licitações x Produtos (Itens do Contrato)

**Depende de:** Empresas x Licitações (Etapa 5) + Produtos (3.6)

| Endpoint                                             | Método | Descrição                                  |
| ---------------------------------------------------- | ------ | ------------------------------------------ |
| `/api/{ente}/empresas-licitacoes/{id}/produtos`             | GET    | Lista itens com saldo disponível calculado |
| `/api/{ente}/empresas-licitacoes/{id}/produtos`             | POST   | Incluir item (produto + quant, preço, marca) |
| `/api/{ente}/empresas-licitacoes/{id}/produtos/{itemId}`    | PUT    | Alterar item                               |
| `/api/{ente}/empresas-licitacoes/{id}/produtos/{itemId}`    | DELETE | Remover item do contrato                   |
| `/api/{ente}/empresas-licitacoes/{id}/produtos/preview`     | POST   | Preview da importação CSV (sem salvar)     |
| `/api/{ente}/empresas-licitacoes/{id}/produtos/importar`    | POST   | Confirmar e importar produtos via CSV      |

> **Sobre o CSV:** O fluxo é sempre preview → importar. Nunca chame `/importar` sem antes apresentar o resultado do `/preview` ao usuário.

Consulte: [docs/licitacoes-produtos.md](licitacoes-produtos.md)

---

## Etapa 7 — Aditivos

**Depende de:** Licitações x Produtos (Etapa 6)

| Endpoint             | Método    | Descrição                                     |
| -------------------- | --------- | --------------------------------------------- |
| `/api/{ente}/aditivos`      | GET       | Listagem                                      |
| `/api/{ente}/aditivos`      | POST      | Criar aditivo (requer `licitacao_produto_id`) |
| `/api/{ente}/aditivos/{id}` | GET       | Detalhe                                       |
| `/api/{ente}/aditivos/{id}` | PUT/PATCH | Atualizar                                     |
| `/api/{ente}/aditivos/{id}` | DELETE    | Excluir                                       |

Consulte: [docs/aditivos.md](aditivos.md)

---

## Etapa 8 — Solicitações

**Depende de:** Empresas (3.4) + Responsáveis (3.3) + Contratos (Etapa 5, licitada) ou Naturezas (3.6b, direta)

| Endpoint                                | Método    | Descrição                                                                       |
| --------------------------------------- | --------- | ------------------------------------------------------------------------------- |
| `/api/{ente}/solicitacoes`                     | GET       | Listagem (filtros: `?empresa_id=`, `?responsavel_id=`, `?ehlicitada=`, `?ano=`) |
| `/api/{ente}/solicitacoes`                     | POST      | Criar (gera `guia` e `protocolo` automaticamente)                               |
| `/api/{ente}/solicitacoes/{id}`                | GET       | Detalhe                                                                         |
| `/api/{ente}/solicitacoes/{id}`                | PUT/PATCH | Atualizar                                                                       |
| `/api/{ente}/solicitacoes/{id}`                | DELETE    | Excluir                                                                         |
| `/api/{ente}/solicitacoes/totais-diretos?ano=` | GET       | Compras diretas do ano por natureza × limite anual                              |

> **Atenção:** A rota `/totais-diretos` deve ser chamada **antes** de `/solicitacoes/{id}` na configuração do roteador do frontend, pois é uma rota estática que pode colidir com o parâmetro `{id}`.

Consulte: [docs/solicitacoes.md](solicitacoes.md)

---

## Etapa 9 — Solicitações x Produtos (Itens da Solicitação)

**Depende de:** Solicitações (Etapa 8) + Itens do contrato (Etapa 6)

| Endpoint                                   | Método    | Descrição                       |
| ------------------------------------------ | --------- | ------------------------------- |
| `/api/{ente}/solicitacoes/{id}/produtos`          | GET       | Lista itens da solicitação      |
| `/api/{ente}/solicitacoes/{id}/produtos`          | POST      | Adicionar produto à solicitação |
| `/api/{ente}/solicitacoes/{id}/produtos/{itemId}` | PUT/PATCH | Atualizar item                  |
| `/api/{ente}/solicitacoes/{id}/produtos/{itemId}` | DELETE    | Remover item                    |

> Licitada: envie `licitacao_produto_id` + `quant` (preço = preço atual do item; quantidade ≤ saldo). Direta: `produto_id` + `quant` + `preco_unitario` (limite anual da natureza).

Consulte: [docs/solicitacoes.md](solicitacoes.md)

---

## Resumo da Ordem

| Etapa | Módulo                  | Depende de                       |
| ----- | ----------------------- | -------------------------------- |
| 0     | Área de admin (instituições) | —                           |
| 1     | Autenticação            | Instituição criada na etapa 0    |
| 2     | Instituição             | Auth                             |
| 3.1   | Secretarias             | —                                |
| 3.2   | Órgãos                  | —                                |
| 3.3   | Responsáveis            | —                                |
| 3.4   | Empresas                | —                                |
| 3.5   | Modalidades             | —                                |
| 3.6   | Produtos                | —                                |
| 3.6b  | Naturezas de despesa    | —                                |
| 3.7   | Usuários                | Auth                             |
| 4     | Licitações              | Modalidades                      |
| 5     | Empresas x Licitações   | Empresas + Licitações            |
| 6     | Licitações x Produtos   | Empresas x Licitações + Produtos |
| 7     | Aditivos                | Licitações x Produtos            |
| 8     | Solicitações            | Empresas + Responsáveis + Contratos / Naturezas |
| 9     | Solicitações x Produtos | Solicitações + Itens do contrato |
