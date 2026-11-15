---
layout: post
title: "Il ritorno dei classificatori: i modelli di decisione"
date: 2026-11-15 10:00:00 +0100
description: "Dal 15 settembre 2026 esiste una famiglia di modelli che non scrivono testo: scelgono tra risposte predefinite e assegnano una probabilità a ciascuna. In due mesi sono arrivati OpenAI, Microsoft e Google. Cosa sono, come funzionano, dove hanno senso e perché vanno validati sui propri dati."
categories: ai
tags: ai llm classificatori modelli-di-decisione agenti-ai system-one
toc:
  sidebar: left
---

Dal 15 settembre 2026 esiste una famiglia di modelli che **non scrivono testo**: scelgono tra
risposte predefinite e assegnano una probabilità a ciascuna. In due mesi la categoria è passata da
una startup a OpenAI, Microsoft e, dall'11 novembre, Google.

È un ritorno ai classificatori, ma con la flessibilità di un modello linguistico: il compito si
descrive in una frase invece di addestrarlo.

## Un classificatore per ogni compito

Per anni un classificatore ha fatto una cosa sola. Riconoscere lo spam, il tono di un messaggio o
una specie di uccello richiedeva un dataset etichettato, un addestramento e una manutenzione
dedicati. Con BERT il progresso è stato partire da un modello pre-addestrato e rifinirlo, ma il
risultato restava **un modello per compito**.

Il costo vero è l'etichettatura: per questo molte decisioni piccole restano senza classificatore
([Parallel](https://parallel.ai/articles/what-is-a-decisions-api)). Gli LLM generici hanno coperto
parte del vuoto, perché basta descrivere il compito nel prompt. Ma significa generare token,
interpretare una stringa, validarla e riprovare quando il formato si rompe. Su milioni di chiamate
al giorno il costo si somma, e un agente resta in attesa di ciascuna.

L'idea non è nuova: esistevano già classificatori basati sull'inferenza del linguaggio naturale e
modelli come UniMC e Universal Discriminator ([arXiv 2609.30216](https://arxiv.org/pdf/2609.30216)).
Ciò che cambia è la scala e l'interfaccia.

## Un modello che non scrive

Un modello di decisione riceve uno **stato** (un ticket, un log, una policy) e alcune **domande con
risposte ammesse**. Restituisce una probabilità per ogni opzione, in un solo passaggio e senza
generare testo.

Il primo è **Jev**, di TypeSafe AI, startup fondata da Diogo Almeida, ex OpenAI
([The Batch](https://www.deeplearning.ai/the-batch/models-built-to-do-one-thing-well)), uscito in
accesso anticipato il 15 settembre. TypeSafe ha battezzato la categoria "System One", dalla
distinzione di Kahneman tra giudizio rapido e ragionamento lento. Sul nome non c'è accordo: Simon
Willison preferisce [decision models](https://simonwillison.net/2026/Sep/21/jev/), ed è
l'espressione che uso qui; TypeSafe ha invece
[chiesto di non chiamarli così](https://typesafe.ai/blog/dont-call-it-a-decision-model), perché Jev
formula giudizi e a decidere è il codice che li combina.

Un esempio: un messaggio dice _"mi avete addebitato due volte"_. Si chiede a quale team va
inoltrato, quanto è urgente e se richiede un rimborso. La risposta è una distribuzione di
probabilità per ciascuna domanda, che il codice confronta con una soglia.

Le domande sono di tre tipi:

| Tipo                  | Cosa restituisce                                               |
| --------------------- | -------------------------------------------------------------- |
| Sì o no               | Probabilità, da 0 a 1, che la condizione sia vera              |
| Scelta (_choice_)     | Opzione più probabile, probabilità di ogni opzione, confidenza |
| Valutazione (_score_) | Probabilità per ogni livello ordinato e media pesata           |

Una precisazione per chi fa divulgazione: "non può allucinare" vale **solo per il formato**, perché
la risposta è sempre una delle opzioni. Il modello può comunque assegnare una probabilità alta a una
risposta sbagliata.

## Stesso filone dei transformer, testa diversa

TypeSafe non dichiara l'architettura di Jev: dice soltanto che non è un LLM, non è autoregressivo
ed è basato su transformer. Più corretto, quindi, dire che i modelli di decisione discendono dallo
stesso filone dei transformer, non che derivano dagli LLM. Dei concorrenti l'architettura si
conosce, e si divide in tre famiglie.

- **Backbone da LLM, senza generazione.** Cloudflare Clef usa Qwen congelato con adattatori LoRA, una
  sola lettura del contesto e poi la valutazione in parallelo delle opzioni
  ([Cloudflare](https://blog.cloudflare.com/clef-decision-models/)). Microsoft-Decision-1 è un
  Qwen3.5-9B post-addestrato.
- **Encoder alla BERT.** Laya (ModernBERT-large), Von e GLiNER2.5-Decide (340 milioni di parametri)
  riportano gli encoder al centro, e girano anche su CPU.
- **Generalisti dietro un endpoint dedicato.** OpenAI serve le decisioni con gpt-6-luna; Gemini
  Decision 1 poggia sull'architettura multimodale di Gemini.

La seconda famiglia rende preciso il "ritorno ai classificatori": si riparte dagli encoder, ma con
**una testa unica guidata da istruzioni in linguaggio naturale**, al posto di una testa per ogni
compito. Cambia anche l'obiettivo di addestramento: TypeSafe descrive RLCD, apprendimento per
rinforzo su decisioni calibrate, che premia probabilità affidabili più che risposte singole
([Browserbase](https://browserbase.com/blog/what-is-jev)).

## Il mercato a metà novembre 2026

Al 12 novembre un censimento indipendente conta **33 modelli di 26 fornitori**
([systemonemodels.org](https://systemonemodels.org/guides/what-is-a-system-one-model/)). Alcuni
esempi tra i commerciali, con prezzi in dollari per milione di token in ingresso (l'output è
gratuito):

| Fornitore | Modello                  | Prezzo input |
| --------- | ------------------------ | ------------ |
| TypeSafe  | Jev 1.14                 | 0,042        |
| OpenAI    | gpt-6-luna               | 0,10 (0,01 con cache) |
| Google    | gemini-decision-1        | 0,10         |
| Microsoft | Microsoft-Decision-1     | 0,042        |
| Perplexity | pplx-decider-v1.2 e flash | 0,02 e 0,01 |
| Cloudflare | Clef, Clef-flash, Clef-omni | 0,24, 0,038 e 0,15 |

I prezzi scendono in fretta: Perplexity è partita da 0,04 dollari e ha dimezzato; Cloudflare ha
portato Clef-flash da 0,09 a 0,038, sotto Jev, riducendo però il contesto ospitato da 64K a 24K
token.

Sul fronte dei pesi aperti, Hugging Face ne ospita oltre 200. Quasi tutti sono fine-tuning di
Qwen con licenza Apache 2.0 o MIT (Cloudflare, Perplexity, vLLM Semantic Router, Base Labs, Respan
su Gemma 4, Together AI, H2O.ai e molti altri). Servirli in locale è semplice: llama.cpp espone
`/v1/systemone` dal 2 ottobre e vLLM supporta Decision 3.0 nelle build notturne. Attenzione al peso
in memoria: i modelli da 27B richiedono una GPU grossa, mentre Laya, GLiNER2.5-Decide e jevos girano
su CPU.

Queste tabelle non sono esaustive, e alcune delle fonti (Parallel, Respan, Laya AI) vendono un
prodotto concorrente.

## Un contratto condiviso: l'API System One

In due mesi i modelli di decisione sono passati dall'esperimento di una startup a una categoria, e
a standardizzarsi più in fretta è lo **schema di richiesta e risposta di TypeSafe**: Ollama (0.35),
llama.cpp, vLLM, Liquid AI, Baseten e Microsoft Foundry espongono lo stesso endpoint, Cloudflare
dichiara Clef compatibile con l'API di Jev, Perplexity e Google usano le stesse tre domande su un
percorso `/decisions` proprio. Solo OpenAI ha uno schema suo (`POST /v1/decisions`, con `predicate`
per il sì/no).

Attorno al contratto è nato uno strato di integrazione: Pydantic AI ha una classe `SystemOneModel`
e il gateway Bifrost traduce un'unica richiesta verso otto fornitori.

Il risultato pratico è la **portabilità a livello di codice**: si passa da Jev ospitato a un
modello locale cambiando l'URL base. Ma le risposte non sono equivalenti: una probabilità di 0,7 di
un modello non vale quella di un altro, e le soglie vanno riconvalidate a ogni cambio di versione.
Come per le Chat Completions di OpenAI, la competizione si sposterà dall'interfaccia alla qualità e
alla calibrazione.

## Dove hanno senso: routing, gate, cascata

Il modello di decisione lavora accanto all'LLM: l'LLM pianifica, scrive e chiama strumenti, il
modello di decisione prende le tante piccole decisioni in mezzo. Quattro schemi ricorrono.

- **Routing.** OpenRouter ha lanciato Jev Router, che sceglie modello e livello di ragionamento per
  ogni richiesta.
- **Gate sulle azioni.** AWS Strands Decider 2B controlla una chiamata a uno strumento prima che
  l'agente la esegua. Secondo AWS, su compiti brevi mai visti le risposte con confidenza almeno 0,9
  sono corrette circa nel 95% dei casi: dato del produttore, da rivalidare sul proprio traffico.
- **Ripiego.** Bifrost accetta un elenco di modelli di riserva per la stessa decisione.
- **Cascata.** Il modello gestisce ciò di cui è sicuro, i casi dubbi vanno a un modello costoso o a
  una persona. In Stagehand, Browserbase accetta l'azione scelta da Jev solo sopra una soglia di
  0,7 e altrimenti passa a un LLM; la latenza mediana è scesa da 1,97 a 0,46 secondi.

La cascata funziona solo se la probabilità è **calibrata**, ed è il tema della prossima sezione.

I dati d'uso confermano il quadro: su 2.170 progetti pubblici che integrano Jev, la domanda a scelta
compare nell'81,0%, il sì/no nel 72,2% e la valutazione nel 45,4%. Su Requesty Jev pesa per il 4,8%
delle richieste ma solo per lo 0,4% dei token: molte chiamate, tutte piccole.

Un dato di ricerca mostra che il compromesso è reale. Nella classificazione di debolezze software
(CWE) in 50 classi, rispetto a GPT-5.6-Sol Jev ha una latenza mediana 6,27 volte più bassa e un
costo stimato 55,9 volte più basso, ma accuratezza Top-1 inferiore (0,458 contro 0,495) e Macro-F1
inferiore (0,269 contro 0,360) ([arXiv 2609.34963](https://arxiv.org/pdf/2609.34963)).

## Le ombre: calibrazione, robustezza, benchmark

**La confidenza non è la probabilità di avere ragione.** Per le domande a scelta Jev la calcola come
(n × p − 1) / (n − 1), con n opzioni e p la probabilità più alta: misura quanto è _concentrata_ la
distribuzione. Con tre opzioni, una confidenza di 0,9 corrisponde a una probabilità di 0,933. La
calibrazione si misura su molte decisioni, non su una.

**Robustezza.** Un gruppo di USC, MBZUAI e Arizona State University ha mostrato che un ottimizzatore
riusciva a deviare verso un'opzione sbagliata prefissata il **61,4%** di 508 decisioni di Jev
inizialmente corrette, aggiungendo un breve contesto (mediana 31 parole), giudicato naturale nel
91,6% dei casi. Su un'implementazione aperta con Qwen3-1.7B il tasso è stato del 72,6%, quindi la
debolezza sembra della categoria ([arXiv 2609.30243](https://arxiv.org/pdf/2609.30243)). Non è la
frequenza di errore ordinaria, ma conta se il testo in ingresso è controllato da un avversario, come
una mail di spam.

**Lingue.** Su dieci lingue regionali indonesiane, con 500 frasi etichettate per lingua un
classificatore classico resta sotto Jev zero-shot in 9 lingue su 10
([arXiv 2610.14112](https://arxiv.org/pdf/2610.14112)). All'opposto, il checkpoint inglese di Laya
sul khmer ha accuratezza 0,000 con confidenza 0,952: una soglia di confidenza non protegge. Per
l'italiano non ho trovato valutazioni dedicate.

**Benchmark.** Le misure disponibili non sono confrontabili e vengono quasi sempre da chi vende un
concorrente. Alcuni esempi: Cloudflare mostra Clef che batte Jev su BANKING77 (macro-F1 94,20
contro 79,74) ma perde su When2Call (72,37 contro 80,97); Google dichiara Gemini Decision 1 primo
in 5 test su 7; Respan misura un'accuratezza di 0,932 per Jev e 0,901 per Tev1, con calibrazione
migliore per Tev1. Ognuno vince nella corsa scelta da sé. Universale, quindi, vale per
**l'interfaccia più che per la qualità**.

## Come scegliere: validare sui propri dati

"Senza addestramento" non significa "senza validazione". Il costo si sposta dall'addestrare un
modello al descrivere bene il compito e al misurare quanto funziona. In pratica:

1. Scrivere domande e opzioni come si farebbe con un revisore umano, con una descrizione per ogni
   opzione.
2. Etichettare un campione realistico (poche centinaia di esempi), con casi lunghi, ambigui e privi
   dell'evidenza necessaria.
3. Misurare accuratezza e calibrazione su più modelli, non solo sul più noto.
4. Fissare la soglia di confidenza sui dati, e rifarlo a ogni cambio di versione o di modello.
5. Instradare i casi sotto soglia a un LLM o a una persona.
6. Se il testo in ingresso può essere controllato da un avversario, aggiungere controlli oltre la
   soglia.

Per scegliere tra servizio e pesi aperti contano dati e volumi: i pesi aperti tengono lo stato
dentro la propria rete e sostituiscono il prezzo per token con costi di infrastruttura. Con un
prezzo ospitato sceso fino a 0,01 dollari per milione di token, però, il conto dell'hardware va
fatto davvero.

## Conclusione

Il classificatore universale esiste come **interfaccia**, non ancora come garanzia di qualità. Il
compito si cambia riscrivendo una domanda, ma quanto il modello sia affidabile su quel compito va
misurato ogni volta. Non è un ritorno al passato né la fine dei classificatori addestrati: è uno
spostamento del costo dal training alla specifica del compito e alla validazione.

L'arrivo di OpenAI, Microsoft e Google in meno di due mesi dice che la categoria resterà. Dice
anche che il prezzo per token conterà sempre meno, e che la differenza la faranno calibrazione,
robustezza e copertura delle lingue: tre cose che, per ora, ciascuno deve verificare sui propri dati.

## Perimetro e fonti

Dati e prezzi sono quelli pubblicati nelle pagine consultate il 14 novembre 2026; il mercato cambia
di settimana in settimana, e molte cifre di prestazioni vengono da chi vende un prodotto. Non ho
misurato nessun modello.

- [Parallel, What is a Decisions API](https://parallel.ai/articles/what-is-a-decisions-api)
- [systemonemodels.org, What is a System One model](https://systemonemodels.org/guides/what-is-a-system-one-model/)
- [Respan, Jev alternatives](https://www.respan.ai/articles/jev-alternatives)
- [ScriptByAI, Open-source Jev alternatives](https://www.scriptbyai.com/jev-open-source-alternatives/)
- [Laya AI, System One decision models](https://laya-ai.com/system-one-models)
- [Google, Gemini Decisions API](https://blog.google/innovation-and-ai/technology/developers-tools/gemini-decisions-api/)
- [TypeSafe AI, Don't Call It a Decision Model](https://typesafe.ai/blog/dont-call-it-a-decision-model)
- [Requesty, TypeSafe Jev explained](https://www.requesty.ai/blog/typesafe-jev-explained)
- [Cloudflare, Introducing Clef](https://blog.cloudflare.com/clef-decision-models/)
- [The Batch, Models Built to Do One Thing Well](https://www.deeplearning.ai/the-batch/models-built-to-do-one-thing-well)
- [Browserbase, What is Jev?](https://browserbase.com/blog/what-is-jev)
- [arXiv 2609.30216, Jev in the Wild](https://arxiv.org/pdf/2609.30216)
- [arXiv 2609.34963, JevVibe](https://arxiv.org/pdf/2609.34963)
- [arXiv 2609.30243, JevOut](https://arxiv.org/pdf/2609.30243)
- [arXiv 2610.14112, lingue regionali indonesiane](https://arxiv.org/pdf/2610.14112)
