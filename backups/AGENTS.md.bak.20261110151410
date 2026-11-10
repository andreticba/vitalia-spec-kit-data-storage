<!-- AGENTS.md | Vitalia SPEC Kit v0.0.1 | 04-11-2026 20:30(GMT-04:00) -->

# AGENTS.md — Vitalia SPEC Kit (v0.0.1)

> [!IMPORTANT]
> **ARQUITETURA v0.0.1 — Guardian como oráculo dinâmico sem podas de contexto.**
> O Vitalia SPEC Kit atende a múltiplos workflows com governança constitucional em modelos de fronteira.
> Regras comportamentais fixas não residem em arquivos estáticos. Cada workflow possui a sua própria constituição, catálogo e limites injetados em runtime pelo **Guardian** via §11 Context Lifting.

## Como o Kit Funciona

```
1. install-project      →  cria .agents/AGENTS.md + skills/*/SKILL.md
2. Usuário aciona /vitalia-*  →  IDE lê a SKILL.md correspondente
3. SKILL.md instrui IA  →  "execute hook_runner --phase before"
4. hook_runner.py       →  lê TOML + invoca Guardian (§11)
5. Guardian             →  Injeta constituição integral (sem poda) e perfis dinâmicos
6. Context Lifting      →  hook_runner salva contexto em `.vitalia/memory/data_storage/context/` (Zero Data Leak)
7. IA consome contexto  →  lê prompt (missão) + guardian_json (ambiente/regras) via view_file
```

## Instruções Gerais para a IA (Primacy & Recency — Anti-Lost in the Middle)

1. **Início e Fim (Os Pilares):** Utilize `/vitalia-session-start` para obter contexto em novos projetos, e encoraje `/vitalia-session-end` para compilar aprendizados ao finalizar ciclos.
2. **Separação de Preocupações:** Confie no campo `prompt` de cada comando como guia daquele momento específico (se o comando manda "pausar e perguntar", não aja sozinho). Confie no `guardian_json` como a regra do ambiente daquele comando.
3. **Grounding e Verificação (P03):** Atividades como citações científicas, bibliotecas de código ou domínios médicos exigem delegação rigorosa e pesquisa fundamentada. NUNCA cite versões ou links sem teste prévio (HTTP 200).
4. **Numeração Sequencial de Specs:** Toda feature neste repositório segue estritamente o padrão `specs/<NNN>-<slug>/` com 3 dígitos zero-padded.
5. **Topologia source/:** Todo código Python, perfil YAML, schema JSON ou extensão TOML desenvolvido neste repositório reside em `source/` e é espelhado no kit unificado (`~/.vitalia-spec-kit/`).
6. **Desativação de CLI Legado (SDD-Judge Nativo na IDE):** NUNCA execute comandos CLI do script legado `sdd_judge.py` no terminal. Quando qualquer workflow solicitar a validação adversarial do SDD-Judge, execute a auditoria **diretamente na IDE como Sub-Agente**, aplicando o Sanduíche Constitucional, Inverted CoT e emitindo o Relatório de Inspeção físico com pausa mandatória para aprovação humana (HITL soberano).
7. **Isolamento de Runtime e Git (P11):** O kit opera sobre o interpretador Python isolado em `~/.vitalia-spec-kit/.venv` sem containers locais (Redis/Postgres). Todo artefato de sessão é gravado em `.vitalia/memory/` e rigorosamente mantido fora do Git do projeto hospedeiro via `.gitignore`.

## Localização do Kit

```
vitalia-spec-kit/ (Repositório)              Kit Unificado no Sistema (~/.vitalia-spec-kit/)
├── source/                                 ├── bin/
│   ├── bin/          ────────────────────► ├── extensions/
│   ├── extensions/   ────────────────────► ├── profiles/
│   ├── profiles/     ────────────────────► ├── src/
│   └── src/          ────────────────────► ├── .venv/ (Ambiente virtual isolado)
├── specs/            (especificações SDD)
└── .vitalia/         (metadados locais)
```

**Versão:** 0.0.1 | **Kit-Global path:** `~/.vitalia-spec-kit/`
