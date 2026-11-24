---
title: API SETOR COMPRAS
---

API REST do setor de compras de prefeituras e órgãos públicos. Cada instituição tem banco próprio e é acessada pelo
slug na URL: `/api/{ente}/...` (ex.: `/api/agua-branca/login`).

## Ambiente de homologação

| | |
|---|---|
| Área de admin (cadastro de instituições) | `https://setorcompras-api.viniciusm.com.br/api/admin` — [admin](docs/admin.md) |
| Base URL da instituição | `https://setorcompras-api.viniciusm.com.br/api/homologacao` |
| Ente (slug) | `homologacao` |
| Saúde | `GET https://setorcompras-api.viniciusm.com.br/up` → `200` |

- Login do administrador do sistema: `POST /api/admin/login` (token só vale em `/api/admin/...`).
- Login na instituição: `POST /api/homologacao/login` com `{ "email", "password" }` → token. Envie `Authorization: Bearer {token}`
  e `Accept: application/json` em todas as chamadas.
- O primeiro acesso usa senha provisória: só `/me`, `/logout` e `PUT /senha` funcionam até a troca ([auth](docs/auth.md)).
- CORS liberado para qualquer origem (o front usa token, sem cookies).
- Credenciais de acesso são passadas à parte — nunca neste repositório.
- Dados de homologação podem ser apagados a qualquer momento.

## Comece por aqui

- **[Guia de migração do front](docs/migracao-front.md)** — tudo o que mudou e o que ajustar no front
- [Padrões da API](docs/padroes.md) — paginação, formato das respostas, tipos e erros
- [Perfis e permissões](docs/permissoes.md)
- [Ordem de implementação para o frontend](docs/ordem-implementacao.md)

## Acesso

- [Autenticação e troca de senha](docs/auth.md)
- [Usuários](docs/users.md)
- [Instituição](docs/instituicao.md)
- [Área de administração do sistema](docs/admin.md)

## Cadastros

- [Secretarias](docs/secretarias.md)
- [Órgãos](docs/orgaos.md)
- [Responsáveis](docs/responsaveis.md)
- [Empresas](docs/empresas.md)
- [Modalidades](docs/modalidades.md)
- [Produtos (catálogo)](docs/produtos.md)
- [Naturezas de despesa](docs/naturezas.md)

## Licitações e contratos

- [Licitações](docs/licitacoes.md)
- [Contratos (empresas × licitações)](docs/empresas-licitacoes.md)
- [Itens do contrato](docs/licitacoes-produtos.md)
- [Aditivos](docs/aditivos.md)

## Compras

- [Solicitações e itens](docs/solicitacoes.md)
