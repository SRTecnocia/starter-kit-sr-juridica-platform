# Plano de Execução por Fases — Plataforma All‑in‑One de Gestão e Automação Jurídica

Fonte única de planejamento baseada exclusivamente em:
- `docs/README.md`
- `docs/07-planning/PRD.md`
- `docs/07-planning/mermaid.md`
- `README.md` (raiz do projeto)

Este documento organiza todas as funcionalidades mapeadas no PRD e no diagrama (mermaid) em fases sequenciais, com objetivos, entregáveis, responsáveis e dependências. Inclui especificações detalhadas de campos (máscaras, obrigatoriedade, validações) conforme stack e capacidades descritas na documentação.

## 1) Agentes (Papéis e Responsabilidades)
Com base na stack e nos módulos descritos:
- Product Manager (PM): priorização, escopo e critérios de aceite.
- Tech Lead: decisões técnicas globais, padrões e revisão.
- UX/UI Designer: fluxos (onboarding, portais, dashboards) e diretrizes visuais (tema claro/escuro, gradientes, iconografia).
- Frontend Engineer (Next.js + React + Tailwind + Radix): App Router, páginas, componentes, i18n (`next-intl`).
- Backend/API Engineer (Next.js API Routes): endpoints protegidos, webhooks, integrações server-side.
- Database/Supabase Engineer: schema, RLS, migrations, realtime.
- Integrations Engineer: APIs de Tribunais, Calendários, Financeiros, Armazenamento em nuvem.
- Data & BI Engineer: jurimetria, painéis e exportações.
- DevOps/SRE: deploy, observabilidade, SLO/SLA, automação.
- Security & Compliance: LGPD, OAB 205/2021, ISO 27001/SOC2, revisão de acesso.
- QA Engineer: testes funcionais, regressão, acessibilidade.
- Analytics Specialist: GA4, GTM, Meta Pixel, LogRocket.
- Technical Writer: documentação de uso/operacional.

Observação: Autenticação via Clerk, banco Supabase, estado cliente (Zustand) e servidor (TanStack Query), e integrações de analytics — já previstos nas docs — são tratados como capacidades habilitadoras nas fases.

## 2) Metas e Métricas (alinhadas ao PRD)
- Adoção de módulos‑chave (>70% MAU).
- Reduzir tempo de faturamento mensal em 30%.
- >=10 relatórios/mês por escritório (jurimetria).
- Churn <1% ao mês. NPS >=70.
- Dashboard com múltiplos gráficos <4s de carregamento.
- Disponibilidade 99.95%.

---

## 3) Fase 0 — Fundação do Produto e Design System
- Objetivo: habilitar rapidamente o desenvolvimento seguro e consistente.
- Módulos habilitados (docs): Autenticação (Clerk), i18n, Configuração Central, Analytics, Estado (Zustand + TanStack Query), Notificações Realtime, Tema claro/escuro.
- Entregáveis:
  - Layout base (`App Router`) com guardas de rota e sessão Clerk.
  - Tema claro/escuro com gradientes destaque; iconografia alinhada ao PRD.
  - Internacionalização pt‑BR/en (`next-intl`), navegação e breadcrumbs.
  - Providers: Zustand, TanStack Query, Analytics (GA4, GTM, Meta Pixel, LogRocket) configuráveis via `.env`.
  - Estrutura de navegação para módulos do PRD e Dashboard Principal.
- Responsáveis: Tech Lead, Frontend, Backend/API, DevOps, Analytics, Security.
- Dependências: nenhuma.
- Especificações (campos e validações de base):
  - Usuário: email (obrigatório, RFC 5322), senha (regras do Clerk), nome completo (obrigatório), telefone (opcional com máscara BR, ver abaixo).
  - Máscaras BR (conforme `README.md`):
    - CPF: `000.000.000-00` (validação de dígitos verificadores; obrigatório quando aplicável a pessoa física).
    - CNPJ: `00.000.000/0000-00` (validação DV; obrigatório quando aplicável a pessoa jurídica).
    - CEP: `00000-000` (validação e auto-complete via ViaCEP quando usado em formulários de endereço).
    - Telefone BR: `(00) 0000-0000` ou `(00) 00000-0000` (dinâmico 9º dígito; DDD obrigatório quando coletado).
  - Acessibilidade ARIA e estados de erro/sucesso/loading.
- Critérios de aceite:
  - Login/registro funcionando com Clerk; troca de idioma; tema persistente; analytics carregando sob flags.

## 4) Fase 1 — Onboarding, Autenticação e Dashboard Principal
- Objetivo: primeira experiência guiada e acesso ao hub.
- Funcionalidades (conforme `mermaid.md`):
  - Onboarding guiado para usuários novos; fluxo retorna para Login/Autenticação; Dashboard Principal.
- Entregáveis:
  - Tour interativo contextual e progressivo (tooltips/steps) cobrindo módulos principais.
  - Dashboard Principal com cartões/resumos conectados (dados demo inicialmente; realtime habilitado).
- Responsáveis: UX/UI, Frontend, Analytics, QA.
- Dependências: Fase 0.
- Especificações:
  - Eventos de analytics para cada passo do onboarding (start, step, complete).
  - Conteúdo i18n; compatível com modo escuro.
- Critérios de aceite: conclusão de onboarding rastreável, taxa de conclusão >80% em pilotos.

## 5) Fase 2 — Módulo de Gestão de Processos e Casos (Core)
- Objetivo: CRUD completo de processos, listagem e detalhes com timeline.
- Funcionalidades (PRD + `mermaid.md`):
  - Acessar módulo de processos; Visualizar Lista; Selecionar Processo; Visualizar Detalhes; Cadastrar Novo Processo.
  - Detalhes incluem: Linha do Tempo Interativa; Documentos do Processo; Tarefas do Processo (Delegar, Atualizar Status); Comunicações do Processo.
- Entregáveis:
  - Páginas: lista, criação, detalhes com abas (Timeline, Documentos, Tarefas, Comunicações).
  - Modelagem Supabase para processos, partes envolvidas, eventos/timeline, tarefas e comunicações.
  - Realtime para novos eventos na timeline.
- Responsáveis: Backend/API, Database/Supabase, Frontend, QA.
- Dependências: Fase 0, Fase 1.
- Especificações (campos):
  - Processo (criação/edição):
    - Título/Nome do Caso (obrigatório, string 3–120).
    - Tipo de Processo (obrigatório, enum: judicial/administrativo/consultivo).
    - Área (obrigatório, enum conforme catálogo interno).
    - Cliente (obrigatório; PF usa CPF máscara/validação, PJ usa CNPJ máscara/validação).
    - Partes Envolvidas (lista de pessoas físicas/jurídicas; cada item com nome, tipo, documento CPF/CNPJ conforme aplicável; todos obrigatórios nos itens).
    - Descrição/Resumo (opcional, até 2.000).
    - Endereço (opcional com CEP máscara `00000-000` e auto‑preenchimento ViaCEP quando informado).
    - Data de Início (obrigatório, data válida; formato UI `dd/mm/aaaa`).
    - Responsável Interno (obrigatório, usuário da organização).
  - Anexar Documentos Iniciais: upload múltiplo (tamanho máx. definido pelo storage), descrição opcional, confidencialidade (checkbox).
  - Timeline/Eventos: tipo do evento (obrigatório), data/hora (obrigatório), descrição (obrigatório 5–1000), anexos (opcional).
  - Tarefas do Processo: título (obrigatório), responsável (obrigatório), prioridade (enum), status (to‑do/doing/done), prazo (data/hora; obrigatório se prior > baixa), comentários (opcional).
  - Comunicações: destinatários (obrigatório), canal (email/chat interno), mensagem (obrigatória), anexos (opcional).
- Critérios de aceite: CRUD completo; filtros por responsável, tipo, status; timeline com realtime; validações BR aplicadas.

## 6) Fase 3 — Controle de Alçadas e Aprovações
- Objetivo: fluxo de aprovação para atos processuais críticos.
- Funcionalidades (PRD + `mermaid.md`):
  - Controle de Alçadas e Aprovações: Solicitar Aprovação de Ato; Decisão (Sim/Não); Executar Ato; Revisar e Ajustar.
- Entregáveis:
  - Workflow configurável por perfil/nível; registro de trilha de auditoria.
  - UI para solicitar aprovação, aprovar/recusar com justificativa, e executar atos aprovados.
- Responsáveis: Backend/API, Frontend, Security & Compliance, QA.
- Dependências: Fase 2.
- Especificações (campos):
  - Solicitação: ato (obrigatório), processo (obrigatório), justificativa (obrigatória 10–1000), prazo de aprovação (data/hora), aprovadores (lista obrigatória).
  - Decisão: aprovado (boolean obrigatório), comentário (obrigatório ao recusar), timestamp.
  - Auditoria: usuário, ação, hash/assinatura interna, IP.
- Critérios de aceite: bloqueio de execução sem aprovação; log imutável de auditoria; notificações ao status mudar.

## 7) Fase 4 — GED Avançado (Documentos)
- Objetivo: armazenamento em nuvem com versionamento, busca, automação de documentos e assinatura.
- Funcionalidades (PRD + `mermaid.md`):
  - Armazenamento em Nuvem: Upload, Buscar, Histórico de Versões.
  - Automação de Documentos (Document Assembly): Selecionar Modelo; Responder Questionário Dinâmico; Gerar Documento Completo.
  - Assinatura Eletrônica/Digital: Enviar p/ Assinatura; Acompanhar Status.
- Entregáveis:
  - Tabelas de documentos, versões, metadados e permissões (controle de acesso granular).
  - Busca por nome, tags, processo, partes e conteúdo indexado (escopo inicial por metadados; conteúdo pleno em fase futura, se aplicável).
  - Motor de templates com questionários dinâmicos (campos obrigatórios por template).
  - Integração inicial com provedor de assinatura (status: pendente, assinado, recusado, expirado).
- Responsáveis: Backend/API, Database, Integrations, Frontend, Security, QA.
- Dependências: Fases 2–3.
- Especificações (campos):
  - Documento: nome (obrigatório), tipo (obrigatório), processo (obrigatório), visibilidade (privado/time/cliente), tags (opcional), arquivo (obrigatório), versão (auto), autoria (auto), trilha de auditoria.
  - Questionário Dinâmico: cada pergunta com tipo (texto, número, data, CPF, CNPJ, CEP, telefone), obrigatório (boolean), máscara aplicada conforme tipo BR quando pertinente.
  - Assinatura: signatários (nome, email ou telefone), ordem de assinatura (opcional), mensagem, prazo (data), lembretes (boolean), status.
- Critérios de aceite: versionamento funcional; geração de documento a partir de modelo; status de assinatura refletido na UI.

## 8) Fase 5 — Automação e Inteligência de Prazos/Publicações
- Objetivo: captura automática de andamentos/publicações; sugestão de prazos com confirmação/ajuste.
- Funcionalidades (PRD + `mermaid.md`):
  - Captura de Andamentos e Publicações: Notificação de Nova Publicação; Distribuição Inteligente para Advogado.
  - Inteligência de Prazos: Sugestão Automática; Confirmação/Ajuste.
- Entregáveis:
  - Conectores/rotinas para ingestão de publicações (fonte definida por integração futura; nesta fase estruturar jobs e storage).
  - Motor de regras para prazos por tipo de publicação; UI de revisão e confirmação.
  - Notificações e distribuição ao responsável pelo caso.
- Responsáveis: Integrations, Backend/API, Frontend, QA, Security.
- Dependências: Fase 2, Fase 4.
- Especificações (campos):
  - Publicação: origem (obrigatório), data (obrigatória), referência de processo (obrigatória), conteúdo (obrigatório), classificação (opcional), anexos (opcional).
  - Sugestão de Prazo: tipo (obrigatório), data limite (obrigatória), base legal (opcional), responsável sugerido (obrigatório), necessidade de confirmação (boolean).
  - Confirmação/Ajuste: conferente (obrigatório), nova data (opcional), observações (opcional), criar tarefa automática (boolean).
- Critérios de aceite: publicações geram notificações e prazos sugeridos; confirmação cria tarefas vinculadas ao processo.

## 9) Fase 6 — Módulo Financeiro Completo
- Objetivo: time tracking, honorários, faturamento em lote, contas a pagar/receber e DRE.
- Funcionalidades (PRD + `mermaid.md`):
  - Timesheet Inteligente: Lançar Horas; Relatórios de Horas.
  - Gestão de Honorários Flexível: Configurar Modelo de Honorários; Registrar Pagamentos Recebidos.
  - Faturamento em Lote (Mass Billing): Gerar Faturas Automaticamente; Enviar Faturas.
  - Contas a Pagar e Receber: Registrar Despesas/Receitas; Conciliação Bancária; Relatório DRE.
- Entregáveis:
  - Tabelas de lançamentos de horas, contratos/modelos de honorários, faturas, lançamentos financeiros, categorias, clientes.
  - Assistente de faturamento em lote com seleção por contrato/recorrência.
  - Relatórios de horas por cliente/caso/advogado; DRE básico a partir de planos de contas.
- Responsáveis: Backend/API, Database, Frontend, Integrations (posterior para ERPs), QA.
- Dependências: Fases 2, 4, 5.
- Especificações (campos):
  - Timesheet: data (obrigatória), horas (obrigatória, decimal 0.25 steps), faturável (boolean), descrição (obrigatória 5–500), cliente/projeto/processo (obrigatório), taxa (derivada do modelo when faturável).
  - Honorários: tipo (hora/fixo/success fee/retainer/híbrido) obrigatório; valores e regras conforme tipo; vigência (datas obrigatórias), índice de reajuste (opcional enum), centro de custo (opcional).
  - Fatura: cliente (obrigatório), período (obrigatório), itens (min 1), moeda (BRL), impostos/ret/ISS (opcional), data de vencimento (obrigatória), envio por email (opcional), status (aberta/paga/atrasada/cancelada).
  - Financeiro: lançamento (receita/despesa), valor (obrigatório, moeda BRL), data (obrigatória), categoria (obrigatória), documento fiscal (opcional), forma de pagamento (enum), conciliação (flag/ID extrato).
  - Máscaras: valores monetários com formatação `R$ 1.234,56`; datas `dd/mm/aaaa`.
- Critérios de aceite: timesheet com relatórios; geração e envio de faturas; lançamentos e DRE básico.

## 10) Fase 7 — CRM & Portal do Cliente
- Objetivo: relacionamento e área do cliente com visibilidade de casos e comunicações.
- Funcionalidades (PRD + `mermaid.md`):
  - Portal do Cliente: Acesso; Visualizar Andamentos; Baixar Documentos Liberados; Chat.
  - Automação de Comunicação: Configurar Envio de Relatórios de Status; Enviar Relatórios Personalizados.
  - Gestão de Leads: Adicionar Lead; Mover no Funil; Converter em Cliente.
- Entregáveis:
  - Portal autenticado (Clerk) com escopo restrito a casos/documentos liberados.
  - Chat interno contexto processo; logs de comunicação.
  - Módulo simples de CRM (pipeline, estágios, conversão para cliente/contato).
- Responsáveis: Frontend, Backend/API, Database, UX/UI, QA.
- Dependências: Fases 2, 4, 5.
- Especificações (campos):
  - Lead: nome (obrigatório), email (obrigatório RFC), telefone (opcional máscara BR), empresa (opcional), origem (enum), estágio (enum), responsável (obrigatório), observações (opcional).
  - Relatórios de Status: seleção de cliente/casos (obrigatório), período (obrigatório), template (obrigatório), canais (email/portal), agendamento (opcional), anexos (opcional).
  - Portal Documentos: somente documentos com visibilidade para cliente; registro de download (timestamp, IP).
- Critérios de aceite: clientes acessam, visualizam andamentos e baixam documentos; leads percorrem funil; envio de relatórios.

## 11) Fase 8 — Gestão da Equipe e Produtividade
- Objetivo: visão Kanban/Lista de tarefas, filtros e relatórios de produtividade.
- Funcionalidades (PRD + `mermaid.md`):
  - Dashboard de Tarefas e Delegação: Visualizar (Kanban/Lista); Filtrar; Delegar.
  - Relatórios de Produtividade: Horas Trabalhadas vs Faturadas; Identificar Oportunidades.
- Entregáveis:
  - Board de tarefas integrado aos processos e prazos; filtros por responsável, processo, prioridade, status.
  - Relatórios de produtividade por advogado/equipe/departamento.
- Responsáveis: Frontend, Backend/API, Database, QA.
- Dependências: Fases 2, 6.
- Especificações (campos):
  - Tarefa: título (obrigatório), descrição (opcional), processo (opcional/relacionado), responsável (obrigatório), prioridade (enum), status (enum), prazo (opcional), checklists (opcional), comentários (opcional).
  - Relatório: intervalo de datas (obrigatório), agrupamento (enum), métricas (horas trabalhadas, faturáveis, taxa de utilização).
- Critérios de aceite: movimentação drag‑and‑drop; filtros funcionais; relatórios exportáveis (CSV/Excel).

## 12) Fase 9 — Jurimetria (Business Intelligence)
- Objetivo: painéis e relatórios personalizáveis; exportações.
- Funcionalidades (PRD + `mermaid.md`):
  - Visualizar Painéis e Relatórios; Personalizar Relatórios; Exportar Dados para Análise.
- Entregáveis:
  - Modelos de relatórios (tempo médio por comarca/juiz, probabilidade de êxito por tese, valores médios por tipo de ação, tendências).
  - Personalização (filtros salvos, campos visíveis, ordenação).
  - Exportação CSV/Excel para análise externa.
- Responsáveis: Data & BI, Backend/API, Frontend.
- Dependências: Fases 2, 5, 6.
- Especificações:
  - Definição de métricas, dimensões e catálogos (comarcas, juízes, tipos de ação).
  - Agregações eficientes; paginação/virtualização para grandes volumes.
- Critérios de aceite: >=10 relatórios/mês por escritório (meta) e exportações disponíveis.

## 13) Fase 10 — Integrações (Ecossistema)
- Objetivo: conectar com sistemas externos priorizados.
- Funcionalidades (PRD + `mermaid.md`):
  - APIs de Tribunais: Configurar Conexão; Monitorar Status.
  - Calendários: Sincronizar Google e Outlook.
  - Sistemas Financeiros: Integrar ContaAzul/QuickBooks; Mapear Contas Contábeis.
  - Armazenamento em Nuvem: Conectar Google Drive/Dropbox/OneDrive; Configurar Pastas Sincronizadas.
- Entregáveis:
  - Conectores com chaves/segredos armazenados de forma segura; healthchecks e telas de status.
  - Jobs de sincronização e reconciliação; logs e alertas.
- Responsáveis: Integrations, Backend/API, Security, DevOps, QA.
- Dependências: Fases 4, 5, 6.
- Especificações:
  - Campos de configuração: credenciais (obrigatórias), escopos (obrigatórios conforme provedor), webhooks (URLs validadas), mapeamentos (obrigatórios para contas contábeis), pastas sincronizadas (obrigatórias quando habilitadas).
- Critérios de aceite: sincronização inicial e periódica; status visível; erros rastreáveis.

## 14) Fase 11 — Requisitos Não‑Funcionais e Conformidade
- Objetivo: segurança, multi‑tenant, performance e disponibilidade.
- Itens (PRD): LGPD; Provimento OAB 205/2021; ISO 27001/SOC2 (processual); arquitetura multi‑tenant; performance (<4s dashboards); SLA 99.95%; escalabilidade.
- Entregáveis:
  - Isolamento lógico de dados por escritório (tenant); políticas RLS.
  - Gestão de chaves e segredos; revisão de acessos; auditorias periódicas.
  - Testes de carga; otimizações de queries/caching; CDN/edge quando aplicável.
  - Runbooks de incidentes; SLOs/SLA; páginas de status.
- Responsáveis: Security & Compliance, DevOps/SRE, Database, Backend/API.
- Dependências: Fases 2–10.
- Especificações:
  - Logs de auditoria para ações críticas; retenção e trilhas completas.
  - Backups e testes de restauração; RTO/RPO definidos.
- Critérios de aceite: cumprir metas de performance; evidências de conformidade; testes de carga aprovados.

## 15) Fase 12 — Deploy, Observabilidade e Suporte
- Objetivo: entrega contínua com monitoramento e suporte.
- Entregáveis:
  - Pipelines CI/CD; ambientes (dev/stage/prod); feature flags.
  - Observabilidade: logs, métricas, traces; alertas proativos.
  - Documentação operacional e de uso final (técnica e help center).
- Responsáveis: DevOps/SRE, Technical Writer, Analytics, QA.
- Dependências: Fases 0–11.
- Critérios de aceite: deploy sem downtime perceptível; dashboards de saúde; playbooks de suporte.

---

## 16) Mapeamento de Funcionalidades (Rastreamento)
Todas as funcionalidades do PRD e do diagrama foram alocadas:
- Onboarding/Autenticação/Dashboard: Fases 1–2.
- Gestão de Processos (cadastro, lista, detalhes, timeline, docs, tarefas, comunicações): Fase 2.
- Alçadas/Aprovações: Fase 3.
- GED (upload, busca, versões), Assembly e Assinatura: Fase 4.
- Publicações e Inteligência de Prazos: Fase 5.
- Financeiro (timesheet, honorários, faturamento, AP/AR, DRE): Fase 6.
- CRM/Portal/Chat/Relatórios de Status/Leads: Fase 7.
- Equipe e Produtividade (Kanban/Relatórios): Fase 8.
- Jurimetria/BI (painéis, personalização, export): Fase 9.
- Integrações (Tribunais, Calendários, Financeiros, Cloud Storage): Fase 10.
- Não‑Funcionais (LGPD, OAB, Multi‑tenant, Performance, SLA, Escala): Fase 11.
- Deploy/Observabilidade/Docs: Fase 12.

## 17) Dependências Cruzadas (Resumo)
- Fase 0 → habilita todas as demais.
- Fase 2 é base para 3, 4, 5, 7, 8, 9.
- Fase 4 (GED) é base para 5 (publicações com anexos), 7 (documentos no portal) e 10 (cloud sync).
- Fase 6 (financeiro) alimenta 8 (produtividade) e 10 (ERPs).

## 18) Critérios de Conclusão por Versão
- V1 (MVP Escalável): Fases 0–5 + requisitos mínimos de 11 + 12 (operacional básico). Fora de escopo conforme PRD: contencioso de massa; pesquisa de jurisprudência integrada; apps mobile nativos.
- V1.1–V1.3: Fases 6–10 + expansão de 11–12.

## 19) Anexos — Especificações de Máscaras e Validações (Consolidadas)
- CPF: `000.000.000-00` com validação de dígitos verificadores.
- CNPJ: `00.000.000/0000-00` com validação de dígitos verificadores.
- CEP: `00000-000` com consulta opcional ViaCEP para auto‑preencher endereço.
- Telefone BR: `(00) 0000-0000` e `(00) 00000-0000` (9º dígito). DDD obrigatório quando requerido.
- Email: RFC 5322; normalização para lower case.
- Datas: `dd/mm/aaaa` na UI; timezone consistente.
- Moeda: BRL com formatação `R$ 1.234,56` (armazenar em centavos/integer para precisão).
- Acessibilidade: suporte ARIA; foco/teclado; contrastes adequados para temas claro/escuro.

---

Este plano é estritamente derivado dos documentos referenciados e não implementa funcionalidades; serve como guia executivo e técnico para execução por fases.
