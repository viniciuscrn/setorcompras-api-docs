# Ordem de Implementação dos Endpoints — Frontend

Este documento define a ordem recomendada para o frontend implementar a integração com a API, respeitando as dependências entre os recursos.

---

## Visão Geral das Dependências

```
Auth
 └── Instituição (configuração inicial)
 └── Cadastros Base (independentes entre si)
      ├── Secretarias
      ├── Órgãos
      ├── Responsáveis
      ├── Empresas
      ├── Modalidades
      ├── Produtos
      └── Usuários
           └── Licitações (requer Modalidade)
                └── Empresas x Licitações (requer Empresa + Licitação)
                     └── Licitações x Produtos (requer EmpresaLicitação + Produto)
                          └── Aditivos (requer LicitaçãoProduto)
                               └── Solicitações (requer Empresa + Responsável)
                                    └── Solicitações x Produtos (requer Solicitação + Aditivo/Produto)
```

---

## Etapa 1 — Autenticação

**Prioridade:** Obrigatório antes de qualquer outra tela.

| Endpoint      | Método | Descrição                              |
| ------------- | ------ | -------------------------------------- |
| `/api/login`  | POST   | Login do usuário, retorna token Bearer |
| `/api/logout` | POST   | Logout e revogação do token            |
| `/api/me`     | GET    | Dados do usuário autenticado           |

> Todos os demais endpoints exigem o header `Authorization: Bearer {token}`.

Consulte: [docs/auth.md](auth.md)

---

## Etapa 2 — Configuração Inicial da Instituição

**Prioridade:** Fazer logo após o login. A API retorna alertas enquanto a instituição não estiver configurada.

| Endpoint           | Método    | Descrição                  |
| ------------------ | --------- | -------------------------- |
| `/api/instituicao` | GET       | Busca dados da instituição |
| `/api/instituicao` | POST      | Cria a instituição         |
| `/api/instituicao` | PUT/PATCH | Atualiza os dados          |

Consulte: [docs/instituicao.md](instituicao.md)

---

## Etapa 3 — Cadastros Base (sem dependências)

Estes módulos são independentes entre si e podem ser implementados em paralelo. Devem ser feitos antes dos módulos de Licitação e Solicitação.

### 3.1 Secretarias

| Endpoint                | Método    | Descrição         |
| ----------------------- | --------- | ----------------- |
| `/api/secretarias`      | GET       | Listagem paginada |
| `/api/secretarias`      | POST      | Criar             |
| `/api/secretarias/{id}` | GET       | Detalhe           |
| `/api/secretarias/{id}` | PUT/PATCH | Atualizar         |
| `/api/secretarias/{id}` | DELETE    | Excluir           |

Consulte: [docs/secretarias.md](secretarias.md)

### 3.2 Órgãos

| Endpoint           | Método    | Descrição         |
| ------------------ | --------- | ----------------- |
| `/api/orgaos`      | GET       | Listagem paginada |
| `/api/orgaos`      | POST      | Criar             |
| `/api/orgaos/{id}` | GET       | Detalhe           |
| `/api/orgaos/{id}` | PUT/PATCH | Atualizar         |
| `/api/orgaos/{id}` | DELETE    | Excluir           |

Consulte: [docs/orgaos.md](orgaos.md)

### 3.3 Responsáveis

| Endpoint                 | Método    | Descrição         |
| ------------------------ | --------- | ----------------- |
| `/api/responsaveis`      | GET       | Listagem paginada |
| `/api/responsaveis`      | POST      | Criar             |
| `/api/responsaveis/{id}` | GET       | Detalhe           |
| `/api/responsaveis/{id}` | PUT/PATCH | Atualizar         |
| `/api/responsaveis/{id}` | DELETE    | Excluir           |

Consulte: [docs/responsaveis.md](responsaveis.md)

### 3.4 Empresas

| Endpoint             | Método    | Descrição         |
| -------------------- | --------- | ----------------- |
| `/api/empresas`      | GET       | Listagem paginada |
| `/api/empresas`      | POST      | Criar             |
| `/api/empresas/{id}` | GET       | Detalhe           |
| `/api/empresas/{id}` | PUT/PATCH | Atualizar         |
| `/api/empresas/{id}` | DELETE    | Excluir           |

Consulte: [docs/empresas.md](empresas.md)

### 3.5 Modalidades

| Endpoint                | Método    | Descrição         |
| ----------------------- | --------- | ----------------- |
| `/api/modalidades`      | GET       | Listagem paginada |
| `/api/modalidades`      | POST      | Criar             |
| `/api/modalidades/{id}` | GET       | Detalhe           |
| `/api/modalidades/{id}` | PUT/PATCH | Atualizar         |
| `/api/modalidades/{id}` | DELETE    | Excluir           |

Consulte: [docs/modalidades.md](modalidades.md)

### 3.6 Produtos

| Endpoint             | Método    | Descrição         |
| -------------------- | --------- | ----------------- |
| `/api/produtos`      | GET       | Listagem paginada |
| `/api/produtos`      | POST      | Criar             |
| `/api/produtos/{id}` | GET       | Detalhe           |
| `/api/produtos/{id}` | PUT/PATCH | Atualizar         |
| `/api/produtos/{id}` | DELETE    | Excluir           |

Consulte: [docs/produtos.md](produtos.md)

### 3.7 Usuários

| Endpoint          | Método    | Descrição         |
| ----------------- | --------- | ----------------- |
| `/api/users`      | GET       | Listagem paginada |
| `/api/users`      | POST      | Criar             |
| `/api/users/{id}` | GET       | Detalhe           |
| `/api/users/{id}` | PUT/PATCH | Atualizar         |
| `/api/users/{id}` | DELETE    | Excluir           |

Consulte: [docs/users.md](users.md)

---

## Etapa 4 — Licitações

**Depende de:** Modalidades (Etapa 3.5)

| Endpoint               | Método    | Descrição                            |
| ---------------------- | --------- | ------------------------------------ |
| `/api/licitacoes`      | GET       | Listagem (retorna `{ data: [...] }`) |
| `/api/licitacoes`      | POST      | Criar (requer `modalidade_id`)       |
| `/api/licitacoes/{id}` | GET       | Detalhe                              |
| `/api/licitacoes/{id}` | PUT/PATCH | Atualizar                            |
| `/api/licitacoes/{id}` | DELETE    | Excluir                              |

Consulte: [docs/licitacoes.md](licitacoes.md)

---

## Etapa 5 — Empresas x Licitações (Vencedores/Contratos)

**Depende de:** Empresas (3.4) + Licitações (Etapa 4)

| Endpoint                        | Método    | Descrição                                            |
| ------------------------------- | --------- | ---------------------------------------------------- |
| `/api/empresas-licitacoes`      | GET       | Listagem (filtros: `?licitacao_id=`, `?empresa_id=`) |
| `/api/empresas-licitacoes`      | POST      | Vincular empresa vencedora à licitação               |
| `/api/empresas-licitacoes/{id}` | GET       | Detalhe                                              |
| `/api/empresas-licitacoes/{id}` | PUT/PATCH | Atualizar                                            |
| `/api/empresas-licitacoes/{id}` | DELETE    | Excluir                                              |

Consulte: [docs/empresas-licitacoes.md](empresas-licitacoes.md)

---

## Etapa 6 — Licitações x Produtos (Itens do Contrato)

**Depende de:** Empresas x Licitações (Etapa 5) + Produtos (3.6)

| Endpoint                                             | Método | Descrição                                  |
| ---------------------------------------------------- | ------ | ------------------------------------------ |
| `/api/empresas-licitacoes/{id}/produtos`             | GET    | Lista itens com saldo disponível calculado |
| `/api/empresas-licitacoes/{id}/produtos`             | POST   | Vincular produto ao contrato               |
| `/api/empresas-licitacoes/{id}/produtos/{produtoId}` | DELETE | Remover item do contrato                   |
| `/api/empresas-licitacoes/{id}/produtos/preview`     | POST   | Preview da importação CSV (sem salvar)     |
| `/api/empresas-licitacoes/{id}/produtos/importar`    | POST   | Confirmar e importar produtos via CSV      |

> **Sobre o CSV:** O fluxo é sempre preview → importar. Nunca chame `/importar` sem antes apresentar o resultado do `/preview` ao usuário.

Consulte: [docs/licitacoes-produtos.md](licitacoes-produtos.md)

---

## Etapa 7 — Aditivos

**Depende de:** Licitações x Produtos (Etapa 6)

| Endpoint             | Método    | Descrição                                     |
| -------------------- | --------- | --------------------------------------------- |
| `/api/aditivos`      | GET       | Listagem                                      |
| `/api/aditivos`      | POST      | Criar aditivo (requer `licitacao_produto_id`) |
| `/api/aditivos/{id}` | GET       | Detalhe                                       |
| `/api/aditivos/{id}` | PUT/PATCH | Atualizar                                     |
| `/api/aditivos/{id}` | DELETE    | Excluir                                       |

Consulte: [docs/aditivos.md](aditivos.md)

---

## Etapa 8 — Solicitações

**Depende de:** Empresas (3.4) + Responsáveis (3.3) + Auth (Etapa 1)

| Endpoint                                | Método    | Descrição                                                                       |
| --------------------------------------- | --------- | ------------------------------------------------------------------------------- |
| `/api/solicitacoes`                     | GET       | Listagem (filtros: `?empresa_id=`, `?responsavel_id=`, `?ehlicitada=`, `?ano=`) |
| `/api/solicitacoes`                     | POST      | Criar (gera `guia` e `protocolo` automaticamente)                               |
| `/api/solicitacoes/{id}`                | GET       | Detalhe                                                                         |
| `/api/solicitacoes/{id}`                | PUT/PATCH | Atualizar                                                                       |
| `/api/solicitacoes/{id}`                | DELETE    | Excluir                                                                         |
| `/api/solicitacoes/totais-diretos?ano=` | GET       | Relatório de totais de compras diretas por empresa/ano                          |

> **Atenção:** A rota `/totais-diretos` deve ser chamada **antes** de `/solicitacoes/{id}` na configuração do roteador do frontend, pois é uma rota estática que pode colidir com o parâmetro `{id}`.

Consulte: [docs/solicitacoes.md](solicitacoes.md)

---

## Etapa 9 — Solicitações x Produtos (Itens da Solicitação)

**Depende de:** Solicitações (Etapa 8) + Licitações x Produtos (Etapa 6) / Aditivos (Etapa 7)

| Endpoint                                   | Método    | Descrição                       |
| ------------------------------------------ | --------- | ------------------------------- |
| `/api/solicitacoes/{id}/produtos`          | GET       | Lista itens da solicitação      |
| `/api/solicitacoes/{id}/produtos`          | POST      | Adicionar produto à solicitação |
| `/api/solicitacoes/{id}/produtos/{itemId}` | PUT/PATCH | Atualizar item                  |
| `/api/solicitacoes/{id}/produtos/{itemId}` | DELETE    | Remover item                    |

> Itens licitados referenciam um `aditivo_id` para rastrear o saldo consumido do contrato.

Consulte: [docs/solicitacoes.md](solicitacoes.md)

---

## Resumo da Ordem

| Etapa | Módulo                  | Depende de                       |
| ----- | ----------------------- | -------------------------------- |
| 1     | Autenticação            | —                                |
| 2     | Instituição             | Auth                             |
| 3.1   | Secretarias             | —                                |
| 3.2   | Órgãos                  | —                                |
| 3.3   | Responsáveis            | —                                |
| 3.4   | Empresas                | —                                |
| 3.5   | Modalidades             | —                                |
| 3.6   | Produtos                | —                                |
| 3.7   | Usuários                | Auth                             |
| 4     | Licitações              | Modalidades                      |
| 5     | Empresas x Licitações   | Empresas + Licitações            |
| 6     | Licitações x Produtos   | Empresas x Licitações + Produtos |
| 7     | Aditivos                | Licitações x Produtos            |
| 8     | Solicitações            | Empresas + Responsáveis + Auth   |
| 9     | Solicitações x Produtos | Solicitações + Aditivos          |
