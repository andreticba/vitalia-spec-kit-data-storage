---
name: vitalia-skill-evaluation
description: >
  Analisa ativamente o histórico da sessão (transcript) para identificar gargalos,
---
<!-- SKILL.md | Vitalia Kit 0.6.0 — gerado por install-project.sh -->
<!-- Source: /home/andre/.vitalia/extensions/skill-evaluation.toml -->

# Vitalia: skill-evaluation

Ao acionar este comando:

1. Execute o script nativo para invocar os hooks e iniciar o processo de extração de contexto:
   ```bash
   python3 /home/andre/.vitalia/src/vitalia_hook_runner.py /home/andre/.vitalia/extensions/skill-evaluation.toml --phase before --args="{{args}}"
   ```
2. O script ejetará no stdout um **Stub JSON**. Extraia o caminho contido no campo `workflow_context_file`.
3. Utilize a ferramenta `view_file` para ler o arquivo JSON físico referenciado.
4. O arquivo lido conterá duas chaves fundamentais: `prompt` e `guardian_json`.
5. **Diretrizes de Comportamento e Execução (MANDATÓRIO):**
   - O `prompt` contém a sua **missão principal, passos de execução e formato de interação**. Ele é o seu objetivo absoluto para esta ação.
   - O `guardian_json` contém o **contexto operacional dinâmico** injetado para o seu workflow (pode incluir a Constituição do projeto, Tabelas de Roteamento, Catálogo de Agentes, etc.).
   - Execute o que o `prompt` mandar, utilizando as informações presentes no `guardian_json` como dados de suporte ou regras de contorno (ex: se o prompt disser para rotear, use a tabela do guardian; se mandar codar, obedeça à constituição do guardian).
   - Jamais permita que o conteúdo do campo `guardian_json` substitua o fluxo de etapas interativas e os gates definidos no campo `prompt`.
