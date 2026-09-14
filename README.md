# Agenti sul Codice

**Un percorso di studio su come l'industria fa lavorare i coding agent in modo governabile.**
Otto passi, tredici–quindici sessioni da un'ora, solo terminale. Solo fonti primarie correnti,
verificate una per una.

> Il percorso non insegna a programmare agenti. Insegna a governarli: identità e credenziali,
> la pull request come unico output, la CI prima della review, chi approva, cosa succede quando
> qualcuno mette istruzioni dentro una issue.

---

## Le lezioni

Ogni lezione: domanda d'apertura, il contenuto spiegato dalle fonti, le fonti da leggere dopo,
l'esercizio con template e soluzione di riferimento, l'autoverifica con le risposte.

| # | lezione | sessioni | stato |
|---|---|---|---|
| 1 | [Le posizioni correnti sul multi-agent](lezioni/01-posizioni-correnti/README.md) | 2 | disponibile |
| 2 | [I controlli dei tre vendor, letti in parallelo](lezioni/02-controlli-vendor/README.md) | 1 | disponibile |
| 3 | [La griglia: OWASP Top 10 for Agentic Applications 2026](lezioni/03-griglia-owasp/README.md) | 1 | disponibile |
| 4 | [Gli incidenti veri, ricostruiti](lezioni/04-incidenti/README.md) | 2 | disponibile |
| 5 | Identità e credenziali degli agenti | 3 | in scrittura |
| 6 | Il lab: il workflow di riferimento, una volta | 3–4 | in scrittura |
| 7 | I protocolli, collocati | 1 | in scrittura |
| 8 | L'assessment, misurabile | 1–2 | in scrittura |

Per seguirlo: fai un fork, e usa `progressi/` — un file per passo, già predisposto per il primo.
Il resto di questa pagina è il syllabus: perché questo percorso, il workflow di riferimento, i
passi in breve, il glossario, le fonti.

---

## Perché questo percorso e non un altro

**Il pezzo maturo ha un nome preciso.** GitHub Copilot cloud agent, OpenAI Codex, Claude Code,
Devin, Cursor, Jules fanno tutti la stessa cosa: task → ambiente isolato → branch proprio →
**l'unico output è una pull request** → CI prima che qualcuno legga → un umano approva → l'agente
non può approvarsi. Da qui si impara.

**Il multi-agent non è più «immaturo», è delimitato.** Nel 2026 il punto contestato è uno solo:
*più scrittori in parallelo sullo stesso codice*. In produzione ci sono un writer più agenti che
leggono e rivedono, e molti agenti indipendenti ciascuno sulla propria PR con l'umano al merge.
Anthropic e Cognition, partite nel 2025 da posizioni opposte, nel 2026 convergono sul valutatore separato e restano diverse solo sulla scrittura in parallelo.

**La governance ha una griglia, e gli incidenti hanno un nome.** OWASP Top 10 for Agentic
Applications 2026 (ASI01–ASI10) è la rubrica con cui un cliente e un auditor misurano. Comment
and Control, CamoLeak, il GitHub MCP server, s1ngularity sono i casi da cui i controlli
discendono.

---

## Il workflow di riferimento

Quello che i tre vendor fanno uguale, con i controlli al posto giusto:

| fase | controllo dichiarato da ogni vendor |
|---|---|
| task / issue | solo chi ha `write` può assegnarlo all'agente |
| sandbox | ambiente isolato, `egress allowlist` sulla rete in uscita |
| branch proprio | identità del bot separata, push solo su quel branch |
| pull request | draft, unico output dell'agente |
| CI | required status checks prima di qualunque lettura |
| reviewer | da contesto pulito: non ha visto come è nata la modifica |
| merge | un umano, mai l'autore |

Negli incidenti del 2025–2026 ha ceduto quasi sempre lo stesso anello: la rete in uscita, e un
contenuto non fidato (issue, commento, titolo di PR) letto come istruzione.

---

## Gli otto passi

Ogni passo legge la fonte, non un riassunto; produce un oggetto — una checklist, una tabella,
una rubrica — non un'opinione; ed è chiuso quando lo si sa raccontare in tre frasi senza
rileggere gli appunti. Si leggono **solo testi correnti**: nessuna fonte superata o ritrattata
entra nelle letture; se viene nominata è perché la si sentirà citare e va saputa collocare.

### 1 · Le posizioni correnti sul multi-agent — 2 sessioni

**Leggi.** Anthropic, *Building multi-agent systems: When and how to use them*, gen. 2026.
Cognition, *Multi-Agents: What's Actually Working*, apr. 2026. Solo questi due: il «Don't build
multi-agents» del 2025 non si legge come posizione corrente, l'autore l'ha ristretto; basta
sapere che esiste.

**Capisci.** Single agent prima. Un secondo agente è giustificato in tre situazioni:
`context protection`, parallelizzazione (per copertura, non per velocità), specializzazione per
strumenti, comportamento o dominio. Cognition: `writes stay single-threaded`; Anthropic ammette
scritture parallele solo con contesto davvero isolato. Entrambe: il valutatore lavora da contesto
pulito.

**Fai.** La checklist «quando un secondo agente è giustificato», una pagina.

### 2 · I controlli dei tre vendor, letti in parallelo — 1 sessione

**Leggi.** GitHub, *Risks and mitigations for Copilot cloud agent*. OpenAI, *Codex — Agent
approvals & security*. Anthropic, `claude-code-action/docs/security.md`.

**Capisci.** Tre vendor, gli stessi controlli: trigger solo da chi ha `write`, push solo sul
branch dell'agente, PR draft che un umano deve mergiare, firewall sulla rete in uscita, filtro
dei caratteri nascosti. La convergenza *è* la prova che esiste un «modo giusto».

**Fai.** Una tabella controllo × vendor: presente, assente, diverso. Dove divergono ci sono le
domande da fare a un fornitore.

### 3 · La griglia: OWASP Top 10 for Agentic Applications 2026 — 1 sessione

**Leggi.** OWASP GenAI Security Project, *Top 10 for Agentic Applications 2026*, ASI01–ASI10.
Solo questo: la voce `LLM06 Excessive Agency` dell'edizione 2025 è un riferimento da riconoscere
quando viene citata, non una lettura.

**Capisci.** I dieci rischi con nome e sigla, e i tre root cause dell'`excessive agency`: troppa
funzionalità, troppi permessi, troppa autonomia. È la lingua in cui un auditor scrive.

**Fai.** La rubrica per l'assessment, ancora vuota: dieci righe ASI, colonne «presente / assente /
ignoto» e «evidenza».

### 4 · Gli incidenti veri, ricostruiti — 2 sessioni

**Leggi.** *Comment and Control* (apr. 2026): Claude Code Security Review action, Gemini CLI
action e Copilot agent dirottati da un titolo di PR o da un commento HTML in una issue; su Copilot
tre mitigazioni runtime aggirate leggendo `/proc/[pid]/environ`. Invariant Labs, *GitHub MCP
server* (mag. 2025): una issue pubblica fa trapelare repo privati. *CamoLeak* (ott. 2025, CVSS
9.6). Wiz, *s1ngularity* (ago. 2025): il primo supply chain attack che usa Claude Code e Gemini
CLI installati localmente per cercare segreti. Replit/SaaStr (lug. 2025) come nota: agente con
credenziali di produzione, nessuna PR.

**Capisci.** Il pattern è uno: `indirect prompt injection` attraverso contenuto non fidato che
l'agente legge come istruzione, più una rete in uscita aperta o un segreto raggiungibile. È il
`confused deputy` con un nome nuovo. Replit insegna un'altra cosa: permessi di produzione in mano
a un agente, senza gate.

**Fai.** Per ogni incidente, una riga: quale controllo del passo 2 mancava, quale riga ASI del
passo 3 lo classifica, cosa l'avrebbe fermato.

### 5 · Identità e credenziali degli agenti — 3 sessioni

**Leggi.** MCP specification `2026-07-28`: *Security Best Practices* (il divieto testuale di
`token passthrough`, il confused deputy nei proxy) e *Authorization* (audience binding con
`RFC 8707 resource`). GitHub: *installation access token* di una GitHub App (scade in un'ora,
restringibile per repo e permessi) e *OpenID Connect* in Actions. I tre vendor sui segreti:
OpenAI, *Codex cloud environments*; GitHub, *Configure secrets and variables for Copilot cloud
agent* e *Give access to resources*; Anthropic, `claude-code-action/docs/setup.md`. OWASP
Agentic 2026, `ASI03 Identity & Privilege Abuse`. Come tassonomia, la lista della OWASP
*Non-Human Identities Top 10* 2025: non parla di agenti, ma dà i nomi.

**Capisci.** L'identità di un agente è una `non-human identity` separata da quella dell'umano,
con uno **short-lived, audience-bound token** scoped al repo, non una credenziale conservata. I
tre vendor divergono su un punto: **Codex rimuove i segreti prima della fase agente**; **Copilot e
claude-code-action li espongono come variabili d'ambiente** al processo dell'agente. Nessuna
norma, una scelta di design — ed è da `/proc/[pid]/environ` che Comment and Control li ha letti.
Il `token passthrough` è l'anti-pattern che rende un server MCP un confused deputy; l'audience
binding è il meccanismo che lo impedisce. E l'identità comprende *chi può innescare* l'agente: il
bypass di gennaio 2026 su claude-code-action (Flatt Security, CVSS 7.8) rubava l'OIDC token
facendosi passare per `[bot]` — il token era a vita breve, il trigger non era controllato.

**Fai.** La tabella **identità e segreti × vendor**, colonne verificabili in documentazione:
`identity of record` (chi firma il commit), `token type`, `scope`, `lifetime`, dove vivono i
segreti durante la fase agente, chi può innescare, chi approva il merge. Sotto, una riga per
anti-pattern: PAT personale passato all'agente, `CLAUDE_CODE_OAUTH_TOKEN` personale, token
passthrough.

La workload identity in senso pieno (SPIFFE/SPIRE, federazione OIDC verso il cloud) è materia di
un percorso sulla software supply chain: qui si nomina soltanto.

### 6 · Il lab: il workflow di riferimento, una volta, con un agente vero — 3–4 sessioni

**Prima.** L'identità, dal passo 5. Da terminale, le CLI di coding agent girano con il `gh auth`
e la chiave ssh della persona che le lancia: l'agente *è* quella persona, e «least privilege»
resta una parola. Serve una GitHub App con installation token in un clone dedicato, con i soli
permessi Contents / Pull requests / Issues. Poi verificare che il piano GitHub del repo supporti i
`rulesets`; se no, repo pubblico di prova.

**Fai.** Ruleset sul branch di integrazione: `require pull request`, `required status checks`,
`bypass list` vuota, l'identità dell'agente fuori da ogni bypass. Poi **tre PR reali**, una per
sessione, con un agente da terminale che usa l'identità del bot: un bump di dipendenza, un test
mancante, una documentazione fuori sincrono. Alla terza, un **reviewer da contesto pulito**: una
seconda sessione, dell'altro agente, che vede solo la PR. L'umano mergia.

**Esce.** Tre PR mergiate con la storia leggibile, e una nota: cosa ha funzionato, cosa è stato
rifiutato, quale controllo del passo 2 implementa ogni pezzo. È una **demo**, non una misura: tre
campioni non misurano niente.

### 7 · I protocolli, collocati — 1 sessione

**Leggi.** MCP: la pagina di overview e quella sull'`authorization` della specifica corrente, non
la specifica intera. A2A: l'overview e il comunicato della Linux Foundation sulla 1.0.

**Capisci.** `MCP` collega un modello ai suoi `tool`: verticale, client–server, ovunque, ed è il
vettore di due incidenti del passo 4. `A2A` collega agenti ad agenti: `agent card`, `task` con
ciclo di vita, `artifact`. Reale nelle piattaforme enterprise, **assente nella pipeline di
codice**: là l'handoff si chiama pull request.

**Fai.** Una tabella a due righe, MCP e A2A: cosa standardizza, cosa non copre, dove lo si è visto
nei passi 4 e 6.

### 8 · L'assessment, misurabile — 1–2 sessioni

**Fai.** Una pipeline vera — quella del lab, o una di produzione a cui si ha accesso in lettura —
contro la rubrica del passo 3: dieci righe ASI01–ASI10, ogni riga `presente / assente / ignoto`
con l'evidenza (il file, la regola, lo screenshot). Sotto, la checklist del passo 1: il secondo
agente, se c'è, è giustificato? Chiudi con il prossimo passo e il suo costo.

**Esce.** Un documento che qualcuno che non ha visto niente legge e sa dire cosa manca. Quella
pagina è la competenza.

---

## Il pattern che funziona in produzione

Un solo scrittore, un reviewer da contesto pulito, l'umano al merge.

```
 SESSIONE A · WRITER            pull request           SESSIONE B · REVIEWER          UMANO
 legge il task            ──▶   diff + CI verde   ──▶  vede solo la PR            ──▶  mergia
 lavora nel sandbox                                    non ha visto come è nata
 committa sul suo branch                               giudica forma, correttezza, sicurezza
 (il ragionamento resta qui)                           (può essere l'altro vendor)
```

Cognition 2026 lo chiama «coder + reviewer da contesto pulito»; Copilot cloud agent lo offre come
code review di default; GitHub Agent HQ lo fa con Claude, Codex e Copilot sullo stesso repo. La
scrittura resta a un solo thread.

---

## Il glossario giusto

Termini in inglese, spiegazione in italiano: quelli che si trovano nelle fonti e che un cliente
riconosce.

| termine | cosa significa | passi |
|---|---|---|
| `agent / workflow` | un *workflow* ha il percorso fissato dal codice; un *agent* decide da sé il passo successivo | 1 |
| `orchestrator–workers` | un agente scompone e delega, altri eseguono. L'unico pattern multi-agent usato stabilmente, quasi sempre con workers in sola lettura | 1 |
| `single writer` | uno solo scrive codice per volta; gli altri leggono, pianificano, rivedono | 1, 6 |
| `reviewer (clean context)` | un agente in una sessione separata che valuta una PR senza aver visto come è nata | 1, 6 |
| `handoff` | nei framework: un agente passa il controllo a un altro con un payload esplicito. Nel coding si chiama pull request | 1, 7 |
| `coding agent / cloud agent` | un agente che riceve un task, lavora in un ambiente isolato e produce una PR. «Cloud agent» è il nome GitHub da apr. 2026 | 2, 6 |
| `sandbox, egress allowlist` | l'ambiente isolato, e la lista dei soli host raggiungibili in uscita. L'anello che ha ceduto negli incidenti | 2, 4 |
| `non-human identity, GitHub App, installation token` | l'identità con cui l'agente parla al forge, separata da quella umana: un token a vita breve (un'ora), scoped al repo e ai permessi | 5, 6 |
| `short-lived, audience-bound token` | un token che scade presto ed è valido per un solo destinatario. Il linguaggio di GitHub e della spec MCP | 5 |
| `token passthrough` | un server MCP che accetta e rigira un token non emesso per lui. Vietato testualmente dalla spec | 5 |
| `secret scanning, push protection` | il forge blocca il push di un segreto prima che entri nella storia | 5, 6 |
| `ruleset, required status checks, bypass list` | le regole del forge su un branch: PR obbligatoria, check obbligatori, e la lista di chi può saltarle — che deve essere vuota | 6 |
| `CODEOWNERS` | il file che dice chi deve approvare cosa, per path | 6 |
| `indirect prompt injection` | istruzioni nascoste in contenuto che l'agente legge (issue, commento, risultato di un tool) ed esegue | 4 |
| `excessive agency` | OWASP: un agente con più funzionalità, permessi o autonomia di quanto il task richieda | 3, 4 |
| `confused deputy` | un componente privilegiato indotto ad agire per conto di uno meno privilegiato | 4 |
| `human-in-the-loop, approval gate` | una decisione che per disegno spetta a una persona: il merge, un'azione con conseguenze | 2, 6 |
| `MCP` | Model Context Protocol: come un modello accede a tool e dati. Verticale | 5, 7 |
| `A2A, agent card, task, artifact` | Agent2Agent: come un agente chiede a un altro di fare un task. Orizzontale | 7 |
| `audit trail, signed commits, provenance` | chi ha fatto cosa, verificabile dopo. Copilot cloud agent firma i commit da apr. 2026 | 4, 5, 8 |

---

## Le fonti

Di ogni fonte sono stati controllati esistenza, data e titolo corrente, e se il contenuto dice
qualcosa di più sfumato di come viene di solito citato (settembre 2026). La colonna a destra
riporta quel controllo.

| fonte | data | passo | nota |
|---|---|---|---|
| [Anthropic — Building multi-agent systems: When and how to use them](https://claude.com/blog/building-multi-agent-systems-when-and-how-to-use-them) | 23 gen. 2026 | 1 | posizione corrente; *Building effective agents* (dic. 2024) resta valido solo per il vocabolario |
| [Cognition — Multi-Agents: What's Actually Working](https://cognition.com/blog/multi-agents-working) | apr. 2026 | 1 | restringe [Don't build multi-agents](https://cognition.com/blog/dont-build-multi-agents) (giu. 2025), stesso autore: i principi «still hold» per i parallel-writer swarms |
| [GitHub — Risks and mitigations for Copilot cloud agent](https://docs.github.com/en/copilot/concepts/agents/cloud-agent/risks-and-mitigations) | 2026 | 2 | il prodotto si chiama *cloud agent* da apr. 2026, non più *coding agent* |
| [OpenAI — Codex: Agent approvals & security](https://developers.openai.com/codex/agent-approvals-security) | 2026 | 2 | sandbox offline di default con allowlist |
| [Anthropic — claude-code-action, security.md](https://github.com/anthropics/claude-code-action/blob/main/docs/security.md) | 2026 | 2 | |
| [OWASP — Top 10 for Agentic Applications 2026](https://genai.owasp.org/resource/owasp-top-10-for-agentic-applications-for-2026/) | dic. 2025 | 3 | edizione corrente per gli agenti; [LLM06 Excessive Agency](https://owasp.org/www-project-top-10-for-large-language-model-applications/2_0_vulns/LLM06_ExcessiveAgency.html) (2025) resta la definizione dei tre root cause |
| [Comment and Control](https://oddguan.com/blog/comment-and-control-prompt-injection-credential-theft-claude-code-gemini-cli-github-copilot/) | apr. 2026 | 4 | tre vendor, stesso workflow del passo 6, mitigazioni aggirate |
| [Invariant Labs — GitHub MCP server](https://invariantlabs.ai/blog/mcp-github-vulnerability) | mag. 2025 | 4 | «nessun fix ovvio, è architetturale» |
| [CamoLeak](https://www.legitsecurity.com/blog/camoleak-critical-github-copilot-vulnerability-leaks-private-source-code) | ott. 2025 | 4 | CVSS 9.6 |
| [Wiz — s1ngularity](https://www.wiz.io/blog/s1ngularitys-aftermath) | ago. 2025 | 4 | agenti installati in locale usati per cercare segreti: il punto di contatto con la software supply chain |
| [Replit / SaaStr](https://www.theregister.com/2025/07/21/replit_saastr_vibe_coding_incident/) | lug. 2025 | 4 | nessuna PR, nessuna CI: insegna excessive permissions, non il workflow |
| [MCP — Security Best Practices, spec 2026-07-28](https://modelcontextprotocol.io/specification/2026-07-28/basic/security_best_practices) | lug. 2026 | 5 | «Token passthrough is explicitly forbidden»; il confused deputy è specifico dei proxy con static client ID. Le versioni 2025 sono superate |
| [MCP — Authorization (audience binding, RFC 8707)](https://modelcontextprotocol.io/specification/2025-11-25/basic/authorization) | 2025–26 | 5 | `resource` obbligatorio e validazione dell'audience lato server |
| [GitHub — installation access token](https://docs.github.com/en/apps/creating-github-apps/authenticating-with-a-github-app/generating-an-installation-access-token-for-a-github-app) | 2026 | 5 | «will expire after 1 hour», restringibile per repositories e permissions |
| [GitHub Actions — OpenID Connect](https://docs.github.com/en/actions/concepts/security/openid-connect) | 2026 | 5 | token «only valid for a single job» |
| [GitHub — push protection](https://docs.github.com/en/code-security/secret-scanning/introduction/about-push-protection) | 2026 | 5 | controllo di uscita sul codice che l'agente committa; attivo di default sui repository pubblici |
| [OpenAI — Codex cloud environments](https://learn.chatgpt.com/docs/environments/cloud-environment) | 2026 | 5 | «secrets are removed before the agent phase starts»; le *environment variables* invece restano: la distinzione è il punto |
| [GitHub — secrets and variables for Copilot cloud agent](https://docs.github.com/en/copilot/how-tos/copilot-on-github/customize-copilot/customize-cloud-agent/configure-secrets-and-variables) | mag. 2026 | 5 | attenzione alla data: l'environment `copilot` è il meccanismo pre-maggio 2026; dall'8/05/2026 sono gli «Agents secrets and variables», esposti come env var al processo agente |
| [GitHub — Copilot cloud agent, access to resources](https://docs.github.com/en/copilot/tutorials/cloud-agent/give-access-to-resources) | 2026 | 5 | token «limited to the repository where it's running»; commit firmati come Copilot con co-author umano |
| [Anthropic — claude-code-action, setup.md](https://github.com/anthropics/claude-code-action/blob/main/docs/setup.md) | 2026 | 5 | GitHub App di default, OIDC del workflow scambiato per un installation token (`id-token: write`); da non confondere con l'OIDC verso Anthropic; `CLAUDE_CODE_OAUTH_TOKEN` personale resta possibile, sconsigliato a livello org |
| [Flatt Security — claude-code-action, bypass `[bot]`](https://flatt.tech/research/posts/poisoning-claude-code-one-github-issue-to-break-the-supply-chain/) | gen. 2026 | 5 | furto dell'OIDC token via attore `[bot]`, fix v1.0.94, CVSS 7.8 |
| [OWASP — Non-Human Identities Top 10 2025](https://owasp.org/www-project-non-human-identities-top-10/2025/top-10-2025/) | 2025 | 5 | edizione corrente; non parla di agenti: si usa come tassonomia |
| [GitHub — policy sui personal access token](https://docs.github.com/en/enterprise-cloud@latest/admin/enforcing-policies/enforcing-policies-for-your-enterprise/enforcing-policies-for-personal-access-tokens-in-your-enterprise) | 2026 | 5 | massimo 366 giorni; il PAT personale passato all'agente è un anti-pattern |
| [MCP — specifica corrente](https://modelcontextprotocol.io/specification/latest) | 2026 | 7 | si leggono overview e authorization, non la specifica intera |
| [Linux Foundation — A2A 1.0](https://www.linuxfoundation.org/press/a2a-protocol-surpasses-150-organizations-lands-in-major-cloud-platforms-and-sees-enterprise-production-use-in-first-year) | apr. 2026 | 7 | 150+ organizzazioni; reale nelle piattaforme enterprise, assente nei coding agent |
| [Linux Foundation — Agentic AI Foundation (MCP)](https://www.linuxfoundation.org/press/linux-foundation-announces-the-formation-of-the-agentic-ai-foundation) | 2025 | 7 | |
| [GitHub Agent HQ — multi-agent development](https://code.visualstudio.com/blogs/2026/02/05/multi-agent-development) | feb. 2026 | 1, 6 | Claude, Codex e Copilot sullo stesso repo, l'umano mergia |
| [Microsoft Agent Framework — handoff orchestration](https://devblogs.microsoft.com/agent-framework/a-tour-of-handoff-orchestration-pattern/) | 2026 | 1, 7 | l'handoff come routing fuori dal coding; il loop A→B→C→A come failure mode numero uno |

In questo campo le date contano: prima di usare il percorso, ricontrollare che le pagine dei vendor
siano ancora quelle.

---

## Le regole del percorso

1. **Un passo per sessione**, circa un'ora. Un passo è chiuso quando lo si sa raccontare in tre
   frasi senza rileggere gli appunti.
2. **Solo fonti primarie correnti.** Nessun testo superato o ritrattato entra fra le letture.
3. **Ogni passo produce un oggetto** — una checklist, una tabella, una rubrica — non un'opinione.
   Chi studia scrive; chi affianca dice cosa manca e non riscrive.
4. **Domande prima delle risposte.** Si prova a rispondere, poi si legge.
5. **Niente infrastruttura prima del primo oggetto.** Costruire strumenti per studiare è la forma
   più elegante di rinvio.

## Come si inizia

1. **Fai un fork** (o un clone) di questo repository e crea una cartella `notes/`. Ogni passo
   produce un file lì dentro: è l'unica cosa che alla fine conta.
2. **Apri il passo 1 e, prima di leggere, rispondi per iscritto alla sua domanda** in una riga:
   *a cosa serve, secondo te, un secondo agente?* Poi leggi le due fonti. Poi scrivi l'oggetto
   del passo — la checklist — e confronta con quello che avevi risposto. Questo ordine vale per
   ogni passo: prova, leggi, scrivi.
3. **Un passo per sessione**, un'ora. Chiudi il passo solo quando lo racconti in tre frasi a
   qualcuno senza rileggere gli appunti. Se non riesci, la sessione dopo si ricomincia dallo stesso
   passo: nessuno si salta perché «sembra ovvio».
4. **Se ti affianca un agente**, usalo come si usa un buon collega: fagli domande, non chiedergli
   riassunti; non lasciargli scrivere le note al posto tuo; prima di leggere una fonte, fagli
   verificare che esista ancora con quel titolo e quella data. In questo campo le pagine cambiano
   nome ogni pochi mesi.
5. **Per il lab del passo 6** ti serve un repository di prova separato, su cui puoi attivare i
   `rulesets` (se il piano non li supporta sul privato, usa un repo pubblico di prova) e una
   GitHub App dedicata all'agente. Preparali durante il passo 5, non prima.
6. **Tieni il log** in fondo al tuo `notes/README.md`:

   | passo | iniziato | chiuso | oggetto prodotto |
   |---|---|---|---|
   | 1 | | | |
   | 2 | | | |
   | 3 | | | |
   | 4 | | | |
   | 5 | | | |
   | 6 | | | |
   | 7 | | | |
   | 8 | | | |

Questo repository è una fotografia di settembre 2026 e non viene aggiornato: se una fonte è
cambiata, annotalo nel tuo fork.

## Cosa serve

Un terminale con `git` e `gh`, una CLI di coding agent (`claude`, `codex` o equivalente), un
repository di prova su cui si possano attivare i `rulesets`, e un browser per le fonti. Nessuna
piattaforma da comprare, nessun ambiente da montare. Python compare solo se un required check
del lab richiede uno script: in quel caso si legge e si modifica, non si scrive da zero.

---

## Licenza

Testo e pagina: [CC BY 4.0](LICENSE). Autore: Franco Geraci. Le fonti linkate restano dei
rispettivi autori.
