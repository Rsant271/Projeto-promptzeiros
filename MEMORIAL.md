# Memorial de Construção — Promptzeiros

**Disciplina:** Engenharia de Prompt e Aplicações em IA — UNICID 2026.1
**Grupo:** Promptzeiros
**Data de entrega:** 05/05/2026

---

## Equipe

| RGM | Nome | Papel |
|---|---|---|
| 47236232 | Gustavo Cavalcante de Amorim | **Líder do grupo** |
| 47203099 | Renan Silva dos Santos | Integrante |
| 47290986 | Mateus Felicio | Integrante |
| 48100170 | Cauã Martins Aziago | Integrante |
| 47323485 | Leonardo Alves | Integrante |
| 47197871 | Laura Maciel | Integrante |
| 04733816-4 | Denner Souza | Integrante |
| 47290986 | Pedro Souza | Integrante |
| 047331801 | Kayky Fernandes | Integrante |
| 48162086 | João Lucas Moreira Dos Reis | Integrante |

---

## 1. Problema e público-alvo

### Problema
Pequenas e médias empresas (PMEs) brasileiras — clínicas, imobiliárias, escritórios, lojas — precisam estar presentes no WhatsApp para captar leads, mas:

- Não conseguem responder em poucos segundos (a maioria dos leads "esfria" se demorar).
- Não têm tempo de qualificar manualmente cada contato (perguntar orçamento, urgência, intenção).
- Têm receio de bloqueio do número quando fazem disparos sem critério.
- Não têm verba para contratar um time de SDR humano nem ferramenta corporativa.

### Público-alvo
- **Primário:** PMEs com 1 a 50 funcionários (clínicas, imobiliárias, agências, advocacia).
- **Secundário:** Profissionais autônomos que recebem leads de redes sociais (corretores, consultores).

### Objetivo
Construir um ecossistema de IA que:
1. Atende mensagens recebidas (Inbound) com uma triagem em 4 perguntas e devolve um lead já qualificado para o humano.
2. Faz disparos controlados (Outbound) a partir de uma planilha, com travas para reduzir risco de bloqueio.
3. Pode ser adaptado a outros nichos trocando apenas o system prompt — a estrutura dos workflows não muda.

---

## 2. Solução de automação

### Ferramentas escolhidas
- **n8n Cloud** — orquestração visual dos workflows.
- **Evolution API** — gateway WhatsApp (hospedado em Railway).
- **OpenAI GPT-4o-mini** — modelo de linguagem que conduz a conversa.
- **Google Sheets** — planilha onde ficam os leads, status de envio e marcação de opt-out.

### Workflow 1 — Inbound (atendimento)

```
Webhook Evolution
   └── Filtra mensagem pessoal (ignora grupo / broadcast / eco do bot)
          └── Extrai dados da mensagem
                 └── Detecta opt-out ("SAIR" / "PARAR" / "CANCELAR")
                        ├── (sim) → Resposta automática + atualiza planilha
                        └── (não) → AI Agent (GPT-4o-mini)
                                       └── Envia resposta pelo WhatsApp
```

11 nós no total.

### Workflow 2 — Outbound (disparo)

```
Manual Trigger
   └── Lê leads da planilha (filtros: status pendente, consentimento TRUE)
          └── Loop por lead (1 por vez)
                 └── Valida horário, template e palavras proibidas
                        └── Verifica se o número tem WhatsApp
                               └── Espera (delay aleatório)
                                      └── Mostra "digitando..."
                                             └── Envia mensagem
                                                    └── Marca status na planilha
```

15 nós no total.

### Travas anti-bloqueio aplicadas no Outbound

| Trava | O que faz |
|---|---|
| 1 lead por vez | Processa um lead de cada vez (sem rajada) |
| Delay aleatório | Espera entre 15 e 90 segundos entre disparos |
| Janela horária | Só dispara em horário comercial (9h–12h e 13h–18h, fuso SP) |
| 3 templates A/B/C | Alterna a redação para o WhatsApp não detectar repetição |
| Personalização | Substitui `{{nome}}` pelo nome real do lead |
| Palavras proibidas | Bloqueia "promoção", "desconto", "urgente", "imperdível", etc. |
| Sem links na 1ª mensagem | Nem URL nem `www.` |
| Tamanho variável | Mensagens entre 80 e 600 caracteres |
| Validação do número | Confirma se é WhatsApp antes de enviar |
| "Digitando..." | Mostra status de digitação como faria um humano |
| Opt-out explícito | Todo template termina com "responda SAIR" |

### LGPD
- **Consentimento explícito:** a planilha tem uma coluna `consentimento` — leads sem `TRUE` não são lidos.
- **Opt-out 1-clique:** quem responder `SAIR` no WhatsApp não é mais contactado.
- **Coleta mínima:** só armazenamos nome + telefone + nicho.

---

## 3. Solução de desenvolvimento

A "aplicação" do projeto é a própria conversa via WhatsApp — o usuário interage por mensagens nativas, sem precisar instalar nada.

A planilha do Google Sheets funciona como CRM-leve para o gestor consultar leads e triagens.

O editor visual do n8n serve como dashboard para acompanhar execuções, logs e erros.

A pasta [`prompts/`](prompts/) contém o system prompt em produção, que é o "coração" da aplicação — é ele que define como o agente se comporta.

---

## 4. Diagrama do ecossistema

Ver [`DIAGRAMA.md`](DIAGRAMA.md) — três diagramas em Mermaid (renderizado pelo GitHub):
1. Camadas funcionais do agente.
2. Fluxo de dados (Inbound + Outbound).
3. Branch de decisão LGPD.

Resumo das 4 camadas (Murta 2023):

| Camada | Função | No projeto |
|---|---|---|
| Perceptiva | Recebe a entrada | Webhook Evolution → n8n |
| Cognitiva | Compreende e decide o que responder | AI Agent (GPT-4o-mini + system prompt) |
| Decisória | Aplica políticas e filtros | Detecta opt-out, valida templates, anti-ban |
| Executiva | Executa a ação | Envia mensagem pela Evolution + atualiza planilha |

---

## 5. Engenharia de prompt — papel da IA

O system prompt é o que dá personalidade e regras ao agente. Usamos os **5 elementos de Carraro (2023, p. 165)** organizados em tags XML, porque facilita o modelo identificar cada parte:

```xml
<contexto>...</contexto>
<instrucao>...</instrucao>
<formato_obrigatorio>...</formato_obrigatorio>
<exemplos>...</exemplos>
<constraints>...</constraints>
```

### O que cada elemento faz no projeto

- **`<contexto>`** — descreve o negócio (imobiliária no exemplo entregue), a persona do agente e o tom de voz.
- **`<instrucao>`** — diz para o agente fazer **uma pergunta de cada vez**, seguindo o roteiro BANT (Budget, Authority, Need, Timeline) adaptado.
- **`<formato_obrigatorio>`** — define como a resposta tem que sair: texto puro, sem emoji, sem markdown, no máximo 2 linhas (best practice WhatsApp).
- **`<exemplos>`** — mostra um diálogo completo de exemplo, do "oi" até o fim da triagem.
- **`<constraints>`** — limites: nunca prometer preço, nunca inventar dados, sempre transferir para humano se o usuário pedir.

### Como adaptar para outro nicho

Trocando o conteúdo da tag `<contexto>` (e ajustando os exemplos), o mesmo bot atende uma clínica odontológica, um escritório de advocacia ou uma barbearia. **A estrutura dos workflows não muda — só o prompt.**

O system prompt em produção está em [`prompts/system-prompt-v7.md`](prompts/system-prompt-v7.md).

---

## 6. Justificativa das ferramentas

### Por que GPT-4o-mini?
- Custo baixo: US$ 0,15 por milhão de tokens de entrada.
- Latência baixa: o agente responde em menos de 1 segundo.
- Tokenizer otimizado para português brasileiro.
- Suficiente para a tarefa (não precisa de modelo "topo de gama" para fazer 4 perguntas).

### Por que Evolution API e não a oficial da Meta?
- É gratuita e self-hosted (cabe no orçamento de um trabalho acadêmico).
- Boa para protótipo / MVP educacional.
- **Limitação assumida:** Evolution não é oficial e tem risco de bloqueio. Para uso real em produção, recomendamos um BSP oficial (YCloud, Twilio, Meta Cloud API). Documentado em §8.

### Por que n8n?
- Visual e auditável — a banca consegue ver o fluxo na hora da apresentação.
- Cloud gerenciado, sem precisar manter servidor.
- Tem suporte nativo a memória, AI Agent e function calling.

### Por que Google Sheets?
- Equipe não-técnica consegue editar a lista de leads.
- Custo zero.
- **Limitação assumida:** não escala acima de ~10 mil leads/dia. Em produção real precisaria migrar para um banco (Postgres ou similar).

---

## 7. Limitações conhecidas

1. **Evolution API não-oficial:** risco de bloqueio do número. Em produção deveria ser BSP oficial.
2. **Memória in-memory:** o histórico da conversa é perdido se o n8n reiniciar. Em produção, usaríamos Postgres Chat Memory.
3. **Google Sheets como banco:** não escala. Em produção, migrar para Postgres.
4. **Sem captura de áudio nem imagem:** o usuário precisa digitar. Extensão futura usaria Whisper (áudio) e Vision (imagens).
5. **Validação manual:** não temos suíte automatizada de testes de regressão.

---

## 8. Melhorias futuras

| Versão | Melhoria | Por quê |
|---|---|---|
| v2 | Postgres no lugar do Sheets | Escala para milhares de leads |
| v2 | Postgres Chat Memory | Não perde histórico em restart |
| v2 | Function Calling no GPT-4o-mini | Estruturar a saída sem depender de marker textual |
| v3 | Whisper para áudio | Lead mandar áudio e o bot transcrever |
| v3 | Vision para imagens | Lead mandar print de produto / imóvel |
| v3 | Integração com CRM (HubSpot / Pipedrive) | Sair da planilha e entrar em ferramenta corporativa |

---

## 9. Referências

- **Carraro, R. (2023).** *Engenharia de Prompts para Sistemas de IA.* — 5 elementos do prompt (p. 165).
- **Mello, F. (2024).** *Agentes de IA: Arquiteturas Adaptativas.* — Template parametrizado e limites éticos da autonomia (Unidade IV).
- **Murta, D. (2023).** *Interpretabilidade em Sistemas de IA.* — 4 camadas do agente (Unidade IV p. 2).
- **Lee, Goldberg, Kohane (2024).** *Privacy-by-Design in Conversational AI.* — Privacidade como arquitetura (Unidade IV p. 4).

---

Grupo Promptzeiros — UNICID 2026.1
