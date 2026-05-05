# Prompts

Pasta com o **system prompt** em produção, usado pelo agente de IA do workflow Inbound.

| Arquivo | O que é | Onde aplicar |
|---|---|---|
| [`system-prompt-v7.md`](system-prompt-v7.md) | System prompt em produção (5 elementos de Carraro 2023 organizados em XML) | Workflow Inbound → nó `Extract Message Data` → campo `system_prompt` |

## Para adaptar a outro nicho

Trocar o conteúdo da tag `<contexto>` (negócio, persona, tom de voz) e ajustar os exemplos da tag `<exemplos>`. A estrutura dos workflows não muda.
