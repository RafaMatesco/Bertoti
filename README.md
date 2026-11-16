# Bertoti — Engenharia de Software & Projeto Integrador (4º Semestre)

Repositório de registro das entregas e contribuições individuais de **Rafael Matesco** nos projetos desenvolvidos no 4º semestre da **FATEC São José dos Campos**:

1. **[Agente de Triagem de Incidentes de Segurança (SOC Agent)](#1-projeto-agente-de-triagem-de-incidentes-de-segurança-soc-agent--es-iii)** — Engenharia de Software III ([Galaticos-API/ES-III](https://github.com/Galaticos-API/ES-III))
2. **[Sinapse — Plataforma de Gestão de Requisitos com IA](#2-projeto-sinapse--plataforma-de-gestão-de-requisitos-com-ia-api-4)** — Projeto Integrador / API 4º Semestre para a **PRO4TECH** ([Galaticos-API/API-4](https://github.com/Galaticos-API/API-4))

---

## 1. Projeto: Agente de Triagem de Incidentes de Segurança (SOC Agent — ES-III)

- **Repositório do Projeto:** [Galaticos-API/ES-III](https://github.com/Galaticos-API/ES-III)
- **Descrição:** Agente autônomo para classificação de alertas de segurança, correlação de evidências (MITRE ATT&CK) e proposição de ações de contenção com barreira obrigatória de aprovação humana (*Human-in-the-Loop*), construído com Java 21, Spring Boot 3.5, Spring AI, Ollama (`qwen2.5:7b`) e interface gráfica Swing.

### Minhas Contribuições (SOC Agent)

No projeto **SOC Agent**, atuei na frente de **Testes Automatizados e Validação dos Padrões de Projeto**, além da **organização, criação e atribuição das tarefas** da equipe.

#### 1.1. Gestão e Divisão de Tarefas da Equipe
- Estruturação das frentes de trabalho, criação das *tasks* do projeto e distribuição das responsabilidades entre os 10 integrantes da equipe cobrindo todos os padrões GoF exigidos, integração com IA local, interface gráfica e documentação.

| Integrante | Responsabilidade |
|---|---|
| **Daniel** | Integração do fluxo sem interface gráfica (*headless*) |
| **Gustavo** | Validação com o Ollama |
| **Gabriel** | Padrão *State* |
| **Emmanuel** | Integração da IA, padrões de projeto e interface |
| **Giovanni** | Fluxo de triagem, interface, testes e documentação |
| **Lucas** | Padrão *Observer* |
| **Heitor** | Padrão *Composite* na interface |
| **Cauan** | Padrão *Strategy* |
| **Rafael Matesco** | **Testes automatizados e criação/atribuição das tarefas** |
| **Victor** | Guia de apresentação |

#### 1.2. Suíte de Testes Automatizados (JUnit 5 + AssertJ)

Desenvolvimento e validação dos **24 testes unitários automatizados** que verificam o comportamento dos 5 padrões de projeto (GoF) e as garantias arquiteturais de segurança do agente. Toda a suíte foi desenhada para executar **100% offline** (sem dependência do runtime do Ollama ou de rede), utilizando dublês de teste determinísticos (`AnalistaFalso` e `SimulatedContainmentGateway`).

- **Padrão *Command* (`CommandInvokerTest.java`):**
  - `recusaSemAprovacao`: Garante o bloqueio de comandos destrutivos (`IsolateHostCommand`) antes da aprovação humana.
  - `executaAposAprovacao`: Valida execução após atingir a fase `CONTAINING`.
  - `leituraDispensaAprovacao`: Confirma execução livre de comandos somente leitura (`CollectForensicsCommand`).
  - `desfazContencao`: Valida ação compensatória (`undoLast`) restaurando o host isolado.
  - `processaFila`: Assegura ordem estrita de execução da fila (`collect_forensics` -> `isolate_host` -> `block_ip`).
  - `falhaNaoInterrompeFila`: Garante que falha de um comando não interrompa os demais da fila.
- **Padrão *State* (`IncidentStateTest.java`):**
  - `fluxoCompleto`: Percorre todas as transições válidas (`RECEIVED` -> `TRIAGING` -> `CORRELATING` -> `AWAITING_APPROVAL` -> `CONTAINING` -> `CLOSED`).
  - `contencaoExigeAprovacao`: Prova que `allowsContainmentCommands()` é `false` antes da aprovação e lança `IllegalTransitionException` fora de `AWAITING_APPROVAL`.
  - `aprovacaoNegadaEncerra`: Verifica que `deny` transiciona para `CLOSED` sem contenção.
  - `estadoTerminalRejeitaTudo`: Confirma rejeição de qualquer operação após `CLOSED`.
- **Padrão *Strategy* (`TriagePlannerTest.java`):**
  - Isolamento dos planejadores com o dublê determinístico `AnalistaFalso` (`ThreatAnalyst`).
  - `planoCompleto`, `falsoPositivoNaoAge`, `ferramentaNaoPermitida` (*allowlist* da `CommandFactory` descartando ferramentas alucinadas como `delete_all_logs`), `reactSeparaLeituraDeContencao` e `humanoNoCircuitoForcaAprovacao`.
- **Padrão *Observer* (`AgentEventBusTest.java`):**
  - `difundeParaTodos`, `falhaIsolada` (isolamento de exceções entre observadores), `registrarInscreveBeans` (`AuditTrailListener`, `ConsoleListener` via `EventListenerRegistrar`) e `filtraPorTipo` (`supports(AgentEvent)`).
- **Padrão *Composite* (`EvidenceNodeTest.java`):**
  - `contaFolhas`, `agregaSeveridade` (propagação *bottom-up* de severidade), `tratamentoUniforme`, `folhaRecusaFilhos` (`UnsupportedOperationException`) e `calculaProfundidade`.
- **Inspeção Visual da UI (`UiSnapshot.java`):**
  - Utilitário em `src/test/java/br/fatec/esiii/socagent/gui/UiSnapshot.java` para renderizar o `SocDashboardFrame` em PNG nos temas claro/escuro e estados `VAZIO`, `AGUARDANDO_APROVACAO` e `ENCERRADO` sem subir Spring ou Ollama.

**Execução dos testes no ES-III:**
```bash
./mvnw test
```

---

## 2. Projeto: Sinapse — Plataforma de Gestão de Requisitos com IA (API-4)

- **Repositório do Projeto:** [Galaticos-API/API-4](https://github.com/Galaticos-API/API-4)
- **Cliente:** PRO4TECH
- **Descrição:** Plataforma para organização e gerenciamento de requisitos de software (Projetos, Épicos, Features, PBIs e Critérios de Aceitação), análise automatizada de repositórios, busca semântica e chat conversacional com RAG (*Retrieval-Augmented Generation*), construída com **React 19 + TypeScript + Vite** (frontend), **Node.js + Express + TypeScript + PostgreSQL/pgvector** (backend) e **Python + FastAPI + Ollama** (`ai-service`).

### Minhas Contribuições (Sinapse / API-4)

No projeto **Sinapse**, atuei de forma *full-stack* e em arquitetura/documentação desde o kick-off até a integração de IA, RAG e infraestrutura de desenvolvimento:

#### 2.1. Arquitetura, Pesquisa Técnica (Spike) e Documentação
- **Kick-off e Contexto para Agentes:** Documentação inicial de alinhamento com o cliente (`docs/Kick-off/Kick-off.md`) e criação do guia de contexto técnico (`docs/AGENTS.md`).
- **Definição Arquitetural e Diagramas:** Modelagem e diagramas de **Entidade-Relacionamento (ERD)**, **Visão Geral da Arquitetura e Ingestão** e **Fluxo de Consulta Semântica (RAG)** em `docs/Architecture/`.
- **Design System:** Definição de diretrizes visuais e componentes em `docs/DESIGN_SYSTEM.md`.
- **Spike Técnico de Embeddings (`PRE-07`):** Estudo comparativo de modelos de *embedding* e script de reprodução em `docs/Spike Embeddings.md` e `scripts/spike_embeddings.py`.
- **Refinamento e Auditoria Documental:** Consolidação do `README.md`, `docs/PLANEJAMENTO_SCRUM.md`, `docs/SETUP_GUIDE.md` e remoção de duplicidades documentais.

#### 2.2. Integração Full-Stack do *RepoAnalyzer* (Análise de Repositórios com IA)
- **AI Service (Python):** Estruturação do módulo `ai-service/analyzer/` (`config.py`, `models.py`, `ollama_client.py`, `pipeline.py`, `prompts.py`, `scanner.py`), integração no `ai-service/main.py` e testes unitários em `ai-service/tests/test_analyzer.py`.
- **Banco de Dados e Backend (Node.js/TypeScript):** Migration `database/migrations/007_repo_analysis.sql` (e `database/init.sql`) e módulo completo `backend/src/modules/repo-analyses/` (`repository`, `routes`, `service`, `types`).
- **Frontend (React/TypeScript):** Interface da aba de análise de repositórios em `frontend/src/projects/RepoAnalyzerTab.tsx` e cliente API `frontend/src/projects/repo-analyzer.api.ts`.

#### 2.3. Chat Conversacional com RAG e Busca Semântica
- **Módulo de Chat no Backend:** Implementação e evolução de rotas, repositório, serviços, tipos e documentação OpenAPI (`docs/api/openapi.yaml`) em `backend/src/modules/chat/` (`chat.routes.ts`, `chat.repository.ts`, `chat.service.ts`, `rag.service.ts`, `chat.fakes.ts`), com testes de validação em `chat.validation.test.ts`.
- **Busca Semântica e Backfill de Embeddings:** Implementação de consultas semânticas em `backend/src/modules/search/search.repository.ts` e criação do script `backend/scripts/backfill-embeddings.mjs` para geração retroativa de *embeddings*.
- **Interface de Chat no Frontend:** Desenvolvimento e testes da tela de chat em `frontend/src/views/chat/ChatView.tsx`, `frontend/src/api/api_chat.ts`, `frontend/src/assets/styles/chat.css`, `frontend/src/views/chat/ChatView.test.tsx` e `frontend/src/api/api_chat.test.ts`.

#### 2.4. Autenticação, Landing Page e Frontend
- **Autenticação Persistente:** Implementação e correção do fluxo completo de login e cadastro com sessão persistente no navegador via token (`backend/src/modules/auth/`, `backend/src/middleware/requireAuth.ts`, `frontend/src/auth/Auth.tsx` e testes em `auth.routes.test.ts`).
- **Landing Page e Dashboard:** Desenvolvimento da *landing page* (`frontend/src/components/LandingPageView.tsx`), painel do desenvolvedor (`frontend/src/components/DeveloperDashboard.tsx`), melhorias de acessibilidade (`aria-label`) e refatoração estrutural do frontend (`frontend/src/App.tsx`).

#### 2.5. Infraestrutura, Docker, Scripts de Desenvolvimento e Banco de Dados
- **Scripts de Ambiente Local (`start-dev.ps1` e `stop-dev.ps1`):** Automação de inicialização do ambiente local, configuração do Docker para comunicação com o serviço de IA/Ollama no host (usando loopback IPv4), *pull* automático de modelos, parâmetros para desativar modelos e descarregamento de modelos do Ollama da memória ao encerrar (`stop-dev.ps1`).
- **Migrations e Seed Completo de Desenvolvimento:** Suporte automático a scripts de migração e carga inicial no `docker-compose.yml` e `backend/src/database/migrate.ts`, além do seed completo em `database/seed/dev_seed.sql` para testes ponta a ponta do acervo.
- **CI e Testes:** Ajustes de pipeline em `.github/workflows/ci.yml` e correções em suítes de testes de backend e frontend.
