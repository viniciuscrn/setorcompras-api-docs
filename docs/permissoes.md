# Perfis e permissões

> Formatos comuns (paginação `?per_page=`/`?q=`, `{ data, message }`, números, erros): [padroes.md](padroes.md).

Cada usuário de uma instituição tem **um perfil**: `admin`, `gerente`, `licitacao` ou `compras`.
O perfil define as **permissões**, no formato `{area}.ver` (consultas, `GET`) e `{area}.gerenciar` (criar, editar, excluir).
Quem gerencia uma área sempre também vê.

O login e o `/me` devolvem `user.permissoes` — use essa lista no front para mostrar ou esconder telas e botões.
Uma ação sem permissão responde `403` com `"message": "Seu perfil não tem permissão para esta ação."`.

> A matriz fica em `App\Support\MatrizPermissoes` (fonte única). É uma **proposta a validar com o setor**; ao alterá-la, `php artisan ente:migrar` (ou `POST /api/admin/entes/migrar`) sincroniza os perfis de todas as instituições.

## Áreas

| Área | Rotas |
|------|-------|
| `usuarios` | `/users` |
| `instituicao` | `/instituicao` (o `GET /instituicao/logo` é público) |
| `estrutura` | `/secretarias`, `/orgaos`, `/responsaveis` |
| `catalogo` | `/empresas`, `/produtos`, `/modalidades`, `/naturezas` |
| `licitacoes` | `/licitacoes`, `/empresas-licitacoes` (incl. itens — criar, alterar, remover — preview e importação CSV), `/aditivos` |
| `solicitacoes` | `/solicitacoes` e `/solicitacoes/{id}/produtos` |
| `relatorios` | `/solicitacoes/totais-diretos` (só `ver`) |

Sem permissão específica: `/login`, `/logout`, `/me`, `PUT /senha`.

## Matriz

| Área | admin | gerente | licitacao | compras |
|------|:-----:|:-------:|:---------:|:-------:|
| usuarios | ver + gerenciar | ver | — | — |
| instituicao | ver + gerenciar | ver | ver | ver |
| estrutura | ver + gerenciar | ver + gerenciar | ver | ver |
| catalogo | ver + gerenciar | ver + gerenciar | ver + gerenciar | ver |
| licitacoes | ver + gerenciar | ver + gerenciar | ver + gerenciar | ver |
| solicitacoes | ver + gerenciar | ver + gerenciar | ver | ver + gerenciar |
| relatorios | ver | ver | ver | ver |

## Regras dos usuários

- Só `admin` cria, edita e exclui usuários.
- Ninguém altera o próprio perfil nem exclui a própria conta; a instituição nunca fica sem `admin`.
- Senha definida por outra pessoa é provisória (ver [auth.md](auth.md#put-senha)).
