# Lezione 6 — Il lab: il workflow di riferimento, una volta, con un agente vero

**Fase B · 3–4 sessioni · produce: tre pull request mergiate con la storia leggibile, e una nota di
lab**

---

## Prima di cominciare: la domanda d'apertura

Scrivi la risposta in `progressi/06.md`, sezione «Prima», in non più di cinque righe.

> **Nel repository dove lavori oggi, cosa impedisce fisicamente — non per convenzione — a un
> agente di fare merge del proprio lavoro? Elenca le regole attive, se le conosci.**

Se la risposta è «niente» o «non lo so», il lab serve a te prima che a chiunque altro.

---

## Cosa costruisci, e perché in quest'ordine

Il lab riproduce, su un repository di prova, il workflow che i tre vendor hanno adottato
(lezione 2): un agente con **identità propria** scrive su un **branch proprio**; l'unico output è
una **pull request**; la **CI** gira prima di qualunque lettura; un **reviewer separato** giudica;
un **umano** mergia. Non si costruisce niente di nuovo: si configurano cose che GitHub offre già,
nell'ordine in cui si sostengono a vicenda.

| sessione | cosa | perché prima di ciò che segue |
|---|---|---|
| 1 | il repository, il piano GitHub, i **ruleset** | senza regole sul branch di integrazione, il resto è convenzione |
| 1–2 | la **GitHub App** come identità dell'agente, il suo installation token | senza identità separata, l'agente sei tu e i ruleset non ti fermano |
| 2 | la CI come **required status check** | senza check obbligatorio, «CI prima della review» è un'abitudine |
| 2–3 | **tre pull request** vere, aperte dall'agente con l'identità dell'App | è il workflow, eseguito |
| 3 | il **reviewer da contesto pulito** e il merge umano | la lezione 1 messa in pratica |
| 3–4 | la nota di lab | l'oggetto del passo |

È una **demo**, non una misura: tre PR non dimostrano un tasso di successo. Dimostrano che ogni
controllo della lezione 2 ha un posto preciso nella configurazione, e che tu sai dov'è.

---

## Sessione 1 — Il repository e i ruleset

### Il piano GitHub decide dove fai il lab

I ruleset — le regole moderne sui branch — sono disponibili, dice GitHub, «in public repositories
with GitHub Free and GitHub Free for organizations, and in public and private repositories with
GitHub Pro, GitHub Team, and GitHub Enterprise Cloud». Quindi: **su un repository privato con
account Free i ruleset non ci sono**. Il repository di prova o è pubblico, o sta su un account Pro
o Team. Deciderlo prima evita di scoprirlo alla terza sessione.

Crea il repository di prova con qualcosa dentro che abbia un test: un piccolo progetto in
qualunque linguaggio, con un comando di test che passa. Il branch di integrazione è `main`.

### Ruleset e non branch protection

GitHub oggi raccomanda i ruleset rispetto alle branch protection rules classiche: «Multiple
rulesets can apply to the same branch at the same time, while only one branch protection rule
applies», e quando si sovrappongono «the most restrictive version of the rule applies». Un
ruleset si può mettere in `evaluate` per vederne l'effetto senza imporlo, e «Anyone with read
access to a repository can view its active rulesets» — la regola è leggibile da chiunque, il che
per un assessment vale oro.

Le regole che servono, con i nomi della documentazione: **Require a pull request before merging**
(con *Required approvals*, *Dismiss stale pull request approvals when new commits are pushed*),
**Require status checks to pass before merging**, **Block force pushes**, **Restrict deletions**.

### La bypass list, vuota

Un ruleset ha una **bypass list**: «you can allow certain users to bypass the rules in the
ruleset» — ruoli, team, GitHub App. Nel lab resta **vuota**. Con la lista vuota, nemmeno
l'amministratore del repository salta le regole: la documentazione lo dice per via indiretta —
«organization owners or repository administrators will be unable to change or rename the default
branch unless they are authorized to bypass the ruleset». Una bypass list vuota è il controllo che
rende vere tutte le altre regole; una con dentro «Repository admin» è una porta.

### Da terminale

`gh` ha `gh ruleset list`, `gh ruleset view` e `gh ruleset check` — utile quest'ultimo per vedere
quali regole si applicherebbero a un branch, anche prima che esista — ma **non ha un comando per
creare** un ruleset. Si usa l'API REST, `POST /repos/{owner}/{repo}/rulesets`, con `gh api`. Il
payload minimo, con i nomi di campo della documentazione:

```json
{
  "name": "integration-branch",
  "target": "branch",
  "enforcement": "active",
  "bypass_actors": [],
  "conditions": { "ref_name": { "include": ["~DEFAULT_BRANCH"], "exclude": [] } },
  "rules": [
    { "type": "deletion" },
    { "type": "non_fast_forward" },
    { "type": "pull_request", "parameters": {
        "required_approving_review_count": 1,
        "dismiss_stale_reviews_on_push": true,
        "require_code_owner_review": false,
        "require_last_push_approval": false,
        "required_review_thread_resolution": false,
        "allowed_merge_methods": ["squash"] } },
    { "type": "required_status_checks", "parameters": {
        "strict_required_status_checks_policy": true,
        "do_not_enforce_on_create": false,
        "required_status_checks": [ { "context": "test" } ] } }
  ]
}
```

```bash
gh api -X POST repos/OWNER/REPO/rulesets --input ruleset.json
gh ruleset check main
```

`~DEFAULT_BRANCH` è il pattern speciale per il branch di default. `"context": "test"` è il **nome
del job** della CI che renderai obbligatorio nella sessione 2: deve coincidere. `bypass_actors` è
`[]`, e resta così.

---

## Sessione 1–2 — L'identità dell'agente: una GitHub App

### Perché un'App e non un secondo account

La lezione 5 ha dato la ragione: l'agente deve avere un'identità **separata dalla tua**, con un
token **short-lived e scoped**. Una GitHub App è esattamente questo: si registra una volta, si
installa sul solo repository di prova, e genera **installation access token** che «will expire
after 1 hour», restringibili per repository e permessi. Un secondo account umano darebbe un PAT a
366 giorni e un'identità che un auditor confonderebbe con una persona.

### Registrazione

Dal tuo profilo: *Settings → Developer settings → GitHub Apps → New GitHub App*. Nome unico,
massimo 34 caratteri; una homepage URL qualsiasi; webhook **disattivato** (deseleziona *Active*);
installabile «Only on this account».

### Permessi minimi

«GitHub Apps don't have any permissions by default... You should select the minimum permissions
required for the app.» Per il lab: **Contents: write** (per pushare sul branch — «you should
request the "Contents" repository permission»), **Pull requests: write** (per aprire la PR). Issues
solo se l'agente dovrà leggerle o aprirle. Nient'altro.

### Installazione e token

Installa l'App sul repository di prova: *Edit → Install App → Only select repositories*. Poi il
token, in tre passi che la documentazione descrive con esempi in Bash:

1. un **JWT** firmato RS256 con la private key dell'App — `iat` sessanta secondi nel passato,
   `exp` al massimo dieci minuti nel futuro, `iss` uguale al client ID;
2. l'**installation ID**, da `GET /repos/{owner}/{repo}/installation` con quel JWT;
3. il **token**: `POST /app/installations/{id}/access_tokens`, opzionalmente con `repositories` e
   `permissions` per restringerlo ancora. «You must use a JWT to access this endpoint.»

Il token si usa con git come password HTTP — `git clone https://x-access-token:TOKEN@github.com/OWNER/REPO.git`
— e con `gh` attraverso la variabile `GH_TOKEN`, che «takes precedence over previously stored
credentials». Dentro GitHub Actions esiste una scorciatoia, `actions/create-github-app-token`;
fuori da Actions la documentazione rimanda alle librerie Octokit. Nel lab, da terminale, i tre
passi si fanno a mano una volta, per capirli; poi si mettono in uno script.

Dove tenere la private key: fuori dal repository, sulla macchina, con permessi stretti. È l'unico
segreto a vita lunga del lab, ed è quello da cui derivano tutti i token a vita breve.

### Il clone dedicato

Un secondo clone del repository, in una cartella separata, configurato **solo** con il token
dell'App: nessun `gh auth` tuo, nessuna chiave SSH tua. Da lì lancerai l'agente. Se dal clone
dell'agente riesci a fare `gh api user` e vedi il tuo nome, l'identità non è separata.

---

## Sessione 2 — La CI come required status check

Un workflow GitHub Actions che gira su `pull_request` ed esegue il test del progetto. Due dettagli
che la documentazione sottolinea e che fanno fallire i lab:

- Il required status check è **il nome del job**: «Use `jobs.<job_id>.name` to set a name for the
  job», e «make sure that job names are unique across all workflows». Il nome deve essere `test`,
  come nel ruleset.
- «GitHub Actions generates checks, not commit statuses.» Se il workflow non gira — per un filtro
  su path o branch — «Associated checks stay in a "Pending" state and block merging». E il caso
  opposto, insidioso: «A job that is skipped will report its status as "Success". It will not
  prevent a pull request from merging, even if it is a required check.» Un job con `if:` che lo
  salta non protegge niente.

```yaml
name: ci
on: [pull_request]
jobs:
  test:
    name: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: ./run-tests.sh
```

Verifica: apri tu una PR banale e guarda che il merge sia bloccato finché `test` non è verde e
nessuno ha approvato.

---

## Sessione 2–3 — Tre pull request, aperte dall'agente

Dal clone dedicato, con `GH_TOKEN` impostato al token dell'App e nient'altro. Tre task, uno per
sessione se serve, del tipo per cui l'industria usa i coding agent oggi:

1. **Un bump di dipendenza**, con il test che deve restare verde.
2. **Un test mancante** per una funzione che ne è priva.
3. **Una documentazione fuori sincrono** con il codice, da riallineare.

Per ciascuno: l'agente lavora su un branch `agent/<task>`, committa, pusha con il token dell'App,
apre la PR. Osserva, e annota nella nota di lab:

- **chi è l'autore** dei commit e della PR (deve essere l'App, non tu);
- **cosa succede al merge** prima che la CI sia verde e prima dell'approvazione (bloccato);
- **cosa succede se l'agente prova a pushare su `main`** (rifiutato dal ruleset);
- **cosa l'agente ha toccato** oltre al necessario, se qualcosa.

Un fatto della documentazione da tenere presente: «Pull request authors cannot approve their own
pull requests.» Con un solo account umano che apre la PR e *required approvals: 1*, quella PR
resterebbe non mergiabile per sempre. Con l'App come autore, tu non sei l'autore, e puoi
approvare. È la stessa separazione che Copilot cloud agent impone da solo (lezione 2): qui la
ottieni dal ruleset più l'identità.

---

## Sessione 3 — Il reviewer da contesto pulito, e il merge

Alla terza PR, prima di approvarla, una **seconda sessione** dell'agente — meglio se dell'altro
vendor — riceve un solo input: la pull request. Non il transcript della prima sessione, non il
task originale, non le tue conversazioni. Le domande sono quelle della lezione 1: la forma è
giusta? c'è un errore che il test non vede? c'è una conseguenza di sicurezza? si capirà fra sei
mesi?

Poi decidi tu. Approvi, o chiedi modifiche, e alla fine **mergi tu**, con squash. Nella storia di
`main` resta un commit per task, con l'App come autore del lavoro e tu come chi l'ha ammesso.

Cosa annotare: se il reviewer ha trovato qualcosa che tu non avevi visto; se ha segnalato cose
irrilevanti (lo farà); quanto è costato in tempo rispetto a rivedere da solo.

---

## La nota di lab

`progressi/06.md`, sezione «Oggetto». Non un diario: una tabella che lega **ogni controllo della
lezione 2** al pezzo di configurazione che lo implementa nel tuo lab, con l'evidenza.

### Template

```markdown
| controllo (lezione 2) | dove sta nel lab | evidenza (comando, file, screenshot) | funziona? |
|---|---|---|---|
| chi innesca | | | |
| branch di scrittura | | | |
| approvazione e merge | | | |
| rete in uscita | | | |
| contenuto nascosto | | | |
| segreti nella fase agente | | | |
| identità dei commit | | | |
| CI prima della review | | | |
| reviewer separato | | | |

### Cosa è stato rifiutato, e da cosa
- 

### Cosa non sono riuscito a imporre, e perché
- 
```

### Soluzione di riferimento

| controllo | dove sta nel lab | evidenza |
|---|---|---|
| chi innesca | tu, dal clone dedicato | non c'è trigger automatico: è il limite del lab, e va scritto |
| branch di scrittura | `agent/*`; il ruleset blocca `main` | `gh ruleset check main`; il push su `main` rifiutato |
| approvazione e merge | ruleset: PR obbligatoria, 1 approvazione, bypass list vuota | `gh api repos/O/R/rulesets`; il merge bloccato finché non approvi |
| rete in uscita | **non imposta** dal lab (l'agente gira sulla tua macchina) | va scritto come assente: è la riga rossa più probabile |
| contenuto nascosto | non applicabile: il task lo scrivi tu | — |
| segreti nella fase agente | solo `GH_TOKEN` dell'App nel clone dedicato; la private key fuori dal repo | `env` nel clone dedicato |
| identità dei commit | l'App | `git log --format='%an'` sulle PR |
| CI prima della review | required status check `test` | una PR con test rosso non mergiabile |
| reviewer separato | seconda sessione, solo la PR in input | la review nella terza PR |

Due righe di questa tabella saranno «non imposto»: la rete in uscita e il trigger. È giusto così,
ed è il punto della lezione 8 — un assessment onesto dice cosa manca, non cosa si vorrebbe.

### Dopo

Torna alla domanda d'apertura: cosa impedisce fisicamente il merge nel repository dove lavori.
Ora sai nominare le regole. Ci sono? Tre righe.

---

## Le fonti di questa lezione

Tutte documentazione GitHub. Leggile prima di ciascuna sessione, non tutte insieme.

1. [*About rulesets*](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/about-rulesets) e [*Available rules for rulesets*](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/available-rules-for-rulesets); [*Creating rulesets for a repository*](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/creating-rulesets-for-a-repository) per la bypass list.
2. [*GitHub's plans*](https://docs.github.com/en/get-started/learning-about-github/githubs-plans) — dove i ruleset sono disponibili.
3. REST, [*Create a repository ruleset*](https://docs.github.com/en/rest/repos/rules?apiVersion=2022-11-28#create-a-repository-ruleset); CLI, [`gh ruleset`](https://cli.github.com/manual/gh_ruleset).
4. [*Registering a GitHub App*](https://docs.github.com/en/apps/creating-github-apps/registering-a-github-app/registering-a-github-app), [*Choosing permissions*](https://docs.github.com/en/apps/creating-github-apps/registering-a-github-app/choosing-permissions-for-a-github-app), [*Installing your own GitHub App*](https://docs.github.com/en/apps/using-github-apps/installing-your-own-github-app).
5. [*Generating a JWT for a GitHub App*](https://docs.github.com/en/apps/creating-github-apps/authenticating-with-a-github-app/generating-a-json-web-token-jwt-for-a-github-app), [*Generating an installation access token*](https://docs.github.com/en/apps/creating-github-apps/authenticating-with-a-github-app/generating-an-installation-access-token-for-a-github-app), [*Authenticating as a GitHub App installation*](https://docs.github.com/en/apps/creating-github-apps/authenticating-with-a-github-app/authenticating-as-a-github-app-installation).
6. [*About status checks*](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/collaborating-on-repositories-with-code-quality-features/about-status-checks) e [*Troubleshooting required status checks*](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/collaborating-on-repositories-with-code-quality-features/troubleshooting-required-status-checks).
7. [*Approving a pull request with required reviews*](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/reviewing-changes-in-pull-requests/approving-a-pull-request-with-required-reviews) — «Pull request authors cannot approve their own pull requests».

---

## Autoverifica

1. Perché la bypass list deve essere vuota, e cosa dice la documentazione sugli amministratori?
2. Perché un job con `if:` che viene saltato non protegge il merge, anche se è required?
3. Con un solo account umano e *required approvals: 1*, cosa succede a una PR aperta da quella
   persona — e come l'identità dell'App cambia la situazione?

<details>
<summary>Risposte</summary>

1. Perché chi è nella bypass list salta **tutte** le regole del ruleset, e con la lista vuota
   nemmeno l'amministratore lo può fare: la documentazione dice che «repository administrators
   will be unable to change or rename the default branch unless they are authorized to bypass the
   ruleset». Una bypass list con «Repository admin» dentro rende le altre regole decorative.
2. Perché «A job that is skipped will report its status as "Success". It will not prevent a pull
   request from merging, even if it is a required check». Il check risulta verde senza aver
   eseguito niente. Il job deve girare sempre sulle PR, senza condizioni che lo saltino.
3. Resta **non mergiabile**: «Pull request authors cannot approve their own pull requests», e con
   la bypass list vuota nessuno può forzare. Con la GitHub App come autore della PR, l'umano non è
   l'autore e può approvare: la separazione fra chi scrive e chi ammette è nella struttura, non
   nella buona volontà.

</details>

---

## Chiusura del passo

Chiuso quando le tre PR sono mergiate con l'App come autore, la nota di lab ha tutte le righe
compilate — comprese quelle «non imposto» — e sai dire quali due controlli il lab non copre e
perché. Data in `progressi/README.md`.

**← Lezione 5** [Identità e credenziali](../05-identita-credenziali/README.md) ·
**Prossima →** [07 · I protocolli, collocati](../07-protocolli/README.md)
