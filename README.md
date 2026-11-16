# Bertoti — Engenharia de Software III

Repositório de entregas da disciplina de **Engenharia de Software III** (FATEC) — **Rafael Matesco**.

---

## Projeto: Agente de Triagem de Incidentes de Segurança (SOC Agent)

- **Repositório do Projeto:** [Galaticos-API/ES-III](https://github.com/Galaticos-API/ES-III)
- **Descrição:** Agente autônomo para classificação de alertas de segurança, correlação de evidências (MITRE ATT&CK) e proposição de ações de contenção com barreira obrigatória de aprovação humana (*Human-in-the-Loop*), construído com Java 21, Spring Boot 3.5, Spring AI, Ollama (`qwen2.5:7b`) e interface gráfica Swing.

---

## Minhas Contribuições (Rafael Matesco)

No projeto **SOC Agent**, atuei na frente de **Testes Automatizados e Validação dos Padrões de Projeto**, além da **organização, criação e atribuição das tarefas** da equipe.

### 1. Gestão e Divisão de Tarefas da Equipe
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

---

### 2. Suíte de Testes Automatizados (JUnit 5 + AssertJ)

Desenvolvimento e validação dos **24 testes unitários automatizados** que verificam o comportamento dos 5 padrões de projeto (GoF) e as garantias arquiteturais de segurança do agente. Toda a suíte foi desenhada para executar **100% offline** (sem dependência do runtime do Ollama ou de rede), utilizando dublês de teste determinísticos (`AnalistaFalso` e `SimulatedContainmentGateway`).

#### 2.1. Testes do Padrão *Command* (`CommandInvokerTest.java`)
Validação da execução, enfileiramento, reversibilidade (`undo`) e governança sobre comandos de resposta a incidentes:
- **`recusaSemAprovacao`**: Garante que comandos destrutivos (`IsolateHostCommand`) sejam bloqueados pelo `CommandInvoker` caso o incidente ainda não tenha recebido aprovação humana.
- **`executaAposAprovacao`**: Verifica a execução bem-sucedida de comandos de contenção quando o incidente atinge a fase `CONTAINING`.
- **`leituraDispensaAprovacao`**: Confirma que comandos apenas de investigação/leitura (`CollectForensicsCommand`) podem ser executados sem bloqueio de aprovação.
- **`desfazContencao`**: Valida a ação compensatória (`undoLast`), garantindo que um host isolado seja restaurado ao estado normal e removido da pilha de *undo*.
- **`processaFila`**: Assegura que a fila de comandos (`executeQueue`) preserve estritamente a ordem de enfileiramento (`collect_forensics` -> `isolate_host` -> `block_ip`).
- **`falhaNaoInterrompeFila`**: Garante resiliência na contenção — se um comando lançar exceção (ex.: falha de integração com EDR), o erro é registrado e os comandos seguintes da fila continuam sendo executados.

#### 2.2. Testes do Padrão *State* (`IncidentStateTest.java`)
Validação da máquina de estados do ciclo de vida do incidente e da barreira de governança:
- **`fluxoCompleto`**: Percorre todas as transições válidas (`RECEIVED` -> `TRIAGING` -> `CORRELATING` -> `AWAITING_APPROVAL` -> `CONTAINING` -> `CLOSED`).
- **`contencaoExigeAprovacao`**: Prova que `allowsContainmentCommands()` retorna `false` antes da aprovação e que tentar aprovar um incidente fora do estado `AWAITING_APPROVAL` lança `IllegalTransitionException`.
- **`aprovacaoNegadaEncerra`**: Verifica que a recusa do analista (`deny`) transiciona o incidente diretamente para `CLOSED`, mantendo os comandos de contenção bloqueados.
- **`estadoTerminalRejeitaTudo`**: Confirma que, uma vez em `CLOSED`, qualquer tentativa de nova transição (`correlate`, `approve`) é rejeitada com `IllegalTransitionException`.

#### 2.3. Testes do Padrão *Strategy* (`TriagePlannerTest.java`)
Isolamento das políticas de planejamento através de um dublê determinístico (`AnalistaFalso` implementando `ThreatAnalyst`):
- **`planoCompleto`**: Verifica se o `PlanThenExecutePlanner` traduz as ações propostas em comandos e marca `requiresApproval = true` na presença de ações destrutivas.
- **`falsoPositivoNaoAge`**: Garante que classificações `FALSE_POSITIVE` não gerem comandos de resposta nem solicitem aprovação, mesmo que o modelo sugira ações indevidamente.
- **`ferramentaNaoPermitida`**: Testa a lista de permissão (*allowlist*) da `CommandFactory`, comprovando que ferramentas alucinadas pelo LLM (como `delete_all_logs`) são descartadas com segurança.
- **`reactSeparaLeituraDeContencao`**: Valida que o `ReActPlanner` executa comandos de leitura (`lookup_mitre`, `collect_forensics`) durante o loop de raciocínio e retém os comandos destrutivos (`isolate_host`) para aprovação humana.
- **`humanoNoCircuitoForcaAprovacao`**: Comprova que o decorador/estratégia `HumanInTheLoopPlanner` exige aprovação humana mesmo para planos compostos exclusivamente por comandos de leitura.

#### 2.4. Testes do Padrão *Observer* (`AgentEventBusTest.java`)
Verificação da difusão de eventos e isolamento de falhas entre observadores:
- **`difundeParaTodos`**: Confirma a entrega de eventos publicados no `AgentEventBus` para múltiplos observadores inscritos simultaneamente.
- **`falhaIsolada`**: Garante que uma exceção lançada por um observador defeituoso seja capturada pelo barramento sem interromper a notificação dos demais observadores (preservando a trilha de auditoria).
- **`registrarInscreveBeans`**: Valida o registro automático de beans Spring (`AuditTrailListener`, `ConsoleListener`) via `EventListenerRegistrar`.
- **`filtraPorTipo`**: Testa a capacidade de inscrição seletiva através do método `supports(AgentEvent)`.

#### 2.5. Testes do Padrão *Composite* (`EvidenceNodeTest.java`)
Validação da estrutura em árvore para exibição de evidências na interface gráfica:
- **`contaFolhas`**: Verifica a contagem recursiva (`leafCount()`) de evidências através de grupos aninhados (`EvidenceGroup` e `EvidenceLeaf`).
- **`agregaSeveridade`**: Comprova a propagação *bottom-up* da maior criticidade (`highestSeverity()`), fazendo com que um grupo de nível `INFO` ou `LOW` reflita a severidade `CRITICAL` de seus descendentes.
- **`tratamentoUniforme`**: Demonstra que folhas e nós compostos respondem de maneira uniforme à interface `EvidenceNode`.
- **`folhaRecusaFilhos`**: Assegura que a tentativa de adicionar nós filhos a uma `EvidenceLeaf` lance `UnsupportedOperationException`.
- **`calculaProfundidade`**: Verifica o cálculo recursivo de profundidade (`depth()`) da árvore de evidências.

#### 2.6. Utilitário de Captura e Inspeção Visual da UI (`UiSnapshot.java`)
- Apoio nos testes de interface gráfica em `src/test/java/br/fatec/esiii/socagent/gui/UiSnapshot.java`, permitindo renderizar o `SocDashboardFrame` de forma determinística (temas claro/escuro e estados `VAZIO`, `AGUARDANDO_APROVACAO` e `ENCERRADO`) em arquivos PNG sem necessidade de subir o Spring ou o Ollama.

---

### Como Executar os Testes no Projeto Principal

No repositório [ES-III](https://github.com/Galaticos-API/ES-III), basta executar:

```bash
./mvnw test
```

