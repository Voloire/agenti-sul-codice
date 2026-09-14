# Lezione 7 — I protocolli, collocati: MCP e A2A

**Fase C · 1 sessione · produce: una tabella a due righe, MCP e A2A**

---

## Prima di leggere: la domanda d'apertura

Scrivi la risposta in `progressi/07.md`, sezione «Prima», in non più di cinque righe.

> **Quando il tuo coding agent apre una pull request, quale protocollo sta usando per passare il
> lavoro a chi lo rivede? E quando legge un file o chiama un'API, quale?**

Se la risposta alla prima è «nessuno, è git», hai già metà della lezione.

---

## La lezione

### 1. Due assi, due protocolli

Nel 2026 due protocolli aperti stanno sotto la stessa fondazione — la **Agentic AI Foundation**
della Linux Foundation — e vengono citati insieme così spesso da sembrare due pezzi della stessa
cosa. Non lo sono. Il modo più chiaro per tenerli distinti è quello che usa il progetto A2A stesso:
**«MCP is vertical. It deepens a single agent», «A2A is horizontal. It connects agents across that
boundary»**. E ancora: «A2A connects the agents to each other; MCP connects each agent to its own
tools».

L'asse verticale: un agente e i suoi strumenti. L'asse orizzontale: un agente e altri agenti. Questa
lezione serve a sapere cosa ciascuno standardizza, cosa non copre, e dove — nel caso dei coding
agent — si incontrano nella pratica. Non serve a implementarli.

### 2. MCP — Model Context Protocol

**Cos'è.** «An open protocol that enables seamless integration between LLM applications and
external data sources and tools», su JSON-RPC 2.0, ispirato al Language Server Protocol. La versione
corrente della specifica è **`2026-07-28`**; le versioni sono date, e indicano «the last date
backwards incompatible changes were made». Le versioni 2025 sono superate: leggi sempre quella
corrente. Anthropic ha donato MCP alla Agentic AI Foundation nel dicembre 2025.

**Tre ruoli.** L'**host** è l'applicazione che «initiate[s] connections» — il tuo coding agent —
e «enforces security policies and consent requirements», «handles user authorization decisions».
Il **client** è il connettore dentro l'host, uno per server: «communicates with exactly one
server». Il **server** è il servizio che «expose[s] resources, tools and prompts via MCP
primitives», locale o remoto. Un principio di design che conta per la sicurezza: «Servers should
not be able to read the whole conversation, nor "see into" other servers».

**Tre primitive** lato server: **tools** («Functions for the AI model to execute»), **resources**
(«Context and data, for the user or the AI model to use»), **prompts** («Templated messages and
workflows for users»). Lato client c'è l'**elicitation**, richieste del server all'utente per
informazioni aggiuntive.

**Trasporti.** Due: **stdio**, «newline-delimited messages over the standard streams of a
client-launched subprocess» — il server è un processo locale — e **Streamable HTTP**, «each
message is an HTTP POST to a single MCP endpoint». Il distinguo conta per l'autorizzazione: per
stdio la spec dice che «SHOULD NOT follow this specification, and instead retrieve credentials
from the environment» — cioè le credenziali stanno nell'ambiente, esattamente dove Comment and
Control le ha lette.

**Autorizzazione.** Opzionale, ma se c'è è OAuth 2.1: «Authorization servers MUST implement OAuth
2.1»; il server MCP è un OAuth resource server, il client un OAuth client. E qui tornano le due
frasi della lezione 5: il `resource` di RFC 8707 obbligatorio in ogni richiesta, e i server che
«MUST validate that access tokens were issued specifically for them as the intended audience» —
«MUST NOT accept or transit any other tokens». Il **token passthrough** è vietato per nome.

### 3. A2A — Agent2Agent

**Cos'è.** «An open standard for seamless communication and collaboration between AI agents»:
agenti che possono «delegate sub-tasks, exchange information, and coordinate actions» senza
condividere i propri internals. Nato in Google, donato alla Linux Foundation; versione **1.0.0**
del 12 marzo 2026, «the first stable, production-ready version»; dall'agosto 2026 progetto della
stessa Agentic AI Foundation di MCP. Il comunicato LF dell'aprile 2026 conta «over 150 supporting
organizations», con Google Cloud, Microsoft Azure e AWS Bedrock fra le piattaforme.

**Agent Card.** «A JSON metadata document describing an agent's identity, capabilities, endpoint,
skills, and authentication requirements», pubblicato per convenzione in
`/.well-known/agent-card.json`. È il modo in cui un agente si presenta a un altro.

**Task.** «A stateful unit of work initiated by an agent, with a unique ID and defined lifecycle.»
Gli stati, con i nomi della spec 1.0: `SUBMITTED`, `WORKING`, `INPUT_REQUIRED`, `AUTH_REQUIRED`,
`COMPLETED`, `FAILED`, `CANCELED`, `REJECTED` (più `UNSPECIFIED`). Gli ultimi quattro sono
terminali: «Once a task reaches a terminal state... it cannot restart»; un seguito è un nuovo task
nello stesso `contextId`. `INPUT_REQUIRED` e `AUTH_REQUIRED` sono stati di interruzione: il task
aspetta qualcuno.

**Message e Artifact.** Un **message** è «a single turn of communication between a client and an
agent» — per interazioni brevi. Un **artifact** è «a tangible output generated by an agent during a
task (for example, a document, image, or structured data)». Entrambi fatti di `Part`: testo, file,
dati. L'agente risponde con un task quando il lavoro richiede «substantial, trackable work over an
extended period».

### 4. Cosa nessuno dei due copre

Vale la pena dirlo esplicitamente, perché è dove le aspettative sbagliano. Stando ai testi:

- **L'audit trail.** MCP lo nomina solo come *rischio* del token passthrough («Accountability and
  Audit Trail Issues»); non definisce nessun formato né obbligo di log. A2A non lo tratta. Chi ha
  fatto cosa resta a carico dell'host o dell'orchestratore.
- **La separazione fra chi scrive e chi approva.** MCP dice che «Hosts must obtain explicit user
  consent before invoking any tool» e che «MCP itself cannot enforce these security principles at
  the protocol level»: il consenso è un requisito sull'host, non un ruolo nel protocollo. A2A ha
  stati di interruzione, non un approvatore terzo.
- **Il codice sorgente.** Nessuna delle due specifiche parla di repository, branch, diff, pull
  request o commit. Nel caso dei coding agent il lavoro — il diff — è fuori da entrambi i
  protocolli: A2A al più lo trasporta come `Artifact`, MCP come risultato di un tool.
- **L'identità dell'agente come principal.** MCP autorizza il *client* per conto di un *resource
  owner*; A2A descrive gli schemi di sicurezza nell'Agent Card. Nessuno dei due, nei testi letti,
  definisce un'identità dell'agente distinta dall'utente — ed è il tema della lezione 5.

Queste quattro assenze sono inferenze di chi scrive dai testi, non frasi delle specifiche. Ma sono
esattamente le quattro cose che le lezioni 5 e 6 hanno costruito **fuori dai protocolli**: con i
ruleset, la GitHub App, la pull request.

### 5. Dove stanno nei coding agent

| | MCP | A2A |
|---|---|---|
| Copilot cloud agent | sì: «You can use MCP to extend the capabilities of Copilot cloud agent»; solo tools, non resources o prompts; niente server remoti con OAuth | non dichiarato |
| Codex (CLI, IDE, cloud) | sì: stdio e Streamable HTTP; OAuth con Client ID Metadata Documents e Dynamic Client Registration | non dichiarato |
| Claude Code | sì: HTTP, stdio, WebSocket; OAuth 2.0; può fare anche da server (`claude mcp serve`) | non dichiarato |

Tutti e tre parlano MCP; **nessuno dei tre dichiara A2A**. Non è un giudizio su A2A — è reale, ha
150 organizzazioni dietro e sta nelle piattaforme enterprise — è la descrizione di dove sta oggi:
nelle piattaforme di orchestrazione, non nella pipeline di codice. Nella pipeline di codice il
passaggio di lavoro fra un agente e chi lo rivede ha già un protocollo, e si chiama **pull
request**: ha un ciclo di vita (aperta, in revisione, approvata, mergiata, chiusa), un artefatto
(il diff), un'identità dell'autore, un audit trail (la storia di git), e un approvatore separato
imposto dal ruleset. Sono le quattro cose che i protocolli non coprono.

### 6. Cosa te ne fai

Derivazione di chi scrive. Quando qualcuno propone «un'architettura A2A» o «tutto via MCP», tre
domande:

1. **Su quale asse siamo?** Se è un agente che deve usare strumenti, è MCP. Se sono agenti che si
   passano lavoro, è A2A — o, nel codice, una pull request.
2. **Dove sta la verità del lavoro?** Se l'artefatto è codice, sta in git, e il protocollo lo
   trasporta al massimo. Se qualcuno vuole che il protocollo *sia* il record, chiedere dov'è
   l'audit trail.
3. **Chi tiene il token, e per chi è emesso?** La domanda della lezione 5, che MCP rende
   normativa: audience binding, niente passthrough. Un server MCP che «rigira» token è la
   prima cosa da cercare in un assessment.

---

## Le fonti di questa lezione

Overview e concetti, non le specifiche intere.

**MCP** — [Versioning](https://modelcontextprotocol.io/specification/versioning) (qual è la corrente), [Architecture](https://modelcontextprotocol.io/specification/latest/architecture), [Transports](https://modelcontextprotocol.io/specification/latest/basic/transports), [Authorization](https://modelcontextprotocol.io/specification/latest/basic/authorization), [Security Best Practices](https://modelcontextprotocol.io/specification/latest/basic/security_best_practices); il [post sulla donazione alla AAIF](https://blog.modelcontextprotocol.io/posts/2025-12-09-mcp-joins-agentic-ai-foundation/).

**A2A** — [Key concepts](https://a2a-protocol.org/latest/topics/key-concepts/), [Life of a Task](https://a2a-protocol.org/latest/topics/life-of-a-task/), [A2A and MCP](https://a2a-protocol.org/latest/topics/a2a-and-mcp/); il [post della 1.0](https://a2a-protocol.org/latest/blog/2026/03/12/a2a-protocol-ships-v10-production-ready-standard-for-agent-to-agent-communication/) e il [comunicato LF](https://www.linuxfoundation.org/press/a2a-protocol-surpasses-150-organizations-lands-in-major-cloud-platforms-and-sees-enterprise-production-use-in-first-year).

**Nei coding agent** — GitHub, [MCP and Copilot cloud agent](https://docs.github.com/en/copilot/concepts/agents/cloud-agent/mcp-and-cloud-agent); OpenAI, [Codex MCP](https://developers.openai.com/codex/mcp); Anthropic, [Claude Code MCP](https://code.claude.com/docs/en/mcp).

---

## Esercizio: la tabella a due righe

Scrivi in `progressi/07.md`, sezione «Oggetto».

### Template

```markdown
| | cosa standardizza | cosa non copre | dove l'ho visto nel percorso (lezioni 4, 5, 6) |
|---|---|---|---|
| MCP | | | |
| A2A | | | |
```

### Soluzione di riferimento

| | cosa standardizza | cosa non copre | dove l'ho visto |
|---|---|---|---|
| MCP | come un agente accede a tool, risorse e prompt di un server; il trasporto (stdio, HTTP); l'autorizzazione OAuth 2.1 con audience binding | l'audit trail; chi approva; il codice sorgente; l'identità dell'agente come principal | lezione 4: il GitHub MCP server con il token dell'utente; il proxy di Codex che non filtra le connessioni MCP. Lezione 5: il divieto di token passthrough |
| A2A | come un agente si presenta (Agent Card) e chiede a un altro di fare un task, con stati, messaggi e artefatti | lo stesso; e nessun coding agent lo parla | lezione 6: il passaggio di lavoro nel codice è la pull request, con ciclo di vita, artefatto, autore e approvatore |

### Dopo

Torna alla domanda d'apertura. Le due risposte — quale protocollo per passare il lavoro, quale per
usare strumenti — sono cambiate? Tre righe.

---

## Autoverifica

1. La frase con cui il progetto A2A distingue i due protocolli, e cosa significa per un coding
   agent.
2. Quattro cose che nessuno dei due copre, e cosa nel percorso le ha coperte.
3. Cosa dice la spec MCP sulle credenziali per il trasporto stdio, e quale incidente della
   lezione 4 rende quella frase concreta.

<details>
<summary>Risposte</summary>

1. «MCP is vertical. It deepens a single agent», «A2A is horizontal. It connects agents across that
   boundary». Per un coding agent: gli strumenti — file, API, database — passano da MCP; il
   passaggio del lavoro a chi lo rivede non passa da A2A, che nessuno dei tre vendor dichiara,
   ma dalla pull request.
2. **Audit trail**, **separazione fra chi scrive e chi approva**, **codice sorgente** (repository,
   diff, PR), **identità dell'agente come principal**. Le hanno coperte, fuori dai protocolli: la
   storia di git, i ruleset con la bypass list vuota, la pull request, la GitHub App con
   l'installation token (lezioni 5 e 6).
3. Per stdio i client «SHOULD NOT follow this specification, and instead retrieve credentials from
   the environment»: le credenziali dei server MCP locali stanno nelle variabili d'ambiente. Comment
   and Control le ha lette da lì — il processo MCP «retain[s] full environment» — con `ps auxeww`.

</details>

---

## Chiusura del passo

Chiuso quando, senza appunti, sai dire i due assi con la frase di A2A, le quattro assenze, e perché
nella pipeline di codice il protocollo di handoff è la pull request. Data in `progressi/README.md`.

**← Lezione 6** [Il lab](../06-lab/README.md) ·
**Prossima →** [08 · L'assessment, misurabile](../08-assessment/README.md)
