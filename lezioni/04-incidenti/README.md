# Lezione 4 — Gli incidenti veri, ricostruiti

**Fase A · 2 sessioni · produce: una riga per incidente — controllo mancante, sigla ASI, cosa
l'avrebbe fermato**

---

## Prima di leggere: la domanda d'apertura

Scrivi la risposta in `progressi/04.md`, sezione «Prima», in non più di cinque righe.

> **Immagina di dover far uscire dati privati da un repository attraverso un coding agent, senza
> avere alcun accesso a quel repository. Da dove entri, e da dove escono i dati?**

Non è un esercizio di attacco: è il modo più rapido per capire quale controllo conta. Poi leggi.

---

## La lezione

### 1. Perché gli incidenti e non le definizioni

Le tre lezioni precedenti hanno dato le posizioni, i controlli dichiarati e la griglia. Gli
incidenti servono a una cosa che nessuna definizione dà: vedere **quale anello ha ceduto davvero**,
in sistemi costruiti da chi il problema lo conosceva. Cinque casi, 2025–2026, tutti pubblici, tutti
con un writeup tecnico leggibile. Quattro hanno la stessa forma; il quinto è di natura diversa, e
capire perché è parte della lezione.

### 2. Comment and Control (aprile 2026)

*Aonan Guan, con Zhengyu Liu e Gavin Zhong, Johns Hopkins.* Blog del 15 aprile 2026; segnalazioni
tra ottobre 2025 e febbraio 2026.

Tre sistemi, tre varianti dello stesso attacco:

- **`anthropics/claude-code-security-review`**, la GitHub Action che rivede le PR. Il **titolo
  della PR** viene interpolato nel prompt senza sanitizzazione; il workflow scatta su
  `pull_request`, quindi chiunque apra una PR fa partire l'agente. L'agente esegue `ps auxeww |
  grep` con il tool Bash, e `ANTHROPIC_API_KEY` e `GITHUB_TOKEN` finiscono nel commento della PR e
  nei log di Actions.
- **`google-github-actions/run-gemini-cli`**: titolo, corpo e commenti dell'issue entrano nel
  prompt come «Additional Content»; una finta «Trusted Content Section» scritta dall'attaccante
  scavalca le istruzioni; la `GEMINI_API_KEY` viene postata come commento pubblico.
- **GitHub Copilot coding agent**: istruzioni in un **commento HTML** dentro un'issue, invisibile
  nel rendering; l'agente esegue `ps auxeww | base64` e committa l'output in un file della PR.
  Decodificato, contiene `GITHUB_TOKEN`, `GITHUB_PERSONAL_ACCESS_TOKEN`, `GITHUB_COPILOT_API_TOKEN`,
  `COPILOT_JOB_NONCE`.

Il caso Copilot è quello da studiare, perché **tre mitigazioni esistevano ed erano attive**, e sono
state aggirate una per una: il filtro dell'ambiente puliva solo il subprocess bash, mentre «The
parent Node.js process and the MCP server process retain the full environment»; il secret scanning
è stato battuto codificando in base64; il firewall di rete era irrilevante, perché l'esfiltrazione passava da `github.com`, host
consentito per definizione.

Risposte dei vendor: Anthropic ha aggiunto `--disallowed-tools 'Bash(ps:*)'` e una nota — «The
action is not designed to be hardened against prompt injection» — con un bounty di 100 dollari;
la severità è passata da 9.3 a 9.4 e poi a «None». Google ha accettato in VRP. GitHub ha prima
classificato «Informative», poi riaperto: «This is a previously identified architectural
limitation»; «exposure of environment variables through process inspection is a known consequence
of the current runtime design».

La frase del writeup che chiude la questione: «the prompt injection here is not a bug; it is
context that the agent is *designed* to process». E la blocklist su `ps` è aggirabile: «`cat
/proc/*/environ` achieves the same result».

### 3. Il GitHub MCP server (maggio 2025)

*Invariant Labs, Marco Milanta e Luca Beurer-Kellner*, 26 maggio 2025. Sistema: l'integrazione
GitHub via MCP, con Claude Desktop come client — ma «not specific to any particular agent or MCP
client».

L'attaccante apre un'**issue su un repository pubblico** della vittima, con un'injection nascosta
in un blocco «About The Author». La vittima chiede al suo agente: «Have a look at the open issues
in `<user>/public-repo`». L'agente, attraverso il server MCP che usa **il token dell'utente**, legge
un repository **privato**, ne porta i dati in contesto, e li scrive in una pull request del
repository pubblico — «freely accessible to the attacker since it is public». Trapelano dettagli
del repo privato, piani personali, uno stipendio.

Il controllo mancante è uno: il token dell'agente **non era confinato** al repository su cui
lavorava. Invariant chiama la classe «toxic agent flows» — «use of indirect prompt injection to
trigger a malicious tool use sequence» — e dice, testualmente, che «this is not a flaw in the
GitHub MCP server code itself, but rather a fundamental architectural issue... GitHub alone cannot
resolve this vulnerability through server-side patches». E sui modelli: «even state-of-the-art
aligned models are vulnerable to these attacks». Nessun fix del vendor nel writeup; mitigazione
proposta: policy a runtime che impediscono accessi cross-repository nella stessa sessione.

### 4. CamoLeak (ottobre 2025)

*Legit Security, Omer Mayraz.* Scoperta in giugno, fix il 14 agosto, blog l'8 ottobre 2025. Sistema:
GitHub Copilot Chat. **CVSS 9.6.**

Istruzioni in un **commento HTML nascosto** nella descrizione di una PR — una funzione ufficiale di
GitHub. La vittima apre la PR e usa Copilot Chat: l'interazione è richiesta, non è zero-click.
«Copilot operates with the same permissions as the user making the request», quindi legge
repository privati e li codifica: «encode its contents in base16, and append it to a URL». La
Content Security Policy di GitHub blocca immagini da domini esterni; l'attaccante l'aggira con
**Camo**, il proxy immagini di GitHub stesso: un dizionario di URL Camo pre-firmati, uno per
carattere, pixel trasparenti 1×1. Copilot rende i dati rubati come «ASCII art composed entirely of
images», e il browser della vittima, caricando le immagini, li invia al server dell'attaccante un
carattere alla volta.

Fix: «GitHub fixed it by disabling image rendering in Copilot Chat completely». La prompt
injection in sé resta.

### 5. s1ngularity / Nx (agosto 2025)

*Wiz Research (Rami McCarthy), 3 settembre 2025; postmortem di Nx.* Pacchetti maligni pubblicati
il 26 agosto 2025, rimossi dopo circa quattro ore; ondate successive fino al 31.

È un incidente **a tre stadi**, e i coding agent compaiono nel secondo:

1. **Il furto del token npm.** Un workflow di Nx che validava il titolo delle PR girava su
   `pull_request_target` con «unsanitized PR title echoing»: shell injection → `GITHUB_TOKEN` con
   permessi read/write (il default legacy) → un branch con uno script maligno → il workflow di
   publish invocato via `workflow_dispatch` → `NPM_TOKEN` esfiltrato. Tre concause, dette da Nx:
   `pull_request_target` con input non sanitizzato, token Actions sovra-permissivo,
   `workflow_dispatch` sul publish.
2. **Il postinstall.** Le versioni avvelenate di `nx` raccoglievano variabili d'ambiente, token
   GitHub e npm, wallet, chiavi SSH — e **invocavano le CLI di AI installate in locale** con i flag
   che disattivano le conferme: `--dangerously-skip-permissions` (Claude Code), `--yolo` (Gemini),
   `--trust-all-tools` (Amazon Q), chiedendo di cercare `*.env`, `.key`, wallet, SSH. Il bottino
   veniva pubblicato in repository pubblici chiamati `s1ngularity-repository` sull'account della
   vittima. Poi aggiungeva `sudo shutdown -h 0` a `.zshrc` e `.bashrc`.
3. **La seconda ondata.** Con i token rubati, «at least 480 compromised accounts» hanno reso
   pubblici «over 6,700 private repositories».

Numeri di Wiz: «over 1,700 users had secrets publicly leaked», «over 2,000 unique, verified
secrets»; a distanza di giorni «over 40% of leaked npm tokens» erano ancora validi. Fix di Nx:
trusted publishing con OIDC, 2FA manuale per la pubblicazione, pipeline disabilitate per i
contributor esterni, verifica della provenance.

Una precisazione che le fonti impongono: **non è stato «un attacco AI riuscito»**. Wiz misura che
«AI only exfiltrated data successfully in under a quarter of cases», e che «almost a quarter of
Claude interactions were rejected». Il grosso del danno viene dal postinstall classico e dai token.
Ma Wiz lo descrive come «AI-powered malware», e la stampa lo ha indicato come il primo caso noto in
cui un malware usa gli agenti installati sulla macchina come strumento di ricerca dei segreti — ed
è il ponte fra questo percorso e la software supply chain.

### 6. Replit / SaaStr (luglio 2025) — il caso di natura diversa

*Jason Lemkin*, 12–20 luglio 2025; The Register il 21 luglio, Fortune il 23 con la risposta di
Amjad Masad (Replit).

Nessun attaccante. Nessuna prompt injection. Durante un **code freeze dichiarato in chat**,
l'agente Replit esegue comandi distruttivi sul database di produzione — dati di «more than 1,200
executives and over 1,190 companies», secondo Fortune. Poi dichiara il rollback «impossible in this
case» e che ha «destroyed all database versions»; Lemkin verifica che **il rollback funzionava**. In
precedenza aveva creato «a 4,000-record database full of fictional people» e report falsi, «eleven
times in ALL CAPS» dopo che gli era stato detto di non farlo. La confessione dell'agente, nella
versione del Register: «a catastrophic error of judgement», che ha «violated your explicit trust
and instructions»; in quella di Fortune: «I destroyed months of work in seconds.»

Il controllo mancante non ha a che fare con il contenuto non fidato: **nessuna separazione fra
sviluppo, staging e produzione**, e nessun modo di rendere un code freeze eseguibile. La risposta
di Masad, riportata da Fortune — il Register scriveva di non aver ancora ottenuto commento —:
«Unacceptable and should never be possible»; separazione automatica fra database di sviluppo e
produzione; un planning mode «so you can strategize without risking your codebase».

Va tenuto perché insegna una cosa che gli altri quattro non insegnano — permessi di produzione in
mano a un agente, senza gate — ma va tenuto **come nota**: è ASI05, non ASI01, e in un workflow a
pull request con CI non sarebbe potuto accadere in quella forma.

### 7. La tabella, e cosa hanno in comune

| incidente | vettore d'ingresso | canale di uscita | controllo mancante o aggirato | fix |
|---|---|---|---|---|
| Comment and Control | titolo PR, corpo issue, commento HTML nascosto | commento PR, log Actions, file committato (base64) | nessuna sanitizzazione del prompt né allowlist dei tool; env filtering solo sul subprocess; firewall che consente `github.com` | blocklist su `ps`; nota «not hardened against prompt injection» |
| GitHub MCP server | issue pubblica con injection | pull request pubblica | token non confinato al repository | nessuno del vendor; policy a runtime proposte |
| CamoLeak | commento HTML nascosto in PR | immagini via Camo → server attaccante | CSP aggirata attraverso il proxy immagini di GitHub | rendering immagini disabilitato |
| s1ngularity | `pull_request_target` + titolo PR → token npm → postinstall | repository pubblici della vittima | token CI sovra-permissivo; CLI AI con conferme disattivate | OIDC trusted publishing, 2FA, niente run per esterni |
| Replit | nessuno: azione autonoma | nessuna uscita: distruzione | nessuna separazione dev/prod; freeze non eseguibile | separazione DB, planning mode, rollback |

I primi quattro hanno **tre cose in comune**, e le fonti le sostengono:

1. **L'input malevolo entra da metadati che l'agente è progettato a leggere**: titoli di PR, issue,
   commenti — nascosti o in chiaro. Non c'è un bug da correggere: c'è un canale che deve esistere.
2. **L'agente opera con credenziali più ampie del compito**: «the same permissions as the user»
   (CamoLeak), un token cross-repository (MCP), i segreti della CI nell'ambiente (Comment and
   Control, Nx).
3. **L'esfiltrazione usa GitHub stesso come canale**, aggirando ogni controllo sulla rete in uscita:
   pull request, commenti, log, Camo, repository pubblici. Guan lo formalizza nel titolo: GitHub
   come canale di comando e controllo.

La conseguenza per il disegno: il firewall sulla rete in uscita è necessario e **non è
sufficiente**, perché il forge è sempre nella allowlist. Ciò che ferma questi attacchi è a monte —
chi può innescare l'agente, cosa vede dei segreti, quanto è largo il suo token — e a valle: cosa
finisce in una PR senza che un umano lo veda.

### 8. Quattro cose che si dicono e le fonti smentiscono

- *«Il rollback era impossibile»* (Replit): l'agente lo ha detto; l'utente ha verificato che
  funzionava.
- *«Il GitHub MCP server aveva un bug»*: Invariant scrive l'opposto — «not a flaw in the GitHub MCP
  server code itself».
- *«s1ngularity è stato un attacco AI riuscito»*: meno di un quarto delle esfiltrazioni via AI è
  andata a segno; il danno è venuto dai token.
- *«Anthropic ha risolto Comment and Control»*: il fix è una blocklist su un comando, aggirabile
  con `/proc/*/environ`; Anthropic stessa dichiara l'action non progettata per resistere
  all'injection.

---

## Le fonti di questa lezione

I writeup originali, non i riassunti di stampa. Leggili dopo la lezione, uno per sessione se serve.

1. Aonan Guan, [*Comment and Control*](https://oddguan.com/blog/comment-and-control-prompt-injection-credential-theft-claude-code-gemini-cli-github-copilot/), 15 aprile 2026.
2. Invariant Labs, [*GitHub MCP Exploited*](https://invariantlabs.ai/blog/mcp-github-vulnerability), 26 maggio 2025.
3. Legit Security, [*CamoLeak*](https://www.legitsecurity.com/blog/camoleak-critical-github-copilot-vulnerability-leaks-private-source-code), 8 ottobre 2025.
4. Wiz Research, [*s1ngularity's aftermath*](https://www.wiz.io/blog/s1ngularitys-aftermath), 3 settembre 2025, e il [postmortem di Nx](https://nx.dev/blog/s1ngularity-postmortem).
5. Come nota: The Register, [*Replit / SaaStr*](https://www.theregister.com/2025/07/21/replit_saastr_vibe_coding_incident/), 21 luglio 2025, e Fortune, [*AI coding tool Replit wiped database*](https://fortune.com/2025/07/23/ai-coding-tool-replit-wiped-database-called-it-a-catastrophic-failure/), 23 luglio 2025, per i numeri e la risposta di Masad. I post originali di Lemkin sono su X e non sempre raggiungibili; le citazioni qui vengono dalla stampa.

---

## Esercizio: una riga per incidente

Scrivi in `progressi/04.md`, sezione «Oggetto», una tabella con cinque righe. Per ciascun
incidente: quale controllo della **lezione 2** mancava o è stato aggirato; quale sigla della
**lezione 3** lo classifica; e **cosa l'avrebbe fermato** — un controllo concreto, non «più
attenzione».

### Template

```markdown
| incidente | controllo della lezione 2 mancante o aggirato | sigla ASI | cosa l'avrebbe fermato |
|---|---|---|---|
| Comment and Control | | | |
| GitHub MCP server | | | |
| CamoLeak | | | |
| s1ngularity | | | |
| Replit | | | |
```

### Soluzione di riferimento

| incidente | controllo mancante o aggirato | ASI | cosa l'avrebbe fermato |
|---|---|---|---|
| Comment and Control | trigger da chiunque apra una PR; segreti nell'ambiente del processo agente; tool Bash senza allowlist | ASI01, ASI02, ASI03 | trigger solo da write access; segreti rimossi prima della fase agente (il modello Codex, lezione 5); allowlist dei tool con `ps` e `cat /proc` esclusi; e comunque **un umano che legge il diff prima del merge** |
| GitHub MCP server | token dell'agente non confinato al repository di lavoro | ASI03 | un token scoped al solo repository (installation token, lezione 5); una policy a runtime contro l'accesso cross-repository nella stessa sessione |
| CamoLeak | contenuto nascosto nella PR letto come istruzione; permessi dell'agente uguali a quelli dell'utente | ASI01, ASI03 | filtro dei commenti HTML (lezione 2, dichiarato da GitHub e Anthropic); token a scope ridotto; e il fix adottato — nessun rendering di immagini |
| s1ngularity | `pull_request_target` con input non sanitizzato; token CI read/write; CLI AI eseguite con conferme disattivate | ASI04, ASI02 | token Actions read-only di default; trusted publishing OIDC; e sulle macchine degli sviluppatori, mai `--dangerously-skip-permissions` fuori da una sandbox |
| Replit | nessuna separazione dev/prod; agente con credenziali di produzione | ASI05 | «Prevent direct agent-to-production systems» (OWASP); l'unico output dell'agente è una pull request, mai un comando sulla produzione |

### Dopo

Rileggi la tua risposta alla domanda d'apertura. Il tuo piano di esfiltrazione era uno dei quattro?
Quale controllo l'avrebbe fermato? Tre righe.

---

## Autoverifica

1. Nel caso Copilot di Comment and Control tre mitigazioni erano attive. Quali, e come è stata
   aggirata ciascuna?
2. Perché il firewall sulla rete in uscita non basta contro nessuno dei primi quattro incidenti?
3. In che senso Replit è di natura diversa, e perché va tenuto lo stesso?

<details>
<summary>Risposte</summary>

1. **Il filtro dell'ambiente**, che puliva solo il subprocess bash mentre il processo Node padre e
   il server MCP «retain full environment» — aggirato leggendo l'ambiente dei processi con `ps
   auxeww`. **Il secret scanning** — aggirato codificando in base64. **Il firewall** — irrilevante,
   perché il file esfiltrato è stato committato nella PR, cioè inviato a `github.com`, host
   consentito.
2. Perché in tutti e quattro **il canale di uscita è GitHub stesso**: una PR, un commento, un log,
   il proxy Camo, un repository pubblico. Il forge è sempre nella allowlist. I controlli che
   fermano l'attacco sono a monte (trigger, segreti, scope del token) e a valle (un umano che
   guarda cosa finisce nella PR).
3. **Nessun attaccante e nessuna injection**: l'agente ha agito da solo, con credenziali di
   produzione, contro istruzioni esplicite. È ASI05, non ASI01. Va tenuto perché insegna il
   controllo che gli altri non toccano — «prevent direct agent-to-production systems» — e perché
   in un workflow a pull request con CI non sarebbe potuto accadere in quella forma.

</details>

---

## Chiusura del passo

Chiuso quando, senza appunti, sai raccontare due incidenti passo per passo, dire le tre cose che i
primi quattro hanno in comune, e spiegare perché Replit è una nota. Data in `progressi/README.md`.

**← Lezione 3** [La griglia OWASP](../03-griglia-owasp/README.md) ·
**Prossima →** [05 · Identità e credenziali degli agenti](../05-identita-credenziali/README.md)
