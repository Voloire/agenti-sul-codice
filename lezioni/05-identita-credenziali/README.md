# Lezione 5 — Identità e credenziali degli agenti

**Fase B · 3 sessioni · produce: la tabella identità e segreti × vendor**

---

## Prima di leggere: la domanda d'apertura

Scrivi la risposta in `progressi/05.md`, sezione «Prima», in non più di cinque righe.

> **Quando un coding agent fa `git push`, chi è che sta pushando? Con quale credenziale, ottenuta
> come, valida per quanto, e chi l'ha autorizzata?** Rispondi per l'agente che usi davvero, non in
> astratto.

Se non sai rispondere a una delle cinque domande, hai già trovato il tema della lezione.

---

## La lezione

### 1. L'agente è qualcuno

Un coding agent che tocca un repository ha un'identità: commit con un autore, chiamate API con un
token, un'installazione con dei permessi. L'industria ha un nome per questa classe di identità —
**non-human identity**, NHI — e una lista OWASP dedicata, la *Non-Human Identities Top 10* del
2025. Quella lista non parla di agenti: parla di service account, API key, pipeline. Ma i nomi
valgono anche qui, e tre in particolare: **NHI7 Long-Lived Secrets** (il segreto che non scade),
**NHI1 Improper Offboarding** (l'identità che nessuno revoca), **NHI10 Human Use of NHI** (la
persona che usa la credenziale della macchina — o la macchina che usa quella della persona).

Nella griglia della lezione 3 il tema ha una riga: **ASI03 Identity and Privilege Abuse**,
«the agentic evolution of Excessive Agency», con la mitigazione scritta in tre parole: token
«short-lived, narrowly scoped... per task».

Il problema pratico si vede subito con un esperimento mentale. Da terminale, Codex CLI o Claude
Code girano con il `gh auth` e la chiave SSH **di chi li lancia**: quando l'agente pusha, sta
pushando quella persona. «Least privilege» non è una parola vuota per cattiva volontà: è vuota
perché non c'è nessuna identità separata a cui applicarlo. Questa lezione serve a costruirla.

### 2. Il token giusto: short-lived e audience-bound

Due proprietà, con le parole di chi le implementa.

**Short-lived.** GitHub, sugli installation access token di una GitHub App: «will expire after
1 hour», e alla generazione si può restringere per `repositories` e `permissions`. GitHub Actions,
sui token OIDC: «a short-lived access token that is only valid for a single job». Il contrario è il
personal access token: per i **fine-grained PAT** la policy di default di organizzazione ed
enterprise fissa una durata massima di **366 giorni**; i PAT *classic* non hanno alcun obbligo di
scadenza. Un PAT personale nelle mani di un agente è l'anti-pattern per eccellenza — la lezione 2 ha
già mostrato Anthropic che scrive, nel suo `security.md`, «Do not use a personal access token...
could be partially or fully recovered over time via prompt injection».

**Audience-bound.** Un token valido per un solo destinatario. La specifica MCP, versione
`2026-07-28`, pagina *Authorization*: i client «MUST implement Resource Indicators for OAuth 2.0
as defined in RFC 8707», il parametro `resource` «MUST identify the MCP server that the client
intends to use the token with», e i server «MUST validate that access tokens were issued
specifically for them as the intended audience». Tradotto: un token rubato a un server non apre
un altro server.

Il linguaggio conta. Non «capability a vita breve», non «credenziale temporanea»: **short-lived,
audience-bound token**. Sono le parole di GitHub e della spec MCP, e sono quelle che un
interlocutore tecnico riconosce.

### 3. L'anti-pattern con un nome: token passthrough

La pagina *Security Best Practices* della spec MCP definisce un anti-pattern e lo vieta:
«"Token passthrough" is an anti-pattern where an MCP server accepts tokens from an MCP client
without validating that the tokens were properly issued *to the MCP server* and passes them
through to the downstream API.» La frase normativa: «MCP servers **MUST NOT** accept any tokens
that were not explicitly issued for the MCP server.» Tra i rischi elencati: «Accountability and
Audit Trail Issues» — non si sa più chi ha fatto cosa.

È la forma moderna del **confused deputy**: un componente privilegiato indotto ad agire per conto
di uno meno privilegiato. La spec lo tratta in una sezione a parte, per il caso specifico dei
proxy MCP con static client ID; il principio è lo stesso. Nella lezione 4 l'hai già visto
all'opera: il GitHub MCP server usava il token dell'utente — valido su tutti i suoi repository —
per conto di un'issue scritta da un estraneo.

### 4. Dove vivono i segreti mentre l'agente lavora: i tre vendor divergono

Questo è il punto della lezione, e va detto con precisione perché è facile sbagliarlo: **non c'è
una norma dell'industria**. C'è una scelta di design, e i tre vendor l'hanno fatta in modo diverso.

**OpenAI Codex cloud** — dalla documentazione degli environment, sui secrets: «They are only
available to setup scripts. For security reasons, secrets are removed before the agent phase
starts.» I segreti servono a installare le dipendenze, poi spariscono. Attenzione alla distinzione
che la pagina fa: le **environment variables** normali sono «set for the full duration of the chat
(including setup scripts and the agent phase)»; sono i *secrets* a essere rimossi. E la rete, nella
fase agente, «is off by default».

**GitHub Copilot cloud agent** — dall'8 maggio 2026 (changelog GitHub, *More flexible secrets
and variables for Copilot cloud agent*) esistono gli «Agents secrets and variables», a livello di
repository o organizzazione, «exposed to the agent as environment variables in its development
environment». Cioè: **i segreti stanno nell'ambiente del processo dell'agente**. Il meccanismo
precedente — l'environment GitHub chiamato `copilot` — è quello che si trova ancora in molti
articoli; è stato migrato automaticamente. Il token con cui l'agente parla a GitHub «is limited to
the repository where it's running»; e, dalla pagina dei rischi già letta nella lezione 2, i commit
sono firmati, con Copilot come autore e l'umano come co-author.

**claude-code-action** — dal `setup.md`: l'autenticazione verso GitHub usa di default la **Claude
GitHub App**; il workflow deve avere `id-token: write`, perché l'action scambia il token OIDC del
workflow con un installation token dell'App — che il `security.md` descrive come «short-lived token
scoped specifically to the repository». La API key di Anthropic, però, sta nell'ambiente del job,
come secret di Actions: esposta al processo. Esiste anche un'autenticazione OIDC **verso Anthropic**
(workload identity federation, «exchanging the workflow's GitHub Actions OIDC token for a
short-lived Anthropic access token»), che sostituisce la API key: è un secondo OIDC, con un altro
scopo, da non confondere con il primo. E un terzo modo: `CLAUDE_CODE_OAUTH_TOKEN`, generato da un
utente Pro o Max con `claude setup-token` — un token **personale**. Il `setup.md` lo presenta come
alternativa alla pari; che non vada in un job condiviso è una derivazione di chi scrive, da
[NHI10](#1-lagente-è-qualcuno), non una raccomandazione di Anthropic.

| | Codex cloud | Copilot cloud agent | claude-code-action |
|---|---|---|---|
| segreti durante la fase agente | rimossi prima che l'agente giri | presenti, come env var | presenti, come env var (API key); token GitHub scambiato via OIDC |
| token verso il forge | — (l'ambiente) | scoped al repository | installation token dell'App, scoped al repository, un'ora |
| identità dei commit | non dichiarata | Copilot, co-author umano, firmati | App, non firmati di default |

Perché conta: nella lezione 4, Comment and Control ha letto i segreti proprio dall'ambiente del
processo — `ps auxeww`, `/proc/[pid]/environ` — su Copilot e sulla Claude Code action. Dove il
segreto non c'è, non si può leggere. Il modello Codex non è «la norma»; è la scelta che quel tipo di
attacco non raggiunge.

### 5. L'identità comprende chi può innescare

Un token a vita breve non basta, se chiunque può far partire il job che lo genera.

**Flatt Security** (RyotaK), su `claude-code-action`: segnalato il 12 gennaio 2026, pubblicato il
1 giugno. Il controllo sull'attore che innesca l'action — solo chi ha write — poteva essere aggirato
facendo passare l'attore per un `[bot]`: la funzione di verifica restituiva `true` per qualsiasi
GitHub App. Ottenuto il trigger, l'attaccante leggeva da `/proc/self/environ` le credenziali
`ACTIONS_ID_TOKEN_REQUEST_TOKEN` e `ACTIONS_ID_TOKEN_REQUEST_URL`, con cui **richiedeva** un token
OIDC del workflow, e lo scambiava con un installation token della Claude GitHub App con permessi
di scrittura. Fix nella versione 1.0.94, **CVSS 7.8**. Il token era short-lived e scoped,
esattamente come da manuale; **il trigger non era controllato**. Chi può innescare l'agente è la
prima colonna della tabella dell'identità, non un dettaglio del workflow.

### 6. Il controllo di uscita: push protection

L'ultima difesa non è sull'agente ma sul forge. Il **secret scanning con push protection** blocca
un push che contiene un segreto riconoscibile prima che entri nella storia; su GitHub è attivo di
default per i repository pubblici. Non ferma un'esfiltrazione codificata in base64 — Comment and
Control l'ha mostrato — ma ferma l'errore più comune: un agente che committa un `.env`.

### 7. Cosa te ne fai

Derivazione di chi scrive. Per qualunque agente che tocca un repository, otto colonne, tutte
verificabili nella documentazione o nelle impostazioni — mai a parole:

**identity of record** (chi firma il commit) · **token type** (installation token, OIDC, PAT) ·
**scope** (quali repository, quali permessi) · **lifetime** · **dove vivono i segreti** durante la
fase agente · **chi può innescare** · **chi approva il merge** · **push protection** attiva o no.

E tre anti-pattern da riconoscere a vista: un PAT personale passato all'agente; un token personale
come `CLAUDE_CODE_OAUTH_TOKEN` in un job condiviso; un server MCP che rigira token non emessi per
lui.

---

## Le fonti di questa lezione

Tre sessioni: la prima per MCP e GitHub, la seconda per i tre vendor, la terza per gli incidenti e
la tabella.

**Spec e forge**
1. MCP, [*Security Best Practices*](https://modelcontextprotocol.io/specification/2026-07-28/basic/security_best_practices), spec `2026-07-28` — la versione corrente; le versioni 2025 sono superate.
2. MCP, [*Authorization*](https://modelcontextprotocol.io/specification/latest/basic/authorization) — RFC 8707, audience binding.
3. GitHub, [*Generating an installation access token*](https://docs.github.com/en/apps/creating-github-apps/authenticating-with-a-github-app/generating-an-installation-access-token-for-a-github-app).
4. GitHub Actions, [*About security hardening with OpenID Connect*](https://docs.github.com/en/actions/concepts/security/openid-connect).
5. GitHub, [*About push protection*](https://docs.github.com/en/code-security/secret-scanning/introduction/about-push-protection).

**I tre vendor sui segreti**
6. OpenAI, [*Codex cloud environments*](https://learn.chatgpt.com/docs/environments/cloud-environment).
7. GitHub, [*Configure secrets and variables for Copilot cloud agent*](https://docs.github.com/en/copilot/how-tos/copilot-on-github/customize-copilot/customize-cloud-agent/configure-secrets-and-variables) e [*Give Copilot cloud agent access to resources*](https://docs.github.com/en/copilot/tutorials/cloud-agent/give-access-to-resources).
8. Anthropic, [`claude-code-action/docs/setup.md`](https://github.com/anthropics/claude-code-action/blob/main/docs/setup.md) per i tre modi di autenticarsi, e il [`security.md`](https://github.com/anthropics/claude-code-action/blob/main/docs/security.md) già letto nella lezione 2 per il token e il divieto di PAT. GitHub, [changelog dell'8 maggio 2026](https://github.blog/changelog/2026-05-08-more-flexible-secrets-and-variables-for-copilot-cloud-agent/) per la data degli Agents secrets.

**Griglia, tassonomia, incidente**
9. OWASP, *Top 10 for Agentic Applications 2026*, voce ASI03 (già letta nella lezione 3).
10. OWASP, [*Non-Human Identities Top 10 2025*](https://owasp.org/www-project-non-human-identities-top-10/2025/top-10-2025/) — solo la lista dei dieci nomi.
11. Flatt Security, [*Poisoning Claude Code: one GitHub issue to break the supply chain*](https://flatt.tech/research/posts/poisoning-claude-code-one-github-issue-to-break-the-supply-chain/), 1 giugno 2026 (segnalazione del 12 gennaio).

Da riconoscere, non da leggere: la [policy GitHub sui personal access token](https://docs.github.com/en/enterprise-cloud@latest/admin/enforcing-policies/enforcing-policies-for-your-enterprise/enforcing-policies-for-personal-access-tokens-in-your-enterprise) — 366 giorni di default per i fine-grained, nessun obbligo per i classic.

---

## Esercizio: la tabella identità e segreti × vendor

Scrivi in `progressi/05.md`, sezione «Oggetto». Tre colonne per i tre vendor, e una quarta per
**l'agente che usi tu, come lo usi oggi** — è la colonna che conta.

### Template

```markdown
| | Codex cloud | Copilot cloud agent | claude-code-action | il mio agente, oggi |
|---|---|---|---|---|
| identity of record (chi firma il commit) | | | | |
| token type | | | | |
| scope | | | | |
| lifetime | | | | |
| dove vivono i segreti nella fase agente | | | | |
| chi può innescare | | | | |
| chi approva il merge | | | | |
| push protection | | | | |

### Anti-pattern presenti nel mio setup
- 
```

### Soluzione di riferimento, per le tre colonne dei vendor

| | Codex cloud | Copilot cloud agent | claude-code-action |
|---|---|---|---|
| identity of record | non dichiarata (la configurazione git dell'ambiente) | Copilot, con l'umano co-author; firmati | la GitHub App; non firmati di default |
| token type | — | token dell'agente, scoped al repo | installation token dell'App via OIDC del workflow |
| scope | — | il repository in cui gira | il repository; nessun accesso cross-repository |
| lifetime | — | non dichiarato in pagina | un'ora |
| segreti nella fase agente | rimossi prima della fase agente (le env var normali restano) | env var del processo agente | env var del job (API key), salvo OIDC verso Anthropic |
| chi può innescare | non dichiarato | write access | write access; bot esclusi di default |
| chi approva il merge | il repository (ruleset) | un umano; il richiedente non può | il repository (ruleset) |
| push protection | del forge, indipendente | del forge | del forge |

La quarta colonna non ha soluzione di riferimento: è la tua. Se in «token type» hai scritto «il mio
`gh auth`» e in «identity of record» il tuo nome, hai trovato il lavoro della lezione 6.

### Dopo

Torna alla domanda d'apertura: chi stava pushando, con quale credenziale, valida per quanto,
autorizzata da chi. Quante delle cinque risposte sono cambiate? Tre righe.

---

## Autoverifica

1. Le due proprietà del token giusto, con la fonte di ciascuna.
2. Dove stanno i segreti durante la fase agente nei tre vendor, e perché questa differenza ha
   contato in Comment and Control.
3. Nel caso Flatt Security il token era short-lived e scoped. Cosa mancava, e cosa insegna sulla
   colonna «chi può innescare»?

<details>
<summary>Risposte</summary>

1. **Short-lived** — GitHub: l'installation token «will expire after 1 hour»; il token OIDC di
   Actions è «only valid for a single job». **Audience-bound** — spec MCP, *Authorization*: RFC 8707,
   il `resource` «MUST identify the MCP server», i server «MUST validate that access tokens were
   issued specifically for them».
2. **Codex cloud**: rimossi prima della fase agente (restano le environment variables normali).
   **Copilot cloud agent**: presenti come variabili d'ambiente («Agents secrets and variables»).
   **claude-code-action**: la API key è nell'ambiente del job; il token GitHub è scambiato via OIDC.
   Comment and Control ha letto i segreti dall'ambiente dei processi (`ps auxeww`, `/proc/[pid]/environ`):
   dove il segreto non c'è, l'attacco non trova nulla.
3. Mancava **il controllo su chi innesca**: facendosi passare per `[bot]` l'attaccante aggirava il
   check sull'attore, faceva partire il job, leggeva dall'ambiente le credenziali per richiedere un
   token OIDC e lo scambiava con un installation token in scrittura. Un token perfetto non protegge un trigger aperto: «chi può
   innescare» è parte dell'identità dell'agente, non del workflow.

</details>

---

## Chiusura del passo

Chiuso quando, senza appunti, sai dire le due proprietà del token giusto con le fonti, la divergenza
dei tre vendor sui segreti, e il nome dei tre anti-pattern. Data in `progressi/README.md`.

**← Lezione 4** [Gli incidenti](../04-incidenti/README.md) ·
**Prossima →** [06 · Il lab: il workflow di riferimento, una volta](../06-lab/README.md)
