<!-- anchor-research-brainstorming-002-003.md | Vitalia SPEC Kit v0.0.1 | 07-11-2026 20:25:00(GMT-04:00) -->
> **anchor-research-brainstorming-002-003.md | Vitalia SPEC Kit v0.0.1 | 07-11-2026 20:25:00(GMT-04:00)**

# Âncora Consolidada e Harmonizada de Concepção e Pesquisa (Specs 002 + 003 + Ciclo Corretivo SDD)

Este documento é a **Fonte Canônica Única de Verdade (Single Source of Truth)** para o próximo turno de engenharia (após o rollback do repositório raiz para a Spec 001 no commit `23bb91b`). Ele reúne e harmoniza **integralmente** as decisões, contratos, códigos canônicos, análises de engenharia reversa e rastros de pesquisa das **Specs 002 e 003** com as correções definitivas do **Ciclo 2 (Pós-Mortem WSL2/zsh)** e com o código já preparado em `~/.vitalia/`, eliminando contradições históricas para evitar alucinações.

---

## 0. Regra Fundamental de Topologia e Isolamento das 4 Camadas (Inviolável)

Todo agente ou desenvolvedor que operar a partir desta âncora DEVE respeitar a separação estrita entre as **4 camadas físicas e de versionamento**:

```
┌──────────────────────────────────────────────────────────────────────────────────────────────┐
│ 1. REPOSITÓRIO RAIZ DO PROJETO (/home/andre/projetos/assistidos/vitalia-spec-kit)            │
│    • Todo o código-fonte do kit mora ESTRITAMENTE dentro da pasta `source/`:                 │
│      source/{bin,extensions,profiles,src,integrations,templates,AGENTS.md}                   │
│    • Nenhum artefato de memória de sessão (.vitalia/memory/*) é commitado neste repositório. │
├──────────────────────────────────────────────────────────────────────────────────────────────┤
│ 2. KIT INSTALADO NA MÁQUINA DO USUÁRIO (Padrão: ~/.vitalia-spec-kit)                         │
│    • Quando o kit é instalado na máquina via `kit-bootstrap.sh`, o código que mora em        │
│      `source/` no projeto raiz é instalado diretamente na pasta escolhida pelo usuário       │
│      (sem o nível intermediário `source/`), sendo `~/.vitalia-spec-kit` a opção padrão.      │
│    • O caminho escolhido é persistido em `VITALIA_KIT_DIR` e abriga também o `.venv/`        │
│      isolado do kit (`${VITALIA_KIT_DIR}/.venv/bin/python3`).                                │
├──────────────────────────────────────────────────────────────────────────────────────────────┤
│ 3. REPOSITÓRIO EXCLUSIVO DE CONTEXTO DE SESSÃO (.vitalia/memory/session/)                    │
│    • Reside dentro do projeto local, mas é versionado em um repositório Git SEPARADO         │
│      exclusivo para contexto (ex: `vitalia-spec-kit-context`).                               │
│    • Cada máquina que trabalha com o kit possui sua própria coleção/shard em                 │
│      `shards/<machine_id>.yaml`, além do ledger compartilhado em `data/decisions.jsonl`,     │
│      `data/learnings.jsonl` e `data/session_history.jsonl`, com timestamps estritamente no   │
│      formato canônico `DD-MM-YYYY HH:MM:SS(GMT-04:00)`.                                      │
│    • As etapas `session-start`, `session-end` e `session-consolidate` são processadas pelos  │
│      scripts CLI do kit (`vitalia_context_engine.py`); seus arquivos `.toml` NÃO devem       │
│      conter código ad-hoc nem gerar arquivos fora desse motor.                               │
├──────────────────────────────────────────────────────────────────────────────────────────────┤
│ 4. REPOSITÓRIO EXCLUSIVO DE DADOS DO PROJETO (.vitalia/memory/data_storage/)                 │
│    • Reside dentro do projeto local, mas é versionado em um repositório Git SEPARADO         │
│      exclusivo para dados do projeto (ex: `vitalia-spec-kit-data`).                          │
│    • Armazena os payloads de Context Lifting (`context/`), handoffs (`handoffs/`) e          │
│      documentos de pesquisa/âncora (`research/`).                                            │
└──────────────────────────────────────────────────────────────────────────────────────────────┘
```

---

# PARTE I — Fundações Arquiteturais e Contratos SDD (Harmonizados da Spec 002 + Código Ativo em `~/.vitalia`)

## 1. Sumário Executivo e Propósito

O objetivo desta arquitetura é transformar o **Vitalia SPEC Kit (v0.0.1)** em um produto **100% autônomo, distribuível e instalável via `git clone` em qualquer máquina limpa**, eliminando dependências de containers Docker locais (Ollama, Redis, Postgres), podas artificiais de contexto de 6 GB de VRAM e caminhos absolutos hardcoded, enquanto blinda o pipeline SDD contra testes rasos e regressões silenciosas.

Esta Parte I consolida:
1. **O Rastro de Pesquisa de 2026** (padrões da Anthropic para Managed Agents, isolamento de runtime e ISO 29148);
2. **Os Pivôs Arquiteturais Críticos** harmonizados com as Lentes de Fase Sem Poda (`gp_*.yaml`);
3. **A Análise Individual e Contratos Formais dos 16 Arquivos `.toml` e Perfis `gp_*.yaml`** alinhados ao código ativo em `~/.vitalia/`;
4. **A Regência do `vitalia-architect` (`architect.toml`)** e o catálogo canônico de 7 domínios de `brainstorming_intents.yaml`;
5. **A Ordem de Instruções em `source/AGENTS.md`** contra a diluição de contexto (*Lost in the Middle*).

---

## 2. Rastro de Pesquisa e Grounding do Estado da Arte (2026)

Em conformidade com o princípio constitucional **P03 (External Verification Protocol)**, toda decisão técnica abaixo é ancorada em literatura e documentação oficial verificada de 2026:

| # | Fonte Canônica | URL Verificada | Data | Princípio Técnico de 2026 Incorporado |
|---|---|---|---|---|
| **E1** | **Anthropic Engineering (Abril 2026)** — *Scaling Managed Agents: Decoupling the brain from the hands* | [`https://www.anthropic.com/engineering/managed-agents`](https://www.anthropic.com/engineering/managed-agents) | 03-11-2026 | **Decouple Brain from Hands & Session:** O harness do LLM (Brain) é *stateless* e descartável (*cattle*). As ferramentas/sandboxes (Hands) operam de forma isolada. O log de eventos (Session) é durável e reside fora da janela de contexto (*"The session is not the context window"*). |
| **E2** | **Anthropic Platform Docs (2026)** — *Claude Managed Agents Overview* | [`https://platform.claude.com/docs/en/managed-agents/overview`](https://platform.claude.com/docs/en/managed-agents/overview) | 03-11-2026 | **Context Interrogation & Positional Slices:** O modelo interroga o histórico sob demanda via fatias posicionais e *Prompt Caching*, em vez de manter históricos monolíticos estáticos. |
| **E3** | **LMSYS / UC Berkeley** — *Judging LLM-as-a-Judge with MT-Bench & Chatbot Arena* | [`https://arxiv.org/abs/2306.05685`](https://arxiv.org/abs/2306.05685) | 03-11-2026 | **Inverted Chain-of-Thought (Critique-First, Verdict-Last):** Proibição de veredito antecipado em avaliações constitucionais para mitigar viés de posição e auto-afirmação. Subsidia o Sub-Agente `/vitalia-judge`. |
| **E4** | **PyPA** — *pipx & venv: Isolated Python Application Environments* | [`https://docs.python.org/3/library/venv.html`](https://docs.python.org/3/library/venv.html) | 06-11-2026 | **Runtime Isolation:** O kit mantém seu próprio interpretador e bibliotecas na pasta de instalação escolhida pelo usuário (padrão `~/.vitalia-spec-kit/.venv/`), expondo shims sem interferir no projeto host. |
| **E5** | **ISO/IEC/IEEE 29148:2018** — *Systems and Software Engineering: Requirements Engineering* | [`https://www.iso.org/standard/72089.html`](https://www.iso.org/standard/72089.html) | 03-11-2026 | **Critérios Canônicos de Requisitos:** Requisitos individuais devem ser estritamente *Unambiguous* (interpretação única), *Measurable* (quantificáveis e observáveis em caixa-preta) e *Singular* (sem conjunções ocultas). Embaseia o checklist de qualidade do `spec-specify.toml`. |

---

## 3. Registro dos Pivôs Críticos da Arquitetura

### Pivô 1: Fim da Poda de Contexto de 6 GB VRAM e Adoção de Lentes de Fase Sem Poda (`gp_*.yaml`)
* **Estado Anterior:** Perfis `gp_*.yaml` impunham tetos artificiais (`total_budget = 1000` ou `4000`) e filtravam domínios para viabilizar inferência em GPUs modestas (GTX 1060 de 6 GB) via Ollama local, mutilando a Constituição e truncando o histórico em `n=5`.
* **Decisão Consolidada:** No Vitalia SPEC Kit, a inferência opera na IDE Antigravity com **modelos de fronteira**. Os arquivos `gp_*.yaml` **NÃO são excluídos**: eles operam como **Lentes Adversariais de Fase Sem Poda** (`no_prune: true`, `total_budget: 0`), fazendo o Guardian (`guardian_context.py`) injetar **100% da `constitution.yaml`**, o histórico completo de decisões/aprendizados (sem corte `n=5`) e os guardrails específicos daquela fase (`phase_guardrails` e `phase_profiles`).

### Pivô 2: Saneamento da Infraestrutura Conteinerizada
* **Estado Anterior:** O `scan_environment.py` e o `kit_env_bootstrap.py` executavam probes de socket TCP nas portas `6379` (Redis) e `5432` (Postgres), disparavam `docker compose up -d vitalia_redis` e geravam status `DEGRADED` se os containers não estivessem ativos.
* **Decisão Consolidada:** Toda dependência de containers locais foi abandonada. O kit opera no filesystem puro. O `scan_environment.py` verifica estritamente: (a) Presença do `.env` local do projeto; (b) Presença de `.agents/AGENTS.md` e pureza de `.agents/skills/`; (c) Integridade do runtime isolado em `${VITALIA_KIT_DIR}/.venv/` (padrão `~/.vitalia-spec-kit/.venv/`).

### Pivô 3: Destino do Context Lifting na Memória 3-Tier
* **Estado Anterior:** Os arquivos de contexto gerados pelo hook runner e pelo Guardian eram despejados na pasta global `~/.vitalia/tmp/`.
* **Decisão Consolidada:** Para cumprir o princípio **P07 (Zero Data Leak)** e respeitar o isolamento por projeto e por máquina, os arquivos de Context Lifting são gerados dentro do repositório de dados do projeto:
  ```
  .vitalia/memory/data_storage/context/<machine_id>/<workflow>_<timestamp>_<hash>.json
  ```

### Pivô 4: Silêncio Arquitetural sobre o CLI Legado `sdd_judge.py`
* **Estado Anterior:** Workflows chamavam `sdd_judge.py` via terminal ou continham instruções negativas contraditórias.
* **Decisão Consolidada:** O script CLI legado `sdd_judge.py` não faz parte do novo produto. Toda auditoria constitucional (`specify`, `plan`, `tasks`, `after_task`, `clarify`, `converge`) é executada **nativamente na IDE Antigravity via Sub-Agente** (`/vitalia-judge`, `/vitalia-task-verifier`, `/vitalia-clarify`, `/vitalia-converge`), aplicando Sanduíche Constitucional, FMEA Mental, *Inverted CoT* e pausa HITL soberana.

### Pivô 5: Papel do Brainstorming, Researcher, Route e Architect
* **Decisão Consolidada:**
  * O **`/vitalia-brainstorming`** é o Supervisor Socrático de concepção (protocolo STOP → SCAN → ASK → WAIT em turnos de até 3 perguntas, realizando Engenharia Reversa Funcional em 3 Camadas e classificando a demanda na Matriz P05 de 4 Modalidades).
  * O **`/vitalia-research` (`vitalia-researcher`)** é o Sub-Agente e Workflow de Pesquisa Profunda com acesso real à rede (`search_web` + `run_command` HTTP 200), leitura integral de documentação oficial, detecção de depreciação e Engenharia Reversa Funcional em 3 Camadas.
  * O **`/vitalia-route`** atua como despachante semântico determinístico consultando `brainstorming_intents.yaml` → `agents_catalog.yaml` → `routing_table.yaml`.
  * O **`/vitalia-architect` (Vitalia Architect)** recebe os consensos de engenharia, valida contra a Constituição integral, registra ADRs e conduz formalmente o pipeline SDD:
    `brainstorming → route → architect → specify → clarify → plan → tasks → task-verifier → analyze → implement → converge → judge → session-end`.

> **Nota sobre a Estratégia de Testes (P05):** O antigo Pivô 6 (bifurcação simples de 3 vias) foi **substituído e expandido pela Decisão 3 da PARTE III** (**Matriz Bidimensional de 4 Modalidades: `[TEST:NEW]`, `[TEST:BUGFIX]`, `[TEST:REFACTOR]`, `[TEST:MIGRATION]` + Cenário de Falsificação obrigatório**).

### Pivô 6 (Ex-Pivô 7): Escopo Focado em Fundações de Runtime, Governança e Fidelidade Funcional
* **Decisão Consolidada:** O escopo da entrega unificada (Specs 002 + 003) concentra-se nas **fundações de runtime, instalação universal, motor de sessão 3-Tier rico e blindagem adversarial do ciclo SDD**. A ativação executável do esquadrão de 5 especialistas de domínio (`Django-Stark`, `React-Vinci`, `Infra-Nomad`, `Cortex-Analyst`, `Figma-Maestro`) permanece alocada para evolução futura.

### Pivô 7 (Ex-Pivô 8): Diagramas Mermaid em Tema Nativo e Programação Defensiva (P16)
* **Decisão Consolidada:** Proibição de classes CSS customizadas (`fill:#...`, `stroke:#...`) nos diagramas Mermaid para herdar a paleta nativa da IDE, e obrigatoriedade de aspas duplas em todos os labels de nós/arestas contendo caracteres especiais `()`, `[]`, `|`, `{}`.

---

## 4. Análise Individual e Contratos dos 16 Arquivos `.toml` (`source/extensions/`) e Lentes `gp_*.yaml` (`source/profiles/`)

Para cada arquivo TOML do novo produto, definem-se a perspectiva do LLM, recursos, lente de fase (`gp_*.yaml` com `no_prune: true` e `total_budget: 0`) e os contratos formais de aceite (`ACs`), alinhados ao código ativo em `/home/andre/.vitalia/`:

### 1. `session-start.toml` (`guardian_profile = "gp_session-start"`)
* **Perspectiva do LLM:** Lê `.vitalia/memory/session/` (repositório compartilhado de contexto). Apresenta a feature ativa, semáforo de sincronização e o próximo passo P0.
* **Recursos:** Ferramentas `run_command` e `read_file` (opcional: `write_file`).
* **Contrato de Aceite (ACs):**
  * `AC-START-01`: Remover `hardware_profile = "lightweight"` em favor de `"frontier"` e `version = "0.0.1"`.
  * `AC-START-02`: Definir `preferred = "filesystem"` no bloco `[transport]` (purgar `redis` e `redis_key`).
  * `AC-START-03`: Executar `git -C .vitalia/memory/session pull --rebase` para sincronizar o contexto compartilhado da equipe. Se a pasta `data/` não existir, inicializar via `python3 {{VITALIA_KIT_DIR}}/src/vitalia_context_engine.py --action init`; se existir, executar `--action consolidate` de forma resiliente.
  * `AC-START-04`: Interrogar o log posicional da sessão (`SESSION_STATE.md`, `LEARNINGS.md` e `README.md`) sem carregar históricos mortos; toda lógica de processamento reside no script CLI `vitalia_context_engine.py` (o `.toml` não contém código nem gera arquivos fora desse motor).

### 2. `spec-specify.toml` (`guardian_profile = "gp_spec-specify"`)
* **Perspectiva do LLM:** Consome `specs/*/research-brainstorming.md` (quando disponível) e o input do usuário. Produz a especificação técnica agnóstica de tecnologia em `spec.md` e valida via checklist ISO 29148.
* **Recursos:** `read_file`, `write_file` (opcionais: `search_web`, `run_command`, `define_subagent`, `invoke_subagent`).
* **Lente de Fase (`gp_spec-specify.yaml`):** `no_prune: true`, `total_budget: 0`, injetando `phase_guardrails` de Engenharia de Requisitos e Contratos.
* **Contrato de Aceite (ACs):**
  * `AC-SPEC-01`: Incluir `specs/*/research-brainstorming.md` na chave `[context] reads`, `preferred = "filesystem"` e `guardian_profile = "gp_spec-specify"`.
  * `AC-SPEC-02`: Requisitos funcionais (`FR-xxx`) estruturados sem detalhes prematuros de implementação, e `SC-xxx` / `Given/When/Then` definindo **comportamentos observáveis de caixa-preta e invariantes de pureza do contrato do consumidor** (proibidos critérios baseados na mera existência sintática de arquivos ou strings).
  * `AC-SPEC-03`: Seção obrigatória `## Suposições Verificadas` preenchida via delegação a Sub-Agente com acesso à rede (`define_subagent` `vitalia-researcher` ou `self` — proibido usar o subagente `research` built-in sem rede), com validação HTTP 200 real, leitura integral da página/documentação oficial e verificação de depreciação (P03).
  * `AC-SPEC-04`: Quando envolver correção de bugs ou migração/refatoração de componentes existentes, incluir a seção `## Comportamentos Existentes a Preservar & Defeitos a Reproduzir`.
  * `AC-SPEC-05`: Disparo compulsório do Sub-Agente `/vitalia-judge --mode=specify` com pausa soberana para aprovação humana (HITL).

### 3. `judge.toml` (`guardian_profile = "gp_sdd-judge"`) — Sub-Agente Constitucional Adversarial
* **Perspectiva do LLM:** Auditor Red Team e *Failure Hunter*. Lê o artefato da fase sob inspeção (`specify`, `plan`, `tasks`, `after_task`, `clarify`, `converge`) e o confronta com a totalidade da Constituição (P00 a P17).
* **Recursos:** `read_file`, `write_file` (opcionais: `grep_search`, `list_dir`).
* **Lente de Fase (`gp_sdd-judge.yaml`):** `no_prune: true`, `total_budget: 0`, contendo `mandatory_fmea_questions` e `phase_profiles` específicos por modo.
* **Contrato de Aceite (ACs):**
  * `AC-JUDGE-01`: Ingestão da Constituição integral via Guardian (`no_prune: true`) sem podas de token budget, `preferred = "filesystem"` e `version = "0.0.1"`.
  * `AC-JUDGE-02`: **Red Team Failure Hunt (FMEA Mental Obrigatório em 4 Questões):**
    1. *Hipóteses de Ambiente Limpo (Clean Machine Assumptions):* Que dependências de SO, variáveis, shells (ex: WSL2/zsh) ou estados prévios falharão numa instalação limpa?
    2. *Auditoria Anti-Shallow Testing (P05):* Os testes ou tarefas de teste validam execução comportamental de caixa-preta com Cenário de Falsificação e modalidade (`[TEST:NEW|BUGFIX|REFACTOR|MIGRATION]`), ou limitam-se a `assert path.is_file()` / `assert "str" in read_text()`? **(Se detectar teste raso → emitir veredito `BLOCK` sumário).**
    3. *Regressão por Simplificação Indevida:* Em refatorações/migrações, algum comportamento estável anterior foi silenciosamente amputado (confrontar com a Matriz de Migração do `plan.md`)?
    4. *Violação de Contrato do Consumidor:* O artefato polui diretórios estritos do cliente (ex: `.trash` ou `.vitalia-manifest` em `.agents/skills/`) ou viola schemas OpenAPI 3.0 limpos?
  * `AC-JUDGE-03`: Aplicação estrita de *Inverted Chain-of-Thought (E3)*: extração literal de evidências com número de linha → respostas às 4 questões do FMEA Mental + análise crítica individual 1-a-1 → emissão do veredito `PASS`, `WARN` ou `BLOCK` por último.
  * `AC-JUDGE-04`: Emissão obrigatória do artefato físico `specs/{FEATURE}/Relatório de Inspeção - {fase}.md` e pausa mandatória para consentimento HITL soberano. Silêncio total sobre CLIs legados (`sdd_judge.py`).

### 4. `spec-plan.toml` (`guardian_profile = "gp_spec-plan"`)
* **Perspectiva do LLM:** Lê `spec.md` aprovada. Desenha arquitetura técnica, modelo de dados, contratos de interface, Matriz de Migração e Test Depth Agreement (TDA).
* **Recursos:** `read_file`, `write_file` (opcionais: `search_web`, `run_command`, `define_subagent`, `invoke_subagent`).
* **Lente de Fase (`gp_spec-plan.yaml`):** `no_prune: true`, `total_budget: 0`.
* **Contrato de Aceite (ACs):**
  * `AC-PLAN-01`: Esquemas em `contracts/` estritamente compatíveis com OpenAPI 3.0 puro (proibidos `$schema`, `title` raiz e `additionalProperties: false`), `preferred = "filesystem"` e `guardian_profile = "gp_spec-plan"`.
  * `AC-PLAN-02`: Decisões técnicas em `research.md` verificadas via Sub-Agente com acesso à rede (`vitalia-researcher` / `self`), exigindo URL com HTTP 200 comprovado, leitura integral da página oficial e checagem de depreciação.
  * `AC-PLAN-03`: **Matriz de Migração e Preservação Comportamental (Obrigatória ao alterar/refatorar componentes existentes):** Baseada na Engenharia Reversa Funcional em 3 Camadas: `| Componente | Comportamento Existente a Preservar (Invariante de Negócio) | Acoplamento / Defeito a Remover | Contrato do Consumidor & Estratégia de Teste ([TEST:MIGRATION]) |`.
  * `AC-PLAN-04`: **Test Depth Agreement (TDA — Matriz Constitucional P05):** Estipular para cada componente a modalidade (`[TEST:NEW]`, `[TEST:BUGFIX]`, `[TEST:REFACTOR]`, `[TEST:MIGRATION]`) e a profundidade mínima de teste, proibindo testes sintáticos `assert file.is_file()`.
  * `AC-PLAN-05`: Submissão mandatória à auditoria `/vitalia-judge --mode=plan`.

### 5. `spec-tasks.toml` (`guardian_profile = "gp_spec-tasks"`)
* **Perspectiva do LLM:** Converte `plan.md` em tarefas atômicas sequenciais (`T001 [P] [US1]`), definindo arquivos exatos, modalidade da Matriz P05 e Cenário de Falsificação.
* **Recursos:** `read_file`, `write_file`.
* **Lente de Fase (`gp_spec-tasks.yaml`):** `no_prune: true`, `total_budget: 0`.
* **Contrato de Aceite (ACs):**
  * `AC-TASKS-01`: Aplicar a **Matriz Bidimensional de 4 Modalidades (P05)** e o **Cenário de Falsificação obrigatório** em toda tarefa de teste:
    `- [ ] T### [P] [USN] [TEST:NEW|BUGFIX|REFACTOR|MIGRATION] Criar/executar teste em caminho/do/teste.py — Cenário de Falsificação: <comportamento de negócio, execução real ou invariante de contrato que fará o teste falhar se quebrado/omitido>`.
  * `AC-TASKS-02`: Proibição absoluta de tarefas de teste rasas/sintáticas (`assert path.is_file()`). Cada linha da Matriz de Migração do `plan.md` deve originar uma tarefa `[TEST:MIGRATION]`.
  * `AC-TASKS-03`: Rastreabilidade explícita com User Stories (`[US1]`), `preferred = "filesystem"` e handoff obrigatório para `/vitalia-task-verifier` (sem auto-aprovação).

### 6. `task-verifier.toml` (`guardian_profile = "gp_task-verifier"`)
* **Perspectiva do LLM:** Auditor Adversarial Red Team Constitucional de Tarefas e Gatekeeper de Profundidade/Falsificação.
* **Recursos:** `read_file`, `write_file` (opcional: `run_command`).
* **Lente de Fase (`gp_task-verifier.yaml`):** `no_prune: true`, `total_budget: 0`.
* **Contrato de Aceite (ACs):**
  * `AC-TVER-01`: **Veto BLOCKING Imediato** para qualquer tarefa de teste sem tag `[TEST:NEW|BUGFIX|REFACTOR|MIGRATION]`, sem **Cenário de Falsificação** explícito ou baseada apenas em existência sintática de arquivos/strings (`is_file()`).
  * `AC-TVER-02`: Verificar correspondência 1-a-1 com o TDA e a Matriz de Migração do `plan.md`, e exigir que tarefas de correção de bug sejam precedidas por `[TEST:BUGFIX]`.
  * `AC-TVER-03`: Gerar `specs/{feature}/Relatório de inspeção.md`, pausar obrigatoriamente para confirmação HITL soberana no chat e, após aprovação, criar `tasks.md.bak` e selar `tasks.md` com o carimbo constitucional.

### 7. `spec-implement.toml` (`guardian_profile = "gp_spec-implement"`)
* **Perspectiva do LLM:** Executa o `tasks.md` aprovado tarefa por tarefa, comprovando Red/Green Phase no terminal, rodando testes e atualizando `[ ]` para `[/]` e `[X]`.
* **Recursos:** `read_file`, `write_file` (opcionais: `web_search`, `run_command`).
* **Lente de Fase (`gp_spec-implement.yaml`):** `no_prune: true`, `total_budget: 0`.
* **Contrato de Aceite (ACs):**
  * `AC-IMPL-01`: **Prova Empírica de Falha Prévia (P05):** Para tarefas `[TEST:NEW]` e `[TEST:BUGFIX]`, executar o teste via `pytest` no terminal **ANTES** de implementar o código/fix e confirmar que ele falha pelo motivo de negócio esperado (se já passar sozinho antes da correção, é falso positivo e deve ser reescrito).
  * `AC-IMPL-02`: **Proibição de Reescrita Cega (`[TEST:REFACTOR]` / `[TEST:MIGRATION]`):** Preservar 100% dos comportamentos estáveis mapeados na Matriz de Migração do `plan.md`, preservar cabeçalhos P04/docstrings e respeitar a pureza de diretórios do consumidor (ex: zero `.trash` ou `.vitalia-manifest` em `.agents/skills/`).
  * `AC-IMPL-03`: Proibição absoluta de marcar `[X]` sem execução comprovada da suíte de testes retornando código 0, com Gate Pós-Task Adversarial nativo na IDE, Trava de Escalação Invertida para `/vitalia-brainstorming` e Context Lifting em `.vitalia/memory/data_storage/context/`.

### 8. `analyze.toml` (`guardian_profile = "gp_analyze"`)
* **Perspectiva do LLM:** Inspetor *read-only* de consistência documental (`after_tasks`) e auditor de deltas de implementação (`after_implement`).
* **Recursos:** `read_file`, `list_dir`, `grep_search`, `write_file`, `run_command`.
* **Lente de Fase (`gp_analyze.yaml`):** `no_prune: true`, `total_budget: 0`.
* **Contrato de Aceite (ACs):**
  * `AC-ANALYZE-01`: Modo `after_tasks`: executar os **7 passes adversariais** sobre `spec.md`, `plan.md` e `tasks.md` (Duplicações, Ambiguidades, Subespecificação/Omissão na Matriz de Migração, Violações da Constituição sem poda, Gaps de Cobertura, Inconsistências/Contrato do Consumidor e Superficialidade de Testes/Ausência de Modalidade P05 ou Cenário de Falsificação).
  * `AC-ANALYZE-02`: Modo `after_implement`: inspecionar Git Diff em **3 camadas** (*Camada 1: Previsto em Task*, *Camada 2: Adaptação Correlata*, *Camada 3: Órfão Real / Drift*) e realizar **verificação estática mandatória de call-sites** (aridade, imports e assinaturas).
  * `AC-ANALYZE-03`: Emissão dual do relatório (no chat da IDE e em `specs/<feature>/Relatório de Análise e Consistência - <modo>.md`) com bloqueio em caso de `CRITICAL` e **pausa mandatória para consentimento humano (HITL Soberano)**.

### 9. `converge.toml` (`guardian_profile = "gp_sdd-judge"`)
* **Perspectiva do LLM:** Auditoria de convergência quadridimensional entre código real implementado e artefatos SDD (`spec.md`, `plan.md`, `tasks.md`).
* **Recursos:** `read_file`, `list_dir`, `grep_search`, `write_file` (opcional: `run_command`).
* **Contrato de Aceite (ACs):**
  * `AC-CONV-01`: Reversão de status no `tasks.md` para qualquer tarefa falsamente marcada como `[X]` que contenha stubs vazios (`pass`), testes sintáticos/rasos (`assert path.is_file()`), amputação de comportamentos estáveis da Matriz de Migração ou falha em cenário de máquina limpa.
  * `AC-CONV-02`: Disparo do Sub-Agente `/vitalia-judge --mode=converge` nativo na IDE com `preferred = "filesystem"`.

### 10. `clarify.toml` (`guardian_profile = "gp_sdd-judge"`)
* **Perspectiva do LLM:** Detecta e resolve ambiguidades cirurgicamente no `spec.md` ativo antes do `spec-plan`.
* **Recursos:** `read_file`, `write_file`.
* **Contrato de Aceite (ACs):**
  * `AC-CLAR-01`: Aplicar a **Taxonomia de 10 Categorias** (ISO/IEC/IEEE 29148 & Berry et al., incluindo *Consumer Contract Omission*) via Sub-Agente SDD-Judge nativo (`--mode clarify`, sem CLI legado `sdd_judge.py`), calculando `Priority Score` e gerando `specs/*/ambiguity_metrics.json`.
  * `AC-CLAR-02`: Apresentar uma pergunta por vez e integrar a resposta imediatamente no `spec.md`.

### 11. `research.toml` (`guardian_profile = "gp_research"`) — Workflow `/vitalia-research` & Sub-Agente `vitalia-researcher`
* **Perspectiva do LLM:** Vitalia Researcher — especialista em pesquisa profunda de fronteira, verificação empírica HTTP 200 (P03), análise de contratos do consumidor e engenharia reversa funcional.
* **Recursos:** `read_file`, `write_file` (opcionais: `search_web`, `run_command`, `define_subagent`, `invoke_subagent`).
* **Lente de Fase (`gp_research.yaml`):** `no_prune: true`, `total_budget: 0`.
* **Contrato de Aceite (ACs):**
  * `AC-RES-01`: Operar com acesso real à rede via `define_subagent` (`vitalia-researcher` com `EnableWriteTools: true`) ou `self` (contornando o sandbox sem rede do subagente `research` built-in da IDE).
  * `AC-RES-02`: Executar os **4 Eixos Mandatórios**: (1) Compreensão do Contrato Estrito do Cliente; (2) Busca Web com validação `curl` HTTP 200, leitura integral da documentação oficial e detecção de depreciação; (3) Engenharia Reversa Funcional em 3 Camadas; (4) Emissão da Tabela de Proveniência P03 e Matriz de Migração Funcional em `research.md` ou `research-brainstorming.md`.

### 12. `session-end.toml`
* **Perspectiva do LLM:** Conduz a reflexão cognitiva de fim de ciclo em 4 fases, registrando aprendizados e decisões no Repositório de Contexto Compartilhado (`.vitalia/memory/session/`).
* **Recursos:** `read_file`, `write_file`, `run_command`.
* **Contrato de Aceite (ACs):**
  * `AC-END-01`: Delegação estrita ao motor CLI `python3 {{VITALIA_KIT_DIR}}/src/vitalia_context_engine.py --action end`, gravando em `.vitalia/memory/session/data/learnings.jsonl`, `decisions.jsonl` e `session_history.jsonl` com o `machine_id` da estação local e timestamps no formato canônico **`DD-MM-YYYY HH:MM:SS(GMT-04:00)`**.
  * `AC-END-02`: Atualização do shard `.vitalia/memory/session/shards/<machine_id>.yaml`, tolerante a offline e sem push bloqueante (o `.toml` não contém lógica inline de geração de arquivos de sessão fora do motor CLI).

### 13. `session-consolidate.toml`
* **Perspectiva do LLM:** Sincroniza o repositório `.vitalia/memory/session/`, recompila o dashboard visual `README.md` da sessão e consolida os shards multi-máquina.
* **Recursos:** `read_file`, `run_command`.
* **Contrato de Aceite (ACs):**
  * `AC-CONS-01`: Invocação de `python3 {{VITALIA_KIT_DIR}}/src/vitalia_context_engine.py --action consolidate`, carregando as coleções `.vitalia/memory/session/shards/<machine_id>.yaml` de cada máquina ordenadas por data (`last_sync` no formato `DD-MM-YYYY HH:MM:SS(GMT-04:00)` decrescente).
  * `AC-CONS-02`: Compilação do `README.md` rico via `dashboard_template.md` e das visões puras (`SESSION_STATE.md`, `LEARNINGS.md`, `DECISIONS.md`, `SESSION_HISTORY.md`) operando no filesystem local, seguida de curadoria HITL de domínios de grounding e `git commit` + `git push` no repositório exclusivo de contexto `.vitalia/memory/session/`.

### 14. `brainstorming.toml` (`guardian_profile = "gp_brainstorming"`)
* **Perspectiva do LLM:** Facilitador Socrático Amplo e Supervisor de Concepção (Design Thinking, Grounding e Engenharia Reversa).
* **Recursos:** `read_file` (opcionais: `write_file`, `search_web`, `run_command`, `define_subagent`, `invoke_subagent`).
* **Lente de Fase (`gp_brainstorming.yaml`):** `no_prune: true`, `total_budget: 0`.
* **Contrato de Aceite (ACs):**
  * `AC-BRAIN-01`: Protocolo estrito STOP → SCAN → ASK → WAIT (`preferred = "filesystem"`, `version = "0.0.1"`).
  * `AC-BRAIN-02`: No passo SCAN, executar: (a) Grounding de Fronteira via Sub-Agente com rede (`vitalia-researcher` / `self`) validando HTTP 200 e leitura integral; (b) Engenharia Reversa Funcional em 3 Camadas para refatorações; (c) Classificação na Matriz de Testes P05 (`[TEST:NEW]`, `[TEST:BUGFIX]`, `[TEST:REFACTOR]`, `[TEST:MIGRATION]`).
  * `AC-BRAIN-03`: Formulação de **no máximo 3 perguntas por turno** com tabelas de trade-off, priorizando decisões P0 (saúde, privacidade e segurança) e ativando o Scope Guardian (`🚧 Ideias Estacionadas (Out of Scope)`).
  * `AC-BRAIN-04`: **Loop Contínuo com Soberania HITL:** Novos turnos de até 3 perguntas são iniciados enquanto restarem dúvidas, encerrando-se **apenas** quando o usuário concordar explicitamente que a concepção está madura.
  * `AC-BRAIN-05`: Emissão de `specs/<feature>/research-brainstorming.md` e disparo do handoff `forward` via `python3 {{VITALIA_KIT_DIR}}/src/vitalia_handoff.py create` exclusivamente para o `/vitalia-route`.

### 15. `route.toml` (`guardian_profile = "gp_vitalia-route"`)
* **Perspectiva do LLM:** Despachante Oficial do Kit. Lê o handoff do Brainstorming (ou input direto) e direciona o fluxo para o especialista adequado do catálogo.
* **Recursos:** `read_file` (opcionais: `write_file`, `run_command`).
* **Lente de Fase (`gp_vitalia-route.yaml`):** `no_prune: true`, `total_budget: 0`.
* **Contrato de Aceite (ACs):**
  * `AC-ROUTE-01`: Consulta determinística da tríade de arquivos: `brainstorming_intents.yaml` (domínio) → `agents_catalog.yaml` (`target_agent`) → `routing_table.yaml` (`command`), com `preferred = "filesystem"` e `hardware_profile = "frontier"`.
  * `AC-ROUTE-02`: Encaminhamento obrigatório de demandas de arquitetura/engenharia de software para o **/vitalia-architect** (Arquiteto de Soluções e Tech Lead Sênior).
  * `AC-ROUTE-03`: Confirmação transparente com o usuário antes de efetivar o handoff via `python3 {{VITALIA_KIT_DIR}}/src/vitalia_handoff.py`.

### 16. `architect.toml` (`guardian_profile = "gp_architect"`) — O Condutor Oficial do Ciclo SDD (Vitalia Architect)
* **Perspectiva do LLM:** Você é o **Vitalia Architect**, Arquiteto de Soluções e Tech Lead Sênior. Você é o condutor oficial e soberano do ciclo SDD. Você recebe o bastão do `/vitalia-route` (trazendo o `research-brainstorming.md`), avalia trade-offs de engenharia, isolamento das 4 camadas, conformidade constitucional e impacto arquitetural, cria ADRs quando necessário e aciona formalmente o início do pipeline SDD disparando o `/vitalia-spec-specify`.
* **Recursos:** `read_file`, `write_file` (opcionais: `run_command`, `search_web`, `define_subagent`, `invoke_subagent`).
* **Lente de Fase (`gp_architect.yaml`):** `no_prune: true`, `total_budget: 0`.
* **Contrato de Aceite (ACs):**
  * `AC-ARCH-01`: Ingestão do artefato `specs/<feature>/research-brainstorming.md` encaminhado pelo `route`.
  * `AC-ARCH-02`: **Ingestão e Validação Holística da Constituição Sem Poda:** O agente inspeciona integralmente o bloco `guardian_json.sources.constitution.principles` (P00–P17 sem poda) injetado pelo Guardian, validando a solução proposta contra todos os princípios ativos.
  * `AC-ARCH-03`: Registro de Architecture Decision Record (`docs/adr/ADR-xxxx-<slug>.md`) sempre que houver escolha técnica estruturante, e enquadramento da demanda na **Engenharia Reversa em 3 Camadas** e na **Matriz Bidimensional de 4 Modalidades de Testes (P05)**.
  * `AC-ARCH-04`: Handoff formal e determinístico para `/vitalia-spec-specify`, assegurando o pipeline completo (`brainstorming → route → architect → specify → clarify → plan → tasks → task-verifier → analyze → implement → converge → judge → session-end`).
  * `AC-ARCH-05`: **Contrato TOML Canônico (`source/extensions/architect.toml`):**
  ```toml
  # architect.toml | Vitalia SPEC Kit v0.0.1 | 07-11-2026 20:25(GMT-04:00)
  # Contrato Formal de Extensão Canônica — Vitalia Architect (AC-ARCH-01/05)
  description = "Arquiteto de Soluções e Tech Lead Sênior (Vitalia Architect). Avalia arquitetura contra a Constituição integral, cria ADRs e conduz o ciclo SDD."

  [meta]
  title = "Vitalia Architect — Regência Técnica e Arquitetura de Soluções"
  version = "0.0.1"
  hardware_profile = "frontier"
  model_profile = "pro"
  timeout_seconds = 180
  domain_gates = []

  [hooks]
  before = ["python3 {{VITALIA_KIT_DIR}}/src/hooks/scan_environment.py"]
  after = []

  [tools]
  required = ["read_file", "write_file"]
  optional = ["run_command", "search_web", "define_subagent", "invoke_subagent"]

  [context]
  reads = [
    "session_context",
    "env_summary",
    "context_learnings",
    "context_decisions",
    "routing_table",
    "agents_catalog",
    "specs/*/research-brainstorming.md"
  ]
  writes = ["active_spec", "adr"]

  [transport]
  preferred = "filesystem"
  file_path = ".vitalia/pipeline.json"
  session_shard = true

  [guardian]
  yaml_sources = ["constitution", "routing_table", "agents_catalog", "grounding_domains"]
  guardian_profile = "gp_architect"
  inject_constitution = true
  inject_session_context = true

  prompt = """
  # 🏛️ PAPEL E REGÊNCIA DO VITALIA ARCHITECT (TECH LEAD SÊNIOR)

  Você atua como o **Vitalia Architect**, o Arquiteto de Soluções e Tech Lead Sênior responsável pela regência técnica do Vitalia SPEC Kit.
  Você é o guardião da integridade arquitetural e o condutor da transição entre a ideação (`research-brainstorming.md`) e a execução formal no pipeline SDD.

  ---

  ## SUA LEI FUNDAMENTAL: A CONSTITUIÇÃO INJETADA SEM PODA
  Leia integralmente as regras constitucionais presentes em `guardian_json.sources.constitution.principles` (P00 a P17). A Constituição é viva, dinâmica e soberana.

  ## MISSÃO E RESPONSABILIDADES:
  1. **Auditoria de Conformidade Constitucional (P00–P17):** Confronte a solução concebida no Brainstorming contra cada princípio da Constituição ativa sem concessões.
  2. **Ciclo SDD Estrito (P01):** Assegurar o pipeline contínuo:
     `brainstorming → route → architect → specify → clarify → plan → tasks → task-verifier → analyze → implement → converge → judge → session-end`.
  3. **Desacoplamento, Topologia e Autonomia (P11):**
     - No repositório raiz, todo o código do kit reside em `source/`. Quando instalado na máquina via `kit-bootstrap.sh`, o código vai para a pasta escolhida pelo usuário (padrão `~/.vitalia-spec-kit`).
     - Erradicar dependências de containers locais (Redis/Postgres), podas artificiais de VRAM e caminhos hardcoded.
  4. **Engenharia Reversa em 3 Camadas e Enquadramento de Testes (P05):**
     - Para refatorações ou migrações, exigir decomposição em: (1) Intenção de negócio pura a preservar; (2) Infraestrutura legada a purgar; (3) Contrato estrito do consumidor.
     - Enquadrar a estratégia de testes na **Matriz Bidimensional de 4 Modalidades (P05)**: `[TEST:NEW]`, `[TEST:BUGFIX]`, `[TEST:REFACTOR]` e `[TEST:MIGRATION]`, exigindo Cenário de Falsificação comportamental e vetando testes sintáticos (`assert path.is_file()`).
  5. **Decisão Arquitetural (ADR) e Comunicação Estruturada:**
     - Se a demanda exigir escolha técnica estruturante, registre o ADR em `docs/adr/ADR-xxxx-<slug>.md`.
     - Produza planos de turno claros com diagramas Mermaid nativos (labels entre aspas duplas — P16), declarando premissas, riscos e escopo (In-Scope / Out-of-Scope).
     - Pause para consentimento humano soberano (HITL) e conduza o handoff para `/vitalia-spec-specify`.
  """
  ```

---

## 5. Código Canônico de `source/profiles/brainstorming_intents.yaml` (7 Domínios Estruturados — C8)

O arquivo `brainstorming_intents.yaml` contém exclusivamente dados estruturados (sem metadados comportamentais ou comandos hardcoded, que pertencem a `agents_catalog.yaml` e `routing_table.yaml`), abrangendo os **7 domínios semânticos** escritos em `source/profiles/brainstorming_intents.yaml`:

```yaml
# brainstorming_intents.yaml | Vitalia SPEC Kit v0.0.1 | 06-11-2026 22:55:00(GMT-04:00)
# Catálogo de Intenções Semânticas para Roteamento e Classificação
# Apenas dados estruturados — zero metadados comportamentais ou prompts

schema_type: "brainstorming_intents"
schema_version: "0.0.1"

domains:
  estrategia_produto:
    description: "Concepção de novas funcionalidades, alinhamento de roadmap, proposta de valor, trade-offs e definição de MVP."
    threshold: 0.80
    p0_flag: false
    utterances:
      - "Quero criar uma nova funcionalidade no sistema"
      - "Preciso estruturar a ideia de um novo módulo"
      - "Como podemos desenhar a proposta deste produto?"
      - "Quero fazer um brainstorming de produto"
      - "Preciso decidir os trade-offs desta feature"
      - "Como posso melhorar a experiência do usuário nesta tela?"
      - "Quero propor um novo fluxo de onboarding"
      - "Qual o escopo mínimo viável (MVP) para esta entrega?"

  arquitetura_software:
    description: "Decisões arquiteturais estruturantes, componentes atômicos orientados a SRP, contratos tipados, isolamento de dependências e Clean Architecture."
    threshold: 0.82
    p0_flag: false
    utterances:
      - "Quero definir a arquitetura de software para este subsistema"
      - "Preciso desenhar o diagrama de componentes e fluxos técnicos"
      - "Como organizar o código em componentes desacoplados com responsabilidade única (SRP)?"
      - "Quero planejar o turno com a regência do Architect"
      - "Quero documentar um Architecture Decision Record (ADR)"
      - "Preciso isolar as dependências e criar um runtime idempotente"
      - "Como desacoplar a infraestrutura externa e operar em filesystem puro?"

  engenharia_web_frontend:
    description: "Desenvolvimento de interfaces web modernas, Vanilla CSS rico, tokens de design, layouts responsivos, Vite e SEO."
    threshold: 0.80
    p0_flag: false
    utterances:
      - "Quero criar uma interface web moderna e atraente"
      - "Como implementar um design system usando Vanilla CSS e glassmorphism?"
      - "Preciso estruturar o frontend usando Vite e componentes desacoplados"
      - "Como melhorar o SEO e a hierarquia semântica HTML desta página?"
      - "Desenvolva uma tela rica com micro-animações e tipografia profissional"

  infraestrutura_devops:
    description: "Automação de ambientes, shims de sistema, pipelines de CI/CD, isolamento de virtualenv e esteiras de instalação."
    threshold: 0.82
    p0_flag: false
    utterances:
      - "Como automatizar o bootstrap do ambiente em máquinas novas?"
      - "Preciso configurar pipelines de CI/CD para executar os testes"
      - "Como garantir que o virtualenv seja 100% isolado do sistema host?"
      - "Quero estruturar a esteira de instalação átomo a átomo"
      - "Como gerar shims parametrizados no diretório ~/.local/bin?"

  seguranca_compliance:
    description: "Análise de vulnerabilidades, conformidade LGPD/HIPAA/GDPR, sanitização de inputs, Zero Data Leak e auditoria de código."
    threshold: 0.85
    p0_flag: true
    utterances:
      - "Preciso auditar o sistema contra vulnerabilidades do OWASP Top 10"
      - "Como garantir que os logs não vazem PII ou credenciais?"
      - "Quero avaliar a conformidade com as regras da LGPD e HIPAA"
      - "Como sanitizar os inputs e blindar os contratos de API?"
      - "Preciso implementar isolamento de dados com Zero Data Leak"

  educacao_workshop:
    description: "Criação de material didático, estruturação de cursos, workshops técnicos, apresentações acadêmicas ou roteiros de treinamento."
    threshold: 0.78
    p0_flag: false
    utterances:
      - "Crie uma apresentação técnica sobre a arquitetura do kit"
      - "Preciso de um roteiro detalhado para um workshop prático"
      - "Como estruturar uma aula sobre componentes desacoplados e SRP?"
      - "Desenvolva um plano de ensino para capacitação em SDD"
      - "Monte um tutorial passo a passo para novos desenvolvedores"

  saude_clinica:
    description: "Sistemas médicos, prontuários, biomarcadores clínicos, suporte à decisão diagnóstica e salvaguarda HITL obrigatória (P09)."
    threshold: 0.85
    p0_flag: true
    utterances:
      - "Desenvolver módulo de prontuário eletrônico ou dados clínicos"
      - "Integração com exames laboratoriais ou biomarcadores"
      - "Algoritmo de triagem ou suporte à decisão médica"
      - "Sistema para uso por profissionais de saúde em clínica"
      - "Tratamento de dados sensíveis de pacientes (LGPD Art. 11/HIPAA)"
      - "Quero adicionar recomendação de dosagem ou frequência terapêutica"
```

---

## 6. Ordem de Instruções no `source/AGENTS.md` (Anti-Lost in the Middle)

A estrutura do `source/AGENTS.md` obedece aos princípios de **Primacy Effect** e **Recency Effect**:

```markdown
<!-- AGENTS.md | Vitalia SPEC Kit v0.0.1 | 07-11-2026 20:25:00(GMT-04:00) -->

# AGENTS.md — Vitalia SPEC Kit (v0.0.1)

> [!IMPORTANT]
> **ARQUITETURA v0.0.1 — Runtime Autônomo, Lentes de Fase Sem Poda e Context Lifting Seguro.**
> O Vitalia SPEC Kit opera desacoplado de containers locais na IDE Antigravity com modelos de fronteira. O Guardian (§11) injeta 100% da Constituição sem truncamento de tokens, combinada com as Lentes Adversariais de Fase (`gp_*.yaml`). O contexto de execução reside em `.vitalia/memory/data_storage/context/`.

## 1. Topologia do Kit e Isolamento das 4 Camadas

1. **Repositório Raiz (`source/`):** No repositório de desenvolvimento do kit, todo o código-fonte mora dentro da pasta `source/` (`bin/`, `extensions/`, `profiles/`, `src/`, `integrations/`, `templates/`).
2. **Kit Instalado na Máquina (`~/.vitalia-spec-kit` por padrão):** Quando instalado via `kit-bootstrap.sh`, o código de `source/` vai para a pasta escolhida pelo usuário (sendo `~/.vitalia-spec-kit` a opção padrão em `VITALIA_KIT_DIR`), contendo `.venv/` isolado.
3. **Repositório de Contexto (`.vitalia/memory/session/`):** Repositório Git separado exclusivo para contexto de sessão (shards por máquina em `shards/<machine_id>.yaml`, `decisions.jsonl`, `learnings.jsonl`, `session_history.jsonl` e dashboard `README.md`). Nunca é commitado no repositório raiz.
4. **Repositório de Dados (`.vitalia/memory/data_storage/`):** Repositório Git separado exclusivo para dados operacionais do projeto (`context/`, `handoffs/`, `research/`). Nunca é commitado no repositório raiz.

## 2. Como o Kit Funciona (Context Lifting)

1. Usuário aciona um comando `/vitalia-*` na IDE.
2. A SKILL correspondente invoca `{{VITALIA_KIT_DIR}}/.venv/bin/python3 .../vitalia_hook_runner.py`.
3. O hook runner processa o TOML, aciona o Guardian (sem poda, `no_prune: true`) e grava o contexto completo em `.vitalia/memory/data_storage/context/<machine_id>/`.
4. A IA consome o arquivo referenciado no Stub JSON via `view_file` / `read_file`.

## 3. Diretrizes Comportamentais Invioláveis (Zona de Máxima Atenção)

1. **Início e Fim de Sessão via CLI:** Toda sessão inicia com `/vitalia-session-start` e encerra com `/vitalia-session-end` + `/vitalia-session-consolidate`, processados por `vitalia_context_engine.py` com datas em `DD-MM-YYYY HH:MM:SS(GMT-04:00)`.
2. **Brainstorming, Pesquisa Profunda e Regência do Architect:** Requisitos novos ou refatorações passam por `/vitalia-brainstorming` (com Engenharia Reversa em 3 Camadas e Grounding via Sub-Agente `/vitalia-research` com validação HTTP 200 e leitura integral), são despachados por `/vitalia-route` ao `/vitalia-architect` e ingressam em `/vitalia-spec-specify`.
3. **Numeração Sequencial de Specs:** Toda feature segue estritamente o padrão `specs/<NNN>-<slug>/` com 3 dígitos zero-padded.
4. **Matriz Bidimensional de Testes (P05 — 4 Modalidades + Cenário de Falsificação):**
   * `[TEST:NEW]` (Código Novo): TDD Red-Green-Refactor executável de caixa-preta (falha comprovada antes do código).
   * `[TEST:BUGFIX]` (Correção de Bug): Bug Reproduction First — o teste DEVE falhar contra o código atual antes do fix.
   * `[TEST:REFACTOR]` (Refatoração Pura): Behavioral Characterization garantindo zero regressão nas entradas/saídas.
   * `[TEST:MIGRATION]` (Adaptação Arquitetural): Contract Preservation + Clean Purge — comprova preservação dos comportamentos estáveis antigos, remoção do legado e pureza do contrato do consumidor.
   * **Veto BLOCKING:** Proibidas asserções puramente sintáticas (`assert path.is_file()`) para atestar entrega.
5. **Auditoria Adversarial Nativa na IDE (`/vitalia-judge`, `/vitalia-task-verifier`, `/vitalia-analyze`):** Nunca execute o CLI legado `sdd_judge.py`. Toda auditoria opera nativamente na IDE com FMEA Mental (4 questões), *Inverted CoT*, emissão física do `Relatório de Inspeção` e pausa obrigatória para aprovação humana (HITL soberano).
```

---

# PARTE II — Análise Forense de Componentes, Catálogos Temporais e Dashboard Rico (Harmonizados da Spec 003)

## 1. Sumário Executivo e Propósito da Remediação Arquitetural

Esta Parte II detalha a análise componente a componente para que o instalador global (`kit-bootstrap.sh`), o ativador de projetos (`install-project`) e o motor de sessão (`vitalia_context_engine.py`) operem de forma 100% autônoma, modular (SRP), auditável e fiel em qualquer máquina limpa (*fresh install*, incluindo WSL2/zsh).

Fica estabelecido o princípio fundamental: **eliminar tanto o espelhamento em lote cego quanto a reescrita cega que amputa funcionalidades**. Cada componente de `source/` é instalado na pasta escolhida pelo usuário (padrão `~/.vitalia-spec-kit`), purgado de débitos legados (Redis, Celery, Ollama, VRAM), mas preservando integralmente suas intenções de negócio estáveis (como clonagem de repositórios remotos de sessão/dados e renderização visual rica do dashboard).

---

## 2. Root Cause Analysis (RCA) Consolidada (Spec 002 + Spec 003)

### RCA-01: Falso Positivo em Testes Unitários e E2E Rasos (Shallow Testing)
- **O que ocorreu:** Os testes `test_bootstrap.py`, `test_bootstrap_idempotency.py` e `test_bootstrap_e2e_project.py` limitaram-se a checar criação de pastas e presença sintática de arquivos (`assert path.is_file()`) ou strings estáticas, passando com 100% de sucesso enquanto o produto falhava na execução real em máquina limpa.
- **Regra Corretiva (P05):** Todo teste deve ser comportamental de caixa-preta (`[TEST:NEW]`, `[TEST:BUGFIX]`, `[TEST:REFACTOR]`, `[TEST:MIGRATION]`) com **Cenário de Falsificação** explícito: executar os comandos reais via `subprocess` em sandbox (`tmp_path`), verificar códigos de retorno, testar clonagem Git real contra um repositório bare local de teste, validar a execução de `vitalia_hook_runner.py` usando o `.venv/bin/python3` instalado e inspecionar a pureza estrita do diretório `.agents/skills/`.

### RCA-02: Omissão de Templates e Extensões no `kit-bootstrap.sh` e Ausência de Pré-Validação de `venv`
- **O que ocorreu:** O `kit-bootstrap.sh` omitiu `integrations/antigravity/SKILL.md.template`, `templates/dashboard_template.md`, `AGENTS.md` e extensões como `analyze.toml` e `research.toml`, além de não validar previamente se o pacote de sistema `python3-venv` estava funcional na máquina host antes de criar o ambiente virtual.
- **Regra Corretiva:** O `kit-bootstrap.sh` deve: (1) permitir ao usuário escolher a pasta de instalação (com `~/.vitalia-spec-kit` como padrão); (2) pré-validar `python3 -m venv` com diagnóstico claro se ausente; (3) instalar explicitamente cada subdiretório e arquivo de `source/` para o destino escolhido e verificar o binário `${VITALIA_DIR}/.venv/bin/python3` ao final.

### RCA-03: Convivência Suja com Resquícios de Caminhos Hardcoded (`~/.vitalia`)
- **O que ocorreu:** Arquivos em `source/` mantinham referências duras a `~/.vitalia` em vez de resolver dinamicamente o diretório escolhido pelo usuário.
- **Regra Corretiva:** Todo o código em `source/` resolve o diretório do kit instalado via `VITALIA_KIT_DIR` (variável de ambiente $\rightarrow$ `.env` do projeto $\rightarrow$ fallback padrão `~/.vitalia-spec-kit`).

### RCA-04: Violação do Contrato do Consumidor em `.agents/skills/` e Reescrita Cega de `git_storage_manager.sh`
- **O que ocorreu:**
  1. O `agent_preserver.sh` criou `.agents/skills/.trash/` e `.agents/skills/.vitalia-manifest` dentro de `.agents/skills/`, violando o contrato do indexador da Antigravity IDE (que exige que `.agents/skills/` contenha exclusivamente diretórios de skills com `SKILL.md`) e impedindo que os slash commands aparecessem.
  2. O `git_storage_manager.sh` substituiu a lógica funcional de verificação e clonagem remota (`git clone --depth=1`) do `install-project` original por um `git init` vazio com `.gitkeep`.
- **Regra Corretiva:**
  1. Manter `.agents/skills/` 100% puro (qualquer manifesto ou backup deve residir em `.vitalia/` ou `.vitalia/memory/data_storage/`).
  2. Preservar em `git_storage_manager.sh` a lógica completa de verificação (`git ls-remote`) e clonagem (`git clone --depth=1`) quando o usuário informar URLs remotas válidas para os repositórios de `session` e `data_storage`, usando `git init` local apenas como fallback quando o repositório remoto estiver vazio ou inacessível.

### RCA-05: Regressão no Motor de Sessão (`vitalia_context_engine.py`) e no Dashboard (`README.md`)
- **O que ocorreu:** Na Spec 002, `source/src/vitalia_context_engine.py` foi reescrito de forma minimalista: usava `datetime.now().isoformat()` em vez do formato canônico `DD-MM-YYYY HH:MM:SS(GMT-04:00)`, salvava `shards/servido.json` em vez de `shards/<machine_id>.yaml`, ignorava `--learnings-json` e `--decisions-json` no `--action end`, e gerava um `README.md` empobrecido sem `dashboard_template.md`.
- **Regra Corretiva:** Preservar no motor de contexto (`vitalia_context_engine.py` / `core/context_engine.py` / `core/shard_manager.py` / `core/utils.py` / `rendering/dashboard_renderer.py`):
  - Formato de data estritamente canônico **`DD-MM-YYYY HH:MM:SS(GMT-04:00)`** (`get_timestamp()`).
  - Gravação de shards por máquina em `.vitalia/memory/session/shards/<machine_id>.yaml`, ordenados por `last_sync` decrescente (para que a máquina que executou a sessão mais recente fique em destaque em `SESSION_STATE.md` e `README.md`).
  - Persistência integral de `data/learnings.jsonl`, `data/decisions.jsonl` e `data/session_history.jsonl` no `--action end`, seguida de `--action consolidate` renderizando o template rico `source/templates/dashboard_template.md`.

---

## 3. Análise Individual Exaustiva de Componentes em `source/`

### 3.1. Pasta `source/profiles/`

| Componente | Função & Contrato | Ação Canônica Consolidada |
| :--- | :--- | :--- |
| `gp_*.yaml` (`gp_brainstorming`, `gp_spec-specify`, `gp_spec-plan`, `gp_spec-tasks`, `gp_task-verifier`, `gp_spec-implement`, `gp_sdd-judge`, `gp_analyze`, `gp_research`, `gp_session-start`, `gp_vitalia-route`, `gp_architect`) | **Lentes Adversariais de Fase Sem Poda** | **Manter e instalar todos os perfis `gp_*.yaml`.** Configurados com `no_prune: true`, `total_budget: 0`, `hardware_profile: "frontier"`, injetando 100% da Constituição + `phase_guardrails` (e `phase_profiles` no `gp_sdd-judge.yaml`). |
| `constitution.yaml` | Princípios Constitucionais P00–P17 | Manter os 18 princípios (P00–P17) com a **Matriz Bidimensional de 4 Modalidades no P05** (`[TEST:NEW]`, `[TEST:BUGFIX]`, `[TEST:REFACTOR]`, `[TEST:MIGRATION]` + Cenário de Falsificação e veto BLOCKING a testes rasos) e validade temporal no P03. |
| `dynamic_questioning.yaml` | Perguntas socráticas do Brainstorming | 9 domínios canônicos livres de Redis/Celery/Ollama/VRAM, com metadados de governança temporal (`last_reviewed_at`, `valid_until`, `validity_window_days`, `status`, `last_update_source`). |
| `grounding_domains.yaml` | Dicionário de domínios para verificação externa P03 | 10 domínios canônicos cobrindo todo o ecossistema tecnológico, com dimensão temporal (`valid_until`), status (`ACTIVE`/`STALE`) e histórico de fontes (`last_update_source`, `update_history`). |
| `brainstorming_intents.yaml` | Mapeamento de intenções semânticas (C8) | 7 domínios estruturados (`estrategia_produto`, `arquitetura_software`, `engenharia_web_frontend`, `infraestrutura_devops`, `seguranca_compliance`, `educacao_workshop`, `saude_clinica`). |
| `agents_catalog.yaml` | Catálogo de agentes do ecossistema | Inclui `architect` (`vitalia-architect`) e `research` (`vitalia-researcher`) em conformidade OpenAPI 3.0 limpa. |
| `routing_table.yaml` | Roteamento determinístico de comandos | Sincronizado com todos os slash commands nativos da IDE (incluindo `/vitalia-research`, `/vitalia-analyze`, `/vitalia-architect`, `/vitalia-task-verifier`). |

---

### 3.2. Catálogo Canônico Exaustivo: `source/profiles/grounding_domains.yaml`

```yaml
# grounding_domains.yaml | Vitalia SPEC Kit v0.0.1 | 07-11-2026 21:00:00(GMT-04:00)
schema_type: "grounding_domains"
schema_version: "0.0.1"
last_catalog_review: "07-11-2026 21:00:00(GMT-04:00)"
default_validity_days: 90

domains:
  - id: "provedores_de_ia"
    name: "Modelos de Fronteira e Orquestração Híbrida de IA"
    description: "Modelos vigentes de fronteira (Google Gemini 3.x: gemini-3.6-flash, gemini-3.1-pro-preview; Anthropic Claude; OpenAI), janelas de contexto, pricing, políticas de retry (Exponential Backoff) e arquiteturas híbridas em projetos assistidos (triagem local PII/PHI via Ollama/Open WebUI + geração analítica cloud)."
    scope: "global"
    trigger: "blocking"
    last_verified_at: "07-11-2026 21:00:00(GMT-04:00)"
    valid_until: "05-02-2027 21:00:00(GMT-04:00)"
    validity_window_days: 90
    status: "ACTIVE"
    last_update_source:
      url: "https://ai.google.dev/gemini-api/docs/models/gemini"
      source_type: "official_vendor_docs"
      verified_by: "session_20261107_2100"
      machine_id: "42c8e746"
      summary_of_changes: "Alinhamento com a família Gemini 3.x (gemini-3.6-flash e gemini-3.1-pro-preview) e padrões de orquestração híbrida (Cloud + Local Triage) do AI-Control."
    update_history:
      - verified_at: "07-11-2026 21:00:00(GMT-04:00)"
        url: "https://ai.google.dev/gemini-api/docs/models/gemini"
        verified_by: "session_20261107_2100"
        summary: "Calibração para modelos Gemini 3.x e roteamento inteligente Flash/Pro."
    authoritative_sources:
      - "https://ai.google.dev/gemini-api/docs/models/gemini"
      - "https://docs.anthropic.com/en/docs/about-claude/models"
      - "https://platform.openai.com/docs/models"
      - "https://github.com/ollama/ollama/blob/main/docs/api.md"

  - id: "python_packages"
    name: "Ecossistema Python, Django 6.x, ASGI & Data Vault"
    description: "Compatibilidade com Python 3.10–3.14+, Django 6.1+, Daphne (ASGI), Django REST Framework (~3.18), SimpleJWT, drf-spectacular (OpenAPI 3.0), Django Channels 4 + channels-redis, Celery 5.6+, pgvector (0.5+), django-crypto-fields (1.0+), cryptography, Pydantic v2, pytest/pytest-django e pacotes core do kit (pyyaml, jinja2, jsonschema)."
    scope: "global"
    trigger: "blocking"
    last_verified_at: "07-11-2026 21:00:00(GMT-04:00)"
    valid_until: "05-02-2027 21:00:00(GMT-04:00)"
    validity_window_days: 90
    status: "ACTIVE"
    last_update_source:
      url: "https://pypi.org"
      source_type: "package_repository"
      verified_by: "session_20261107_2100"
      machine_id: "42c8e746"
      summary_of_changes: "Incorporação da stack Python 3.14 / Django 6.1 ASGI, DRF, Channels, Celery, pgvector e django-crypto-fields utilizada no AI-Control."
    update_history:
      - verified_at: "07-11-2026 21:00:00(GMT-04:00)"
        url: "https://docs.djangoproject.com/en/stable/releases/"
        verified_by: "session_20261107_2100"
        summary: "Validação da stack full-stack Python/Django 6.1 e dependências do kit isolado."
    authoritative_sources:
      - "https://pypi.org"
      - "https://docs.djangoproject.com"
      - "https://www.django-rest-framework.org"
      - "https://drf-spectacular.readthedocs.io"
      - "https://docs.celeryq.dev"
      - "https://channels.readthedocs.io"

  - id: "web_technologies"
    name: "Frontend Moderno: Next.js 16, React 19, TypeScript 5 & TailwindCSS v4"
    description: "Arquitetura App Router e Turbopack (Next.js 16+), React 19, TypeScript 5 (tsc --noEmit), TailwindCSS v4 (@tailwindcss/postcss, tailwind-merge, clsx), separação Server-State (TanStack React Query v5 com SSR factory e Optimistic Updates) vs Client-State (Redux Toolkit), validação com Zod, Axios com interceptors concorrentes (isRefreshing) e geração de tipos via openapi-typescript v7."
    scope: "global"
    trigger: "blocking"
    last_verified_at: "07-11-2026 21:00:00(GMT-04:00)"
    valid_until: "05-02-2027 21:00:00(GMT-04:00)"
    validity_window_days: 90
    status: "ACTIVE"
    last_update_source:
      url: "https://nextjs.org/docs"
      source_type: "official_framework_docs"
      verified_by: "session_20261107_2100"
      machine_id: "42c8e746"
      summary_of_changes: "Alinhamento com a stack frontend do AI-Control: Next.js 16.4+, React 19.2, TailwindCSS v4, TanStack Query v5, Redux Toolkit 2.11, Zod e openapi-typescript."
    update_history:
      - verified_at: "07-11-2026 21:00:00(GMT-04:00)"
        url: "https://nextjs.org/docs"
        verified_by: "session_20261107_2100"
        summary: "Substituição do perfil genérico por Next.js 16, React 19, TailwindCSS v4 e TanStack Query v5."
    authoritative_sources:
      - "https://nextjs.org/docs"
      - "https://react.dev"
      - "https://tailwindcss.com/docs"
      - "https://tanstack.com/query/latest/docs"
      - "https://redux-toolkit.js.org"
      - "https://openapi-ts.dev"
      - "https://zod.dev"

  - id: "external_apis"
    name: "APIs Externas, Gateways Omnichannel, WebSockets & Contratos OpenAPI 3.0"
    description: "Contratos OpenAPI 3.0 estritos (drf-spectacular <-> openapi-typescript), autenticação JWT Stateless com Refresh Rotation, WebSockets ASGI em tempo real, integração com Gateways Omnichannel/WhatsApp (Evolution API v2, webhooks MESSAGES_UPSERT/CONNECTION_UPDATE, despacho síncrono com falha atômica HTTP 502/400 e descarte O(1) de presence.update)."
    scope: "global"
    trigger: "blocking"
    last_verified_at: "07-11-2026 21:00:00(GMT-04:00)"
    valid_until: "05-02-2027 21:00:00(GMT-04:00)"
    validity_window_days: 90
    status: "ACTIVE"
    last_update_source:
      url: "https://spec.openapis.org/oas/v3.0.3.html"
      source_type: "standards_rfc"
      verified_by: "session_20261107_2100"
      machine_id: "42c8e746"
      summary_of_changes: "Inclusão de padrões de integração com Evolution API v2, WebSockets Django Channels e sincronização tipada OpenAPI 3.0."
    update_history:
      - verified_at: "07-11-2026 21:00:00(GMT-04:00)"
        url: "https://spec.openapis.org/oas/v3.0.3.html"
        verified_by: "session_20261107_2100"
        summary: "Padronização de contratos REST/WebSocket e gateways de mensageria."
    authoritative_sources:
      - "https://spec.openapis.org/oas/latest.html"
      - "https://doc.evolution-api.com"
      - "https://www.rfc-editor.org/rfc/rfc6749"

  - id: "security_practices"
    name: "Segurança de Aplicação, Data Vault Criptográfico & OWASP (API/LLM)"
    description: "Criptografia em repouso de PII/PHI (AES-256 e RSA via django-crypto-fields), Blind Indexing com SHA-256 para busca exata sem decriptação, criptografia simétrica de contexto em memória no Redis, mascaramento de credenciais em logs (SensitiveLogMaskingMiddleware: apikey=[REDACTED]), proteção condicional de schemas em produção (OWASP API9:2023) e diretrizes OWASP Top 10 for LLM Applications 2025."
    scope: "global"
    trigger: "blocking"
    last_verified_at: "07-11-2026 21:00:00(GMT-04:00)"
    valid_until: "06-01-2027 21:00:00(GMT-04:00)"
    validity_window_days: 60
    status: "ACTIVE"
    last_update_source:
      url: "https://owasp.org/www-project-top-10-for-large-language-model-applications"
      source_type: "security_standard"
      verified_by: "session_20261107_2100"
      machine_id: "42c8e746"
      summary_of_changes: "Incorporação dos padrões Data Vault (AES-256/RSA + Blind Indexing SHA-256), mascaramento de logs e OWASP API9:2023 do AI-Control."
    update_history:
      - verified_at: "07-11-2026 21:00:00(GMT-04:00)"
        url: "https://owasp.org/API-Security/"
        verified_by: "session_20261107_2100"
        summary: "Alinhamento com OWASP API Security 2023 e OWASP LLM Top 10 2025."
    authoritative_sources:
      - "https://owasp.org/www-project-top-10-for-large-language-model-applications"
      - "https://owasp.org/API-Security/"
      - "https://nvd.nist.gov"
      - "https://cve.mitre.org"

  - id: "regulations_compliance"
    name: "Regulações, Privacidade, Trilha Imutável e Compliance Legal"
    description: "LGPD (Lei 13.709/2018 — Art. 11, 14 e 18), GDPR, HIPAA Security Rule e EU AI Act (Art. 12 e 14). Governança de consentimento explícito (ConsentLog), concessão temporária de acesso (DataAccessGrant) e trilha de auditoria imutável (AuditLog)."
    scope: "global"
    trigger: "blocking"
    constitution_principles: ["P07", "P09", "P17"]
    last_verified_at: "07-11-2026 21:00:00(GMT-04:00)"
    valid_until: "07-11-2027 21:00:00(GMT-04:00)"
    validity_window_days: 365
    status: "ACTIVE"
    last_update_source:
      url: "https://www.planalto.gov.br/ccivil_03/_ato2015-2018/2018/lei/l13709.htm"
      source_type: "legislation"
      verified_by: "session_20261107_2100"
      machine_id: "42c8e746"
      summary_of_changes: "Integração dos modelos de conformidade ConsentLog, DataAccessGrant e AuditLog imutável."
    update_history:
      - verified_at: "07-11-2026 21:00:00(GMT-04:00)"
        url: "https://www.planalto.gov.br/ccivil_03/_ato2015-2018/2018/lei/l13709.htm"
        verified_by: "session_20261107_2100"
        summary: "Calibração dos princípios P07, P09 e P17 com mecanismos auditáveis de consentimento."
    authoritative_sources:
      - "https://www.planalto.gov.br/ccivil_03/_ato2015-2018/2018/lei/l13709.htm"
      - "https://eur-lex.europa.eu/eli/reg/2016/679/oj"
      - "https://www.hhs.gov/hipaa"
      - "https://artificialintelligenceact.eu"

  - id: "cloud_services"
    name: "Infraestrutura Conteinerizada de Aplicação, Banco Vetorial & Cloud"
    description: "Orquestração Docker Compose para projetos full-stack (PostgreSQL 16 com pgvector, Redis particionado por DB lógico, Evolution API, Ollama com GPU pass-through), além de precificação, quotas e SLAs em provedores Cloud (GCP, AWS, Cloudflare)."
    scope: "global"
    trigger: "warning"
    last_verified_at: "07-11-2026 21:00:00(GMT-04:00)"
    valid_until: "05-02-2027 21:00:00(GMT-04:00)"
    validity_window_days: 90
    status: "ACTIVE"
    last_update_source:
      url: "https://github.com/pgvector/pgvector"
      source_type: "infrastructure_docs"
      verified_by: "session_20261107_2100"
      machine_id: "42c8e746"
      summary_of_changes: "Distinção clara: o Kit SDD opera em filesystem puro, enquanto aplicações assistidas (ex: AI-Control) utilizam PostgreSQL 16 + pgvector e Redis particionado via Docker Compose."
    update_history:
      - verified_at: "07-11-2026 21:00:00(GMT-04:00)"
        url: "https://cloud.google.com/pricing"
        verified_by: "session_20261107_2100"
        summary: "Inclusão da topologia de containers de aplicação (pgvector:pg16, redis:alpine) e cloud pricing."
    authoritative_sources:
      - "https://github.com/pgvector/pgvector"
      - "https://docs.docker.com/compose/"
      - "https://cloud.google.com/pricing"
      - "https://aws.amazon.com/pricing"

  - id: "scientific_claims"
    name: "Afirmações Científicas e Revisão de Literatura"
    description: "Citações científicas, evidências empíricas, dosagens, biomarcadores e revisões sistemáticas (PRISMA)."
    scope: "global"
    trigger: "blocking"
    constitution_principles: ["P03", "P09"]
    last_verified_at: "07-11-2026 21:00:00(GMT-04:00)"
    valid_until: "06-05-2027 21:00:00(GMT-04:00)"
    validity_window_days: 180
    status: "ACTIVE"
    last_update_source:
      url: "https://pubmed.ncbi.nlm.nih.gov"
      source_type: "scientific_indexer"
      verified_by: "session_20261107_2100"
      machine_id: "42c8e746"
      summary_of_changes: "Rigor estrito em citações com links verificáveis (HTTP 200) e identificador DOI."
    update_history:
      - verified_at: "07-11-2026 21:00:00(GMT-04:00)"
        url: "https://pubmed.ncbi.nlm.nih.gov"
        verified_by: "session_20261107_2100"
        summary: "Blindagem de alegações médicas sob o princípio P03."
    authoritative_sources:
      - "https://pubmed.ncbi.nlm.nih.gov"
      - "https://europepmc.org"
      - "https://www.cochranelibrary.com"

  - id: "clinical_safety"
    name: "Segurança Clínica, Safety Gate e Transbordo HITL"
    description: "Detecção determinística de risco clínico (CLINICAL_KEYWORDS / Safety Gate), interrupção imediata de fluxo autônomo de IA com transbordo humano (ACTIVE_HUMAN), ranges de biomarcadores e salvaguarda HITL tripla (DRAFT -> REVIEW -> ACTIVE)."
    scope: "global"
    trigger: "blocking"
    constitution_principles: ["P00", "P09", "P14"]
    last_verified_at: "07-11-2026 21:00:00(GMT-04:00)"
    valid_until: "07-11-2027 21:00:00(GMT-04:00)"
    validity_window_days: 365
    status: "ACTIVE"
    last_update_source:
      url: "https://www.who.int/publications"
      source_type: "global_health_authority"
      verified_by: "session_20261107_2100"
      machine_id: "42c8e746"
      summary_of_changes: "Inclusão do padrão Safety Gate e transbordo imediato para operador/profissional humano em conversas omnichannel."
    update_history:
      - verified_at: "07-11-2026 21:00:00(GMT-04:00)"
        url: "https://www.who.int/publications"
        verified_by: "session_20261107_2100"
        summary: "Inviolabilidade do princípio P00 e P09 com Safety Gate em tempo real."
    authoritative_sources:
      - "https://www.who.int"
      - "https://www.ms.gov.br"

  - id: "software_architecture"
    name: "Engenharia de Software, RBAC 4-Tier, RAG Scoped & SRP"
    description: "Componentes atômicos orientados a SRP (Single Responsibility Principle), Service Layer com falha atômica, governança multi-tenant RBAC Hierárquico 4-Tier (Platform -> Org -> Dept -> Project -> Squad) com Containment Rule e permissões por slug, RAG Scoped (pré-filtragem relacional antes da busca vetorial HNSW), prevenção estrita de queries N+1 (batching O(1)) e Matriz de Testes Comportamentais P05."
    scope: "global"
    trigger: "warning"
    last_verified_at: "07-11-2026 21:00:00(GMT-04:00)"
    valid_until: "07-11-2027 21:00:00(GMT-04:00)"
    validity_window_days: 365
    status: "ACTIVE"
    last_update_source:
      url: "https://blog.cleancoder.com/uncle-bob/2014/05/08/SingleReponsibilityPrinciple.html"
      source_type: "engineering_authority"
      verified_by: "session_20261107_2100"
      machine_id: "42c8e746"
      summary_of_changes: "Consolidação dos padrões arquiteturais do AI-Control (RBAC 4-Tier, RAG Scoped, Service Layer atômica, otimização anti-N+1 O(1) e SRP)."
    update_history:
      - verified_at: "07-11-2026 21:00:00(GMT-04:00)"
        url: "https://martinfowler.com/eaaCatalog/serviceLayer.html"
        verified_by: "session_20261107_2100"
        summary: "Integração de Service Layer, RBAC 4-Tier e RAG Scoped aos padrões de arquitetura do kit."
    authoritative_sources:
      - "https://blog.cleancoder.com"
      - "https://martinfowler.com"
      - "https://refactoring.guru"
```

---

### 3.3. Registry Canônico Exaustivo: `source/profiles/dynamic_questioning.yaml`

O arquivo `dynamic_questioning.yaml` define o questionamento socrático durante o `/vitalia-brainstorming` em **9 domínios canônicos** livres de condicionais de Redis/Celery/Ollama/VRAM, todos com metadados temporais (`last_reviewed_at`, `valid_until`, `validity_window_days`, `status` e `last_update_source`):

1. **`protecao_de_dados_e_privacidade` (P0 CRITICAL):** Soberania/Localização de Dados e Modelo de Consentimento.
2. **`autenticacao_e_identidade` (P0 CRITICAL):** Gestão de Identidade e Controle de Acesso (RBAC vs ABAC multi-tenant).
3. **`seguranca_de_api_e_comunicacao` (P0 CRITICAL):** Autenticação de Endpoints e Sanitização/Validação estrita em fronteira.
4. **`arquitetura_de_aplicacao` (HIGH-LEVERAGE):** Modularidade e Coesão (SRP e contratos estritos) e Comunicação Inter-Componentes.
5. **`comunicacao_e_sincronizacao` (HIGH-LEVERAGE):** Atualização de Estado e Gestão de Estado Distribuído (Filesystem com semáforo e Hub Git).
6. **`processamento_assincrono` (HIGH-LEVERAGE):** Desacoplamento de Tarefas Longas.
7. **`arquitetura_de_dados_e_persistencia` (HIGH-LEVERAGE):** Modelo de Consistência e Migrações Versionadas.
8. **`machine_learning_e_ia_de_fronteira` (HIGH-LEVERAGE):** Privacidade de Inferência e Orquestração de Contexto via Guardian sem podas.
9. **`observabilidade_e_auditoria` (MEDIUM):** Structured Logging sanitizado e Ledger de Sessão em `.jsonl`.

---

### 3.4. Pasta `source/extensions/`

| Componente | Ação Consolidada |
| :--- | :--- |
| **Todas as 16 extensões `.toml`** (`session-start`, `spec-specify`, `judge`, `spec-plan`, `spec-tasks`, `task-verifier`, `spec-implement`, `analyze`, `converge`, `clarify`, `research`, `session-end`, `session-consolidate`, `brainstorming`, `route`, `architect`) | - Definir `version = "0.0.1"`, `hardware_profile = "frontier"` e `preferred = "filesystem"` no bloco `[transport]` (purgando `redis` e `redis_key`). <br/>- Substituir qualquer caminho hardcoded `~/.vitalia` por `{{VITALIA_KIT_DIR}}`. <br/>- Vincular `guardian_profile = "gp_<workflow>"` apontando para as Lentes de Fase Sem Poda (`no_prune: true`, `total_budget: 0`). <br/>- Incorporar a Matriz Bidimensional de 4 Modalidades P05 (`[TEST:NEW]`, `[TEST:BUGFIX]`, `[TEST:REFACTOR]`, `[TEST:MIGRATION]`) + Cenário de Falsificação, TDA, Matriz de Migração em 3 Camadas e FMEA Mental (conforme detalhado na Seção 4 da PARTE I). |
| **`analyze.toml` (Dual Mode: `after_tasks` + `after_implement`)** | - Modo `after_tasks`: 7 passes adversariais de consistência entre `spec.md`, `plan.md` (TDA e Matriz de Migração) e `tasks.md` (P05 + Cenário de Falsificação). <br/>- Modo `after_implement`: auditoria de código real via Git Diff em 3 camadas (*Previsto em Task*, *Adaptação Correlata*, *Órfão Real / Drift*) + verificação estática mandatória de aridade/imports em call-sites modificados. <br/>- Emissão dual de relatório (chat da IDE e `specs/<feature>/Relatório de Análise e Consistência - <modo>.md`) com pausa mandatória para consentimento humano (HITL Soberano). |
| **`research.toml` (Novo)** | Workflow `/vitalia-research` e Sub-Agente `vitalia-researcher` operando nos 4 Eixos: Contrato Estrito do Cliente, Busca Web com validação HTTP 200 e leitura integral (detecção de depreciação), Engenharia Reversa Funcional em 3 Camadas e Emissão do Rastro Forense de Pesquisa (P03). |

---

### 3.5. Pasta `source/bin/` e `source/bin/lib/`

| Componente | Ação Consolidada |
| :--- | :--- |
| `lib/redis_detector.sh` | **Exclusão definitiva.** |
| `lib/logger.sh` | Acumulador em memória `_VITALIA_LOG_BUFFER` e função `write_install_log`. |
| `lib/agent_preserver.sh` | Resolver `${VITALIA_KIT_DIR:-${HOME}/.vitalia-spec-kit}` (zero referência a `~/.vitalia`). **Regra de Pureza de Contrato da Antigravity IDE:** NUNCA criar `.trash/` ou `.vitalia-manifest` dentro de `.agents/skills/`; salvar manifestos e backups fora de `.agents/skills/` (ex: em `.vitalia/`). |
| `lib/env_manager.sh` | Garantir que `VITALIA_KIT_DIR`, `SESSION_REPO` e `DATA_REPO` sejam preservados e gravados no `.env` do projeto. |
| `lib/git_storage_manager.sh` | **Preservação Comportamental (`[TEST:MIGRATION]`):** Quando uma URL remota for fornecida para `session` ou `data_storage`, verificar acessibilidade (`git ls-remote`) e executar **`git clone --depth=1`** do repositório existente; usar `git init` apenas quando nenhuma URL remota for fornecida ou quando o repositório remoto for novo/vazio. |
| `install-project` | - Foco exclusivo na Antigravity IDE (sem prompts de Gemini CLI). <br/>- Resolver `VITALIA_KIT_DIR` dinamicamente (respeitando a pasta escolhida pelo usuário ou o padrão `~/.vitalia-spec-kit`). <br/>- Gerar `.agents/skills/vitalia-*/SKILL.md` a partir de todas as extensões `.toml` instaladas (incluindo `vitalia-analyze`, `vitalia-research`, `vitalia-architect`, `vitalia-task-verifier`, `vitalia-clarify`). <br/>- Gravar log de instalação no destino condicional. |

---

### 3.6. Pasta `source/src/` e Motor de Contexto

| Componente | Ação Consolidada |
| :--- | :--- |
| `kit_env_bootstrap.py` | Resolução em 3 níveis (`os.environ["VITALIA_KIT_DIR"]` $\rightarrow$ `.env` do projeto $\rightarrow$ fallback padrão `$HOME/.vitalia-spec-kit`). Zero busca em `~/.vitalia`. |
| `vitalia_hook_runner.py` | Resolução via `kit_env_bootstrap.resolve_kit_dir()`, precedência de PATH com `.venv/bin` e Context Lifting gravado em `.vitalia/memory/data_storage/context/<machine_id>/`. |
| `hooks/guardian_context.py` | - **Sem Poda (`no_prune: true`):** Quando o perfil `gp_*.yaml` definir `no_prune: true` (ou filtros vazios), injetar **100% da `constitution.yaml`** sem tetos de tokens (`1000`/`4000`). <br/>- Expor `phase_guardrails` e `phase_profiles` do `gp_*.yaml` diretamente no `guardian_json`. <br/>- Carregar o histórico completo de `decisions.jsonl` e `learnings.jsonl` sem truncamento `n=5`. <br/>- Validar expiração temporal (`valid_until`) dos domínios de `grounding_domains.yaml`. |
| `vitalia_handoff.py` | Validações de Zero Data Leak compatíveis com `VITALIA_KIT_DIR` customizado ou padrão `~/.vitalia-spec-kit`. |
| `vitalia_context_engine.py` (e módulos `core/context_engine.py`, `core/shard_manager.py`, `core/utils.py`, `rendering/dashboard_renderer.py`) | **Preservação Integral do Motor de Sessão 3-Tier e Dashboard Rico:** <br/>- Timestamps gerados exclusivamente no formato canônico **`DD-MM-YYYY HH:MM:SS(GMT-04:00)`** (`get_timestamp()`). <br/>- Shards salvos por máquina em `.vitalia/memory/session/shards/<machine_id>.yaml` e ordenados por timestamp `last_sync` decrescente (garantindo que a máquina com sincronização mais recente lidere o destaque em `SESSION_STATE.md` e `README.md`). <br/>- `action_end`: persiste `learnings.jsonl`, `decisions.jsonl` e `session_history.jsonl` (com `machine_id` e data `DD-MM-YYYY HH:MM:SS(GMT-04:00)`), atualiza o shard YAML da máquina atual e invoca `action_consolidate()`. <br/>- `action_consolidate`: carrega `templates/dashboard_template.md` via `resolve_kit_dir()` e renderiza `README.md` com badges, diagrama Mermaid, tabela de máquinas, sessão ativa em destaque e seções `<details>` colapsáveis (Histórico, Decisões Arquiteturais Consolidadas e Guard Rails de Grounding), além de `SESSION_STATE.md`, `DECISIONS.md`, `LEARNINGS.md` e `SESSION_HISTORY.md`. |

---

### 3.7. Código Canônico de `source/templates/dashboard_template.md`

```markdown
<!-- README.md | Atualizado em: {timestamp} -->

# 🧠 Painel de Contexto — {project_name}

{badge_status}
{badge_semaforo}
{badge_modo}
{badge_sync}
{badge_grounding}

> **Vitalia SPEC Kit v0.0.1 — Ledger de Memória Persistente e Orquestração Local.**  
> Este repositório armazena o histórico distribuído, aprendizados consolidados e o controle de concorrência das sessões de trabalho do framework Vitalia.

---

## 📡 Topologia de Shards & Sincronização

<div align="center">

```mermaid
flowchart TD
{mermaid_content}
```

</div>

---

## 🖥️ Máquinas e Status Atual

<table>
  <thead>
    <tr>
      <th align="left">Máquina / ID</th>
      <th align="left">Tarefa Atual</th>
      <th align="center">Ambiente</th>
      <th align="center">Status</th>
      <th align="left">Último Sync</th>
      <th align="left">Próximo Passo (P0)</th>
    </tr>
  </thead>
  <tbody>
{shards_table}
  </tbody>
</table>

---

## 🎯 Sessão Ativa em Destaque

{sessao_ativa}

---

## 📚 Histórico, Decisões & Guard Rails

<details>
<summary><strong>🔍 Clique para expandir o Histórico Completo de Sessões</strong></summary>

<br/>

{historico_sessoes}
</details>

<details>
<summary><strong>⚖️ Clique para expandir as Decisões Arquiteturais Consolidadas</strong></summary>

<br/>

| Máquina (ID) | Decisão Arquitetural | Impacto / Racional |
| :--- | :--- | :--- |
{decisoes_arquiteturais}
</details>

<details>
<summary><strong>🛡️ Clique para expandir os Guard Rails de Grounding e Domínios</strong></summary>

<br/>

| Arquivo de Regras | Status | Domínios Monitorados | Pendentes de Curadoria HITL |
| :--- | :---: | :--- | :---: |
| `grounding_domains.yaml` (Global) | {global_status} | `provedores_de_ia`, `python_packages`, `web_technologies`, `external_apis`, `security_practices`, `regulations_compliance`, `cloud_services`, `scientific_claims`, `clinical_safety`, `software_architecture` | — |
| {local_link} (Projeto) | {local_status} | Domínios locais específicos do workspace | {pending_badge} |

</details>

---

<sub>Painel gerado automaticamente pelo motor de contexto do Vitalia Kit (<code>vitalia_context_engine.py --action consolidate</code>).</sub>
```

---

## 4. O Novo `kit-bootstrap.sh` (Da Pasta `source/` para a Pasta Escolhida pelo Usuário)

O `kit-bootstrap.sh` instala o código que reside em `source/` no repositório raiz diretamente para a pasta escolhida pelo usuário na máquina host:
1. **Passo 1 — Destino Global Interativo:** Solicita o caminho sugerindo **`~/.vitalia-spec-kit`** como opção padrão (ou aceita `VITALIA_KIT_DIR` / `--target-dir` em modo não-interativo) e expande `~` para `${HOME}`.
2. **Passo 2 — Pré-Validação de Runtime Host:** Verifica se `python3` e `python3 -m venv` estão operacionais no sistema operacional antes de criar diretórios; se `ensurepip`/`python3-venv` estiver ausente, emite instrução clara e aborta com código != 0.
3. **Passo 3 — Criação de Diretórios e Virtualenv Isolado:** Cria `${VITALIA_DIR}/{bin,bin/lib,extensions,profiles,presets,src,src/core,src/hooks,src/rendering,integrations/antigravity,templates}`, cria `${VITALIA_DIR}/.venv`, instala `requirements.txt` e valida a execução de `${VITALIA_DIR}/.venv/bin/python3 -c "import yaml, jinja2, jsonschema"`.
4. **Passo 4 — Instalação Componente a Componente a partir de `source/`:**
   - Copia `source/bin/install-project` e `source/bin/lib/{agent_preserver.sh,env_manager.sh,git_storage_manager.sh,logger.sh}` (zero `redis_detector.sh`).
   - Copia todos os `.toml` de `source/extensions/` (incluindo `analyze.toml`, `research.toml`, `architect.toml`, `task-verifier.toml`, `clarify.toml`).
   - Copia todos os perfis de `source/profiles/`: `constitution.yaml`, `routing_table.yaml`, `agents_catalog.yaml`, `brainstorming_intents.yaml`, `dynamic_questioning.yaml`, `grounding_domains.yaml` **e todas as Lentes de Fase Sem Poda `gp_*.yaml`**.
   - Copia `source/presets/` (se aplicável), `source/src/` (incluindo `core/`, `hooks/`, `rendering/`), `source/templates/dashboard_template.md`, `source/integrations/antigravity/SKILL.md.template` e `source/AGENTS.md`.
5. **Passo 5 — Shim em `~/.local/bin/`:** Cria `vitalia-install-project` exportando `VITALIA_KIT_DIR="${VITALIA_DIR}"` e invocando `${VITALIA_DIR}/bin/install-project`.

---

# PARTE III — Pós-Mortem Empírico (WSL2/zsh) e Blindagem Adversarial SDD (Ciclo 2)

## 1. Resumo Executivo e Diagnóstico de Causa-Raiz (RCA) do Ciclo 2

A validação empírica em máquina limpa (`andrenote` — WSL2 + zsh) revelou por que as entregas anteriores deixaram brechas apesar dos relatórios de aprovação:

1. **Síndrome do Teste Raso (Shallow / Tautological Testing):** Os testes E2E verificavam apenas `assert path.is_file()` e presença de strings estáticas, gerando falsos positivos enquanto o produto falhava em execução real.
2. **Teatro de Conformidade na Auditoria:** Os workflows de verificação validavam apenas correspondência textual superficial com as tarefas, sem executar análise de modos de falha (FMEA Mental) para cenários de máquina limpa.
3. **Reescrita Cega de Comportamentos Estáveis (`git_storage_manager.sh`):** Na extração modular do `install-project`, a lógica funcional de verificação e clonagem do repositório remoto (`git clone --depth=1`) foi substituída por um `git init` vazio com `.gitkeep`, amputando a sincronização de contexto existente.
4. **Violação do Contrato Estrito do Consumidor (Antigravity IDE):** A criação de `.agents/skills/.trash/` e `.agents/skills/.vitalia-manifest` violou o contrato de pureza de diretórios exigido pelo indexador de skills da Antigravity IDE, impedindo o carregamento dos slash commands.

---

## 2. Decisões Arquiteturais Definitivas (Consenso Consolidado do Ciclo 2)

### Decisão 1: Perfis `gp_*.yaml` como Lentes de Fase Sem Poda + `constitution.yaml` Integral
- A `constitution.yaml` é a fonte canônica única dos princípios universais (P00–P17) e é injetada **integralmente (`no_prune: true`, `total_budget: 0`, sem poda de tokens, severidade ou domínio)** em todos os workflows do ciclo SDD.
- Os arquivos `gp_<fase>.yaml` atuam como **Lentes Adversariais de Fase**, injetando `phase_guardrails` e `phase_profiles` estruturados para cada etapa (`brainstorming`, `specify`, `plan`, `tasks`, `task-verifier`, `implement`, `judge`, `analyze`, `research`).

### Decisão 2: Sub-Agente de Pesquisa Profunda (`vitalia-researcher` / `/vitalia-research`)
- Como o subagente nativo `research` da IDE Antigravity opera em sandbox sem acesso à rede, as pesquisas externas de Grounding (P03) utilizam o Sub-Agente `vitalia-researcher` (definido via `define_subagent` com `EnableWriteTools: true`) ou `self`, além da extensão dedicada `/vitalia-research` (`research.toml`).
- **Regra de Grounding Temporal:** Obrigatória validação HTTP 200 de cada link, leitura integral do conteúdo da página/documentação oficial (seguindo referências para versões mais recentes), checagem de depreciação e registro de Tabela de Proveniência.

### Decisão 3: Matriz Bidimensional de Testes P05 (4 Modalidades + Cenário de Falsificação)
O Princípio `P05` estabelece a **Matriz Bidimensional de 4 Modalidades**, exigindo que toda tarefa de teste em `tasks.md` declare sua tag e seu **Cenário de Falsificação** comportamental:
1. **`[TEST:NEW]` (Código Novo):** TDD Red-Green-Refactor executável (caixa-preta); falha comprovadamente no terminal antes da implementação e passa após o código.
2. **`[TEST:BUGFIX]` (Correção de Bug):** *Bug Reproduction First* — o teste reproduz o defeito real e **DEVE falhar contra o código atual no terminal antes do fix**, passando somente após a correção.
3. **`[TEST:REFACTOR]` (Refatoração Pura):** *Behavioral Characterization* — exercita entradas/saídas de negócio antes e depois da reestruturação, garantindo zero perda funcional.
4. **`[TEST:MIGRATION]` (Adaptação/Desacoplamento Arquitetural):** *Contract Preservation + Clean Purge* — comprova que os comportamentos funcionais estáveis do componente legado sobrevivem na nova arquitetura enquanto a infraestrutura obsoleta é purgada e as invariantes do consumidor são respeitadas.
- **Proibição Absoluta (BLOCKING):** Asserções puramente sintáticas/estáticas (`assert path.is_file()`, `assert "str" in read_text()`) nunca são aceitas isoladamente para atestar conclusão de funcionalidade ou bugfix.

### Decisão 4: Engenharia Reversa Funcional em 3 Camadas e Matriz de Migração no `spec-plan`
Sempre que componentes existentes forem refatorados ou adaptados, o `spec-plan` exige a **Matriz de Migração e Preservação Comportamental** baseada em 3 camadas:
- **Camada 1 (Intenção de Negócio Pura):** O que o componente antigo resolvia e deve ser preservado (ex: clone remoto Git `--depth=1`, ordenação de shards por máquina, renderização rica do dashboard, preservação de agentes sem poluir `skills/`).
- **Camada 2 (Purga de Infraestrutura Legada):** O que deve ser eliminado (Redis, Celery, Ollama local, tetos de VRAM).
- **Camada 3 (Contrato do Consumidor):** Como a Antigravity IDE e o ambiente limpo (WSL2/zsh) consomem o artefato (ex: pureza estrita de `.agents/skills/`, pré-validação de `python3 -m venv`, resolução dinâmica de `VITALIA_KIT_DIR`).

### Decisão 5: Estratégia de Rollback para a Spec 001 (`23bb91b`) e Dogfooding
- No próximo turno, o repositório raiz será regredido via Git até o estado íntegro da **Spec 001 (`commit 23bb91b`)**.
- Este documento-âncora (`.vitalia/memory/data_storage/research/anchor-research-brainstorming-002-003.md`), preservado no repositório isolado de `data_storage`, servirá de insumo inicial para executar o ciclo SDD unificado (Specs 002 + 003) utilizando os workflows blindados em `~/.vitalia/` e espelhando o resultado final validado em `source/`.

---

## 3. Rastro de Pesquisa e Grounding Consolidado (P03)

| # | Afirmação / Tecnologia | Fonte Canônica | Local / URL Verificada | Status Temporal | Data |
|---|---|---|---|---|---|
| 1 | Estrutura pura de `.agents/skills/<name>/SKILL.md` e indexação de slash commands na Antigravity IDE | Documentação Oficial Antigravity Customizations (`agy-customizations`) | `/home/andre/.gemini/antigravity/builtin/skills/agy-customizations/docs/skills.md` & `SKILL.md` | Ativo (v2.0 / 2026) | 07-11-2026 |
| 2 | Migração de workflows legados para Skills com suporte nativo a slash commands | Documentação Oficial Antigravity (`migrate-workflows`) | `/home/andre/.gemini/antigravity/builtin/skills/migrate-workflows/SKILL.md` | Ativo (v2.0 / 2026) | 07-11-2026 |
| 3 | Isolamento de rede do subagente nativo `research` vs. `define_subagent` / `self` | Especificação do Sistema de Subagentes Antigravity | Prompt de Sistema `<subagents>` da IDE Antigravity | Ativo (2026) | 07-11-2026 |
| 4 | Lógica original de clonagem remota de sessão (`git clone --depth=1`) | Código-fonte histórico do `install-project` | `/home/andre/.vitalia/bin/install-project` (linhas 100-140) | Referência de Preservação | 07-11-2026 |
| 5 | Formato canônico de timestamps `DD-MM-YYYY HH:MM:SS(GMT-04:00)` e consolidação de shards YAML por máquina | Motor de Contexto Vitalia (`utils.py`, `context_engine.py`, `shard_manager.py`) | `/home/andre/.vitalia/src/core/utils.py` & `context_engine.py` | Referência de Preservação | 07-11-2026 |
