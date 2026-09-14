# Lezione 1 — Le posizioni correnti sul multi-agent

**Fase A · 2 sessioni · produce: la checklist «quando un secondo agente è giustificato»**

---

## Prima di leggere: la domanda d'apertura

Scrivi la risposta in `progressi/01.md`, sezione «Prima», in non più di cinque righe. Non
cercare la risposta giusta: cerca la *tua*, quella che daresti oggi a un collega.

> **A cosa serve, secondo te, un secondo agente? In quale situazione concreta ne vorresti uno,
> e cosa ti aspetti che faccia meglio di un agente solo?**

Poi leggi la lezione. Alla fine tornerai su quella risposta.

---

## La lezione

### 1. Le parole, prima delle opinioni

Il dibattito sul multi-agent è stato per due anni confuso perché le parole erano confuse. Il
vocabolario che ha messo ordine è quello di Anthropic, *Building effective agents* (dicembre
2024), sezione «Building blocks, workflows, and agents» — l'unica parte di quel testo che resta
da leggere:

- Un **workflow** è un sistema in cui il modello e gli strumenti sono orchestrati «through
  predefined code paths»: chi chiama chi è deciso in anticipo. Il testo ne descrive cinque:
  **prompt chaining** (una chiamata dopo l'altra), **routing** (una classificazione manda l'input
  al percorso giusto), **parallelization** (più chiamate indipendenti, poi aggregazione),
  **orchestrator–workers** (una chiamata scompone e delega, poi ricompone), **evaluator–optimizer**
  (una produce, un'altra valuta, si itera).
- Un **agent** è un sistema in cui il modello «dynamically direct[s] [its] own processes and tool
  usage»: decide da sé il passo successivo, in un ciclo.

Quando qualcuno dice «multi-agent», spesso sta descrivendo un orchestrator–workers in cui i
workers sono a loro volta agenti. Tenere le parole separate serve a una cosa: quando un fornitore
o un collega dice «abbiamo un sistema multi-agente», poter chiedere *quale* schema, e chi decide il
passo successivo.

### 2. Il 2025: la tesi «non costruire multi-agent»

Nel giugno 2025 Walden Yan, di Cognition — l'azienda dietro Devin — pubblica *Don't Build
Multi-Agents*. La tesi si regge su due principi, quasi letterali:

1. **«Share context, and share full agent traces, not just individual messages.»** Un sub-agente
   che riceve solo il suo pezzo di task non sa *perché* quel pezzo esiste né cosa fanno gli altri.
2. **«Actions carry implicit decisions, and conflicting decisions carry bad results.»** Due
   sub-agenti in parallelo prendono, ciascuno, micro-decisioni non dichiarate — uno stile, una
   convenzione, un'ipotesi — e nessuno le riconcilia.

L'esempio del post è un clone di Flappy Bird diviso fra due sub-agenti: uno fraintende il proprio
sottotask e produce uno sfondo in stile Super Mario, l'altro un personaggio che non c'entra; i
pezzi non stanno insieme. La conclusione era: un solo agente, a thread singolo, con un modello di
compressione del contesto quando la finestra si riempie.

**Questo testo non si legge come posizione corrente.** Va conosciuto perché è molto citato e lo
sentirai ancora, ma dieci mesi dopo lo stesso autore l'ha **ristretto** — non ritirato — con il
testo del punto 3. Studiare la versione del 2025 senza la correzione del 2026 è il modo più sicuro
per fare una figura mediocre con chi la conosce.

### 3. La correzione: cosa funziona davvero (Cognition, aprile 2026)

*Multi-Agents: What's Actually Working* è lo stesso autore con dieci mesi di produzione alle
spalle. Il punto di partenza è esplicito: *«Our original observations still hold today for
parallel-writer swarms»* — i due principi restano validi per gli sciami di scrittori in parallelo.
Quello che è cambiato è la scoperta di **tre casi** in cui più agenti funzionano:

- **Coder + reviewer da contesto pulito.** Un agente scrive; un secondo agente, con *«a completely
  clean context»*, guarda *«only the diff»* e non riceve nulla di quanto è successo prima (*«do not
  share any context beforehand»*). L'autore lo definisce controintuitivo rispetto al 2025, e dà due
  motivi: la matematica dell'attenzione — il *context rot*, un contesto lungo degrada la
  valutazione — e il fatto che il reviewer, non avendo la spec, *ragiona all'indietro
  dall'implementazione* e vede ciò che il writer dava per scontato. Il coder poi filtra i finding
  del reviewer (un «communication bridge»): non tutto ciò che il reviewer segnala va applicato.
- **Lo «smart friend».** Un tool con cui il modello primario *«could make a call out to»* un altro
  modello di frontiera per un parere: una domanda, una risposta, nessuna delega di lavoro. Ha
  funzionato con un primario di frontiera; con un primario più debole (il loro SWE-1.5) no.
- **Il manager con i child agents.** Un Devin che coordina più Devin figli, in quello che il testo
  chiama «map-reduce-and-manage», e che *«live[s] in Devin today»*. È l'orchestrator–workers del
  vocabolario di Anthropic, con un vincolo preciso.

Il vincolo è la regola che il testo scrive in una riga e che vale come criterio di progetto:
**`writes stay single-threaded`** — le scritture restano a thread singolo; letture, revisioni,
pareri e coordinamento possono essere molti.

### 4. La posizione di Anthropic (gennaio 2026)

*Building multi-agent systems: When and how to use them* (claude.com, 23 gennaio 2026) arriva da un
vendor che di sistemi multi-agente ne ha costruiti e ne ha misurato i costi.

Il default è **un agente solo**, e i costi del multi-agent sono elencati senza sconti: ogni agente
in più è *«another potential point of failure, another set of prompts to maintain, another source
of unexpected behavior»*; i token crescono di **3–10 volte**; a ogni passaggio si perde contesto —
il «telephone game». Perciò un secondo agente si giustifica in **tre situazioni**, con soglie
indicative:

| situazione | cosa significa in pratica |
|---|---|
| **context protection** | il lavoro produce molto materiale intermedio irrilevante per la decisione finale (l'ordine di grandezza indicato è oltre il migliaio di token); un sub-agente lo isola e restituisce solo il risultato |
| **parallelization** | più ricerche o letture indipendenti si possono fare insieme. Attenzione al motivo: *«the primary benefit of parallelization is thoroughness, not speed»* — il beneficio è la copertura, e il tempo totale *spesso cresce* |
| **specialization** | l'agente ha troppi strumenti (*«often 20+»*, *«15-20+ tools»*) o deve tenere insieme modi di comportarsi in conflitto, o domini diversi: dividere per competenza — strumenti, system prompt, dominio — migliora le scelte |

Due precisazioni che il testo fa e che spesso vengono perse. Prima: Anthropic **non vieta** la
scrittura parallela; la ammette quando il contesto *«can be truly isolated»*, per esempio frontend
e backend *«with a well-defined API contract»*, o test di componenti diversi. Seconda: Anthropic
**sconsiglia** la divisione coder / tester / reviewer come agenti separati, perché *«the code
reviewer lacks the context of exploration»* — e la salva in un solo caso, la **verifica a scatola
chiusa**: il *verification subagent*, indicato come *«one multi-agent pattern that consistently
works well across domains»*, perché *«the verifier does not need to understand why the artifact
was built as it was»*.

### 5. Dove le due posizioni si incontrano, e dove no

Due aziende, due punti di partenza opposti, gennaio e aprile 2026.

| | Cognition 2026 | Anthropic 2026 |
|---|---|---|
| default | un agente | un agente |
| scritture in parallelo | **mai** sullo stesso codice: `writes stay single-threaded` | **solo** se il contesto è davvero isolabile (interfacce pulite, componenti separati) |
| letture e ricerche in parallelo | sì | sì, per copertura, non per velocità |
| il secondo agente che funziona | il reviewer da contesto pulito | il verification subagent, che non deve sapere perché l'artefatto è fatto così |
| perché il valutatore non deve condividere il contesto | context rot; ragiona all'indietro dall'implementazione | non ha bisogno del contesto dell'esplorazione per verificare |

Il punto di contatto più forte è il **valutatore separato**: entrambi arrivano, da strade diverse,
a dire che chi giudica un risultato lavora meglio se non ha visto come è nato. La differenza
residua è sulla scrittura parallela: Cognition la esclude, Anthropic la circoscrive. Per chi
governa coding agent la conseguenza pratica è la stessa, e la ritroverai nei passi 2 e 6: un agente
scrive su un branch; un reviewer separato, in un'altra sessione, giudica la pull request; un umano
mergia.

### 6. Cosa te ne fai, se decidi tu

Queste domande sono una derivazione di chi scrive, non un elenco delle fonti. Da fare a un team o
a un fornitore che propone un sistema multi-agente, nell'ordine:

1. **Quale schema è?** Orchestrator–workers, routing, evaluator–optimizer? Se la risposta è vaga, il
   sistema non è stato progettato, è cresciuto.
2. **Quale delle tre situazioni lo giustifica?** Se nessuna, perché non un agente solo con
   strumenti e contesto migliori?
3. **Quanti agenti scrivono sullo stesso artefatto nello stesso momento?** Se più di uno, il
   contesto è davvero isolato — c'è un'interfaccia definita fra i pezzi — o si sta contando sulla
   fortuna?
4. **Il reviewer ha visto come il codice è nato?** Se sì, non è un reviewer nel senso di nessuna
   delle due fonti.
5. **Quanto costa in token rispetto a un agente solo, e chi mantiene i prompt in più?** Anthropic
   dice 3–10x e «another set of prompts to maintain»: chiedere il numero.

---

## Le fonti di questa lezione

Leggile **dopo** la lezione, per verificarla, non prima. In quest'ordine:

1. Anthropic, [*Building multi-agent systems: When and how to use them*](https://claude.com/blog/building-multi-agent-systems-when-and-how-to-use-them), 23 gennaio 2026. Le tre situazioni, i costi, il verification subagent, i confini di decomposizione.
2. Cognition, [*Multi-Agents: What's Actually Working*](https://cognition.com/blog/multi-agents-working), aprile 2026. Coder + reviewer, smart friend, manager con child agents, `writes stay single-threaded`.
3. Solo la sezione «Building blocks, workflows, and agents»: Anthropic,
   [*Building effective agents*](https://www.anthropic.com/engineering/building-effective-agents), dicembre 2024.

Da **non** leggere come posizione corrente, da saper collocare: Cognition, [*Don't Build
Multi-Agents*](https://cognition.com/blog/dont-build-multi-agents), giugno 2025 — ristretto dallo
stesso autore con il testo al punto 2.

---

## Esercizio: la checklist

Scrivi in `progressi/01.md`, sezione «Oggetto», la checklist **«quando un secondo agente è
giustificato»**. Una pagina al massimo. Deve poter essere usata da qualcuno che non ha letto le
fonti, davanti a una proposta concreta, per dire sì o no.

### Template

```markdown
## Quando un secondo agente è giustificato

### Una di queste situazioni deve essere vera (altrimenti: un agente solo)
- [ ] ...
- [ ] ...
- [ ] ...

### Se più agenti scrivono
- ...   (la condizione perché sia accettabile)

### Il valutatore
- ...   (cosa non deve aver visto, e perché)

### Il costo da chiedere
- ...

### Segnali che il sistema non è stato progettato
- ...
```

### Soluzione di riferimento

Confrontala con la tua **dopo** averla scritta. Non è l'unica giusta; è quella che discende dalle
due fonti del 2026. Dove le fonti divergono, lo dice.

```markdown
## Quando un secondo agente è giustificato

### Una di queste situazioni deve essere vera (altrimenti: un agente solo)
- [ ] Context protection: molto materiale intermedio irrilevante per la decisione finale;
      un sub-agente lo isola e restituisce solo il risultato.
- [ ] Parallelization: molte letture o ricerche indipendenti da fare insieme — per copertura,
      non per velocità (il tempo totale spesso cresce).
- [ ] Specialization: troppi strumenti (15-20+), o comportamenti in conflitto, o domini diversi
      da tenere insieme.

### Se più agenti scrivono
- Solo se il contesto è davvero isolabile: componenti separati con un'interfaccia definita
  (Anthropic). Sullo stesso codice, mai: le scritture restano a thread singolo (Cognition).

### Il valutatore
- In una sessione separata, con contesto pulito, che vede solo il risultato (il diff) e non
  come è nato. Su questo le due fonti convergono, per motivi diversi: context rot e ragionamento
  all'indietro dall'implementazione (Cognition); il verificatore non ha bisogno di sapere perché
  l'artefatto è fatto così (Anthropic).

### Il costo da chiedere
- Token: 3-10x rispetto a un agente solo. Più prompt da mantenere, più punti di guasto,
  contesto perso a ogni passaggio.

### Segnali che il sistema non è stato progettato
- Nessuno sa dire quale schema è (chaining, routing, parallelization, orchestrator-workers,
  evaluator-optimizer).
- Il motivo dichiarato è «per andare più veloci»: nessuna delle tre situazioni promette velocità.
- Più agenti scrivono sullo stesso codice senza un'interfaccia che li separi.
- Il reviewer condivide il contesto del writer.
```

### Dopo: torna alla domanda d'apertura

Rileggi la tua risposta di prima. Scrivi tre righe in «Dopo»: cosa confermeresti, cosa cambieresti,
e quale delle tre situazioni — se una — descriveva il caso che avevi in mente.

---

## Autoverifica

Rispondi senza rileggere. Le risposte sono in fondo; se ne sbagli una, la lezione non è chiusa.

1. Sulla scrittura in parallelo le due fonti del 2026 non dicono la stessa cosa. Qual è la
   posizione di ciascuna, e qual è la condizione che Anthropic pone?
2. Un fornitore propone tre agenti che lavorano insieme su una feature «per finire prima». Quale
   frase di Anthropic sulla parallelizzazione gli citi, e quale domanda fai per prima?
3. Perché il valutatore lavora meglio senza il contesto del writer? Dà i due motivi di Cognition e
   quello di Anthropic, e spiega perché questo non è una ritrattazione dei principi del 2025.

<details>
<summary>Risposte</summary>

1. **Cognition**: mai scritture parallele sullo stesso codice — `writes stay single-threaded`; i
   principi del 2025 «still hold» per i parallel-writer swarms. **Anthropic**: scritture parallele
   ammesse solo quando il contesto «can be truly isolated», per esempio frontend e backend con un
   «well-defined API contract», o test di componenti diversi.
2. *«The primary benefit of parallelization is thoroughness, not speed»* — e i sistemi
   multi-agente spesso richiedono più tempo totale, non meno. La prima domanda: quale delle tre
   situazioni (context protection, parallelization, specialization) giustifica i tre agenti; «finire
   prima» non ne è una.
3. **Cognition**: il *context rot* — un contesto lungo degrada la valutazione — e il fatto che il
   reviewer, senza la spec, ragiona all'indietro dall'implementazione e vede ciò che il writer dava
   per scontato. **Anthropic**: *«the verifier does not need to understand why the artifact was
   built as it was»*. Non è una ritrattazione perché Cognition dichiara che i due principi del 2025
   restano validi per gli sciami di scrittori paralleli: il reviewer a contesto pulito è un caso
   ristretto — la valutazione — in cui condividere il contesto fa peggio, non un ripensamento
   generale.

</details>

---

## Chiusura del passo

Il passo è chiuso quando, senza appunti, sai raccontare in tre frasi: cosa diceva Cognition nel
2025, cosa dicono Cognition e Anthropic nel 2026 e dove divergono, e qual è la regola in una riga.
Segna la data in `progressi/README.md`.

**Prossima lezione →** [02 · I controlli dei tre vendor, letti in parallelo](../02-controlli-vendor/README.md)
