# Promptzeiros

Trabalho final da disciplina **Engenharia de Prompt e Aplicações em IA** — Análise e Desenvolvimento de Sistemas, UNICID 2026.1.

Ecossistema de IA que automatiza atendimento por WhatsApp para pequenas e médias empresas (PMEs), em duas frentes:

- **Inbound** — quando alguém manda mensagem, um agente de IA responde, faz uma triagem em quatro perguntas (BANT) e encerra com handoff para o humano.
- **Outbound** — partindo de uma planilha de leads, o sistema envia mensagens proativas com travas que reduzem o risco de bloqueio do número.

---

## O que tem nesta entrega

| Arquivo / pasta | O que é |
|---|---|
| [`MEMORIAL.md`](MEMORIAL.md) | Memorial de construção — problema, público, ferramentas, papel da IA, prompts, limitações e melhorias futuras |
| [`DIAGRAMA.md`](DIAGRAMA.md) | Diagrama do ecossistema (Mermaid — renderiza no GitHub) |
| [`workflows/`](workflows/) | **Solução de automação** — fluxos n8n exportados em JSON (Inbound + Outbound) |
| [`prompts/`](prompts/) | **Solução de desenvolvimento** — system prompt em produção |
| [`Apresentacao-Final-0505.pdf`](Apresentacao-Final-0505.pdf) | Slides usados na apresentação do dia 05/05 |

---

## Stack utilizada

- **n8n Cloud** — orquestração visual de workflows
- **OpenAI GPT-4o-mini** — modelo de linguagem
- **Evolution API** — gateway WhatsApp (hospedado em Railway)
- **Google Sheets** — planilha de leads e auditoria

Justificativas técnicas em [`MEMORIAL.md`](MEMORIAL.md).

---

## Equipe

| Nome | RGM |
|---|---|
| **Gustavo Cavalcante de Amorim** (líder) | 47236232 |
| Renan Silva dos Santos | 47203099 |
| Mateus Felicio | 47290986 |
| Cauã Martins Aziago | 48100170 |
| Leonardo Alves | 47323485 |
| Laura Maciel | 47197871 |
| Denner Souza | 04733816-4 |
| Pedro Souza | 47290986 |
| Kayky Fernandes | 047331801 |
| João Lucas Moreira Dos Reis | 48162086 |

---

## Como rodar

1. Importar os dois JSON em [`workflows/`](workflows/) no n8n (Cloud ou self-hosted).
2. Conectar credenciais nos nós que pedem (Google Sheets e HTTP Header Auth da Evolution).
3. Trocar a URL da Evolution e o ID da planilha pelos seus.
4. Ativar os dois workflows.
5. Mandar `oi` no WhatsApp pareado — o bot responde.

---

## Licença

MIT — ver [`LICENSE`](LICENSE).
