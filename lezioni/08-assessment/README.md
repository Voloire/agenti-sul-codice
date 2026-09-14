# Lezione 8 — L'assessment, misurabile

**Fase C · 1–2 sessioni · produce: `docs/assessment-<pipeline>.md`, il documento finale del percorso**

---

## Prima di cominciare: la domanda d'apertura

Scrivi la risposta in `progressi/08.md`, sezione «Prima», in non più di cinque righe.

> **Se domani dovessi dire a qualcuno «la vostra pipeline con agenti è sicura» oppure «non lo è»,
> su quale evidenza lo diresti? Elenca le prime tre cose che andresti a guardare.**

Poi confronta con la rubrica che hai costruito nella lezione 3: quante delle tue tre cose sono
righe della rubrica?

---

## La lezione

### 1. Cos'è un assessment, e cosa non è

Un assessment non è un'opinione argomentata. È una **griglia compilata con evidenze**, in cui ogni
riga ha uno di tre valori — *presente*, *assente*, *ignoto* — e accanto a ciascuno la prova: un
file, una regola, un comando, uno screenshot. Il valore *ignoto* è legittimo e va scritto: un
assessment che non ha righe «ignoto» o ha avuto accesso a tutto, o sta bluffando.

Tre proprietà lo distinguono da una relazione:

- **Misurabile**: due persone con lo stesso accesso arrivano alle stesse righe.
- **Attribuibile**: ogni giudizio rimanda a una fonte del percorso — una pagina di vendor, una
  voce OWASP, un incidente — non a «buona pratica».
- **Attuabile**: chiude con il prossimo passo e il suo costo, non con una lista di desideri.

Tutto ciò che hai prodotto nei sette passi precedenti confluisce qui: la checklist della lezione 1,
la tabella dei vendor della lezione 2, la rubrica della lezione 3, gli incidenti della 4, la
tabella dell'identità della 5, la nota di lab della 6, le due righe della 7.

### 2. Quale pipeline

Una pipeline **vera**, in quest'ordine di preferenza:

1. Una pipeline di produzione a cui hai accesso **in lettura** — repository, workflow, ruleset,
   impostazioni dell'agente — e il permesso di scriverne. È il caso che vale di più e va chiesto
   presto, perché è l'unico che dipende da qualcun altro.
2. Il **lab della lezione 6**. Vale meno perché l'hai costruito tu sapendo cosa cercare, ma è
   completo e onesto: ha già due righe rosse note.
3. Il disegno pubblico di un vendor — le pagine della lezione 2 — trattato come se fosse la tua
   pipeline. Utile come esercizio, da dichiarare come tale.

Il documento dice quale delle tre, e quale accesso hai avuto. Un assessment senza la riga
«accesso avuto» non si può leggere.

### 3. La struttura del documento

Sei sezioni, sempre le stesse, in quest'ordine. Ogni sezione risponde a una domanda e a nessun'altra.

**§1 Oggetto e accesso.** Quale pipeline, quali agenti (prodotto e versione), quali repository,
cosa hai potuto leggere e cosa no. Due paragrafi.

**§2 Il workflow com'è.** Una riga per fase, con l'evidenza: chi innesca l'agente e come; dove
scrive; cosa produce; cosa gira prima della review; chi rivede; chi mergia. È la tabella della
lezione 2 applicata alla pipeline reale, e la colonna «se non dichiarato, chi lo impone» compilata
con quello che c'è davvero.

**§3 La rubrica ASI01–ASI10.** La tabella della lezione 3, compilata. Dieci righe, ogni riga
*presente / assente / ignoto* con l'evidenza. Le tre righe che decidono il colore della pagina —
ASI01, ASI03, ASI05 — per prime o in evidenza.

**§4 Identità e credenziali.** La tabella della lezione 5 per gli agenti di questa pipeline: chi
firma, che token, che scope, che durata, dove stanno i segreti nella fase agente, chi può
innescare, chi approva, push protection. Gli anti-pattern trovati, per nome.

**§5 Il secondo agente, se c'è.** La checklist della lezione 1 applicata: quale delle tre
situazioni lo giustifica; chi scrive e quanti scrivono; il reviewer ha visto come è nato il codice?
Se non c'è un secondo agente, la sezione dice «non applicabile» e perché è un bene o un male.

**§6 Il prossimo passo, e il suo costo.** Uno solo. La riga rossa che, chiusa, cambia più righe
della rubrica; cosa serve per chiuderla — una configurazione, un cambio di identità, una policy — e
quanto costa in giornate e in attrito. Poi, eventualmente, il secondo e il terzo. Non una lista di
tutto.

### 4. Come si compila una riga

Prendi ASI03, *Identity and Privilege Abuse*, con la domanda della tua rubrica: *l'agente ha
un'identità propria con credenziali short-lived per task, non il PAT di uno sviluppatore?*

- **Presente**: l'agente committa come GitHub App; `git log --format='%an'` lo mostra;
  l'installation token ha scadenza a un'ora, documentata; il clone dell'agente non ha credenziali
  umane. Evidenza: l'output del comando, il link alla configurazione dell'App.
- **Assente**: l'agente committa con il nome di una persona; nel workflow c'è un `secrets.PAT`;
  nessuna GitHub App installata. Evidenza: la riga del workflow, l'autore dei commit.
- **Ignoto**: non hai accesso alle impostazioni dell'App né ai secret del repository, e i commit
  non bastano a stabilirlo. Evidenza: cosa hai chiesto e a chi.

Nessuna di queste tre è un'opinione. Il lettore può rifare il controllo.

### 5. Gli errori tipici, da evitare

- **Confondere dichiarato e imposto.** Un vendor dichiara un controllo (lezione 2); la pipeline lo
  usa solo se è configurato. «Copilot filtra i commenti HTML» non è evidenza che *questa* pipeline
  lo faccia, se l'agente non è Copilot.
- **Scrivere «presente» sulla parola di qualcuno.** Se te l'hanno detto e non l'hai visto, è
  *ignoto* con la nota «riferito da».
- **Il firewall come prova sufficiente.** La lezione 4 ha mostrato che l'esfiltrazione passa da
  GitHub, host sempre consentito. La rete in uscita è una riga; non chiude ASI01.
- **La lista dei desideri al posto del prossimo passo.** Dieci raccomandazioni non sono un
  assessment, sono un catalogo. Una, con il costo, è una decisione.
- **Dimenticare la data.** Le pagine dei vendor cambiano nome ogni pochi mesi; l'assessment dice
  quando le hai lette.

### 6. Cosa te ne fai

L'assessment è il documento che dimostra che il percorso è finito, ed è anche l'unico che si porta
via da qui: la stessa struttura vale per la prossima pipeline, il prossimo team, il prossimo
fornitore. Chi lo legge deve poter dire, senza aver visto niente, cosa quella pipeline fa bene e
cosa le manca — e trovare la riga dove andare a verificare.

---

## Le fonti di questa lezione

Nessuna nuova. L'assessment usa quelle delle lezioni 1–7, e le cita riga per riga.

---

## Esercizio: l'assessment

Scrivi `docs/assessment-<pipeline>.md` — non in `progressi/`: è un documento, non una nota — con le
sei sezioni. In `progressi/08.md`, sezione «Oggetto», solo il link e la data.

### Template

```markdown
# Assessment — <pipeline>

Data: <data>. Fonti dei vendor lette il: <data>.

## 1. Oggetto e accesso
Pipeline, agenti (prodotto, versione), repository. Accesso avuto: … Accesso non avuto: …

## 2. Il workflow com'è
| fase | com'è qui | evidenza | chi lo impone |
|---|---|---|---|
| chi innesca | | | |
| dove scrive | | | |
| cosa produce | | | |
| cosa gira prima della review | | | |
| chi rivede | | | |
| chi mergia | | | |

## 3. Rubrica ASI01–ASI10
| ASI | domanda | presente / assente / ignoto | evidenza |
|---|---|---|---|
| ASI01 | | | |
| … | | | |

## 4. Identità e credenziali
| | agente A | agente B |
|---|---|---|
| identity of record | | |
| token type · scope · lifetime | | |
| segreti nella fase agente | | |
| chi può innescare · chi approva | | |
| push protection | | |
Anti-pattern trovati: …

## 5. Il secondo agente
Situazione che lo giustifica: … Chi scrive: … Il reviewer ha visto come è nato il codice: …
(oppure: non applicabile, perché …)

## 6. Il prossimo passo
Uno. Quale riga chiude, cosa serve, quanto costa.
```

### Soluzione di riferimento

Non c'è: ogni pipeline ha la sua. C'è il **criterio con cui giudicarla**. Fai leggere il documento
a qualcuno che non ha visto la pipeline né il percorso, e chiedigli tre cose:

1. Cosa fa bene questa pipeline? — deve rispondere citando righe *presenti*.
2. Cosa le manca? — deve citare righe *assenti*, e trovare l'evidenza.
3. Qual è la prima cosa da fare? — deve dire il tuo §6, non un'altra.

Se risponde a tutte e tre, l'assessment è misurabile e il percorso è finito. Se su una esita,
quella sezione va riscritta.

### Dopo

Torna alla domanda d'apertura. Le tre cose che avresti guardato: sono nel documento? Erano quelle
giuste? Tre righe.

---

## Autoverifica

1. I tre valori di una riga della rubrica, e perché «ignoto» è legittimo.
2. La differenza fra un controllo dichiarato e uno imposto, con un esempio.
3. Perché §6 ha un solo elemento.

<details>
<summary>Risposte</summary>

1. **Presente, assente, ignoto**, ciascuno con evidenza. «Ignoto» è legittimo perché un assessment
   dichiara l'accesso che ha avuto: una riga che non si è potuta verificare, scritta come ignota
   con la nota di cosa si è chiesto, è più onesta di un «presente» sulla parola di qualcuno.
2. Un controllo **dichiarato** è quello che il vendor scrive nella sua pagina (lezione 2); uno
   **imposto** è quello configurato nella pipeline che stai guardando. Esempio: «il richiedente non
   può approvare la PR» è dichiarato da Copilot cloud agent, ma in una pipeline con Codex lo impone
   solo un ruleset con *required approvals* e bypass list vuota — o nessuno.
3. Perché **una decisione** si prende, un catalogo si archivia. Il prossimo passo è la riga rossa
   che, chiusa, cambia più righe della rubrica, con il suo costo: è ciò che chi legge può approvare
   o rifiutare. Il secondo e il terzo, se ci sono, vengono dopo, e dichiaratamente dopo.

</details>

---

## Chiusura del percorso

Il percorso è finito quando il documento esiste, un lettore esterno risponde alle tre domande, e la
riga di `progressi/README.md` ha la data. Da lì in avanti la stessa griglia vale per ogni pipeline
che incontri: è la competenza che il percorso lascia.

**← Lezione 7** [I protocolli](../07-protocolli/README.md) · **Indice** [README](../../README.md)
