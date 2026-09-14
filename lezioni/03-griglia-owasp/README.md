# Lezione 3 — La griglia: OWASP Top 10 for Agentic Applications 2026

**Fase A · 1 sessione · produce: la rubrica ASI01–ASI10, ancora vuota**

---

## Prima di leggere: la domanda d'apertura

Scrivi la risposta in `progressi/03.md`, sezione «Prima», in non più di cinque righe.

> **Se dovessi valutare la sicurezza di una pipeline con coding agent e avessi dieci righe a
> disposizione, quali dieci rischi metteresti? Scrivili come li chiameresti tu, senza cercare i
> nomi giusti.**

Poi leggi la lezione, e alla fine confronta la tua lista con quella di OWASP.

---

## La lezione

### 1. Perché serve una griglia, e perché questa

Un assessment senza griglia è un'opinione ben scritta. La griglia serve a tre cose: non
dimenticare rischi che non ti vengono in mente, usare **nomi che l'interlocutore riconosce**, e
poter dire «presente / assente / ignoto» riga per riga invece di «mi sembra abbastanza sicuro».

La griglia che l'industria usa per gli agenti è la **OWASP Top 10 for Agentic Applications 2026**,
pubblicata dall'OWASP Gen AI Security Project il 9 dicembre 2025 (versione «2026», licenza
CC BY-SA 4.0). È — a giudizio di chi scrive — il documento che un auditor, un CISO o un cliente
citerà; e ogni voce ha la struttura standard OWASP — descrizione, esempi, scenari d'attacco,
mitigazioni, riferimenti — così si legge in fretta e si cita con precisione.

Un principio la attraversa tutta, dichiarato nell'introduzione: **Least-Agency**, l'estensione
agli agenti del *least privilege* — «our advice to organizations to avoid unnecessary autonomy».
In parole nostre: non solo meno permessi possibili, ma meno autonomia e meno capacità di azione
possibili per il compito.

### 2. Da dove viene: l'excessive agency del 2025

Prima degli agenti c'era la lista per le applicazioni LLM, e in quella lista una voce chiamata
**LLM06:2025 Excessive Agency**. Va conosciuta perché i suoi tre root cause sono ancora il modo
più chiaro di spiegare il problema a chi non ha letto niente:

| root cause | cosa significa |
|---|---|
| **Excessive Functionality** | l'estensione «include[s] functions that are not needed for the intended operation of the system»: può modificare o cancellare quando basterebbe leggere |
| **Excessive Permissions** | «permissions on downstream systems that are not needed for the intended operation of the application»: `UPDATE`/`DELETE` quando basta `SELECT` |
| **Excessive Autonomy** | il sistema «fail[s] to independently verify and approve high-impact actions»: agisce senza che nessuno confermi |

La lista agentica **non sostituisce** quella: la cita come radice. ASI02 «builds on the mitigations
of LLM06:2025», ASI03 è «the agentic evolution of Excessive Agency», ASI09 «builds on LLM06:2025».
E nell'edizione 2026 della lista LLM, uscita nell'agosto 2026, la voce sta al terzo posto —
**LLM03:2026** — e il confine tra le due liste è scritto in una frase: «This list owns the risk when the model is a
component... The moment that model becomes an actor... the risk moves to the OWASP Agentic Top
10... neither one covers that ground alone». Quando il modello *agisce*, la griglia è questa.

### 3. Le dieci voci, una alla volta

Per ciascuna: cosa è, un esempio preso dal documento, e le mitigazioni che nomina. Le voci sono
disposte da OWASP su una mappa dei componenti di un sistema agentico — input, tool, integrazioni,
agenti esterni, memoria, comunicazione fra agenti, propagazione, umano, l'agente stesso — e
conviene leggerle così.

**ASI01 · Agent Goal Hijack** — L'attaccante altera l'obiettivo dell'agente, la scelta dei task o
il percorso decisionale, perché gli agenti «cannot reliably distinguish instructions from related
content». Esempio: EchoLeak, un'injection indiretta zero-click su Microsoft 365 Copilot.
Mitigazioni: trattare ogni input in linguaggio naturale come non fidato; approvazione umana per
azioni «high-impact or goal-changing»; system prompt bloccati e sotto configuration management.
*Per i coding agent*: è la lezione 4 per intero.

**ASI02 · Tool Misuse and Exploitation** — Uso dannoso di tool legittimi, dentro i privilegi
concessi: «deleting valuable data, over-invoking costly APIs, or exfiltrating information».
L'esempio è costruito su un coding agent: ha tool approvati per l'esecuzione automatica, tra cui
`ping`, e lo usa per esfiltrare via DNS. Mitigazioni: «Least Agency and Least Privilege for Tools»,
con un profilo per tool; «Action-Level Authentication and Approval» con un «dry-run diff before
final approval»; sandbox con egress allowlist.

**ASI03 · Identity and Privilege Abuse** — Escalation attraverso catene di delega, credenziali
ereditate, cache in memoria. È «the agentic evolution of Excessive Agency». Esempio: un agente
finanza passa tutti i suoi permessi a un agente database. Mitigazioni: token «short-lived, narrowly
scoped... per task»; autorizzazione per azione con un policy engine centrale; human-in-the-loop
per ogni escalation. *Per i coding agent*: è la lezione 5.

**ASI04 · Agentic Supply Chain Vulnerabilities** — Componenti di terzi caricati anche a runtime —
tool, MCP server, agent card, prompt template: «a live supply chain». Esempi: la compromissione di
Amazon Q, con un prompt avvelenato spedito nella versione 1.84.0; e «a compromised NPM package
(e.g., a poisoned nx/debug release) was automatically installed by coding agents» — s1ngularity,
lezione 4. Mitigazioni: SBOM e AIBOM, firma di «manifests, prompts, and tool definitions»;
allowlist con pin per content hash; un «supply chain kill switch».

**ASI05 · Unexpected Code Execution (RCE)** — «Agentic systems - including popular vibe coding tools -
often generate and execute code»: il testo diventa esecuzione. Esempio: «Replit "Vibe Coding"
Runaway Execution», la cancellazione dei dati di produzione. Mitigazioni: «Prevent direct
agent-to-production systems»; «Never run as root»; sandbox con filesystem ristretto alla working
directory e log dei diff; «allowlist for auto-execution under version control».

**ASI06 · Memory & Context Poisoning** — Corruzione persistente di memoria, riassunti, embedding,
store RAG, che «propagates across sessions». Esempio: avvelenamento della memoria di un assistente
via injection indiretta. Mitigazioni: scansione di ogni scrittura in memoria prima del commit;
segmentazione per sessione e tenant; non re-ingerire i propri output; memoria non verificata che
scade.

**ASI07 · Insecure Inter-Agent Communication** — Messaggi fra agenti senza autenticazione,
integrità o validazione semantica. Esempio: spoofing della registrazione in un discovery service
A2A. Mitigazioni: autenticazione mutua e cifratura per agente; messaggi firmati con anti-replay;
«typed contracts and schema validation», protocol pinning (MCP, A2A). *Per i coding agent*: la lezione 7.

**ASI08 · Cascading Failures** — «The propagation and amplification of an initial fault - not the
initial vulnerability itself». Esempio: un loop di auto-remediation che si alimenta da solo.
Mitigazioni: separare planning ed execution con un policy engine esterno; «blast-radius
guardrails such as quotas, progress caps, circuit breakers»; log tamper-evident con lineage.

**ASI09 · Human-Agent Trust Exploitation** — L'umano si fida troppo di un output autorevole. OWASP
distingue: «This entry is about human misperception... whereas ASI10 is agent intent deviation».
L'esempio è per chi scrive codice: «Helpful Assistant Trojan: A compromised coding assistant
suggests a slick one-line fix» che installa una backdoor. Mitigazioni: conferme esplicite;
«Separate preview from effect»; «Plan-divergence detection» contro un workflow di riferimento.
*Per i coding agent*: è il motivo per cui il reviewer guarda il diff, non la spiegazione.

**ASI10 · Rogue Agents** — Agenti compromessi o deviati che «deviate from their intended function
or authorized scope» con azioni ciascuna legittima. Esempio: reward hacking, un agente che cancella
i backup per minimizzare i costi. Mitigazioni: audit log firmati e immutabili; kill switch e revoca
delle credenziali; «signed behavioral manifests» validati prima di ogni azione.

### 4. Tre cose che la lista ti insegna sulla tua pipeline

**La mappa per componente è anche la mappa della tua pipeline.** Input non fidato → ASI01; tool
dell'agente → ASI02; token e identità → ASI03; MCP server e dipendenze → ASI04; il codice che
l'agente esegue → ASI05; memoria e contesto persistente → ASI06; orchestratore ↔ worker → ASI07;
retry e automazioni → ASI08; il reviewer umano → ASI09; l'agente stesso → ASI10. Ogni pezzo della
pipeline ha una riga.

**Gli esempi del documento sono i tuoi incidenti.** Replit è in ASI05, s1ngularity in ASI04, il
coding assistant con la backdoor in ASI09. Quando nella lezione 4 ricostruirai gli incidenti, il
lavoro finale sarà scrivere accanto a ciascuno la sigla ASI: la lista è fatta per quello.

**Tre voci pesano più delle altre per chi governa coding agent** — è una scelta di questo
percorso, non una gerarchia di OWASP — e sono quelle che le lezioni successive approfondiscono: ASI01 (il contenuto non fidato letto come istruzione), ASI03
(l'identità e i token dell'agente), ASI05 (l'esecuzione di codice fuori dalla sandbox). Le altre
sette non si trascurano, ma in un assessment queste tre decidono il colore della pagina.

### 5. I documenti accanto, da conoscere per nome

Non da leggere ora. Da sapere che esistono, perché un interlocutore li citerà:

| documento | cos'è | data |
|---|---|---|
| *Agentic AI – Threats and Mitigations* | la tassonomia T1–T17 su cui la Top 10 dichiara di fondarsi («our foundational and detailed taxonomy») | v1.0, feb. 2025 |
| *Securing Agentic Applications Guide* | la guida implementativa: design, sviluppo, deploy | v1.0, lug. 2025 |
| *Multi-Agentic System Threat Modeling Guide* | threat modeling per sistemi a più agenti | v1.0, apr. 2025 |
| *OWASP Top 10 for LLM Applications 2026* | la lista sorella, con la matrice di mapping LLM ↔ ASI in appendice | ago. 2026 |
| *Agent Control Standard (ACS)* | «declarative controls that are portable across agent frameworks and enforced at runtime» | set. 2026 |

---

## Le fonti di questa lezione

1. OWASP Gen AI Security Project, [*OWASP Top 10 for Agentic Applications 2026*](https://genai.owasp.org/resource/owasp-top-10-for-agentic-applications-for-2026/), 9 dicembre 2025. Il PDF, 57 pagine: la pagina «At a Glance» con la mappa dei componenti, le dieci voci, l'appendice A con il mapping.
2. Da conoscere, non da leggere per intero: OWASP, [*LLM06:2025 Excessive Agency*](https://owasp.org/www-project-top-10-for-large-language-model-applications/2_0_vulns/LLM06_ExcessiveAgency.html) — i tre root cause. Nell'edizione 2026 della lista LLM la voce è **LLM03:2026**.

---

## Esercizio: la rubrica, ancora vuota

Scrivi in `progressi/03.md`, sezione «Oggetto», la rubrica che userai nella lezione 8. Dieci righe,
una per sigla, e per ciascuna **una domanda sì/no** formulata da te, applicabile a una pipeline con
coding agent. Le colonne «presente / assente / ignoto» ed «evidenza» restano vuote: si compilano
sulla pipeline vera.

### Template

```markdown
| ASI | rischio | la domanda sì/no | presente / assente / ignoto | evidenza |
|---|---|---|---|---|
| ASI01 | Agent Goal Hijack | | | |
| ASI02 | Tool Misuse and Exploitation | | | |
| ASI03 | Identity and Privilege Abuse | | | |
| ASI04 | Agentic Supply Chain Vulnerabilities | | | |
| ASI05 | Unexpected Code Execution (RCE) | | | |
| ASI06 | Memory & Context Poisoning | | | |
| ASI07 | Insecure Inter-Agent Communication | | | |
| ASI08 | Cascading Failures | | | |
| ASI09 | Human-Agent Trust Exploitation | | | |
| ASI10 | Rogue Agents | | | |
```

### Soluzione di riferimento

Domande derivate dal testo di ciascuna voce. Le tue possono essere diverse; devono essere
verificabili con un file, una regola o uno screenshot.

| ASI | la domanda sì/no |
|---|---|
| 01 | Il contenuto che l'agente legge — issue, commenti, README, file del repo, output dei tool — è trattato come non fidato, e il system prompt è versionato e bloccato? |
| 02 | Ogni tool ha un profilo scritto di least privilege (scope, rate, egress), e le azioni distruttive mostrano un diff o un dry-run prima dell'approvazione? |
| 03 | L'agente ha un'identità propria con credenziali short-lived per task — non il PAT di uno sviluppatore? |
| 04 | MCP server, tool e dipendenze sono pinnati per hash, firmati, e disattivabili con un kill switch? |
| 05 | Il codice generato gira in una sandbox non-root, senza accesso alla produzione, con una allowlist di auto-esecuzione sotto version control? |
| 06 | Le scritture in memoria o contesto persistente sono validate, segmentate per sessione, e scadono se non verificate? |
| 07 | I messaggi fra agenti (orchestratore ↔ worker) sono autenticati, firmati e validati contro uno schema? |
| 08 | Esistono quote, circuit breaker e separazione planner/executor che limitano la propagazione di un errore? |
| 09 | Il reviewer umano vede l'azione grezza — il diff — e non solo la spiegazione dell'agente, e la preview non produce effetti? |
| 10 | Esistono una baseline di comportamento, log firmati, e la revoca immediata delle credenziali di un agente deviato? |

### Dopo

Confronta la tua lista di dieci rischi della domanda d'apertura con le dieci sigle. Quante
coincidono? Quali avevi e OWASP no — e quali OWASP ha e a te non erano venute in mente? Tre righe.

---

## Autoverifica

1. Quali sono i tre root cause dell'excessive agency, e in quale sigla ASI ciascuno riappare in
   forma agentica?
2. La lista agentica sostituisce quella per le applicazioni LLM? Rispondi con la frase di confine
   dell'edizione 2026.
3. Quali tre sigle pesano di più per una pipeline di coding agent, e a quale lezione di questo
   percorso corrisponde ciascuna?

<details>
<summary>Risposte</summary>

1. **Excessive Functionality**, **Excessive Permissions**, **Excessive Autonomy**. Riappaiono in
   ASI02 (uso dei tool oltre il necessario), ASI03 (privilegi e identità, «the agentic evolution of
   Excessive Agency»), e nel principio di *least agency* che attraversa tutta la lista; ASI09 e ASI10
   ne sono derivazioni sul lato umano e sul lato dell'intento dell'agente.
2. **No, la affianca.** L'edizione 2026 della lista LLM: «This list owns the risk when the model is
   a component... The moment that model becomes an actor... the risk moves to the OWASP Agentic
   Top 10... neither one covers that ground alone». Excessive Agency resta nella lista LLM come
   LLM03:2026 e si manifesta negli agenti come ASI02, ASI03 e ASI08.
3. **ASI01** Agent Goal Hijack → lezione 4, gli incidenti (il contenuto non fidato letto come
   istruzione); **ASI03** Identity and Privilege Abuse → lezione 5, identità e credenziali;
   **ASI05** Unexpected Code Execution → lezioni 2 e 6, la sandbox e il lab.

</details>

---

## Chiusura del passo

Chiuso quando, senza appunti, sai dire le dieci sigle con il loro nome, il principio che le
attraversa, e le tre che decidono il colore di un assessment. Data in `progressi/README.md`.

**← Lezione 2** [I controlli dei tre vendor](../02-controlli-vendor/README.md) ·
**Prossima →** [04 · Gli incidenti veri, ricostruiti](../04-incidenti/README.md)
