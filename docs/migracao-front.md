# Guia de migração do front

O que muda para o front entre a API antiga (publicação inicial desta documentação) e a atual.
Cada item diz **o que era**, **o que é agora** e **o que o front precisa fazer**. Detalhes de cada rota nas páginas de referência.

> **Resumo para planejar:** toda URL ganha o slug da instituição; o login traz permissões e pode exigir troca de senha;
> listas passam a ser paginadas; números chegam como número; erros têm formato único; vários campos foram renomeados;
> o produto virou catálogo e a compra direta passou a ter natureza de despesa e limite anual.

---

## 1. Mudanças que afetam todas as chamadas

### 1.1 URL com o slug da instituição

Cada instituição (prefeitura, câmara...) tem banco próprio e é identificada pelo **slug** na URL.

| Antes | Agora |
|-------|-------|
| `/api/empresas` | `/api/{ente}/empresas` — ex.: `/api/agua-branca/empresas` |
| `/api/login` | `/api/{ente}/login` |

- O token emitido em um ente **só vale nele**. Usar o token de um ente em outro dá `401`.
- Slug inexistente ou instituição desativada: `404`.
- **Front:** guardar o slug (escolhido no login, vindo do subdomínio ou da configuração do deploy) e prefixar todas as chamadas.
  Ver [admin.md](admin.md) para a área que cadastra as instituições — ela segue os mesmos padrões
  (listagem `{ data, links, meta }`, `{ data, message }` nas gravações, UF/CNPJ validados).

### 1.2 Listagens paginadas

| Antes | Agora |
|-------|-------|
| `{ "data": [ ...todos... ] }` (usuários: paginação crua do Laravel, 10 por página) | `{ "data": [...], "links": {...}, "meta": {...} }`, 15 por página |

- Parâmetros: `?per_page=` (1–100), `?page=`, `?q=` (busca) — mais os filtros de cada recurso.
- Selects com cadastros pequenos: `?per_page=100`.
- **Não paginadas:** itens do contrato, itens da solicitação, aditivos de um item, `totais-diretos`, preview do CSV.
- **Front:** ler `meta.total`, `meta.last_page`, `meta.current_page`; trocar a busca local por `?q=`.

### 1.3 Respostas de gravação e exclusão

| Ação | Antes | Agora |
|------|-------|-------|
| Criar / atualizar | `{ message, data }` (usuários: objeto solto) | `{ data, message }` em todos — **usuário também dentro de `data`** |
| Excluir | `{ message }` (usuários: `204` sem corpo) | `200 { message }` em todos |

### 1.4 Números e datas

| Antes | Agora |
|-------|-------|
| Decimais como texto: `"preco_unitario": "4200.00"`, `"quant": "2.00"` | Número: `"preco_unitario": 4200`, `"quant": 2` |

- Quantidades aceitam até 3 casas, preços unitários 4, valores 2.
- Datas de negócio sempre `YYYY-MM-DD` (entrada e saída); a API recusa outro formato (`422`).
- **Front:** remover `parseFloat` de campos vindos da API (não quebra, mas é desnecessário) e formatar moeda na exibição.

### 1.5 CPF, CNPJ, telefone e UF

| Antes | Agora |
|-------|-------|
| Gravados como digitados (`00.000.000/0001-00`) | Aceitos com ou sem máscara, **devolvidos só com dígitos** (`00000000000191`) |
| Sem validação | CPF/CNPJ validados pelos dígitos verificadores; UF deve ser um dos 27 estados (`rn` vira `RN`) |

- **Front:** aplicar máscara na exibição; tratar o `422` de documento inválido.

### 1.6 Erros

Formato único: `{ "message": "..." }` em português; validação (`422`) traz também `errors` por campo, com nomes amigáveis
("razão social", "CPF/CNPJ"). Tabela completa em [padroes.md](padroes.md#erros).

| Status | O que fazer no front |
|--------|----------------------|
| `401` | Sessão acabou (token expira em **12 h**) → voltar ao login |
| `403` com `codigo: "senha_provisoria"` | Ir para a troca de senha |
| `403` | Esconder/desabilitar a ação (perfil sem permissão) |
| `404` | Registro ou rota inexistente |
| `409` | **Novo:** exclusão de algo em uso (ex.: empresa com contrato) → mostrar `message` |
| `422` | Mostrar `errors` nos campos; regras de negócio também chegam aqui (saldo, limite, vigência) |
| `429` | **Novo:** muitas tentativas de login → aguardar (`Retry-After`) |

---

## 2. Login, sessão e permissões

Referência: [auth.md](auth.md), [permissoes.md](permissoes.md).

| Antes | Agora |
|-------|-------|
| `user: { id, name, email, role }` | `user: { id, name, email, role, permissoes: [...], senha_provisoria }` + `alertas` |
| `GET /me` com `created_at`/`updated_at` | `GET /me` com o mesmo formato de `user` do login + `alertas` |
| — | `PUT /senha` (troca da própria senha) |
| Token sem validade | Token expira em 12 h |
| Sem limite de tentativas | 5 tentativas/min por e-mail+IP (`429`) |

- **Senha provisória:** usuário criado por um admin (ou com senha trocada por outra pessoa) entra com `senha_provisoria: true`.
  Até trocar em `PUT /senha` (`senha_atual`, `password`, `password_confirmation`), só `/me`, `/logout` e `/senha` funcionam.
- **Permissões:** cada perfil (`admin`, `gerente`, `licitacao`, `compras`) tem uma lista no formato `{area}.ver` / `{area}.gerenciar`.
  Use `user.permissoes` para montar menus e botões — a matriz está em [permissoes.md](permissoes.md).
- **Usuários:** só `admin` cria, edita e exclui. Ninguém muda o próprio perfil nem se exclui; a instituição nunca fica sem admin (`422`).
  A senha precisa de 8+ caracteres com letras e números.

---

## 3. Mudanças por recurso

### Usuários — [users.md](users.md)
- Respostas dentro de `data` (antes, objeto solto). Exclusão `200 { message }` (antes `204`).
- Listagem: `?role=` e `?q=` (nome, e-mail).

### Instituição — [instituicao.md](instituicao.md)
- **Não há mais criação nem exclusão:** a instituição nasce com o ente. Use `GET` e `PUT/PATCH`.
- Para enviar o brasão, use **`POST /instituicao` em `multipart/form-data`** (atualiza; o PHP não lê arquivos em PUT).
- `logo` (caminho interno) saiu; agora vem **`logo_url`** (rota pública da imagem) ou `null`.
- Só `nome` é obrigatório; os demais campos são opcionais.

### Secretarias, órgãos, responsáveis — [secretarias.md](secretarias.md), [orgaos.md](orgaos.md), [responsaveis.md](responsaveis.md)
- Secretaria: `cnpj` e `tel` **opcionais** (CNPJ validado se informado).
- Responsável: `orgaos_id` → **`orgao_id`**.
- Excluir secretaria com órgãos, órgão com responsáveis ou responsável com solicitações → `409`.

### Empresas — [empresas.md](empresas.md)
- **`limite_compra_direta` saiu** (o limite agora é por natureza de despesa — ver abaixo).
- `cpfcnpj` validado e **único**; novo campo de leitura `tipo_pessoa` (`fisica`/`juridica`).
- Excluir empresa com contratos ou solicitações → `409`.

### Produtos (catálogo) — [produtos.md](produtos.md)
- **Produto agora é só catálogo:** `descricao` e `unidade`. **`quant`, `preco`, `marca` e `numero_item` saíram** — foram para o item do contrato.
- Produto igual (mesma descrição, ignorando maiúsculas/espaços, e unidade) não é duplicado: o `POST` devolve o existente com `200`.
- Unidade é gravada em maiúsculas.

### Modalidades — [modalidades.md](modalidades.md)
- Nome único (`422` se repetido). Com licitações → `409` ao excluir.

### Naturezas de despesa — [naturezas.md](naturezas.md) — **NOVO**
- Cadastro com `nome`, `descricao`, `limite_anual`, `ativo`. Necessário antes da primeira compra direta.
- Já usada → não exclui (`409`); desative com `ativo: false`.
- Área de permissão `catalogo`.

### Licitações — [licitacoes.md](licitacoes.md)
- `vigenciafim` não pode ser antes de `vigenciainicio`.
- Exclusão lógica; com contratos → `409`.
- Listagem: `?modalidade_id=`, `?ano=`, `?q=`. Detalhe traz `contratos`.

### Contratos (empresas × licitações) — [empresas-licitacoes.md](empresas-licitacoes.md)
- Novo campo **`ehreforma`** (reforma de edifício/equipamento: aditivo de acréscimo até 50 %).
- `iniciovigencia` e `fimvigencia` vão juntos; sem datas, vale a vigência da licitação.
- Listagem: `?vigentes_em=YYYY-MM-DD` (contratos válidos na data — use ao escolher o contrato de uma solicitação) e `?q=`.
- **Campos de saldo mudaram:**

| Antes | Agora |
|-------|-------|
| `valor_total_produtos` | `saldo.valor_itens` |
| `quantidade_total_produtos` | — (some `itens[].quant_original`) |
| `valor_gasto_contrato` | `saldo.valor_utilizado` |
| `valor_disponivel_contrato` | `saldo.valor_disponivel` |
| `produtos` (itens crus) | `itens` (com saldo por item) |
| — | `vigente_hoje` |

- O gasto agora é **por contrato** (antes somava todas as solicitações licitadas da empresa — dois contratos da mesma empresa se misturavam).
- Com solicitações: não troca empresa/licitação (`422`) e não exclui (`409`).

### Itens do contrato — [licitacoes-produtos.md](licitacoes-produtos.md)
- **Incluir item** passou a receber os dados do contrato: `produto_id` **ou** `descricao` + `unidade`, mais `quant`, `preco`, `marca`, `numero_item`.
- **Novo:** `PUT/PATCH .../produtos/{item}` para alterar quantidade, preço, marca e nº.
- Remover item com solicitações ou aditivos → `409`. Item de outro contrato na URL → `404` (antes `403`).
- **Campos de saldo mudaram:**

| Antes | Agora |
|-------|-------|
| `quantidade_contratada` | `quant_contratada` (+ `quant_original`, `quant_acrescida`, `quant_suprimida`) |
| `preco_atual` | `preco_atual` (+ `preco_original`) |
| `quantidade_utilizada` | `quant_utilizada` |
| `quantidade_disponivel` | `quant_disponivel` |
| `valor_disponivel` | `valor_disponivel` |
| `produto.quant`, `produto.preco`... | `quant_original`, `preco_original`, `marca`, `numero_item` no próprio item |

- A listagem traz também `saldo` (resumo do contrato) ao lado de `data`.
- **CSV:** o preview devolve as linhas em **`data`** (antes `dados`) e aponta produto repetido no arquivo ou já no contrato.
  A importação reaproveita produtos do catálogo e é tudo ou nada.

### Aditivos — [aditivos.md](aditivos.md)
- **Supressão:** `quant` negativo. Quantidade 0 é recusada.
- **Limite passou a bloquear:** acréscimos ou supressões acumulados acima de 25 % (50 % de acréscimo em reforma) só com **`justificativa`** —
  sem ela, `422` em `justificativa`; com ela, a resposta traz `aviso`.
- Supressão (ou exclusão de acréscimo) que deixaria o item abaixo do já utilizado → `422`.
- O campo que o tipo não usa vem `null` (ex.: `quant` num aditivo só de preço).
- **Resumo mudou:**

| Antes | Agora |
|-------|-------|
| `quant_total_aditivada` | `quant_acrescida` e `quant_suprimida` |
| `percentual_aditivado` | `percentual_acrescido` e `percentual_suprimido` |
| `limite_recomendado: "25%"` | `limite_acrescimo_percentual`, `limite_supressao_percentual` (números) |
| `limite_atingido` | `limite_ultrapassado` |
| — | `quant_contratada`, `quant_utilizada`, `quant_disponivel`, `valor_disponivel` |

- `GET /aditivos/{id}` traz o `resumo`; a relação `licitacaoProduto` saiu da resposta.

### Solicitações — [solicitacoes.md](solicitacoes.md)
- **Campos renomeados:** `empresas_id` → **`empresa_id`**, `responsaveis_id` → **`responsavel_id`**, `users_id` → **`user_id`** (preenchido pela API).
- **Licitada** (`ehlicitada: true`): exige **`empresa_licitacao_id`** — contrato da mesma empresa e **vigente na data**.
- **Direta** (`ehlicitada: false`): exige **`natureza_id`** (natureza ativa).
- Com itens lançados, não muda tipo, empresa, contrato, natureza nem o ano da data (`422`).
- Exclusão lógica: devolve saldo ao contrato e limite à natureza.
- Respostas trazem `contrato` (com `licitacao`) e `natureza`; `user` só com `id` e `name`.
- Listagem: novos filtros `empresa_licitacao_id`, `natureza_id`; `?q=` em guia, protocolo, descrição e razão social.

### Itens da solicitação — [solicitacoes.md](solicitacoes.md)
- **`aditivo_id` saiu.**
- **Licitada:** envie **`licitacao_produto_id`** (item do contrato) + `quant`. A quantidade tem que caber no saldo do item (`422`).
  O **preço é sempre o atual do contrato** — `preco_unitario` enviado é ignorado; `produto_id` vem do item.
- **Direta:** `produto_id` + `quant` + `preco_unitario`. O item que faria o total do ano da natureza passar do limite é **recusado** (`422`, com o disponível na mensagem).
- `valor_total_item` → **`valor_total`**. A listagem traz `valor_total` da solicitação ao lado de `data`.
- Item de outra solicitação na URL → `404` (antes `403`).

### Relatório `totais-diretos` — [solicitacoes.md](solicitacoes.md#get-solicitacoestotais-diretosano)
- **Era por empresa, agora é por natureza de despesa**, com o detalhe por empresa dentro de cada natureza.

| Antes (por empresa) | Agora (por natureza) |
|---------------------|----------------------|
| `empresa_id`, `razaosocial`, `cpfcnpj` | `natureza_id`, `nome`, `ativo` + `empresas: [{ empresa_id, razaosocial, cpfcnpj, total }]` |
| `limite_compra_direta` | `limite_anual` |
| `total_gasto`, `percentual_consumido` | `total_gasto`, `percentual_consumido`, **`disponivel`** |
| `aviso` só acima de 100 % | `aviso` a partir de 80 % ("próximo do fim") e em 100 % ("atingido") |

- Permissão: `relatorios.ver` (todos os perfis).

---

## 4. Ordem sugerida para ajustar o front

1. Prefixo `/api/{ente}` + tratamento de `401`/`403 senha_provisoria`/`429` + tela de troca de senha.
2. Menus e botões por `user.permissoes`.
3. Componente de listagem paginada (`meta`/`links`, `?q=`, `?per_page=`) e leitura de `data` em todas as respostas.
4. Exibição de `errors` (422) e `message` (409) num lugar só.
5. Cadastros: máscaras de CPF/CNPJ/telefone, UF em select, secretaria com CNPJ opcional, `orgao_id`.
6. Produtos como catálogo + **tela de naturezas de despesa** (nova).
7. Contratos: `ehreforma`, novos campos de saldo; itens do contrato com quantidade/preço/marca e edição; CSV com `data`.
8. Aditivos: supressão (negativo), justificativa quando a API pedir, novo resumo.
9. Solicitações: escolher contrato vigente (licitada) ou natureza (direta); itens pelo item do contrato; novos nomes de campo.
10. Relatório de compras diretas por natureza.
11. Instituição: `logo_url` e envio do brasão por `POST` multipart.

---

## 5. Páginas novas nesta documentação

- [padroes.md](padroes.md) — paginação, formatos, tipos e erros.
- [permissoes.md](permissoes.md) — perfis e matriz de permissões.
- [naturezas.md](naturezas.md) — naturezas de despesa e limite de compra direta.
- [admin.md](admin.md) — área de administração do sistema (cadastro das instituições).
