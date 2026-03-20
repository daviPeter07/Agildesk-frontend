# Agildesk Frontend

Frontend web do Agildesk, sistema de help desk para abertura, acompanhamento e resolucao de chamados em tempo real.

## Sobre o projeto

O Agildesk Frontend foi projetado para atender operacoes de suporte com foco em:

- abertura e acompanhamento de chamados
- gestao de status e SLA
- comentarios e historico por ticket
- notificacoes em tempo real
- experiencia orientada a regras de negocio

A aplicacao segue arquitetura modular por feature, com separacao clara entre server state, estado de UI e integracoes de backend.

## Documentacao de referencia

As decisoes arquiteturais e regras detalhadas estao no documento de design:

- [Agildesk Frontend System Design](./Agildesk%20Frontend%20System%20Design.md)

## Stack principal

- React (JavaScript)
- React Router
- Axios
- shadcn-css
- react-hook-form + Zod
- TanStack Query
- nuqs
- socket.io-client
- date-fns
- lucide-react
- ESLint + Prettier

## Principais modulos

- Auth: autenticacao e sessao do usuario
- Tickets: abertura, listagem, detalhe, comentarios e historico
- Chat: comunicacao contextual por chamado
- Users: gestao e visualizacao de usuarios
- Dashboard: indicadores de atendimento e SLA
- Notifications: central de notificacoes e contador de nao lidas

## Arquitetura resumida

Estrutura base do projeto:

src/
app/
assets/
components/
features/
hooks/
lib/
contexts/
store/
styles/
types/

## Requisitos

- Node.js 20 ou superior
- pnpm 9 ou superior

## Instalacao

1. Clone o repositorio.
2. Entre na pasta do frontend.
3. Instale as dependencias com o comando abaixo.

   pnpm install

## Configuracao de ambiente

Crie um arquivo .env na raiz do projeto com as variaveis necessarias para API e realtime.

Exemplo:

    VITE_API_BASE_URL=http://localhost:3000
    VITE_SOCKET_URL=http://localhost:3000

Ajuste os valores conforme seu backend e ambiente de execucao.

## Executando o projeto

Ambiente de desenvolvimento:

    pnpm dev

Build de producao:

    pnpm build

Preview do build:

    pnpm preview

## Scripts uteis

- pnpm dev
- pnpm build
- pnpm preview
- pnpm lint
- pnpm format
- pnpm test

## Diretrizes funcionais importantes

- Sessao baseada em cookie HttpOnly/Secure e validacao por GET /auth/me.
- Server state centralizado no TanStack Query.
- Filtros, busca e paginacao sincronizados na URL com nuqs.
- Listagens com paginacao server-side e busca com debounce no frontend.
- Notificacoes persistidas no backend e atualizadas em tempo real via Socket.IO.
- SLA exibido na interface por estado operacional: no prazo, em alerta e vencido.

## Qualidade e colaboracao

- Gerenciador oficial: pnpm
- Branches: feature/<escopo>, fix/<escopo>, chore/<escopo>, docs/<escopo>
- Commits: Conventional Commits
- Antes de abrir PR, execute:

  pnpm lint
  pnpm test
  pnpm build

- Toda alteracao relevante deve atualizar a documentacao quando necessario.

## Status

Projeto em evolucao continua, com foco em robustez operacional, observabilidade e experiencia de atendimento.
