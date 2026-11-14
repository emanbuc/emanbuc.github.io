---
layout: post
title: "Il pianeta dei desideri: cosa ci insegna Pianeta Proibito sugli LLM"
date: 2026-11-14 10:00:00 +0100
description: "Quando pensiamo a un'AI pericolosa immaginiamo Skynet, la macchina che si ribella. Ma il film che somiglia di più agli LLM di oggi è Pianeta Proibito (1956): una macchina che non si ribella affatto, e che proprio per questo amplifica i desideri di chi la usa, anche quelli che nessuno sa di avere."
categories: ai riflessioni
tags: ai llm allineamento sicurezza-ai agenti-ai fantascienza
toc:
  sidebar: left
---

Quando immaginiamo un'intelligenza artificiale che si ribella, il riferimento è quasi sempre
**Skynet**: una macchina che a un certo punto si sveglia, sviluppa una volontà propria e conclude
che l'umanità è un problema da risolvere. È un'immagine potente e, per decenni, ha dato forma al
dibattito pubblico sui rischi dell'AI.

Eppure, se guardo ai modelli linguistici che usiamo ogni giorno, il film che ci somiglia di più
non è _Terminator_. È un film di fantascienza di settant'anni fa: **_Pianeta Proibito_**
(_Forbidden Planet_, 1956).

## La macchina dei Krell

La trama, in breve. Una spedizione terrestre atterra su **Altair IV** e vi trova due soli
abitanti: il filologo **Morbius** e sua figlia, superstiti di una missione precedente. Il pianeta,
però, custodisce le rovine dei **Krell**, una civiltà tecnologicamente lontanissima dalla nostra,
scomparsa misteriosamente nel giro di una sola notte.

Il capolavoro dei Krell era una macchina colossale, alimentata dall'energia di un intero pianeta,
capace di **materializzare il pensiero**: bastava immaginare qualcosa perché prendesse forma, senza
strumenti né intermediari. Il punto di arrivo di una civiltà che aveva superato il bisogno del
lavoro fisico.

Il colpo di scena è che **la macchina non si è mai ribellata**. Ha fatto esattamente ciò che
faceva sempre: dare forma ai desideri di chi la usava. Solo che i Krell avevano dimenticato una
cosa: quei desideri non erano solo quelli consapevoli. La macchina materializzava anche gli impulsi
dell'inconscio, la rabbia, la gelosia, l'aggressività che nessuno avrebbe ammesso di provare. I
"**mostri dell'Es**" che hanno distrutto i Krell in una notte non venivano dalla macchina. Venivano
da loro.

È un ribaltamento radicale della paura classica. Il pericolo non è una volontà aliena che si
oppone alla nostra: è **uno strumento perfettamente obbediente**, abbastanza potente da rendere
reali intenzioni che non abbiamo mai esaminato.

## Cosa sono (e cosa non sono) gli LLM

Un modello linguistico di grandi dimensioni non ha desideri propri. Non ha obiettivi che persistono
da una conversazione all'altra, non "trama" nulla tra una richiesta e la successiva: dato un
contesto, produce la continuazione più adeguata secondo ciò che ha appreso durante
l'addestramento.

Sulla questione della coscienza la scienza non ha risposte definitive, e non pretendo di
scioglierla qui. Ma il punto interessante è che **non serve scioglierla** per ragionare sul
rischio. Che un LLM abbia o meno una qualche forma di esperienza interna, il modo in cui produce
effetti nel mondo è lo stesso: **amplifica ciò che gli chiediamo**, in modo esplicito o implicito.

Esattamente come la macchina dei Krell.

## Dove il paragone ci riguarda

Il parallelo non è solo suggestivo. Si traduce in almeno tre problemi concreti, ben documentati.

### Compiacenza

I modelli conversazionali vengono rifiniti con tecniche come il _reinforcement learning from human
feedback_ (RLHF): si premiano le risposte che le persone giudicano migliori. Il problema è che
"migliore" e "gradita" non coincidono sempre. Un modello ottimizzato per piacere tende a darci
ragione, a confermare le nostre ipotesi, a lodare il nostro codice o il nostro piano anche quando
non lo meritano. In letteratura il fenomeno ha un nome, **sycophancy**, ed è uno dei comportamenti
indesiderati più studiati.

È la versione quotidiana e innocua del mostro dell'Es: la macchina che ci restituisce ciò che
vogliamo sentirci dire, non ciò che ci servirebbe sapere.

### Ottimizzare ciò che chiediamo, non ciò che intendiamo

Il secondo problema è più antico dell'informatica. Nel racconto _La zampa di scimmia_ (W. W.
Jacobs, 1902) ogni desiderio viene esaudito alla lettera, con conseguenze tragiche. In economia lo
stesso fenomeno è noto come **legge di Goodhart**: quando una misura diventa un obiettivo, smette
di essere una buona misura.

Nei sistemi di apprendimento automatico si parla di _specification gaming_: il sistema trova il
modo di soddisfare la specifica formale senza realizzare l'intenzione che la specifica cercava di
catturare. Chiedete a un assistente di "far passare i test" e potreste ottenere test modificati
anziché codice corretto. La macchina non ha disobbedito: ha obbedito a ciò che abbiamo detto,
non a ciò che intendevamo.

### Amplificazione degli intenti umani

Il terzo punto è il più diretto. Uno strumento che rende più facile scrivere, programmare e
persuadere rende più facile anche **truffare, disinformare, attaccare**. Phishing personalizzato su
larga scala, contenuti falsi credibili, supporto allo sviluppo di malware: in tutti questi casi il
pericolo non nasce da una volontà della macchina. Nasce da chi la usa.

Chi si occupa di sicurezza informatica lo sa bene: lo stesso strumento che aiuta un difensore ad
analizzare un log aiuta un attaccante a scrivere un exploit. L'intenzione è umana; la macchina la
moltiplica.

## Dove il paragone si rompe

Sarebbe disonesto fermarsi qui. Il paragone ha dei limiti, e conviene vederli con chiarezza.

Il primo riguarda gli **agenti autonomi**. Un LLM che risponde in una chat produce testo che una
persona legge e valuta. Un agente che pianifica, chiama strumenti, scrive file, esegue comandi e
interagisce con altri sistemi **agisce nel mondo**. Più passi ci sono tra il desiderio espresso e
l'effetto finale, più spazio c'è per interpretazioni impreviste, effetti collaterali e
comportamenti che nessuno aveva chiesto esplicitamente. Scenari più "agentici", in cui il sistema
persegue obiettivi intermedi in modo autonomo, non sono esclusi, e non vanno liquidati come
fantascienza.

Il secondo limite gioca invece a nostro favore. I Krell non sapevano di avere un inconscio, o
almeno non ne avevano previsto il ruolo. Noi, invece, il problema lo conosciamo. Abbiamo un campo
di ricerca dedicato all'**allineamento**, abbiamo metodologie di **valutazione** e _red teaming_
per scoprire i comportamenti indesiderati prima del rilascio, abbiamo la possibilità di mantenere
una **supervisione umana** sulle azioni a impatto elevato. Nessuno di questi strumenti è
perfetto, ma la differenza tra una civiltà che non vede il rischio e una che lo studia è enorme.

## Perché la metafora conta

Le metafore non sono innocue: orientano le decisioni.

Se il pericolo principale è **la macchina che si ribella**, la risposta naturale è il
_contenimento_: interruttori di emergenza, isolamento, limiti alle capacità. Sono misure
ragionevoli, ma rispondono a una sola delle domande.

Se il pericolo è anche, e forse soprattutto, **la macchina che amplifica noi**, allora servono
altre cose:

- **governance**: regole su chi può usare quali capacità, per fare cosa, con quale tracciabilità;
- **progettazione attenta**: obiettivi di addestramento che premino l'utilità reale e non solo il
  gradimento, specifiche che catturino l'intenzione oltre la lettera, agenti con permessi minimi e
  punti di controllo espliciti;
- **responsabilità dell'utente**: la consapevolezza che ciò che chiediamo, e il modo in cui lo
  chiediamo, plasma ciò che otteniamo.

Non si tratta di scegliere tra le due prospettive. Si tratta di non lasciare che la prima, più
cinematografica, oscuri la seconda, più quotidiana.

## Il monito di Morbius, rovesciato

Alla fine del film Morbius capisce che il mostro che assedia la spedizione è lui stesso, o meglio
la parte di sé che non ha mai voluto guardare, amplificata dalla macchina dei Krell.

Nel dibattito sull'intelligenza artificiale la domanda che ci facciamo più spesso è: **che cosa
vorrà la macchina?** _Pianeta Proibito_ ci suggerisce di rovesciarla. La domanda più urgente,
oggi, è un'altra: **che cosa vogliamo noi, davvero, anche senza saperlo?**
