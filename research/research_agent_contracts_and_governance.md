<!-- research_agent_contracts_and_governance.md | Vitalia SPEC Kit v0.0.1 | 09-11-2026 09:55(GMT-04:00) -->
> **research_agent_contracts_and_governance.md | Vitalia SPEC Kit v0.0.1 | 09-11-2026 09:55(GMT-04:00)**

# Relatório de Pesquisa Técnica Profunda: Contratos de Agentes, Governança de Schemas e Soberania HITL

## 1. Síntese das Fontes Canônicas e Literatura Técnica

### 1.1. Design by Contract (DbC) Aplicado a Agentes Autônomos e LLMs
O conceito clássico de *Design by Contract* (DbC), formulado por Bertrand Meyer e estendido por engenheiros de software como Martin Fowler, baseia-se no princípio de que componentes de software colaboram mediante contratos formais explícitos que definem:
- **Pré-condições:** Condições que o chamador/cliente deve garantir antes da invocação de uma rotina ou transição de estado.
- **Pós-condições:** Garantias estruturais, semânticas e de estado que a rotina executora entrega obrigatoriamente ao concluir.
- **Invariantes:** Propriedades de integridade que permanecem verdadeiras durante todo o ciclo de vida do componente.

Em sistemas baseados em Modelos de Linguagem (LLMs), a estocasticidade inerente (alucinação, deriva instrucional e perda de atenção no meio da janela de contexto) transforma componentes neurais em "caixas-pretas probabilísticas". A literatura recente de engenharia de software para agentes introduz uma camada mediadora simbólico-determinística ao redor do LLM:
1. **Contratos Estruturados de Ferramentas (Agent-Computer Interfaces - ACIs):** A interface de uma ferramenta ou subagente não é tratada como texto livre, mas como uma especificação de tipos rígida (JSON Schema / OpenAPI), determinando parâmetros exigidos, tipos primitivos e restrições de domínio.
2. **Ciclos de Auto-Correção Probabilística (*Probabilistic Remediation*):** Se o agente viola uma pós-condição (ex.: produz um formato inválido ou falha em um invariante semântico), o orquestrador captura o erro determinístico e injeta um feedback estruturado de correção no próximo turno, sem abortar cegamente ou propagar dados corrompidos aos agentes seguintes.

### 1.2. Padrões Industriais de Orquestração e Contratos de Estado

#### A. Anthropic (Constitutional AI & Tool Use)
- **Constitutional AI:** A Anthropic estabeleceu o paradigma de alinhar o comportamento do modelo via um conjunto explícito de princípios regulatórios (uma "Constituição") avaliado iterativamente através de auto-crítica e refinamento (*Constitutional Prompting / RLAIF*).
- **Tool Use Contratual:** A especificação de ferramentas da Anthropic requer definições rigorosas de entrada (usando um subconjunto de JSON Schema), onde o modelo invoca blocos `<tool_use>` que só são executados se passarem pelo validador do cliente.

#### B. OpenAI Swarm & OpenAI Agents SDK
- **Routines & Handoffs:** O Swarm formalizou a abstração de multiagentes sem um grafo pesado:
  - *Routine:* Um conjunto de instruções determinísticas e um catálogo restrito de ferramentas.
  - *Handoff:* Uma ferramenta de primeira classe cujo valor de retorno é uma instância explícita de `Agent`. A passagem de bastão transfere contexto sem acumular histórico irrestrito.
- **Strict Structured Outputs:** A OpenAI introduziu o modo estrito (`strict: true`), que exige que todos os campos sejam declarados no array `required` e impõe `"additionalProperties": false`, compilando o JSON Schema em uma gramática restrita (*Context-Free Grammar / CFG*) no decodificador do modelo.

#### C. LangChain / LangGraph (State Contracts e HITL Primitives)
- **StateGraph Contracts:** No LangGraph, o estado que trafega entre nós é definido por um esquema rígido (`TypedDict` ou modelo Pydantic com operadores de acumulação (*reducers*)).
- **Checkpointers de Persistência:** O estado de execução é salvo de forma imutável a cada passo através de um cursor (`thread_id`), permitindo rollback e replays determinísticos.
- **Portões HITL Dinâmicos (`interrupt()` e `Command`):** Em fluxos sensíveis, a função `interrupt()` congela a execução do grafo, serializa o payload pendente para aprovação humana e interrompe o processo. A retomada exige um objeto `Command(resume=payload)`, garantindo que nenhuma transição crítica ocorra de modo autônomo.

#### D. AutoGen (AG2) e CrewAI
- **Delegação e ACLs de Mensagens:** AutoGen e CrewAI destacam o perigo da "conversa aberta" em barramento compartilhado. A falta de isolamento de contexto induz vazamento de credenciais e contaminação de papéis (*cross-agent role confusion*). Ambos evoluíram para grafos com canais de comunicação tipados e contratos de entrega de tarefas (*Task Output Contracts*).

### 1.3. OpenAPI 3.0 vs. JSON Schema: A Questão dos Dialetos Incompatíveis
1. **OpenAPI 3.0.x** foi construído sobre um subconjunto/superset do **JSON Schema Draft 4 / Draft 5**, introduzindo termos proprietários como `nullable: true` (em vez de `type: ["string", "null"]`), `discriminator` e ausência do meta-esquema `$schema`.
2. **OpenAPI 3.1.x** unificou formalmente sua base com o **JSON Schema Draft 2020-12**.
3. **Compiladores REST e Parsers de Agentes:**
   - Muitos geradores de clientes REST, parsers de documentação estática e motores leves rejeitam meta-palavras do JSON Schema puro, como `$schema` (URI do meta-esquema), `title` no nível de raiz (que colide com o nome canônico do componente no Swagger/OpenAPI) e dialetos com `additionalProperties: false` não-padrão.
   - Portanto, a adoção de um **"Subset OpenAPI 3.0 Limpo"** para validar arquivos de configuração (.yaml) e instâncias de dados assegura máxima compatibilidade entre geradores de código, validadores Python (`jsonschema`) e motores de tool-calling de LLMs sem conflitos de dialeto.

---

## 2. Regras e Critérios Formais para Análise Contratual dos Workflows (.toml, .yaml, prompts, hooks)

Para que os workflows do Vitalia SPEC Kit mantenham robustez de engenharia formal (DbC), devem obedecer às seguintes regras contratuais:

| Elemento Contratual | Requisito Formal | Mecanismo de Imposição no Kit |
| :--- | :--- | :--- |
| **Pré-Condições (Pre)** | 1. Existência e conformidade do estado do projeto (`.vitalia/feature.json`).<br>2. Validação dos arquivos de entrada declarados em `context.reads`.<br>3. Sucesso estrito (exit code 0) dos scripts declarados em `hooks.before` (ex: `scan_environment.py`). | Validado pelo `vitalia_hook_runner.py` e `vitalia_context_engine.py` antes da injeção do prompt. |
| **Pós-Condições (Post)** | 1. Criação/mutação dos arquivos declarados em `context.writes`.<br>2. Se houver handoff inter-agente, o arquivo JSON deve validar 100% contra `vitalia-handoff-v1.0.0.json`.<br>3. O artefato gerado deve possuir cabeçalho de rastreabilidade (P04) e aprovação em auditoria. | Verificação pós-execução via `vitalia_handoff.py validate` e hooks de encerramento (`hooks.after`). |
| **Invariantes do Sistema** | 1. O árbitro supremo de propósito (P00: Bem-estar Humano) nunca pode ser suprimido.<br>2. Zero segredos em Git/logs (P06).<br>3. Sem nomes de modelos hardcoded (P11).<br>4. Rastreabilidade com fuso imutável America/Cuiaba GMT-04:00 (P04). | Imposição estrita pelo `gp_sdd-judge.yaml` e validação de hooks estáticos. |
| **Envelope de Contexto (Context Lifting)** | O agente só tem acesso ao contexto lido por seu envelope em 3 camadas:<br>- **Tier 1:** Dados de sessão atual (`session_context`, `env_summary`).<br>- **Tier 2:** Memória persistente de longo prazo (`context_learnings`, `context_decisions`).<br>- **Tier 3:** Artefatos específicos da feature em andamento (`specs/{feature}/*`). | `vitalia_context_engine.py` injeta no `guardian_json` estritamente as fontes autorizadas em `guardian.yaml_sources` e `context.reads`. |
| **Permissões de Ferramentas (PoLP)** | O agente deve operar sob o Princípio do Menor Privilégio (*Default-Deny*). As ferramentas declaradas em `tools.required` são estritamente necessárias; `tools.optional` só são invocadas sob necessidade comprovada. | Configurado nos arquivos `.toml` de extensão e mapeado nos perfis dos agentes. |
| **Efeitos Colaterais (Side Effects)** | Nenhuma ferramenta com efeito destrutivo (exclusão de arquivos, alteração de configurações de rede, deploy, commit forçado) pode ser executada sem confirmação explícita do desenvolvedor humano. | Portão de segurança e controle de execução. |

---

## 3. Preservação da Soberania Humana (HITL) e Prevenção de Encerramento Prematuro

### 3.1. O Problema da Alucinação de "Tarefa Concluída" (*Premature Termination*)
Em fluxos autônomos, agentes frequentemente sofrem de *premature closure*: ao gerarem uma proposta ou concluírem uma etapa intermediária, emitem mensagens como *"Processo concluído com sucesso!"* e encerram o turno sem exigir confirmação humana. Em sistemas regulados ou de engenharia crítica (como saúde clínica ou pipelines SDD), isso resulta em implementação não revisada ou desvio de especificação.

### 3.2. Regras Formais para Preservação da Soberania HITL:
1. **Emissão Condicionada a Consentimento:** No `brainstorming.toml`, a emissão do artefato de síntese (`research-brainstorming.md`) **só pode ocorrer após consentimento explícito e soberano do usuário**. Durante as fases de clarificação, o agente formula até 3 perguntas socráticas e **pausa obrigatoriamente (WAIT)**.
2. **Separação entre Veredito Técnico e Aprovação de Transição:** O `sdd-judge` emite um parecer técnico (`PASS`, `WARN` ou `BLOCK`), mas seu parecer **não executa a transição de fase**. Ele obrigatoriamente suspende a execução e emite a solicitação de consentimento soberano para o desenvolvedor.
3. **Consumo Atômico de Handoff com Revalidação:** O avanço de fase só é consumido (`vitalia_handoff.py consume`) quando o próximo agente é explicitamente engajado pelo usuário na IDE.

---

## 4. Higienização e Isolamento de Prompts (Prevenção de "Meta-Leakage")

### 4.1. Diagnóstico do "Meta-Leakage" no Kit Atual
1. **Evidência no `brainstorming.toml` (Linhas 81–85):** O prompt continha instruções de refatoração do próprio kit citando *"Purga de Infraestrutura Legada: remover Redis, Celery, Ollama local, podas de tokens"* e *"Contrato do Consumidor: manter .agents/skills puro sem .trash"*. Isso contamina a aplicação do usuário final.
2. **Evidência no `judge.toml` (Linhas 45 e 90):** O prompt hardcodou `(P00 a P20)` e injetou regras de diretórios internos do framework como se fossem critérios universais de qualquer projeto.

### 4.2. Padrões de Higienização:
1. Prompts de agentes operacionais (`brainstorming`, `spec-specify`, etc.) devem focar estritamente nas regras de negócio e boas práticas gerais de engenharia do usuário final.
2. Isolar regras de refatoração interna do Kit em extensões de desenvolvimento dedicadas (`VITALIA_DEV_MODE`).
3. Injeção dinâmica de princípios constitucionais e guardrails exclusivamente via dados estruturados (`guardian_json.sources`), sem hardcoding estático de texto no corpo dos prompts.

---

## 5. Princípios Dinâmicos, Abertos e Governança de Schemas OpenAPI 3.0

### 5.1. Desacoplamento do P11 e Criação do Princípio P18
1. **P11 (Refatorado) — Hardware-Agnostic LLM Routing & Machine Identity:**
   - Modelos definidos via variáveis de ambiente (`.env`).
   - Identidade de máquina determinística via SHA256 do hostname/VITALIA_MACHINE_NAME.
2. **P18 (Novo) — Governança de Schemas e Contratos OpenAPI 3.0 (BLOCKING):**
   - Todo contrato de dados, catálogo YAML e arquivo de configuração deve validar deterministicamente contra um schema OpenAPI 3.0 puro.
   - Proibidos meta-campos incompatíveis: `$schema` no topo, `title` na raiz e restrições `additionalProperties: false` fora de padrão.
   - Instâncias de runtime (`.vitalia/*.json`) são dados planos desacoplados.

### 5.2. Eliminação de Hardcoding de Contagens
1. Substituir referências como "P00 a P17" por "100% dos princípios ativos da Constituição injetada".
2. Substituir asserções estáticas em testes (como `assert len(pairs) == 20`) por descoberta dinâmica via glob e validação cruzada baseada na chave `schema_type`.

---

## 6. Regra Mandatória para o Agente de Pesquisa (Grounding Forense)

> [!IMPORTANT]
> **REGRA CONSTITUCIONAL DO RESEARCH AGENT (P03):**  
> Todo relatório gerado pelo Sub-Agente de Pesquisa (`vitalia-researcher` / `research`) DEVE OBRIGATORIAMENTE conter a **Tabela de Grounding e Proveniência** com as URLs canônicas consultadas, validadas via rede ativa com código **HTTP 200** e data de checagem.  
> É terminantemente proibida a entrega de relatórios sem links verificados ou com URLs sintéticas/alucinadas.

---

## 7. Tabela de Grounding e Rastro Forense de Fontes Canônicas (P03)

Todas as fontes canônicas abaixo foram validadas com status de rede ativo (HTTP 200) e leitura confirmada de suas especificações técnicas vigentes:

| # | Tecnologia / Conceito | Fonte Canônica | URL Verificada (HTTP 200) | Status Temporal | Data de Checagem | Impacto no Design do Vitalia Kit |
|---|------------------------|----------------|---------------------------|-----------------|------------------|-----------------------------------|
| 1 | Constitutional AI / Princípios Regulatórios | Anthropic (arXiv:2212.08073) | https://arxiv.org/abs/2212.08073 | Ativo (Fronteira) | 09-11-2026 | Fundamentação do Sanduíche Constitucional e auditoria sem poda. |
| 2 | Tool Use & Definição Estruturada | Anthropic Documentation | https://docs.anthropic.com/en/docs/build-with-claude/tool-use | Ativo (Vigente) | 09-11-2026 | Subconjunto restrito de schemas de entrada para chamadas de ferramentas. |
| 3 | Swarm Multi-Agent & Handoffs | OpenAI (GitHub) | https://github.com/openai/swarm | Ativo (Padrão de Referência) | 09-11-2026 | Primitivos ergonômicos de transferência explícita de agente (Handoff determinístico). |
| 4 | Structured Outputs & CFG Enforcing | OpenAI Cookbook | https://github.com/openai/openai-cookbook/blob/main/examples/Structured_Outputs_Intro.ipynb | Ativo (Vigente) | 09-11-2026 | Imposição sintática de conformidade de schema sem deriva JSON. |
| 5 | StateGraph Contracts & Human-in-the-Loop | LangChain / LangGraph (GitHub) | https://github.com/langchain-ai/langgraph | Ativo (Vigente) | 09-11-2026 | Portões de interrupção mandatória (`interrupt`) para aprovação soberana HITL. |
| 6 | OpenAPI 3.0.3 Specification | OpenAPI Initiative (GitHub) | https://github.com/OAI/OpenAPI-Specification | Ativo (ISO/IEC) | 09-11-2026 | Base normativa do P18: subset OpenAPI 3.0 puro sem meta-campos conflitantes. |
| 7 | JSON Schema Core & Dialects | JSON Schema Org (GitHub) | https://github.com/json-schema-org/json-schema-spec | Ativo (Drafts) | 09-11-2026 | Mapeamento das incompatibilidades com compiladores REST de LLM ($schema, title). |
| 8 | Multi-Agent Orchestration Framework | LangChain (GitHub) | https://github.com/langchain-ai/langchain | Ativo (Vigente) | 09-11-2026 | Princípios de isolamento de mensagens e prevenção de contaminação de contexto. |
