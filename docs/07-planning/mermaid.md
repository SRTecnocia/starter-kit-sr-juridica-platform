graph TD
    A[Início] --> B{Usuário Novo?};
    B -->|Sim| C[Onboarding Guiado];
    B -->|Não| D[Login/Autenticação];
    C --> D;
    D --> E[Dashboard Principal];

    subgraph "Módulo de Gestão de Processos e Casos"
        E --> F[Acessar Módulo de Processos];
        F --> F1[Visualizar Lista de Processos];
        F1 --> F1a{Selecionar Processo Existente?};
        F1a -->|Sim| F2[Visualizar Detalhes do Processo];
        F1a -->|Não| F3[Cadastrar Novo Processo];
        F2 --> F2a[Linha do Tempo Interativa];
        F2 --> F2b[Documentos do Processo];
        F2 --> F2c[Tarefas do Processo];
        F2 --> F2d[Comunicações do Processo];
        F2c --> F2c1[Delegar Tarefa];
        F2c --> F2c2[Atualizar Status da Tarefa];
        F3 --> F3a[Preencher Dados do Processo];
        F3a --> F3b[Anexar Documentos Iniciais];
        F3b --> F3c[Definir Partes Envolvidas];
        F3c --> F3d[Salvar Processo];
        F2 --> F4[Controle de Alçadas e Aprovações];
        F4 --> F4a[Solicitar Aprovação de Ato Processual];
        F4a --> F4b{Aprovação Concedida?};
        F4b -->|Sim| F4c[Executar Ato Processual];
        F4b -->|Não| F4d[Revisar e Ajustar Ato Processual];
    end

    subgraph "Módulo de Automação e Inteligência de Dados"
        E --> G[Acessar Módulo de Automação];
        G --> G1[Captura de Andamentos e Publicações];
        G1 --> G1a[Notificação de Nova Publicação];
        G1a --> G1b[Distribuição Inteligente para Advogado];
        G --> G2[Inteligência de Prazos];
        G2 --> G2a[Sugestão Automática de Prazos];
        G2a --> G2b[Confirmação/Ajuste de Prazo];
        G --> G3[Módulo de Jurimetria - Business Intelligence];
        G3 --> G3a[Visualizar Painéis e Relatórios];
        G3a --> G3b[Personalizar Relatórios];
        G3a --> G3c[Exportar Dados para Análise];
    end

    subgraph "Módulo de Gestão de Documentos (GED) Avançado"
        E --> H[Acessar Módulo de Documentos];
        H --> H1[Armazenamento em Nuvem];
        H1 --> H1a[Upload de Documento];
        H1 --> H1b[Buscar Documento];
        H1 --> H1c[Visualizar Histórico de Versões];
        H --> H2[Automação de Documentos - Document Assembly];
        H2 --> H2a[Selecionar Modelo de Documento];
        H2a --> H2b[Responder Questionário Dinâmico];
        H2b --> H2c[Gerar Documento Completo];
        H --> H3[Assinatura Eletrônica/Digital];
        H3 --> H3a[Enviar Documento para Assinatura];
        H3a --> H3b[Acompanhar Status da Assinatura];
    end

    subgraph "Módulo Financeiro Completo"
        E --> I[Acessar Módulo Financeiro];
        I --> I1[Timesheet Inteligente];
        I1 --> I1a[Lançar Horas Trabalhadas];
        I1a --> I1b[Visualizar Relatórios de Horas];
        I --> I2[Gestão de Honorários Flexível];
        I2 --> I2a[Configurar Modelo de Honorários];
        I2a --> I2b[Registrar Pagamentos Recebidos];
        I --> I3[Faturamento em Lote - Mass Billing];
        I3 --> I3a[Gerar Faturas Automaticamente];
        I3a --> I3b[Enviar Faturas para Clientes];
        I --> I4[Controle de Contas a Pagar e Receber];
        I4 --> I4a[Registrar Despesas/Receitas];
        I4a --> I4b[Conciliação Bancária];
        I4a --> I4c[Gerar Relatório DRE];
    end

    subgraph "Módulo de Relacionamento com o Cliente (CRM & Portal do Cliente)"
        E --> J[Acessar Módulo CRM];
        J --> J1[Portal do Cliente];
        J1 --> J1a[Cliente Acessa Portal];
        J1a --> J1b[Visualiza Andamento do Processo];
        J1a --> J1c[Baixa Documentos Liberados];
        J1a --> J1d[Comunica-se via Chat];
        J --> J2[Automação de Comunicação];
        J2 --> J2a[Configurar Envio de Relatórios de Status];
        J2a --> J2b[Enviar Relatórios Personalizados];
        J --> J3[Gestão de Leads];
        J3 --> J3a[Adicionar Novo Lead];
        J3a --> J3b[Mover Lead no Funil de Vendas];
        J3a --> J3c[Converter Lead em Cliente];
    end

    subgraph "Módulo de Gestão da Equipe e Produtividade"
        E --> K[Acessar Módulo de Equipe];
        K --> K1[Dashboard de Tarefas e Delegação];
        K1 --> K1a[Visualizar Tarefas - Kanban/Lista];
        K1a --> K1b[Filtrar Tarefas];
        K1a --> K1c[Delegar Tarefa];
        K --> K2[Relatórios de Produtividade];
        K2 --> K2a[Analisar Horas Trabalhadas vs. Faturadas];
        K2a --> K2b[Identificar Oportunidades de Otimização];
    end

    subgraph "Integrações (Ecossistema)"
        E --> L[Acessar Configurações de Integrações];
        L --> L1[APIs de Tribunais];
        L1 --> L1a[Configurar Conexão com Tribunal];
        L1a --> L1b[Monitorar Status da Integração];
        L --> L2[Calendários];
        L2 --> L2a[Sincronizar com Google Calendar];
        L2a --> L2b[Sincronizar com Outlook Calendar];
        L --> L3[Sistemas Financeiros];
        L3 --> L3a[Integrar com ContaAzul/QuickBooks];
        L3a --> L3b[Mapear Contas Contábeis];
        L --> L4[Armazenamento em Nuvem];
        L4 --> L4a[Conectar Google Drive/Dropbox/OneDrive];
        L4a --> L4b[Configurar Pastas Sincronizadas];
    end

    subgraph "Requisitos Não-Funcionais (UX Impacto)"
        E --> M[Considerações de UX para Não-Funcionais];
        M --> M1[Segurança e Conformidade - UX: Confiança, Acesso Seguro];
        M --> M2[Arquitetura Multi-Tenant - UX: Isolamento, Privacidade];
        M --> M3[Performance - UX: Rapidez, Fluidez];
        M --> M4[Disponibilidade - UX: Confiabilidade, Acesso Constante];
        M --> M5[Escalabilidade - UX: Experiência Consistente com Crescimento];
    end

    subgraph "Design e UX (Experiência do Usuário e Identidade Visual)"
        E --> N[Diretrizes de Design e UX];
        N --> N1[Diretriz Principal - Linda, Moderna, Inteligente, Intuitiva];
        N --> N2[Paleta de Cores - Gradientes de Destaque];

        N --> N4[Iconografia - Estilo Linhas de Circuito];
        N --> N5[Intuitividade - Fluxo Lógico, Navegação Clara];
        N --> N6[Onboarding - Guiado, Interativo, Contextual];
    end

    D --> E;
    F3d --> F1;
    F4c --> F2;
    F4d --> F2;
    G1b --> G1;
    G2b --> G2;
    G3c --> G3;
    H1c --> H1;
    H2c --> H2;
    H3b --> H3;
    I1b --> I1;
    I2b --> I2;
    I3b --> I3;
    I4c --> I4;
    J1d --> J1;
    J2b --> J2;
    J3c --> J3;
    K1c --> K1;
    K2b --> K2;
    L1b --> L1;
    L2b --> L2;
    L3b --> L3;
    L4b --> L4;

    P1 --> P2;
    P2 --> P3;
    P3 --> P4;
    P4 --> P1;