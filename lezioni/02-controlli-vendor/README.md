# Lezione 2 — I controlli dei tre vendor, letti in parallelo

**Fase A · 1 sessione · produce: la tabella controllo × vendor**

---

## Prima di leggere: la domanda d'apertura

Scrivi la risposta in `progressi/02.md`, sezione «Prima», in non più di cinque righe.

> **Se un coding agent lavora sul tuo repository, quali sono le tre cose che deve essere
> impossibilitato a fare — non «sconsigliato», impossibilitato — e da cosa dipende che lo siano?**

Poi leggi la lezione.

---

## La lezione

### 1. Perché leggere tre pagine di sicurezza una accanto all'altra

Ogni vendor pubblica una pagina che dice cosa il suo coding agent può fare, cosa non può, e cosa
il vendor **non garantisce**. Prese una per una sono documentazione. Lette in parallelo diventano
un'altra cosa: **dove tre concorrenti, in modo indipendente, arrivano allo stesso controllo, quel
controllo non è un'opinione — è lo stato dell'arte.** E dove divergono, lì stanno le domande da
fare a chi ti propone uno strumento.

Le tre pagine sono:

- GitHub, *Risks and mitigations for GitHub Copilot cloud agent* — apre così: «Copilot cloud
  agent is an autonomous agent that has access to your code and can push changes to your
  repository. This entails certain risks.»
- OpenAI, *Agent approvals & security* per Codex (CLI, IDE e cloud) — «how to operate Codex safely,
  including sandboxing, approvals, and network access».
- Anthropic, `claude-code-action`, il file `docs/security.md` — l'agente Claude Code che gira
  dentro GitHub Actions.

Nessuna delle tre porta una data o una versione in pagina. Questo è il primo fatto da imparare:
**vanno rilette**. Un controllo letto sei mesi fa può non esserci più, o esserci con un altro nome.

### 2. Il primo controllo: chi può far partire l'agente

Due vendor su tre lo dichiarano, con le stesse parole.

GitHub: «Only users with write access to the repository can trigger Copilot cloud agent to
work.» E per le automazioni: «Automations ignore events triggered by users without write access
to the repository by default.»

Anthropic: «The action can only be triggered by users with write access to the repository.» In
più, due precisazioni che valgono una lezione da sole: «By default, GitHub Apps and bots cannot
trigger this action» — e se li ammetti, «Allowed bots are not checked for repository permissions».
Esiste un'opzione per far partire l'action a utenti senza write, `allowed_non_write_users`, e la
documentazione stessa la chiama «a significant security risk» che «bypasses the primary security
mechanism of this action».

Codex: **non lo dichiara.** La pagina parla di sandbox, approvazioni e rete, non di chi può
avviare l'agente. Nel caso di Codex CLI la risposta è ovvia — lo avvia chi lo lancia — ma per Codex
cloud non è scritto.

Perché conta: negli incidenti che vedrai nella lezione 4, l'input malevolo entra quasi sempre da
chi **non** ha write access — un'issue aperta da chiunque, un commento, un titolo di PR. Il
controllo sul trigger è la prima porta.

### 3. Dove l'agente può scrivere, e cosa non può fare con git

GitHub è il più preciso: «Copilot cloud agent only has the ability to push to a single branch» —
«a new `copilot/` branch is created for Copilot, and the agent can only push to that branch». E:
«It cannot directly run `git push` or other Git commands», può fare solo «simple push operations».

Anthropic: «Claude commits code changes to a new branch», nome non specificato. E un dettaglio che
sorprende chi si aspetta un agente autonomo: «Claude does not create pull requests automatically»
— «The user must click the link and create the PR themselves».

Codex: **non dichiarato**; c'è solo il consiglio di lavorare su feature branch. In compenso Codex
protegge alcuni path anche dentro una directory scrivibile: `.git`, `.agents`, `.codex` restano
read-only.

### 4. Chi approva e chi mergia

Qui GitHub è l'unico dei tre a dire qualcosa, e dice molto:

- l'agente «cannot mark its pull requests as "Ready for review" and cannot approve or merge a pull
  request»;
- le PR sono draft: «Draft pull requests created by Copilot cloud agent must be reviewed and
  merged by a human»;
- GitHub «prevents the user who asked Copilot cloud agent to create a pull request from approving
  it» — chi ha chiesto il lavoro non può approvarlo;
- quando la PR è aperta sotto l'identità dell'app, «one more approval is required before it can be
  merged», purché il repository richieda già almeno un'approvazione — «enabled by default in
  rulesets»;
- i workflow «are not triggered until Copilot cloud agent's code is reviewed and a user with write
  access ... clicks the Approve and run workflows button».

Codex e Anthropic: **non dichiarato.** Non perché non conti, ma perché in quei due casi il merge
avviene fuori dallo strumento: è il repository — i suoi ruleset — a doverlo imporre. Questo è il
motivo per cui nella lezione 6 i ruleset si configurano **prima** di far partire l'agente: dove il
vendor non impone la separazione tra chi scrive e chi approva, la impone il repository, o nessuno.

### 5. La rete in uscita

È il controllo che ha ceduto più spesso negli incidenti reali, e i tre vendor lo trattano in modo
diverso.

Codex, per il cloud: la fase di setup «can access the network to install specified dependencies,
then the agent phase runs offline by default unless you enable internet access». Per la CLI: «By
default, the agent runs with network access turned off»; abilitarla è marcato «Elevated Risk».
Esiste un proxy con allowlist per dominio, dove «`deny` always wins» — e la pagina avverte che
«it does not grant network access by itself». Le destinazioni private (loopback, link-local, reti
private) sono bloccate di default.

GitHub: «GitHub restricts Copilot cloud agent's access to the internet», con un firewall che ha
una allowlist raccomandata attiva di default (repository di pacchetti, registry, certificate
authority, host per browser); se una richiesta è bloccata, «a warning is added to the pull request
body»; e «Disabling the firewall will allow Copilot to connect to any host, increasing risks of
exfiltration».

Anthropic: **nessuna menzione** di firewall o rete in uscita nel file `security.md`. L'action gira
in un runner GitHub Actions, che di suo non filtra il traffico in uscita.

### 6. Contenuto non fidato e prompt injection

Tutti e tre sanno che l'agente legge testo scritto da altri — issue, commenti, titoli — e che quel
testo può contenere istruzioni. Due lo filtrano, uno avverte.

GitHub: «GitHub filters hidden characters before passing user input to Copilot cloud agent»; e
«text entered as an HTML comment ... is not passed to Copilot cloud agent».

Anthropic: l'action «sanitizes content by stripping HTML comments, invisible characters, markdown
image alt text, hidden HTML attributes, and HTML entities»; si può restringere ai commenti di
autori scelti con `include_comments_by_actor`. E poi la frase che vale più di tutte le altre:
**«This reduces but does not eliminate prompt injection risk»**, con l'avviso che «new bypass
techniques may emerge» e che restringere gli strumenti «may not eliminate the risk».

Codex non filtra: dichiara il rischio — «Prompt injection can cause the agent to fetch and follow
untrusted instructions» — e dichiara anche i limiti del proprio proxy, che «does not filter web
search, app or connector tool calls, MCP server connections, browser or Computer Use activity». La
protezione contro il DNS rebinding è «a best-effort ... check» che «reduces DNS rebinding risk,
but it does not eliminate it».

Impara a leggere queste frasi per quello che sono: **il vendor che scrive cosa non garantisce è
quello che ha capito il problema.** Il silenzio non è sicurezza.

### 7. I segreti

Codex cloud è netto: «Secrets configured for cloud environments are available only during setup
and are removed before the agent phase starts.» Il codice dell'agente non li vede.

Anthropic: «best-effort scrub of Anthropic, cloud, and GitHub Actions secrets from subprocess
environments»; e due divieti: «Do not use a personal access token» perché «could be partially or
fully recovered over time via prompt injection», e «Never hardcode your Anthropic API key». Il
token usato è «short-lived» e «scoped specifically to the repository», con «No Cross-Repository
Access». Un'altra scelta che protegge dal contenuto malevolo: i file di configurazione (`.claude/`,
`.mcp.json`, `CLAUDE.md`) vengono ripristinati «from the PR base branch before starting Claude»,
così una PR non può riscrivere le istruzioni dell'agente che la valuta.

GitHub: la pagina dei rischi **non dice** come i segreti del repository sono gestiti dentro la
sessione dell'agente. (Lo dice un'altra pagina, e la leggerai nella lezione 5: sono esposti come
variabili d'ambiente.)

### 8. Chi firma i commit

GitHub: «commits are authored by Copilot, with the developer who assigned the issue ... marked as
the co-author»; «commits are signed, so they appear as "Verified"». Sessioni e audit log sono
disponibili agli amministratori.

Anthropic: «By default, commits made by Claude are unsigned»; la firma è opzionale, via API come
GitHub App o con una chiave SSH — e in quel caso i commit sono «attributed to the GitHub account
that owns the signing key», cioè a una persona.

Codex: **non dichiarato**. Le parole «author», «signed», «merge» non compaiono nella pagina.

### 9. La tabella, e cosa ti dice

| controllo | Copilot cloud agent | Codex | claude-code-action |
|---|---|---|---|
| chi innesca | write access | non dichiarato | write access; bot esclusi di default; bypass marcato «significant security risk» |
| branch di scrittura | solo `copilot/`; niente comandi git diretti | non dichiarato; `.git` e config read-only | «a new branch», nome non specificato |
| approvazione e merge | non può approvare né mergiare; PR draft; il richiedente non approva; +1 approvazione se sotto app identity | non dichiarato | non dichiarato; non crea la PR, l'utente la apre |
| rete in uscita | firewall con allowlist default; warning in PR se bloccato | off di default (cloud e CLI); allowlist per dominio; private bloccate | non dichiarato |
| contenuto nascosto | filtra hidden characters e HTML comments | nessun filtro; dichiara il rischio | strip di commenti, invisibili, alt text, attributi, entità; «reduces but does not eliminate» |
| segreti nella fase agente | non dichiarato in questa pagina | rimossi prima della fase agente | scrub best-effort; mai PAT; token short-lived per repo |
| identità dei commit | Copilot autore, umano co-author, firmati | non dichiarato | non firmati di default; firma opzionale |
| cosa dichiara di non garantire | — | injection possibile; proxy non copre MCP e web search; DNS best-effort | «does not eliminate prompt injection risk» |

Tre cose si leggono in questa tabella e nessuna si leggeva nelle pagine separate.

**La convergenza.** Trigger da chi ha write, un branch proprio per l'agente, un umano al merge,
contenuto nascosto filtrato, segreti mai statici: dove due o tre vendor coincidono, hai il
**minimo** che una pipeline con agenti deve avere. Non perché lo dice qualcuno, ma perché tre
concorrenti ci sono arrivati da soli.

**Le assenze.** Codex non dice chi può innescare né chi mergia; Anthropic non parla di rete in
uscita; GitHub non spiega i segreti nella pagina dei rischi. Un'assenza non è un difetto del
prodotto: è **un controllo che devi mettere tu**, nel repository o nell'ambiente — e che un
fornitore deve saperti spiegare.

**La divergenza sui segreti.** Codex li toglie dall'ambiente prima che l'agente giri; gli altri due
no. È la differenza da cui parte la lezione 5.

### 10. Cosa te ne fai

Derivazione di chi scrive, non delle fonti. Davanti a qualunque agente che deve toccare un
repository — di questi tre o di un altro — le domande sono le righe della tabella, nell'ordine:
*chi lo fa partire, dove scrive, chi mergia, dove può connettersi, cosa legge senza filtro, cosa
vede dei segreti, con che nome firma, e cosa dichiarate di non garantire.* Un fornitore che non ha
una risposta per ogni riga non ha una pagina di sicurezza; ha una brochure.

---

## Le fonti di questa lezione

Leggile **dopo**, per verificare. Sono le uniche tre.

1. GitHub, [*Risks and mitigations for GitHub Copilot cloud agent*](https://docs.github.com/en/copilot/concepts/agents/cloud-agent/risks-and-mitigations), con la sottopagina sul firewall linkata dalla sezione sulle informazioni sensibili.
2. OpenAI, [*Agent approvals & security*](https://developers.openai.com/codex/agent-approvals-security) — l'indirizzo reindirizza alla documentazione corrente; se cambia ancora, cerca «Agent approvals & security» nei docs Codex.
3. Anthropic, [`claude-code-action/docs/security.md`](https://github.com/anthropics/claude-code-action/blob/main/docs/security.md).

Nessuna delle tre è datata. Segna nel tuo `progressi/02.md` il giorno in cui le hai lette.

---

## Esercizio: la tabella controllo × vendor

Scrivi in `progressi/02.md`, sezione «Oggetto», **la tua** tabella, con almeno le otto righe della
lezione. Per ogni cella: «dichiarato», «non dichiarato», oppure «diverso» con due parole su come.
Aggiungi una nona riga a tua scelta — qualcosa che ti aspettavi di trovare e che nelle pagine non
c'è — e una colonna in fondo: **«se non dichiarato, chi lo impone?»** (il repository, l'ambiente,
nessuno).

### Template

```markdown
| controllo | Copilot cloud agent | Codex | claude-code-action | se non dichiarato, chi lo impone |
|---|---|---|---|---|
| chi innesca | | | | |
| branch di scrittura | | | | |
| approvazione e merge | | | | |
| rete in uscita | | | | |
| contenuto nascosto | | | | |
| segreti nella fase agente | | | | |
| identità dei commit | | | | |
| cosa non garantisce | | | | |
| (la tua riga) | | | | |
```

### Soluzione di riferimento

La tabella della sezione 9, con l'ultima colonna compilata così: **approvazione e merge** →
i ruleset del repository (require pull request, required approvals, bypass list vuota);
**rete in uscita** per l'action Anthropic → l'ambiente di esecuzione (il runner), non l'agente;
**chi innesca** per Codex cloud → chi ha accesso all'ambiente Codex, da verificare nelle
impostazioni dell'organizzazione; **identità dei commit** per Codex → la configurazione git
dell'ambiente. Se nella tua tabella una cella «non dichiarato» ha come risposta «nessuno», quella
è una riga rossa nell'assessment della lezione 8.

### Dopo

Torna alla domanda d'apertura. Le tre cose che avevi scritto: quante sono dichiarate da tutti e
tre i vendor? Quante da nessuno? Tre righe in «Dopo».

---

## Autoverifica

1. Quale controllo è dichiarato da GitHub e Anthropic con le stesse parole, e cosa dice Anthropic
   dei bot?
2. Su approvazione e merge un solo vendor dichiara qualcosa. Cosa significa per gli altri due, e
   dove si impone quel controllo?
3. Perché la frase «this reduces but does not eliminate prompt injection risk» è un punto a
   favore di chi la scrive?

<details>
<summary>Risposte</summary>

1. **Il trigger solo da chi ha write access.** GitHub: «Only users with write access to the
   repository can trigger»; Anthropic: «can only be triggered by users with write access». Sui bot,
   Anthropic: esclusi di default, e se ammessi «are not checked for repository permissions».
2. Che il vendor **non lo impone**: per Codex e claude-code-action il merge avviene nel repository,
   quindi la separazione tra chi scrive e chi approva la fanno i **ruleset** (require pull request,
   required approvals, bypass list vuota) — o non la fa nessuno. Per questo nella lezione 6 si
   configurano prima di avviare l'agente.
3. Perché chi scrive cosa **non** garantisce ha capito il problema: la prompt injection attraverso
   contenuto letto dall'agente non si elimina con un filtro, si riduce. Il silenzio di un vendor su
   questo punto non è sicurezza, è assenza di dichiarazione.

</details>

---

## Chiusura del passo

Chiuso quando, senza appunti, sai dire: i tre controlli su cui i vendor convergono, le due assenze
più pesanti e chi le deve coprire, e la divergenza sui segreti. Data in `progressi/README.md`.

**← Lezione 1** [Le posizioni correnti sul multi-agent](../01-posizioni-correnti/README.md) ·
**Prossima →** [03 · La griglia: OWASP Top 10 for Agentic Applications 2026](../03-griglia-owasp/README.md)
