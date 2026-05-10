---
layout: post
title: "Dal Vibe Coding all'Agentic Software Engineering: la differenza non è di strumento ma di postura professionale"
date: 2026-05-09 09:30:00
description: "Dal vibe coding all'agentic software engineering: perché la differenza non sta negli strumenti ma nella postura professionale di chi sviluppa software con agenti AI."
categories: ai software-engineering
tags: vibe-coding agentic-engineering ai llm karpathy
---

## Cos'è il Vibe Coding

Nel febbraio 2025 Andrej Karpathy pubblica un post su X in cui descrive il modo in cui sta
costruendo una piccola web app:

> *"I'm building a project or webapp, but it's not really coding — I just see stuff, say stuff,
> run stuff, and copy paste stuff, and it mostly works."*

Il termine *vibe coding* nasce così, e si diffonde in poche settimane. Nel 2025 il Collins
Dictionary lo elegge **parola dell'anno**. Il fenomeno tocca un nervo scoperto: sviluppare software
non assomiglia più a quello che era anche solo due anni prima.

Una definizione operativa: il vibe coding è la pratica di sviluppare software esprimendo intenti
in linguaggio naturale a un agente AI, che produce il codice. Il developer cambia ruolo —
da scrittore di codice riga-per-riga a **direttore e revisore** di output generato.

Punti che vale la pena chiarire subito:

- **Non è "AI che programma da sola"**: è una collaborazione asimmetrica in cui il giudizio
  finale resta umano (o dovrebbe restarlo).
- **Non è "autocomplete potenziato"**: il salto qualitativo sta nella generazione di porzioni
  strutturali di codice, non nel completamento della singola riga.
- **Non è solo uno strumento, è un metodo di lavoro**: i principi dell'ingegneria del software
  restano gli stessi, ma cambiano i metodi per applicarli, gli strumenti che usiamo, e il modo
  in cui usiamo strumenti che già esistevano.
- **Richiede disciplina**: il rischio di ritrovarsi in mano un software di cui lo sviluppatore
  non ha la piena comprensione è altissimo. In alcuni contesti questo è accettabile (a volte
  perfino voluto). In altri è inaccettabile.

## Perché ha avuto successo (e perché diffidarne)

Il vibe coding ha avuto un impatto culturale prima che tecnico:

- Ha **abbassato la barriera d'ingresso** allo sviluppo software. Persone che non sapevano
  programmare in un linguaggio specifico si sono trovate a costruire prototipi funzionanti.
- Ha **accelerato la prototipazione**. Quello che richiedeva giorni si fa in ore.
- Ha cambiato il **modo di esplorare codebase nuove**.

Allo stesso tempo, dal punto di vista dell'ingegneria del software classica, il vibe coding può
includere alcune delle cose peggiori che possono capitare nello sviluppo: codice generato senza
comprensione, senza verifica, senza responsabilità personale dello sviluppatore.

Tutte e due le cose sono vere contemporaneamente. È questa tensione che rende il fenomeno
interessante — e che rende necessario un passo successivo.

## Il passo successivo: Agentic Software Engineering

Nell'aprile 2026, alla conferenza Sequoia AI Ascent, lo stesso Karpathy presenta una distinzione netta. Riprende il termine *vibe coding* e lo affianca a un secondo concetto: *agentic engineering*.
Sono due cose correlate, ma diverse:

- **Vibe coding**: abbassa la barriera di ingresso e permette a chiunque di creare software descrivendo cosa vuole. Va bene per **prototipi e strumenti personali**.
- **Agentic engineering**: è la disciplina professionale di coordinare in modo consapevole agenti AI fallibili preservando correttezza, sicurezza, gusto e manutenibilità. **È ciò che serve a team di sviluppo che producono software in contesti professionali.**

La differenza non è di strumento ma di **postura professionale**.

### Cosa fa l'agentic engineer (e il vibe coder no)

L'agentic engineer non accetta ciecamente il codice generato. Il suo lavoro è:

- progettare specifiche chiare **prima** di delegare
- supervisionare i piani proposti dall'agente
- ispezionare i diff prodotti
- scrivere test e definire criteri di accettazione
- creare evaluation loop per misurare la qualità
- gestire permessi e ambienti di esecuzione (worktree isolati, sandbox)
- preservare nel tempo la qualità del sistema

In altre parole, applica le pratiche dell'ingegneria del software classica a un nuovo tipo
di collaboratore: un agente fallibile, veloce, e pericolosamente plausibile.

### Un esempio: il bug di MenuGen

Karpathy racconta un bug di pagamento accaduto nel suo progetto MenuGen — lo stesso progetto
da cui era nato il post originale del febbraio 2025. L'agente aveva collegato gli acquisti
Stripe agli account Google usando l'indirizzo email come chiave.

Il codice era **plausibile**. Compilava. Girava. Sembrava funzionare. Ma il design era
**sbagliato**: l'email registrata su Stripe e quella usata per il login Google possono differire.
Risultato: utenti che pagano e non vedono il prodotto sbloccato, o viceversa.

Nessun test unitario avrebbe colto questo problema. Nessun linter. Nessun checker statico.
Era un errore di **design di sistema**, ed è un esempio perfetto di cosa significa che l'output
di un LLM è plausibile ma non per questo corretto. Solo un umano con sufficiente giudizio di
prodotto e ingegneristico può imporre la scelta giusta — in questo caso, identificatori utente
persistenti e indipendenti dall'email.

### Cosa resta competenza umana

La frontier skill **non è memorizzare ogni dettaglio delle API**. Gli agenti ricordano
perfettamente se una libreria usa `dim`, `axis`, `keepdim`, `reshape` o `permute`.

L'umano deve capire i concetti sottostanti:

- modelli di storage e di memoria, views vs copie
- invarianti del sistema
- identità e confini di sicurezza
- la forma e l'architettura complessiva del sistema

Come in altre discipline ingegneristiche, il ruolo e il valore dell'ingegnere si spostano
verso l'alto — capire meglio cosa il sistema deve fare e perché — e si separano dalla
realizzazione pratica del manufatto. Non è poi così strano se pensiamo a quello che fa oggi
un ingegnere civile rispetto alla costruzione fisica di un edificio o di un ponte.

## Conclusione

Il vibe coding è una buona porta d'ingresso. È divertente, produttivo, e ha aperto lo sviluppo
software a persone che ne erano escluse. Ma è anche un punto di partenza, non di arrivo.

Chi costruisce software che gli altri useranno — software che gestisce dati, denaro, decisioni —
non può fermarsi al vibe coding. Deve evolvere verso l'agentic engineering: una pratica in cui
gli agenti AI sono collaboratori potenti ma fallibili, e in cui la responsabilità del risultato
resta saldamente umana.

La domanda da porsi non è "userò agenti AI per programmare?" — la risposta è già sì, in un modo
o nell'altro. La domanda è: con quale **disciplina** lo farò?

---

## Riferimenti
- Karpathy, A. (febbraio 2025) — post originale su X che introduce il termine *vibe coding*
- [Collins Dictionary — parola dell'anno 2025](https://blog.collinsdictionary.com/language-lovers/collins-word-of-the-year-2025-ai-meets-authenticity-as-society-shifts/)
- Karpathy, A. (aprile 2026) — [intervento a Sequoia AI Ascent 2026](https://sequoiacap.com/article/ai-ascent-2026/)
- Karpathy, A. — [post su Agentic Software Engineering](https://karpathy.bearblog.dev/sequoia-ascent-2026/)

## Note editoriali
- Tono: divulgativo ma rigoroso — il lettore può essere developer o manager tecnico
- Pezzo lungo: candidato a "post pillar" che si collega ai due post brevi già in bozza
  ("Umani troppo perfetti" sulla plausibilità, "10-80-10" sul metodo di delega)
- Possibile chiusura alternativa: invito ad approfondire nel corso "Vibe Coding"
