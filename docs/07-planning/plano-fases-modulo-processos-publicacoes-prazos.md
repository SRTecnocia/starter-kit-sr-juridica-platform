### Plano por Fases — Módulo Core: Gestão de Processos, Publicações e Prazos Inteligentes

Resumo: planejamento faseado para implementar, no escopo do MVP, o fluxo do PRD e do diagrama Mermaid que cobre: Cadastro/Lista/Detalhe de Processos, Linha do Tempo (eventos), Controle de Alçadas e Aprovações, Captura de Andamentos/Publicações, Inteligência de Prazos (sugestão/ajuste) e Notificações em tempo real. Arquitetura e padrões alinhados ao projeto: Next.js 15 App Router, Clerk, Supabase (RLS, realtime), TanStack Query (server state), Zustand (client state), next-intl.

Escopo MVP (do PRD/Mermaid):
- Cadastro centralizado de processos: preencher dados, anexar documentos iniciais, definir partes, salvar.
- Lista e Detalhe do Processo: timeline de eventos, documentos, tarefas, comunicações.
- Controle de alçadas/aprovações: solicitar, aprovar/reprovar, executar ou ajustar ato processual.
- Captura de andamentos/publicações: ingestão manual (fase 1) e integração externa simulada (fase 2).
- Inteligência de prazos: sugerir prazo com base no tipo de publicação e permitir confirmação/ajuste.
- Notificações em tempo real: novas publicações, aprovações e mudanças de status de tarefas.

Fora do escopo inicial (adiar): BI/Jurimetria avançada, assinatura eletrônica, integrações financeiras, automação de documentos complexa, sincronização com calendários, integrações profundas com APIs de tribunais em produção.

Rotas-alvo (arquitetura Landing + App):
- App protegido: `src/app/[locale]/app/processes/`
  - Lista: `.../processes/page.tsx`
  - Novo: `.../processes/new/page.tsx`
  - Detalhe: `.../processes/[id]/page.tsx` (abas: timeline, documentos, tarefas, aprovações)

Agentes disponíveis (diretório `agents/`) e seus papéis neste plano:
- Backend Database: `backend-database-integration.md`
- Backend API: `backend-api-endpoint.md`
- Feature CRUD: `feature-crud.md`
- Frontend Page/Component/Form/Hook: `frontend-page.md`, `frontend-component.md`, `frontend-form.md`, `frontend-hook.md`
- Estado (client): `frontend-state-management.md`, `global-state-management.md`
- TanStack Query: `tanstack-query-integration.md`
- Realtime/Notificações: `real-time-notifications.md`
- Integração API Externa: `integration-external-api.md`
- Code Review: `code-review.md`

---

### Fase 0 — Alinhamento e Objetivos (1 dia)
- Objetivos: reduzir retrabalho, consolidar visão 360 do caso, minimizar perdas de prazo. Métricas-alvo do PRD: tempo para faturamento não aplicado aqui; foco em adoção do módulo (>70% dos usuários ativos criam/consultam processos) e zero perdas de prazo no piloto.
- Entregáveis: definição de MVP, escopo e critérios de aceite por história.
- Agentes: nenhum executor; referência documental `docs/README.md`, PRD e Mermaid.

Critérios de aceite:
- Fluxos selecionados do Mermaid mapeados para páginas/APIs.
- Itens fora de escopo registrados.

---

### Fase 1 — Modelagem de Dados e Migrações (2–3 dias)
- Tabelas (Supabase):
  - `processes` (id, user_id, número, tribunal, comarca, assunto, status, partes jsonb, created_at, updated_at)
  - `process_events` (id, process_id, type ['init','doc','task','pub','approval','note'], payload jsonb, created_at)
  - `process_documents` (id, process_id, title, url, version, created_at)
  - `process_tasks` (id, process_id, title, assignee_id, status, due_date, created_at, updated_at)
  - `process_approvals` (id, process_id, step, status ['pending','approved','rejected'], approver_id, comment, created_at, decided_at)
  - `publications` (id, process_id, source, content, published_at, classification, created_at)
  - `deadlines` (id, process_id, publication_id, suggested_due_date, confirmed_due_date, status ['suggested','confirmed','adjusted','done'])
- Segurança: habilitar RLS em todas as tabelas, políticas por `user_id` (Clerk sub) e por relacionamento com `processes`.
- Indexação: por `user_id`, `process_id`, `created_at`, e compostos em `deadlines(status, suggested_due_date)`.
- Triggers: atualização de `updated_at`; trigger opcional para criar evento na timeline em inserções relevantes.

Entregáveis:
- Migrações SQL e tipos TypeScript em `src/lib/supabase/types.ts`.

Agentes:
- Backend Database: modelagem + RLS + índices.
- Feature CRUD: referência de patterns para tabelas e Realtime.

Critérios de aceite:
- Migrações executam sem erros; RLS bloqueia acesso cruzado entre tenants.

Riscos/Mitigação:
- Complexidade de RLS em joins: validar com consultas simples e políticas explícitas.

---

### Fase 2 — API Interna (Route Handlers) (2–3 dias)
- Endpoints (protegidos): `src/app/api/protected/`
  - `processes` (GET lista com filtros; POST criar)
  - `processes/[id]` (GET detalhe; PATCH atualizar; DELETE arquivar)
  - `processes/[id]/events` (GET timeline; POST adicionar evento livre/nota)
  - `processes/[id]/tasks` (CRUD de tarefas)
  - `processes/[id]/approvals` (POST solicitar; PATCH decidir)
  - `publications` (POST ingestão manual; GET por processo)
  - `deadlines` (POST sugerir a partir de publicação; PATCH confirmar/ajustar)
- Padrões: autenticação via Clerk, validação Zod, erros consistentes, `NextResponse.json`.

Entregáveis:
- Handlers implementados com integração Supabase server-side.

Agentes:
- Backend API: criação dos endpoints.
- Backend Database: consultas otimizadas/seguras.

Critérios de aceite:
- Testes de contrato (200/4xx/5xx) e RLS efetiva sob Clerk.

---

### Fase 3 — Queries/Mutations (TanStack Query) (1–2 dias)
- Hooks em `src/hooks/queries/` e `src/hooks/mutations/`:
  - Queries: `useProcesses`, `useProcess(id)`, `useProcessEvents(id)`, `usePublications(processId)`, `useDeadlines(processId)`
  - Mutations: `useCreateProcess`, `useUpdateProcess`, `useTaskMutations`, `useApprovalMutations`, `usePublicationIngest`, `useDeadlineMutations`
- Chaves: `['processes', filters]`, `['process', id]`, `['process', id, 'events']`, etc.
- Otimismo e invalidação seletiva após mutações.

Entregáveis:
- Hooks prontos com toasts via Zustand.

Agentes:
- TanStack Query Integration: padrões de chaves, cache e otimização.
- Frontend State/Global State: toasts e UI state.

Critérios de aceite:
- Loading/error states padronizados; sem uso de Zustand para server data.

---

### Fase 4 — Páginas e UX do Módulo de Processos (3–4 dias)
- Páginas:
  - Lista de Processos: busca, filtros (status, tribunal), paginação, ação “Novo”.
  - Novo Processo: formulário com Zod + RHF (dados, partes, anexos iniciais).
  - Detalhe do Processo: abas Timeline, Documentos, Tarefas (delegar/atualizar), Aprovações (solicitar/decidir).
- Componentes:
  - Timeline (renderiza `process_events` por tipo com ícones).
  - Cards/linhas reutilizáveis para tarefas, publicações e prazos.

Entregáveis:
- Páginas em `src/app/[locale]/app/processes/*` e componentes em `src/components/` prontos e responsivos.

Agentes:
- Frontend Page, Frontend Component, Frontend Form, Frontend Hook.

Critérios de aceite:
- i18n via next-intl, dark mode, acessibilidade básica, sem console errors.

---

### Fase 5 — Realtime e Notificações (1–2 dias)
- Subscrições Supabase:
  - Canal por `process_id` para `process_events`, `process_tasks`, `publications`, `deadlines`.
- Notificações:
  - Toasts e contadores via Zustand; integração com hooks de Query para cache.

Entregáveis:
- Atualização em tempo real do detalhe do processo e badges de novos itens.

Agentes:
- Real-time Notifications + TanStack Query Integration.

Critérios de aceite:
- Insersões/updates refletem em UI sem reload; cleanup de canais implementado.

---

### Fase 6 — Integração Externa (stub) e Inteligência de Prazos (2 dias)
- Stub de API externa (ambiente dev) para “nova publicação”.
- Endpoint de ingestão chama serviço de classificação simples (rule-based) e gera sugestão de prazo.
- UI: fluxo de confirmação/ajuste de prazo.

Entregáveis:
- Rota pública/protegida para ingestão, serviço de sugestão, UI para confirmar/ajustar.

Agentes:
- Integration External API, Backend API, Frontend Form/Page.

Critérios de aceite:
- Ao receber publicação, criar registro, evento na timeline, sugestão em `deadlines` e notificar usuário.

---

### Fase 7 — Segurança, Performance e Multi-tenant (1 dia)
- Revisão RLS e políticas por tabela; validação de ownership em todas as queries.
- Índices críticos revisados; limites/paginação padrão; cache headers em GETs públicos.

Entregáveis:
- Checklist de segurança concluído; verificação de tempos de resposta e render.

Agentes:
- Backend Database, Backend API, Code Review.

Critérios de aceite:
- Sem vazamento entre usuários; TTFB e render da lista < 4s (meta PRD para dashboards complexos).

---

### Fase 8 — i18n, A11y e Conteúdo (0,5–1 dia)
- Traduções pt/en para páginas, labels e validações.
- Acessibilidade: landmarks, labels, foco, contraste.

Entregáveis:
- Arquivos em `messages/` atualizados e checagem de acessibilidade básica.

Agentes:
- Frontend Page/Component, Code Review.

Critérios de aceite:
- 100% strings internacionalizadas; navegação por teclado OK.

---

### Fase 9 — Observabilidade e Analytics (0,5 dia)
- Eventos de uso: criação de processo, aprovação, confirmação de prazo.
- Integração com provedores definidos em `docs/02-configuration/analytics.md`.

Entregáveis:
- Chamadas de tracking encapsuladas e testadas.

Agentes:
- Frontend Hook/Component, Code Review.

Critérios de aceite:
- Dashboards de analytics mostram eventos principais após navegação de teste.

---

### Fase 10 — Testes e QA (1–2 dias)
- Testes de rota (smoke), validações e caminhos de erro.
- Testes manuais cross-locale, dark/light, mobile.

Entregáveis:
- Checklist de testes completo; build sem erros; lints/TS OK.

Agentes:
- Code Review (checklist), todos os agentes corrigem regressões.

Critérios de aceite:
- Lint, typecheck e build verdes; fluxos críticos passam.

---

### Fase 11 — Deploy e Rollout (0,5 dia)
- Aplicar migrações, variáveis de ambiente, chaves do Supabase e Clerk.
- Feature flag para liberar módulo a grupo piloto.

Entregáveis:
- Deploy realizado; instruções de rollback; flag configurada.

Agentes:
- Backend Database (migrações), Backend API, Code Review.

Critérios de aceite:
- Módulo acessível para piloto; sem incidentes críticos nas primeiras 24h.

---

### Roadmap Pós-MVP (alto nível)
- Integrações oficiais com tribunais (produção), sincronização com calendários, automação de documentos, BI/Jurimetria, assinatura eletrônica.

---

### Anexos e Referências
- PRD: `docs/07-planning/PRD.md`
- Diagrama: `docs/07-planning/mermaid.md`
- Padrões e guias: `agents/*.md`, `docs/README.md`, `docs/03-development/*`, `docs/05-features/*`

---

### Quadro de Responsabilidades por Fase (Agentes)
- Fase 1: Backend Database, Feature CRUD
- Fase 2: Backend API, Backend Database
- Fase 3: TanStack Query Integration, Global/Frontend State
- Fase 4: Frontend Page/Component/Form/Hook
- Fase 5: Real-time Notifications, TanStack Query Integration
- Fase 6: Integration External API, Backend API, Frontend Form/Page
- Fase 7: Backend Database, Backend API, Code Review
- Fase 8: Frontend Page/Component, Code Review
- Fase 9: Frontend Hook/Component, Code Review
- Fase 10: Code Review (+ todos para correções)
- Fase 11: Backend Database, Backend API, Code Review

Métricas de sucesso (derivadas do PRD):
- Adoção do módulo (>70% usuários ativos criam/consultam processos no mês)
- Zero perdas de prazo reportadas no piloto
- Tempo de carregamento das páginas principais < 4s


