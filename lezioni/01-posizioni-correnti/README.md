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

Il dibattito sul multi-agent è stato per due anni un dibattito confuso perché le parole erano
confuse. Il vocabolario che ha messo ordine è quello proposto da Anthropic nel dicembre 2024 in
*Building effective agents*, ed è l'unico pezzo di quel testo che resta da leggere oggi:

- Un **workflow** è un sistema in cui il percorso è fissato dal codice: il modello viene chiamato
  in punti decisi in anticipo, secondo uno schema. Cinque schemi ricorrono ovunque: **prompt
  chaining** (una chiamata dopo l'altra, l'output di una entra nella successiva), **routing** (una
  classificazione iniziale manda l'input al percorso giusto), **parallelization** (più chiamate
  indipendenti, poi si aggregano i risultati), **orchestrator–workers** (una chiamata scompone il
  problema e delega i pezzi, poi ricompone), **evaluator–optimizer** (una chiamata produce, un'altra
  valuta, e si itera).
- Un **agent** è un sistema in cui è il modello a decidere il passo successivo, in un ciclo:
  osserva, decide, usa uno strumento, osserva di nuovo. Il percorso non è scritto da nessuno.

Quando qualcuno dice «multi-agent», nove volte su dieci sta descrivendo un **orchestrator–workers**
in cui i workers sono a loro volta agenti. Tenere le parole separate serve a una cosa sola: quando
un fornitore o un collega dice «abbiamo un sistema multi-agente», poter chiedere *quale* dei
cinque schemi, e chi decide il passo successivo.

### 2. Il 2025: la tesi «non costruire multi-agent»

Nel giugno 2025 Cognition — l'azienda dietro Devin — pubblica *Don't Build Multi-Agents*. La tesi
è netta e si regge su due principi:

1. **Condividi il contesto, e condividi le tracce complete**, non i riassunti. Un sub-agente che
   riceve solo il suo pezzo di task non sa *perché* quel pezzo esiste né cosa stanno facendo gli
   altri.
2. **Le azioni portano decisioni implicite, e decisioni in conflitto producono risultati
   cattivi.** Due sub-agenti che lavorano in parallelo prendono, ciascuno, decine di micro-decisioni
   non dichiarate — uno stile, una convenzione, un'ipotesi sui dati — e nessuno le riconcilia.

L'esempio del post è un gioco da costruire diviso fra due sub-agenti: uno fa lo sfondo, l'altro il
personaggio, e i due pezzi non stanno insieme perché ognuno ha interpretato a modo suo ciò che il
task lasciava implicito. La conclusione era: un solo agente, a thread singolo, con compressione
del contesto quando la finestra si riempie.

**Questo testo non si legge.** Va conosciuto perché è stato citatissimo e lo sentirai ancora, ma
il suo autore l'ha superato dieci mesi dopo. Studiare la versione ritrattata di una posizione è
il modo più sicuro per fare una figura mediocre con chi la conosce.

### 3. La correzione: cosa funziona davvero (Cognition, aprile 2026)

*Multi-Agents: What's Actually Working* è lo stesso autore che torna sul tema con dieci mesi di
produzione alle spalle, e dice tre cose:

- **Coder + reviewer da contesto pulito.** Un agente scrive; un secondo agente, in una sessione
  separata che **non ha visto** come il codice è nato, lo rivede. Il valore sta esattamente nel
  non condividere il contesto: il reviewer non eredita le ipotesi del writer, quindi le vede.
  Questo rovescia il primo principio del 2025 — «condividi tutto» — per un caso preciso: la
  valutazione.
- **Lo «smart friend».** Un agente che consulta un altro modello di frontiera come si consulta un
  collega bravo: una domanda, una risposta, nessuna delega di lavoro. Non è un sub-agente che
  esegue, è un secondo parere.
- **Le scritture restano a thread singolo.** Il secondo principio del 2025 resta vero: azioni
  parallele sullo stesso artefatto producono decisioni in conflitto. Quindi **un solo scrittore
  per volta**; il parallelismo va bene per leggere, cercare, rivedere, pianificare.

La regola che ne esce si scrive in una riga e vale come criterio di progetto:
**`writes stay single-threaded`; le letture e le revisioni possono essere molte.**

### 4. La posizione di Anthropic (gennaio 2026)

*When to use multi-agent systems (and when not to)* arriva dalla parte opposta — un vendor che di
multi-agent ne ha costruiti, e che ne ha misurato i costi — e converge sullo stesso punto.

Il default è **un agente solo**. Un sistema multi-agente costa più token, più coordinamento, e
soprattutto **più debugging**: quando qualcosa va storto in una catena di agenti, capire dove è
difficile in modo non lineare. Perciò un secondo agente si giustifica solo se una di **tre
condizioni misurabili** è vera:

| condizione | cosa significa in pratica |
|---|---|
| **context protection** | il lavoro produce così tanto materiale intermedio che, tenuto in un solo contesto, degrada le decisioni. Un sub-agente isola quel materiale e restituisce solo il risultato |
| **parallelizzazione in lettura** | ci sono molte ricerche o letture indipendenti da fare, e nessuna modifica: farle in parallelo accorcia il tempo senza creare conflitti |
| **specializzazione oltre una soglia di strumenti** | un agente con troppi tool — l'ordine di grandezza indicato è una ventina — sceglie male; dividerli per competenza migliora la scelta |

Se nessuna delle tre è vera, la risposta è: un agente, strumenti migliori, contesto gestito
meglio. Nessuna delle tre condizioni dice «per andare più veloce a scrivere codice»: la scrittura
parallela non compare tra i motivi validi.

### 5. Dove le due posizioni si incontrano

Due aziende, due punti di partenza opposti, gennaio e aprile 2026: la stessa conclusione.

| | Cognition 2026 | Anthropic 2026 |
|---|---|---|
| default | un agente | un agente |
| parallelismo ammesso | letture, revisioni, pareri | letture, ricerche indipendenti |
| parallelismo vietato | scritture sullo stesso codice | non giustificato da nessuna delle tre condizioni |
| il secondo agente più utile | il reviewer da contesto pulito | il sub-agente che protegge il contesto |
| criterio | `writes stay single-threaded` | tre condizioni misurabili |

Per chi lavora con coding agent la traduzione è immediata, e la ritroverai nei passi 2 e 6: **un
solo agente scrive su un branch; un reviewer separato, in un'altra sessione, giudica la pull
request; un umano mergia.** Il workflow a pull request che i tre vendor hanno adottato non è una
scelta di comodo: è la forma che il dibattito ha assunto quando è finito.

### 6. Cosa te ne fai, se decidi tu

Le domande da fare a un team o a un fornitore che propone un sistema multi-agente, nell'ordine:

1. **Quale schema è?** Orchestrator–workers, routing, evaluator–optimizer? Se la risposta è vaga, il
   sistema non è stato progettato, è cresciuto.
2. **Quale delle tre condizioni lo giustifica?** Se nessuna, chiedi perché non un agente solo con
   strumenti migliori.
3. **Quanti agenti scrivono sullo stesso artefatto nello stesso momento?** La risposta accettabile
   è uno.
4. **Il reviewer ha visto come il codice è nato?** Se sì, non è un reviewer.
5. **Come si fa il debugging quando fallisce?** Se non c'è una risposta, il costo non è stato
   considerato.

---

## Le fonti di questa lezione

Leggile **dopo** la lezione, per verificarla, non prima. In quest'ordine:

1. Anthropic, [*When to use multi-agent systems (and when not to)*](https://claude.com/blog/building-multi-agent-systems-when-and-how-to-use-them), gennaio 2026. Le tre condizioni, il costo del debugging.
2. Cognition, [*Multi-Agents: What's Actually Working*](https://cognition.com/blog/multi-agents-working), aprile 2026. Coder + reviewer, smart friend, single-threaded writes.
3. Solo il vocabolario, sezione «Building blocks, workflows, and agents»: Anthropic,
   [*Building effective agents*](https://www.anthropic.com/engineering/building-effective-agents), dicembre 2024.

Da **non** leggere, da saper collocare: Cognition, [*Don't Build Multi-Agents*](https://cognition.com/blog/dont-build-multi-agents), giugno 2025 — superato dall'autore con il testo al punto 2.

---

## Esercizio: la checklist

Scrivi in `progressi/01.md`, sezione «Oggetto», la checklist **«quando un secondo agente è
giustificato»**. Una pagina al massimo. Deve poter essere usata da qualcuno che non ha letto le
fonti, davanti a una proposta concreta, per dire sì o no.

### Template

```markdown
## Quando un secondo agente è giustificato

### Una di queste condizioni deve essere vera (altrimenti: un agente solo)
- [ ] ...
- [ ] ...
- [ ] ...

### In ogni caso
- ...   (chi scrive)
- ...   (chi rivede, e cosa non deve aver visto)
- ...   (cosa può andare in parallelo)

### Segnali che il sistema non è stato progettato
- ...
```

### Soluzione di riferimento

Confrontala con la tua **dopo** averla scritta. Non è l'unica giusta; è quella che discende dalle
fonti senza aggiungere nulla.

```markdown
## Quando un secondo agente è giustificato

### Una di queste condizioni deve essere vera (altrimenti: un agente solo)
- [ ] Il materiale intermedio è così tanto che in un solo contesto degrada le decisioni
      (context protection).
- [ ] Ci sono molte letture o ricerche indipendenti, senza modifiche, da fare insieme
      (parallelizzazione in lettura).
- [ ] Gli strumenti sono troppi per un agente solo — ordine di grandezza: una ventina — e
      dividerli per competenza migliora la scelta (specializzazione).

### In ogni caso
- Scrive **un solo agente per volta** sullo stesso artefatto: le scritture restano a thread singolo.
- Rivede un agente **in una sessione separata**, che non ha visto come il lavoro è nato.
- In parallelo vanno letture, ricerche, revisioni, pareri. Mai scritture sullo stesso codice.

### Segnali che il sistema non è stato progettato
- Nessuno sa dire quale dei cinque schemi è (chaining, routing, parallelization,
  orchestrator–workers, evaluator–optimizer).
- Il motivo dichiarato è «per andare più veloci a scrivere»: non è tra le condizioni valide.
- Il reviewer condivide il contesto del writer.
- Nessuna risposta a «come si fa il debugging quando fallisce».
```

### Dopo: torna alla domanda d'apertura

Rileggi la tua risposta di prima. Scrivi tre righe in «Dopo»: cosa confermeresti, cosa cambieresti,
e quale delle tre condizioni — se una — descriveva la situazione che avevi in mente.

---

## Autoverifica

Rispondi senza rileggere. Le risposte sono in fondo; se ne sbagli una, la lezione non è chiusa.

1. Qual è l'unico tipo di operazione che nessuna delle due posizioni del 2026 ammette in
   parallelo fra più agenti, e perché?
2. Un fornitore propone tre agenti che lavorano insieme su una feature «per finire prima». Quale
   domanda fai per prima, e quale risposta ti aspetti dalle fonti?
3. Perché il reviewer, per essere utile, **non** deve condividere il contesto del writer — e in che
   senso questo rovescia il primo principio di Cognition del 2025?

<details>
<summary>Risposte</summary>

1. **Le scritture sullo stesso artefatto.** Azioni parallele portano decisioni implicite non
   riconciliate, e decisioni in conflitto producono risultati cattivi (Cognition, principio 2,
   confermato nel 2026: `writes stay single-threaded`); nessuna delle tre condizioni di Anthropic
   riguarda la scrittura parallela.
2. **«Quale delle tre condizioni lo giustifica?»** — context protection, parallelizzazione in
   lettura, specializzazione per numero di strumenti. «Finire prima» non è tra queste: la risposta
   coerente con le fonti è un agente solo, e al massimo un reviewer separato.
3. Perché il valore del reviewer sta nel **non ereditare le ipotesi** del writer: se ha visto come
   il codice è nato, le sue stesse ipotesi gli sembrano ovvie e non le vede. Il principio 2025
   diceva «condividi il contesto completo»; per la valutazione vale l'opposto, ed è stato lo stesso
   autore a dirlo nel 2026.

</details>

---

## Chiusura del passo

Il passo è chiuso quando, senza appunti, sai raccontare in tre frasi: cosa dicevano nel 2025, cosa
dicono nel 2026, e qual è la regola in una riga. Segna la data in `progressi/README.md`.

**Prossima lezione →** [02 · I controlli dei tre vendor, letti in parallelo](../02-controlli-vendor/README.md)
