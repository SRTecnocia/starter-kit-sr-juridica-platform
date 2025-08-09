## Plano de Implementação por Fases – Plataforma All‑in‑One de Gestão e Automação Jurídica

Este documento organiza o planejamento do produto por fases, cobrindo todas as funcionalidades descritas no PRD e no fluxograma Mermaid, com orientações detalhadas de implementação, máscaras de campo, validações, restrições, responsabilidades e mapeamento aos agentes internos (em `agents/`). O objetivo é ser o guia único de execução para times de produto, engenharia e design.

### Agentes internos (responsabilidades)

- **agents/feature-crud.md**: Gerar entidades, CRUDs, listas, detalhes, filtros e paginação.
- **agents/backend-api-endpoint.md**: Definir e implementar endpoints REST/Next.js Route Handlers, autenticação e autorização.
- **agents/backend-database-integration.md**: Modelagem de dados, migrações (Supabase), RLS e integrações com banco.
- **agents/frontend-page.md**: Criação de páginas (Next.js App Router) e roteamento.
- **agents/frontend-form.md**: Formulários com React Hook Form + Zod, máscaras e validações.
- **agents/frontend-component.md**: Componentes de UI reutilizáveis (Radix UI + Tailwind).
- **agents/frontend-hook.md**: Hooks client (UX, estados locais, efeitos).
- **agents/frontend-state-management.md**: Zustand para estado client.
- **agents/global-state-management.md**: Estado compartilhado cross‑app, padrões de sincronização.
- **agents/tanstack-query-integration.md**: Server state (queries, mutations, cache, realtime).
- **agents/integration-external-api.md**: Integrações externas (Tribunais, Calendários, ERPs, Storage).
- **agents/real-time-notifications.md**: Realtime (Supabase Realtime), toasts, in‑app notifications.
- **agents/code-review.md**: Revisão de código, padrões, qualidade, segurança.

### Convenções gerais (aplicáveis a todas as fases)

- **Multi‑tenant**: Isolamento por `organizationId` (Clerk) + RLS no Supabase para todas as tabelas. Toda query filtrada por tenant.
- **i18n**: `next-intl` (pt, en). Não hard‑code textos.
- **A11y e UI**: Radix UI, foco visível, contrastes, navegação por teclado. Temas claro/escuro conforme `tailwind`.
- **Analytics**: GA, GTM, Meta Pixel, LogRocket. Eventos por feature (criação/edição, conversões, erros).
- **Auditoria**: Tabelas de audit log para ações críticas (criar/editar/excluir, aprovar/reprovar, assinar, faturar).
- **Arquitetura**: Next.js 15 App Router, React 19, TS rigoroso, TanStack Query para dados remotos, Zustand para client state.
- **Validações e máscaras padrão (Brasil)**:
  - CPF: `000.000.000-00` (validar dígitos verificadores)
  - CNPJ: `00.000.000/0000-00` (validar dígitos verificadores)
  - CEP: `00000-000`
  - Telefone celular: `(00) 00000-0000`
  - Telefone fixo: `(00) 0000-0000`
  - Data: `DD/MM/AAAA` (timezone padrão America/Sao_Paulo)
  - Hora: `HH:mm`
  - Moeda BRL: `R$ 0.000.000,00` (armazenar como decimal(18,2) no banco)
  - Número do Processo (CNJ): `NNNNNNN-DD.AAAA.J.TR.OOOO` (validar DV do CNJ)
  - OAB: `UF-000000` (UF em `A-Z{2}` + número 1–6 dígitos)
  - Pluralidade de documentos: limites por upload: tamanho máx 25MB/arquivo, tipos: pdf, docx, xlsx, jpg, png.
- **Restrições de segurança**: Rate‑limit por IP/org, anti‑XSS, CSRF (rotas sensíveis), validação completa de entrada com Zod, sanitização HTML.
- **Perf/UX**: Paginação e filtros server‑side, suspense/streaming onde aplicar, skeletons em listas e cards.

---

## Fase 0 – Fundamentos e Plataforma Base

- **Objetivo**: preparar fundações multi‑tenant, autenticação, navegação, perfis e observabilidade.
- **Entregas**:
  - Autenticação Clerk (Sign‑in/Up, SSO) e sincronização de `profiles` no Supabase.
  - RBAC: Papéis mínimos: `admin`, `advogado`, `gestor`, `financeiro`, `assistente`, `cliente` (portal). Mapear permissões gerais.
  - Menu e rotas base (`/app`, `/app/processes`, `/app/settings`, `/app/profile`).
  - Layout responsivo, tema, breadcrumbs, LanguageSwitcher, AuthGuard.
  - Observabilidade: logs estruturados, métricas de API, traços básicos.
  - Auditoria inicial: `audit_logs` (actor, org, ação, target, metadata, timestamp).
- **Agentes**: global-state-management, frontend-page, frontend-component, backend-database-integration, backend-api-endpoint, code-review.
- **Critérios de aceite**: login/SSO funcionais, troca de idioma, tema, criação de perfil e org, RLS ativo, RBAC bloqueando rotas.

---

## Fase 1 – Módulo Core de Processos e Casos

### 1.1 Entidades e campos

- Processo (`processes`):
  - `id` (uuid), `organization_id` (fk), `numero_cnj` (CNJ), `classe`, `assunto`, `tribunal`, `comarca`, `vara`, `instancia`, `fase_procedimental`, `status` (ex.: ativo, suspenso, arquivado), `descricao`, `advogado_responsavel_id`, `tags[]`, `valor_causa` (BRL), `created_at`, `updated_at`.
  - Máscaras/validações: `numero_cnj` obrigatório e único por org; DV CNJ; `valor_causa >= 0`.
- Partes (`process_parties`):
  - `id`, `process_id`, `tipo` (autor, réu, terceiro), `pessoa_tipo` (PF/PJ), `cpf`/`cnpj`, `razao_social`/`nome`, `email`, `telefone`, `endereco` (logradouro, número, compl, bairro, cidade, UF, CEP), `representantes[]` (OAB), `observacoes`.
  - Máscaras: CPF/CNPJ, CEP, telefone, OAB `UF-000000`.
- Advogados vinculados (`process_lawyers`): `id`, `process_id`, `user_id` (interno) ou `oab_externo`.
- Documentos iniciais (`process_documents`): `id`, `process_id`, `nome`, `tipo`, `url`, `versao`, `hash`, `tamanho`, `content_type`, `created_by`.

### 1.2 Funcionalidades

- Cadastrar novo processo (dados + anexos iniciais + partes + advogados).
- Listar processos (filtros: número, parte, responsável, status, classe, assunto, comarca, tags; ordenação).
- Detalhar processo: visão geral + abas (Linha do Tempo, Documentos, Tarefas, Comunicações, Aprovações).
- Editar/arquivar processo; tagging; atribuição/reatribuição de responsável.

### 1.3 Restrições e permissões

- Apenas `admin`/`gestor`/`advogado` com acesso ao processo podem editar; `assistente` pode criar/editar partes e anexos; `cliente` somente leitura se compartilhado.
- Exclusão lógica (soft delete) com auditoria.

### 1.4 UI/UX

- Formulários com RHF + Zod; máscaras (CNJ/CPF/CNPJ/CEP/telefone/moeda/data).
- Tabelas com paginação, colunas configuráveis, quick filters e chips de `tags`.

- **Agentes**: feature-crud, frontend-form, frontend-page, frontend-component, tanstack-query-integration, backend-api-endpoint, backend-database-integration, real-time-notifications, code-review.
- **Critérios de aceite**: CRUD completo, filtros performáticos, validações, auditoria de ações, RLS por org, testes e2e básicos.

---

## Fase 2 – Linha do Tempo e Eventos do Processo

### 2.1 Entidades

- Eventos (`process_events`): `id`, `process_id`, `tipo` (publicacao, andamento, audiencia, despacho, peticao, prazo_criado, tarefa, comentario, aprovacao_solicitada, aprovacao_decidida, documento_anexado), `titulo`, `descricao_rich`, `payload_json`, `actor_id`, `created_at`.

### 2.2 Funcionalidades

- Linha do tempo interativa (scroll infinito, filtros por tipo, busca textual).
- Criação manual e automática (gatilhos de outras features) de eventos.
- Notificações em tempo real ao time do processo.

### 2.3 Restrições

- Edição apenas pelo criador ou `admin`; eventos automáticos são somente leitura.

- **Agentes**: feature-crud, frontend-component, tanstack-query-integration, real-time-notifications, backend-api-endpoint, code-review.
- **Critérios de aceite**: timeline com filtros, eventos automáticos disparados, toasts/notifications, persistência e auditoria.

---

## Fase 3 – Captura de Andamentos/Publicações e Inteligência de Prazos

### 3.1 Integração de publicações

- Conector com provedores/tribunais (webhook + polling fallback).
- Normalização de publicações: `id_externo`, `numero_cnj`, `orgao`, `data`, `texto`, `partes_mencionadas`, `classe/assunto` (quando disponível).
- Desduplicação por `hash(texto)+numero_cnj` e janela temporal.

### 3.2 Inteligência de prazos

- Regras mapeadas por tipo de publicação (ex.: "Publicado acórdão" → sugerir 15 dias para RE/REsp). Campos: `prazo_sugerido_dias`, `tipo_ato`, `base_legal`.
- Pipeline: publicação → sugestão de prazo → confirmação/ajuste → criação de tarefa com due date.

### 3.3 Restrições e máscaras

- `numero_cnj` validado; timezone correta; textos > 10 chars; anexo de fonte quando houver.

- **Agentes**: integration-external-api, backend-api-endpoint, tanstack-query-integration, real-time-notifications, feature-crud, code-review.
- **Critérios de aceite**: ingestão estável (replay idempotente), sugestões coerentes, UX para confirmar/ajustar prazos, eventos na timeline.

---

## Fase 4 – Controle de Alçadas e Aprovações

### 4.1 Configurações

- Regras por org: valores‑limite, tipos de ato que exigem dupla aprovação, papéis aprovadores, SLAs.
- Fluxo: Solicitar → Notificar aprovadores → Aprovar/Reprovar com justificativa → Executar ato/voltar para revisão.

### 4.2 Entidades

- `approvals`: `id`, `process_id`, `tipo_ato`, `solicitante_id`, `status` (pendente, aprovado, reprovado), `aprovadores[]`, `justificativa`, `sla`, `decidido_em`.

### 4.3 Restrições

- Audit trail completo; somente aprovadores decidem; bloqueio de execução de ato enquanto pendente.

- **Agentes**: feature-crud, frontend-component, real-time-notifications, backend-api-endpoint, frontend-page, code-review.
- **Critérios de aceite**: fluxo E2E com notificações, bloqueios e auditoria.

---

## Fase 5 – GED Avançado (Documentos)

### 5.1 Armazenamento e versionamento

- Supabase Storage com ACL por org/processo; versionamento automático por `versao` e `hash`.
- Metadados: `palavras-chave`, `tipo_documento`, `referencia_processo`, `assinaturas`, `origem` (template, upload, gerado).

### 5.2 Funcionalidades

- Upload (arrastar/soltar), busca por texto/nome/tipo, histórico de versões, diff de versões (quando texto), compartilhamento controlado.

### 5.3 Restrições e máscaras

- Tipos permitidos: `pdf, docx, xlsx, jpg, png`; máx 25MB/arquivo; total por processo configurável.
- Scanner antivírus opcional (hook); bloqueio se infectado; retenção e política de exclusão.

- **Agentes**: frontend-component, frontend-form, tanstack-query-integration, backend-api-endpoint, backend-database-integration, code-review.
- **Critérios de aceite**: upload seguro, versionamento, busca eficiente, auditoria, controle de acesso.

---

## Fase 6 – Automação de Documentos (Document Assembly)

### 6.1 Modelos e questionários

- Biblioteca de templates (docx/HTML) com placeholders; questionários dinâmicos (Zod schemas).
- Variáveis padrão: dados do processo, partes, valores, prazos, firma, endereço.

### 6.2 Geração e saída

- Preenchimento → preview → geração em PDF/DOCX; salvamento no GED com vínculo ao processo.

### 6.3 Restrições e máscaras

- Campos com máscaras conforme tipo: CPF/CNPJ/CEP/telefone/data/moeda; validação obrigatória por tipo.

- **Agentes**: frontend-form, frontend-component, backend-api-endpoint, integration-external-api (motores de template se externos), code-review.
- **Critérios de aceite**: geração determinística, numeração de páginas, estilos coerentes, salvamento e versionamento.

---

## Fase 7 – Assinatura Eletrônica/Digital

### 7.1 Integração

- Provedores suportados (ex.: AssinaDoc/DocuSign) via `webhooks` + status: `enviado`, `visualizado`, `assinado`, `cancelado`, `expirado`.

### 7.2 Fluxo

- Enviar para assinatura → acompanhar status → baixar evidências → anexar certificado e trilha.

### 7.3 Restrições

- Conformidade jurídica; carimbo de tempo; CPF/CNPJ obrigatórios; emails válidos; vencimento configurável.

- **Agentes**: integration-external-api, backend-api-endpoint, real-time-notifications, feature-crud, code-review.
- **Critérios de aceite**: ciclo E2E com evidências anexadas ao GED e auditoria.

---

## Fase 8 – Módulo Financeiro

### 8.1 Timesheet Inteligente

- Lançamentos: `data`, `inicio`, `fim` ou `duracao`, `processo/cliente`, `atividade`, `faturavel` (bool), `taxa_hora` (opcional), `notas`.
- Sugestões baseadas em atividades recentes e agenda; arredondamento (6/15 min) configurável.
- Máscaras: data/hora; moeda.

### 8.2 Honorários

- Modelos: por hora, fee fixo, success fee, retainer, híbridos; regras por contrato.

### 8.3 Faturamento em Lote

- Seleção por período/contratos; geração de N faturas; envio por email/portal; status de pagamento.

### 8.4 Contas a Pagar/Receber

- Registros de receitas/despesas, categorias, centros de custo, conciliação bancária (import OFX/CSV), DRE.

### 8.5 Restrições

- Precisão monetária; timezone; permissões (`financeiro`, `gestor`).

- **Agentes**: feature-crud, frontend-form, frontend-page, backend-api-endpoint, backend-database-integration, tanstack-query-integration, integration-external-api (ERPs), code-review.
- **Critérios de aceite**: relatórios consistentes, geração e envio de faturas, conciliação básica, exportações.

---

## Fase 9 – CRM & Portal do Cliente

### 9.1 Portal do Cliente

- Acesso seguro; ver processos compartilhados, andamentos simplificados, baixar documentos liberados, canal de mensagens.
- Permissões: `cliente` somente leitura; logs de acesso a documentos.

### 9.2 Automação de Comunicação

- Relatórios de status agendados; modelos de email; opt‑out; histórico de envios e aberturas (quando possível).

### 9.3 Gestão de Leads

- Pipeline Kanban; etapas configuráveis; atividades; conversão para cliente.

- **Agentes**: frontend-page, frontend-component, tanstack-query-integration, backend-api-endpoint, real-time-notifications, feature-crud, code-review.
- **Critérios de aceite**: portal funcional e seguro, relatórios enviados, pipeline utilizável.

---

## Fase 10 – Equipe e Produtividade

### 10.1 Tarefas e Delegação

- `tasks`: `id`, `process_id?`, `titulo`, `descricao`, `responsavel_id`, `solicitante_id`, `prioridade`, `status` (aberta, em_andamento, bloqueada, concluida), `due_date`, `watchers[]`, `checklist[]`.
- Visões: Kanban/Lista, filtros por responsável, processo, prazo e prioridade.

### 10.2 Relatórios de Produtividade

- Horas trabalhadas vs. faturadas por advogado/equipe/departamento; gráficos e exportações.

- **Agentes**: feature-crud, frontend-component, tanstack-query-integration, real-time-notifications, backend-api-endpoint, code-review.
- **Critérios de aceite**: board fluido, operações drag‑and‑drop, relatórios coerentes.

---

## Fase 11 – Integrações (Calendários, ERPs, Armazenamento Externo)

### 11.1 Calendários (Google/Outlook)

- OAuth, escopos mínimos, 2‑way sync (eventos de prazos, audiências), deduplicação por `external_id`.
- Fallback ICS somente leitura se OAuth indisponível.

### 11.2 ERPs/Contabilidade (ContaAzul/QuickBooks)

- Mapeamento de contas contábeis; sincronização de faturas, pagamentos e clientes; webhooks.

### 11.3 Armazenamento em Nuvem (Drive/Dropbox/OneDrive)

- Linkagem de pastas; espelhamento opcional; controle fino de permissões.

- **Agentes**: integration-external-api, backend-api-endpoint, tanstack-query-integration, code-review.
- **Critérios de aceite**: sync estável, manuseio de conflitos, logs e reprocessamento.

---

## Fase 12 – Jurimetria (BI)

### 12.1 Modelagem analítica

- Tabelas agregadas por comarca, juiz, classe, tempo médio de duração, probabilidade de êxito, valores médios de condenação.

### 12.2 Dashboards

- Painéis personalizáveis; filtros salvos; exportação CSV/Excel.

- **Agentes**: backend-database-integration, services, frontend-component (charts), tanstack-query-integration, code-review.
- **Critérios de aceite**: consultas rápidas (<4s), consistência de KPIs, exportações corretas.

---

## Fase 13 – Segurança, NFRs e Operações

- LGPD: bases legais, consentimentos, anonimização e export de dados por solicitação.
- Multi‑tenant: RLS em todas as tabelas, segregação de storage por org, proibições de cross‑tenant.
- Disponibilidade: backups automáticos, restore testado, SLO 99.95%.
- Performance: budget de carregamento e respostas; caching; índices; EXPLAIN rotineiro.
- Testes: unitários, integração, e2e críticos, testes de carga básicos.
- Observabilidade: tracing de rotas e integrações; alertas SLO/SLA.
- Segurança: rate limiting, WAF/CDN, secret management, rotação de chaves, dependabot.
- **Agentes**: backend-database-integration, backend-api-endpoint, code-review.

---

## Regras de máscaras, validações e restrições (consolidadas)

- **Pessoas/Partes**:
  - PF: `nome` (2–120), `cpf` (DV), `email` (RFC), `telefone`, `endereco` completo, data de nascimento opcional.
  - PJ: `razao_social` (2–160), `cnpj` (DV), `ie` opcional, `telefone`, `endereco`.
  - OAB: `UF-000000` ou `UF 000000` (normalizar para `UF-000000`).
- **Endereço**: `CEP` obrigatório (formato), `UF` (A‑Z{2}), `cidade` (2–80), `logradouro` (2–120), `numero` (até 10), `complemento` (até 60).
- **Processo**: `numero_cnj` único por org, DV válido; `classe/assunto` obrigatórios; `valor_causa >= 0`.
- **Datas e prazos**: `data` e `due_date` em timezone Brasil; validar não retroativas quando aplicável.
- **Documentos**: tipos permitidos; tamanho; metadados obrigatórios (`nome`, `tipo`); proibir executáveis.
- **Financeiro**: precisão decimal(18,2); `valor >= 0`; regras de arredondamento definidas; faturas com somatório coerente com lançamentos.
- **Integrações**: idempotência por `external_id`; assinatura/verificação de webhooks; retries exponenciais.
- **Permissões**: RBAC em todos endpoints e UI; esconder ações não permitidas; logs de acesso.

---

## Entregáveis por fase (DoD)

- Código revisado (agente code-review), testes mínimos, documentação de API/UI, migrações aplicadas, RLS conferido, métricas e logs, i18n.
- Telas: responsivas, acessíveis, com estados de carregamento/erro/empty.
- APIs: documentadas (`/docs` ou markdown), com exemplos e códigos de erro padronizados.
- Auditoria: eventos críticos registrados.
- Segurança: validações, rate‑limit, secrets corretos.

---

## Mapeamento PRD/Mermaid → Fases

- PRD “Módulo de Gestão de Processos e Casos” → Fases 1, 2, 4, 5
- PRD “Automação e Inteligência de Dados” → Fases 3, 6, 12
- PRD “GED Avançado” → Fase 5
- PRD “Assinatura Eletrônica/Digital” → Fase 7
- PRD “Financeiro” → Fase 8
- PRD “CRM & Portal do Cliente” → Fase 9
- PRD “Equipe e Produtividade” → Fase 10
- PRD “Integrações (Ecossistema)” → Fase 11
- PRD “Não‑funcionais/UX impacto” → Fase 13 (com impactos transversais)

---

## Plano de releases (sugestão)

- R0: Fase 0
- R1: Fases 1–2 (MVP operacional de processos)
- R2: Fase 3–4 (publicações + prazos + aprovações)
- R3: Fase 5–6 (GED + automação de documentos)
- R4: Fase 7–8 (assinatura + financeiro)
- R5: Fase 9–10 (portal + produtividade)
- R6: Fase 11–12 (integrações + BI)
- R7: Fase 13 (endurecimento e SLO)

---

## Checklists de execução por agente

- **feature-crud**: modelo, migração, endpoints, lista/detalhe, filtros, paginação, testes.
- **backend-api-endpoint**: schema Zod, auth/RBAC, erros padronizados, rate‑limit, idempotência.
- **backend-database-integration**: modelagem, índices, RLS, seeds, views/materializações quando necessário.
- **frontend-form**: RHF + Zod, máscaras, acessibilidade, estados de erro/sucesso.
- **frontend-component**: reusabilidade, tema, a11y, performance.
- **tanstack-query-integration**: queries/mutations, cache keys, invalidations, realtime.
- **integration-external-api**: OAuth/keys, webhooks, retries, mapeamentos, logs.
- **real-time-notifications**: canais, payloads, UX não intrusiva, preferências do usuário.
- **code-review**: padrões, segurança, legibilidade, testes, docs.

---

Este planejamento é iterativo e deve ser atualizado conforme aprendizados, validações com usuários e mudanças regulatórias.
