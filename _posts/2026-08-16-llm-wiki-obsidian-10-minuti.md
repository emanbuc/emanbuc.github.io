---
layout: post
title: "LLM Wiki con Obsidian - come iniziare in meno di 10 minuti"
date: 2026-08-16 10:00:00
description: "Guida rapida, senza terminale né codice, per far partire una LLM Wiki su Obsidian con il plugin Karpathy LLM Wiki e una chiave API gratuita di Google Gemini."
categories: ai
tags: obsidian llm-wiki gemini quickstart
---

Questa guida presuppone che tu abbia già letto
[Cos'è una LLM Wiki]({% link _posts/2026-08-16-cosa-e-una-llm-wiki.md %}) e sappia, a grandi linee,
di cosa si tratta. Qui ci concentriamo solo su una cosa: **farla partire sul tuo computer**, senza
toccare mai un terminale, senza scrivere una riga di codice.

> Se in futuro vorrai anche il backup automatico su GitHub, la sincronizzazione fra più dispositivi
> o l'ottimizzazione avanzata della quota gratuita, trovi tutto nella
> [guida tecnica completa del progetto]({% link _posts/2026-08-16-creare-llm-wiki-obsidian.md %}).
> Qui invece ci fermiamo al minimo indispensabile per iniziare oggi stesso.

## Cosa ti serve prima di partire

- Un computer (Windows, Mac o Linux) — su smartphone e tablet si può leggere e interrogare la
  wiki, ma è meglio non costruirla lì.
- Una connessione internet.
- Un account Google qualsiasi (anche quello che usi già per Gmail va benissimo).
- Circa 30 minuti per la prima configurazione.

Non ti servono: terminale, Python, Git, account GitHub, carta di credito.

---

## Passo 1 — Installa Obsidian

Obsidian è il programma gratuito su cui si appoggia tutto: è, in sostanza, un editor di note
collegate fra loro con dei link.

1. Vai su **obsidian.md**.
2. Clicca su "Download" e scegli la versione per il tuo sistema operativo.
3. Installa il programma come faresti con qualsiasi altra applicazione.
4. Aprilo. Alla prima schermata, scegli **"Create new vault"** ("vault" è semplicemente il nome
   che Obsidian dà a una cartella di note).
5. Dai un nome al vault (es. "La mia wiki") e scegli dove salvarlo sul tuo computer. Va bene
   qualsiasi posizione: Documenti, Desktop, una cartella dedicata.

A questo punto hai una cartella vuota, gestita da Obsidian. È qui che vivrà tutto: le tue note
originali e la wiki che l'IA costruirà a partire da esse.

## Passo 2 — Installa il "motore" della wiki: il plugin LLM Wiki

Un plugin è un componente aggiuntivo che estende le funzioni di Obsidian. Quello che ci interessa
si chiama **"Karpathy LLM Wiki"** ed è lui a occuparsi di leggere le tue note e costruire la rete
di pagine collegate.

1. In Obsidian, apri le **Impostazioni** (l'icona a forma di ingranaggio, in basso a sinistra).
2. Vai su **Community plugins**.
3. La prima volta ti verrà chiesto di attivare i plugin di comunità: conferma.
4. Clicca su **Browse** e cerca **"Karpathy LLM Wiki"**.
5. Clicca **Install**, poi **Enable**.

## Passo 3 — Procurati una chiave gratuita per l'intelligenza artificiale

La wiki ha bisogno di appoggiarsi a un'intelligenza artificiale per leggere e capire le tue note.
Usiamo **Google Gemini**, che ha un piano gratuito adatto a uso personale. Per usarlo serve una
"chiave API": un codice segreto, legato al tuo account Google, che autorizza il plugin a fare
richieste all'IA per tuo conto — è l'equivalente digitale di una tessera d'accesso.

1. Vai su **aistudio.google.com** e accedi con il tuo account Google.
2. Cerca il pulsante **"Get API key"**.
3. Clicca su **"Create API key"** e scegli l'opzione per crearla in un nuovo progetto.
4. Comparirà un codice che inizia con `AIza...`. **Copialo e conservalo** in un posto sicuro (per
   esempio in un gestore di password): chi lo possiede può usarlo per fare richieste a tuo nome,
   quindi non condividerlo né incollarlo in luoghi pubblici.

Non serve inserire un metodo di pagamento per il piano gratuito.

## Passo 4 — Collega la chiave al plugin

Torna in Obsidian:

1. Impostazioni → cerca **"Karpathy LLM Wiki"** nel menu a sinistra (sotto Community plugins).
2. Nel campo **Provider**, scegli **"Google Gemini"**.
3. Incolla la chiave copiata al passo precedente nel campo **API Key**.
4. Nel campo del modello, scrivi `gemini-2.5-flash` (è un buon punto di partenza: veloce e già
   capace).
5. Clicca su **Test Connection**. Se vedi una spunta verde, sei collegato e pronto.

## Passo 5 — Prepara le tue prime note

Dentro il tuo vault, crea una cartella chiamata `sources` (puoi farlo con il pulsante "nuova
cartella" nel pannello a sinistra di Obsidian). Qui metterai il materiale grezzo di partenza:
appunti, articoli salvati, riassunti di riunioni.

Per iniziare, non serve un archivio enorme: **3-5 note reali** bastano per vedere come funziona,
prima di buttarci dentro tutto quello che hai.

Il formato ideale è testo semplice (file `.md`, quelli che crea Obsidian stesso) o PDF di piccole
dimensioni. Se hai appunti sparsi in Word, email o note del telefono, il modo più semplice è
copiare-incollare il testo in una nuova nota di Obsidian dentro `sources`.

## Passo 6 — Il primo "ingest": trasforma le note in wiki

"Ingest" è il termine che il plugin usa per indicare la lettura e l'elaborazione delle tue note —
pensalo come "far digerire" il materiale all'IA.

1. Apri la palette dei comandi con `Ctrl+P` (Windows/Linux) o `Cmd+P` (Mac).
2. Digita **"Ingest"** e scegli **"🔍 Ingest from folder"**.
3. Seleziona la cartella `sources`.
4. Conferma e attendi. Per poche note, ci vogliono in genere pochi minuti: il plugin sta chiamando
   l'IA per leggere ogni nota, estrarre persone, concetti e collegamenti.

Al termine, troverai una nuova cartella chiamata `wiki` dentro il tuo vault, con dentro le pagine
generate automaticamente.

## Passo 7 — Esplora quello che è stato creato

Dentro `wiki` trovi tre tipi di pagine:

- **`wiki/entities`** — una pagina per ogni persona, azienda o progetto ricorrente nelle tue note.
- **`wiki/concepts`** — una pagina per ogni idea, metodo o argomento ricorrente.
- **`wiki/index.md`** — la pagina "indice", il punto di partenza da cui esplorare tutto il resto
  cliccando sui link.

Apri `wiki/index.md` e comincia a cliccare sui link blu: è la stessa esperienza di navigare
Wikipedia, ma con contenuti generati dal tuo materiale.

## Passo 8 — Fai una domanda alla tua wiki

Questa è la parte più utile nell'uso quotidiano.

1. `Ctrl/Cmd+P` → digita **"Query wiki"**.
2. Scrivi una domanda in linguaggio naturale, per esempio: "Cosa avevo scritto sul progetto X?"
   oppure "Chi ha partecipato alla riunione di marzo?"
3. Il plugin risponde citando le pagine e le note originali da cui ha preso l'informazione, con
   link cliccabili per verificare tu stesso la fonte.

## Passo 9 — Correggi e "congela" le pagine che vuoi tenere così

Se una pagina generata dall'IA non ti convince, o vuoi migliorarla, modificala come faresti con
qualsiasi nota di Obsidian: è testo normale.

Per evitare che una futura elaborazione riscriva una pagina che hai già sistemato, apri la pagina
e nella parte alta (il "frontmatter", cioè il riquadro con le informazioni tecniche della nota)
cambia `reviewed: false` in `reviewed: true`. Da quel momento la pagina è "congelata" e al sicuro
da sovrascritture automatiche.

## Passo 10 — Ripeti nel tempo

Quando accumuli nuove note:

1. Trascinale nella cartella `sources`.
2. Rilancia **"Ingest from folder"**: il plugin salta automaticamente le note già elaborate,
   quindi non consuma tempo né risorse su ciò che ha già letto.
3. Ogni tanto (per esempio una volta a settimana), lancia **"🛠️ Lint wiki"**: è un controllo
   automatico che segnala pagine vuote, duplicati o collegamenti rotti, così puoi sistemarli con
   un clic invece di scoprirli per caso.

---

## Un backup semplice, senza complicazioni

Il tuo vault è solo una cartella sul tuo computer: se il disco si rompe, perdi tutto. La soluzione
più semplice, senza toccare Git o GitHub, è tenerla dentro un servizio di sincronizzazione cloud
che probabilmente già usi — Google Drive, Dropbox, OneDrive, iCloud. Basta creare il vault dentro
quella cartella sincronizzata invece che sul Desktop.

Se in futuro vorrai qualcosa di più robusto — uno storico completo delle modifiche, la possibilità
di lavorare da più dispositivi senza conflitti — quella è la parte "avanzata" descritta nella
[guida tecnica del progetto]({% link _posts/2026-08-16-creare-llm-wiki-obsidian.md %}), che usa
Git e GitHub. Non è necessaria per iniziare.

## Una cosa importante sulla privacy

Il piano gratuito di Google può usare il testo che invii per migliorare i propri prodotti. In
pratica: va benissimo per appunti personali, letture, progetti che non contengono dati sensibili
di altre persone. Se hai materiale riservato — informazioni di lavoro coperte da riservatezza,
dati personali di terzi — non inserirlo in questa wiki collegata al piano gratuito. Per quel tipo
di materiale serve un approccio diverso (un modello che gira interamente sul tuo computer, senza
inviare nulla online): se ti interessa, è un argomento che possiamo approfondire a parte.

## Se qualcosa non funziona

| Problema                                         | Cosa fare                                                                                                                                                                          |
| ------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Il pulsante "Test Connection" resta rosso        | Ricontrolla di aver incollato la chiave per intero, senza spazi extra all'inizio o alla fine                                                                                     |
| L'ingest si ferma con un errore "quota esaurita" | Hai raggiunto il limite giornaliero gratuito. Aspetta il giorno dopo (il conteggio si azzera intorno alle 9-10 del mattino, ora italiana) oppure riduci il numero di note per volta |
| Le pagine generate sembrano confuse o imprecise  | Normale nelle prime prove: correggi manualmente e imposta `reviewed: true` sulle pagine sistemate. Con l'uso, e con un modello leggermente più capace, la qualità migliora        |

---

## Riepilogo in 10 passi

1. Installa Obsidian e crea un vault.
2. Installa il plugin "Karpathy LLM Wiki".
3. Crea una chiave API gratuita su aistudio.google.com.
4. Collega la chiave nelle impostazioni del plugin.
5. Metti 3-5 note in una cartella `sources`.
6. Lancia "Ingest from folder".
7. Esplora `wiki/index.md`.
8. Fai una domanda con "Query wiki".
9. Correggi e congela (`reviewed: true`) le pagine che rivedi.
10. Ripeti con nuove note, e lancia "Lint wiki" ogni tanto.

Da qui in avanti, l'unica cosa che devi ricordarti di fare è continuare a scrivere le tue note come
hai sempre fatto. Il resto — leggere, collegare, aggiornare — non è più un tuo compito.
