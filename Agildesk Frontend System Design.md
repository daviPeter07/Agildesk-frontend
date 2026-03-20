# Agildesk Frontend

## Visão Geral

- Interface web do sistema de help desk Agildesk, focada em abertura, acompanhamento e resolução de chamados.
- Frontend em React com organização modular por features e camadas reutilizáveis para estado, validação, formulários e UI.
- Documentação concentra convenções para manter consistência entre times.

## Tecnologias determinísticas

- React (JavaScript) + React Router.
- Axios para chamadas HTTP e camada `lib/axios` para instâncias configuradas.
- shadcn-css para composição de componentes com CSS puro, apoiado por estilos utilitários em `styles/`.
- react-hook-form + Zod para formulários com validação declarativa e schemas reutilizáveis.
- TanStack Query para server state, cache, refetch e invalidação.
- socket.io-client centralizado em `lib/socket` para comunicação em tempo real.
- nuqs para sincronizar query strings com estado.
- date-fns para manipulação de datas.
- lucide-react para ícones.
- ESLint + Prettier executados antes de qualquer PR.

## Estrutura de diretórios (kebab-case)

```
src/
	app/
		providers/
		layouts/
		routes/

	assets/
		icons/
		images/

	components/
		ui/
		common/
		forms/

	features/
		auth/
			components/
			hooks/
			services/
			schemas/
			queries/
			mutations/
			pages/
		tickets/
			components/
			hooks/
			services/
			schemas/
			queries/
			mutations/
			pages/
		chat/
			components/
			hooks/
			services/
			schemas/
			queries/
			mutations/
			pages/
		users/
			components/
			hooks/
			services/
			schemas/
			queries/
			mutations/
			pages/
		dashboard/
			components/
			hooks/
			services/
			schemas/
			queries/
			mutations/
			pages/

	hooks/

	lib/
		axios/
		socket/
		utils/

	contexts/

	store/

	styles/
		base/
		components/
		pages/
		themes/

	types/
```

## Convenções e responsabilidades

- Pastas sempre em kebab-case (ex.: `ticket-comments`).
- Componentes, páginas e contextos em PascalCase (ex.: `TicketBoard`).
- Hooks iniciam com `use-` no arquivo e `useSomething` na função principal.
- Schemas terminam em `-schema`, services em `-service`, queries em `-query`, mutations em `-mutation`.
- Pages são responsáveis por montar a tela completa e costurar providers específicos.
- Components renderizam partes de UI reutilizáveis ou widgets autocontidos.
- Hooks guardam lógica compartilhada (fetchers, formulários, state machines).
- Services concentram comunicação com o backend (REST, WebSocket, etc.).
- Queries e mutations organizam o server state com TanStack Query, incluindo chaves padronizadas e estratégias de invalidação.
- Schemas (Zod) validam entradas e payloads antes de envio ou persistência.

## Padrões de código

- Funções internas/handlers/utilidades devem ser arrow functions.
- Exportações principais (pages, componentes top-level, hooks, serviços) usam `export function Nome()` para facilitar tree-shaking e consistência com React.
- Comentários sucintos apenas quando o bloco não for autoexplicativo.
- Garantir lint/format via ESLint + Prettier antes de abrir PRs.

## Sessão e autenticação

- Autenticação baseada em cookie de sessão enviado pelo backend via `Set-Cookie` durante o `POST /auth/login`.
- Em produção o cookie deve ser configurado com `HttpOnly` e `Secure`; o frontend não acessa o token diretamente por JavaScript.
- Fluxo mínimo: `POST /auth/login` → `GET /auth/me` → `POST /auth/logout`.
- Estado autenticado determinado pela resposta de `GET /auth/me`; falhas de autenticação invalidam a sessão e redirecionam o usuário para login.
- Instância Axios global deve usar `withCredentials: true` para enviar cookies cross-origin.
- Backend deve responder requisições autenticadas com `Access-Control-Allow-Credentials: true` e **não** pode usar `Access-Control-Allow-Origin: *` quando houver credenciais; a origem precisa ser explícita.
- Respostas `401` gatilham invalidação da sessão local, limpeza de cache sensível e redirecionamento imediato para login.
- Logout invalida a sessão no backend (`POST /auth/logout`) e limpa o estado autenticado (store, React Query, Socket.IO) no frontend.
- Conexão do Socket.IO só inicia após a sessão ser validada via `GET /auth/me`.

## Socket.IO

- Conexão estabelecida apenas após validação da sessão autenticada.
- Desconexão forçada durante o logout para evitar sessões zumbis.
- O handshake do Socket.IO deve respeitar a mesma estratégia de autenticação da aplicação, alinhada à sessão baseada em cookie.
- Canais dedicados por domínio: usuário, ticket e equipe.
- Eventos tratados seguem o padrão `dominio.acao` (ex.: `ticket.created`, `ticket.status.changed`, `ticket.comment.created`, `notification.created`).
- Listeners devem permanecer encapsulados em hooks/services específicos por feature para facilitar limpeza automática e evitar duplicidade de inscrição.

## Regras de negócio (RN)

1. **RN01 Cadastro de usuários**: permitir perfis Usuário, Atendente e Administrador.
2. **RN02 Autenticação**: acesso somente via login + senha.
3. **RN03 Abertura de chamado**: requer usuário autenticado.
4. **RN04 Dados obrigatórios**: título, descrição, categoria, prioridade.
5. **RN05 Status inicial**: todo chamado nasce como "Aberto".
6. **RN06 Status do chamado**: estados válidos – Aberto, Em atendimento, Resolvido, Encerrado.
7. **RN07 Alteração de status**: apenas atendentes podem marcar "Em atendimento".
8. **RN08 Resolução**: só marcar "Resolvido" após resposta do atendente.
9. **RN09 Reabertura**: usuário pode reabrir chamados resolvidos; encerrados não.
10. **RN10 Atribuição**: apenas um atendente por chamado; máximo de 3 chamados ativos por atendente.
11. **RN11 Ações do atendente**: assumir, responder e atualizar chamados.
12. **RN12 Histórico**: registrar todas as interações.
13. **RN13 Encerramento**: apenas usuário ou atendente pode encerrar.
14. **RN14 Condição de encerramento**: só encerrar se o status estiver "Resolvido".
15. **RN15 Pós-encerramento**: chamados encerrados não podem ser alterados.
16. **RN16 Relatórios**: gerar relatórios de desempenho e chamados.
17. **RN17 Prioridade**: alta prioridade deve ser atendida primeiro.
18. **RN18 SLA**: sistema define tempo máximo de atendimento.
19. **RN19 SLA estourado**: sinalizar chamados fora do prazo.
20. **RN20 Notificações usuário**: notificar sempre que houver atualização.
21. **RN21 Notificações atendente**: alertar sobre novos chamados.

## Modelo de dados (referência backend)

- **users**: `id`, `name`, `cpf`, `email`, `password`, `role (ADMIN|USER|TI)`, `departmentId`, `activeTicketsCount`, `isOnline`, `imagem?`, `createdAt`, `updatedAt`.
- **departments**: `id`, `name`, `description?`, `createdAt`.
- **teams**: `id`, `name`, `description`.
- **team_members**: `userId`, `teamId`.
- **services**: `id`, `name`, `description`, `sla_hours`, `priority_suggested (LOW|MEDIUM|HIGH|URGENT)`, `createdAt`, `updatedAt`.
- **tickets**: `id`, `protocol`, `title`, `description`, `status (OPEN|IN_PROGRESS|PENDING|RESOLVED|CLOSED|CANCELED)`, `priority (LOW|MEDIUM|HIGH|URGENT)`, `userId`, `technicianId?`, `teamId?`, `serviceId`, `departmentId`, `hasTechnicianReply`, `deadline`, `closedAt?`, `createdAt`, `updatedAt`.
- **ticket_comments**: `id`, `ticketId`, `userId`, `message`, `isInternal`, `createdAt`.
- **ticket_history**: `id`, `ticketId`, `changedBy`, `action`, `oldValue?`, `newValue?`, `createdAt`.
- **notifications**: `id`, `userId`, `type (NEW_TICKET|TIMEOUT_WARNING|SLA_BREACHED|NEW_REPLY)`, `content`, `readAt?`, `createdAt`.
- **reports**: `id`, `name`, `type (PERFORMANCE|TICKETS)`, `format (EXCEL|PDF)`, `url`, `userId`, `createdAt`.

## Convenções TanStack Query

- Toda query key é um array com o formato `[domínio, contexto, parâmetro?]`.
- Filtros sempre ocupam a terceira posição: `['tickets', 'list', filters]`.
- Chat do chamado pertence ao domínio `tickets` (ex.: `['tickets', 'comments', ticketId]`).
- Dashboard possui chaves independentes por card/bloco para granularidade de invalidação.
- Mutations devem invalidar todas as queries relacionadas ao domínio afetado.
- Lista base de chaves:
  - `['auth', 'me']`
  - `['tickets', 'list']`, `['tickets', 'list', filters]`, `['tickets', 'detail', ticketId]`, `['tickets', 'comments', ticketId]`, `['tickets', 'history', ticketId]`
  - `['notifications', 'list']`, `['notifications', 'unread-count']`
  - `['users', 'list']`, `['users', 'detail', userId]`, `['users', 'technicians']`
  - `['departments', 'list']`, `['departments', 'detail', departmentId]`
  - `['teams', 'list']`, `['teams', 'detail', teamId]`, `['teams', 'members', teamId]`
  - `['services', 'list']`, `['services', 'detail', serviceId]`
  - `['reports', 'list']`, `['reports', 'detail', reportId]`
  - `['dashboard', 'summary']`, `['dashboard', 'tickets-overview']`, `['dashboard', 'sla-overview']`, `['dashboard', 'technician-performance']`

## Estratégia de estado

- TanStack Query é a fonte oficial de server state (cache, refetch, sincronização e invalidação).
- Estados de UI compartilhados ficam em `contexts/` ou `store/`, sempre evitando replicar dados que já estejam em TanStack Query.
- Estados locais simples vivem em `useState` ou hooks escopados ao componente/página.
- Não duplicar dados vindos da API em contextos/stores, salvo justificativa técnica documentada.
- Filtros e paginação persistentes devem preferir nuqs para refletir estado relevante na URL.
- Usuário autenticado sempre deriva de `GET /auth/me`, não de objetos manuais dispersos.
- Estado de conexão do Socket.IO pode ser global para sinalizar disponibilidade realtime.

## Tratamento de erro e feedback

- Toda operação assíncrona deve exibir feedback visual de carregamento e impedir ações repetidas durante o processamento.
- Submissões desabilitam botões/inputs relacionados enquanto aguardam resposta.
- Validações de formulário aparecem inline, próximas ao campo inválido, priorizando mensagens objetivas.
- Erros de regra de negócio trazem mensagens orientadas à ação; erros técnicos ou de comunicação disparam toast/alerta contextual.
- Respostas `401` causam invalidação imediata da sessão e redirecionam para login.
- Listagens precisam prever estados de loading (skeleton/spinner), vazio (mensagem + CTA quando aplicável) e erro (alerta com opção de tentar novamente).
- Filtros persistentes devem indicar quando nenhum resultado foi encontrado e oferecer atalho para limpar critérios.
- Ações indisponíveis por permissão são ocultadas ou bloqueadas com explicação contextual.
- A aplicação deve sinalizar perda e retomada da conexão Socket.IO para informar o usuário sobre disponibilidade realtime.

**Exemplos Agildesk**

- Abrir chamado: botão com estado loading, validações inline por campo, erro de servidor via toast, sucesso com toast ou redirecionamento para o chamado criado.
- Alterar status: botão desabilitado durante envio, atualização perceptível em tela após sucesso, erro de regra com mensagem clara, erro técnico com toast e opção de retry.
- Lista de chamados: skeleton durante fetch, empty state com mensagem e ação para criar filtro/chamado, alerta visível em caso de falha de carregamento.
- Chat/comentários: indicador de envio por mensagem, erro ao enviar com feedback claro e possibilidade de retry, aviso discreto quando o realtime estiver indisponível.

## Paginação, filtros e busca

- Listagens principais usam paginação server-side; sempre enviar `page`, `limit`, filtros, busca e ordenação relevantes ao backend.
- Estado de paginação/filtros/busca/ordenação deve ficar sincronizado na URL via nuqs; qualquer alteração reinicia a contagem para a página 1.
- Busca textual roda no backend e deve usar debounce no frontend para reduzir carga.
- Query keys de listagem precisam incluir paginação, filtros e ordenação atuais para garantir cache consistente.
- Diferenciar estado vazio real (sem registros) de "sem resultado" após filtros; mensagens/contextos distintos.
- Tickets terão como filtros iniciais: `status`, `priority`, busca por protocolo ou título, `page` e `limit`; a evolução prevista inclui `technicianId`, `serviceId`, `departmentId`, `teamId`, `period` e `slaStatus`.
- Filtros persistentes respeitam o formato `[domínio, 'list', filters]`, onde `filters` contém apenas parâmetros relevantes serializáveis.

## Estratégia de notificações

- Notificações são persistidas no backend e expostas via API; eventos relevantes também chegam em tempo real via Socket.IO.
- UI deve oferecer contador global de não lidas (header), painel/lista completa e toasts para eventos imediatos realmente importantes.
- Receber uma notificação **não** a marca como lida; apenas ação explícita do usuário (ou entrada em fluxo definido) altera o status.
- Eventos socket relacionados a notificações devem invalidar queries de lista (`['notifications', 'list']`) e contador (`['notifications', 'unread-count']`).
- Tipos iniciais suportados: `NEW_TICKET`, `NEW_REPLY`, `TIMEOUT_WARNING`, `SLA_BREACHED`. Novos tipos devem ser documentados aqui antes de qualquer uso.
- Toasts devem ser usados com parcimônia, apenas para alertas relevantes/imediatos; demais notificações vivem na lista dedicada.

## Regras de SLA no frontend

- O frontend deve utilizar `tickets.deadline` como referência operacional do prazo de SLA. O valor de `deadline` deriva da regra de negócio baseada em `tickets.createdAt` + `services.sla_hours`.
- Estados visuais mínimos: **no prazo**, **em alerta** (janela final antes do vencimento) e **vencido**.
- Chamados com status `OPEN`, `IN_PROGRESS` e `PENDING` exibem SLA operacional em tempo real. `RESOLVED` e `CLOSED` mostram resultado histórico; `CANCELED` não exigem destaque.
- A interface deve permitir filtros e ordenação pela situação/prazo do SLA (ex.: `slaStatus`, `deadline`, tempo restante).
- Prioridade do ticket (`priority`) e prioridade sugerida do serviço (`priority_suggested`) não substituem a regra de SLA; o frontend deve considerar `deadline` como fonte principal de exibição e ordenação operacional.
- Listagens e dashboards devem indicar visualmente a proximidade do vencimento por meio de labels e destaque visual, além de possibilitar segmentação por SLA (ex.: tickets vencidos, próximos do prazo e dentro do prazo).

## Observabilidade e debug

- Logs de desenvolvimento ficam habilitados apenas em ambientes dev e nunca exibem dados sensíveis (senhas, cookies, tokens).
- Falhas HTTP devem registrar endpoint, método, status e contexto do fluxo para facilitar rastreamento (ex.: `tickets.list`).
- Canal Socket.IO deve registrar em desenvolvimento os eventos de conexão, desconexão, reconexão e falhas de handshake/integração.
- Eventos socket relevantes podem ser observados em desenvolvimento por domínio (ex.: `tickets`, `notifications`).
- UI deve refletir estados críticos: indisponibilidade de backend, falha de carregamento, perda de realtime.
- Fluxos essenciais (auth, listagem de chamados, atualização de status, comentários, notificações) precisam de tratamento explícito de erro e feedback ao usuário.
- Projeto deve prever captura de erros inesperados (error boundaries/contexto) nas áreas críticas.
- Organizar logs por domínio/contexto para facilitar diagnóstico (ex.: `[auth] login failed`).

## Fluxo de trabalho

- Gerenciador padrão: **pnpm** (`pnpm install`, `pnpm dev`, etc.).
- Cada tarefa gera branch dedicada seguindo convenção abaixo.
- Commits seguem Conventional Commits para facilitar changelog e automações.
- Antes de abrir PR: rodar `pnpm lint`, `pnpm test` (quando aplicável) e `pnpm build`.
- Atualizar documentação (ex.: README, docs internos) quando houver mudança de regra ou fluxo.
- Abrir PR usando template do repositório, preenchendo descrição, checklist e evidências.
- Toda feature precisa passar por review humano obrigatório e pode contar com assistência do Copilot Review, quando essa prática fizer parte do fluxo definido pelo time.
- Merge somente após aprovação + status verde do CI; preferir `squash` ou `rebase` conforme política do repositório.

## Convenção de branches

- `feature/<escopo>` para novas funcionalidades (ex.: `feature/tickets-list`).
- `fix/<escopo>` para correções (ex.: `fix/auth-redirect`).
- `chore/<escopo>` para tarefas de manutenção (ex.: `chore/update-eslint`).
- `docs/<escopo>` para ajustes em documentação (ex.: `docs/readme-workflow`).
- Usar kebab-case após o prefixo e manter escopo curto/descritivo.

## Conventional Commits

- `feat: adiciona listagem de chamados`
- `fix: corrige redirecionamento após logout`
- `docs: atualiza convenções de autenticação`
- `refactor: reorganiza hooks de tickets`
- `chore: atualiza dependências pnpm`
- `test: adiciona testes de filtros`
- Mensagens no imperativo, em português, descrevendo a mudança realizada de forma objetiva.

## Padrão de PR

- **Título**: `[tipo] escopo resumido` (ex.: `feature: adiciona listagem de chamados`).
- **Descrição mínima**: o que mudou, por quê e como testar.
- **Checklist antes do review**:
  - [ ] `pnpm lint`
  - [ ] `pnpm test`
  - [ ] `pnpm build`
  - [ ] Documentação/README atualizado quando necessário
  - [ ] Screenshot/GIF anexado para mudanças de UI
- **Passos de teste**: detalhar fluxo manual/automático para validação.
- **Política de merge**: aguardar pelo menos 1 aprovação (ou 2 para mudanças sensíveis) antes de concluir.
