---
layout: post
title: "Creare un LLM Wiki con Obsidian"
date: 2026-08-16 09:30:00
description: "Guida operativa per costruire una LLM Wiki a costo zero con Obsidian, il plugin Karpathy LLM Wiki, l'API gratuita di Google Gemini e un vault versionato su GitHub."
categories: ai
tags: obsidian llm-wiki gemini github litellm
---

**Guida operativa completa** — build di una knowledge base auto-linkata a costo zero.

Se ti sei chiesto [cos'è una LLM Wiki]({% link _posts/2026-08-16-cosa-e-una-llm-wiki.md %}) e vuoi
passare dalla teoria alla pratica, questa guida copre lo stack concreto: Obsidian + plugin
_Karpathy LLM Wiki_ (`green-dalii/obsidian-llm-wiki`) + API key Google AI Studio (piano gratuito) +
vault versionato su GitHub.

> **Data della guida:** agosto 2026. I limiti di quota Google cambiano senza preavviso e senza
> versioning: la sezione 4 va **riverificata sulla pagina ufficiale** prima di dimensionare il
> workflow. Tutti i valori riportati sono indicativi.
>
> **Aggiornamento verificato il 16/08/2026** sulla dashboard `aistudio.google.com/rate-limit` di un
> progetto reale (§4.4, §5.1, §5.4, §5.6): Gemini 2.0 Flash/Flash-Lite risultano **ritirati** (0/0),
> `gemini-2.5-pro` non compare più come opzione free, ed è disponibile una nuova generazione
> (Gemini 3.1/3.5/3.6 Flash e Flash-Lite) con quote più generose sui modelli Lite. I numeri di
> questa revisione sono uno snapshot di un singolo progetto, non un valore universale: **rileggi la
> tua dashboard prima di fidartene**.

---

## 0. Architettura in una pagina

```
┌───────────────────────────────────────────────────────────────┐
│  VAULT OBSIDIAN (cartella locale = working tree git)          │
│                                                                 │
│   sources/          note e PDF di partenza (read-only)         │
│   wiki/                                                        │
│     ├── sources/    riassunti delle fonti + backlink           │
│     ├── entities/   1 pagina per entità (persone, org...)      │
│     ├── concepts/   1 pagina per concetto (teorie, metodi)     │
│     └── index.md    hub centrale con alias                     │
│   .obsidian/        config plugin (ATTENZIONE alle chiavi)     │
└─────────────────┬─────────────────────────┬─────────────────────┘
                   │                         │
        plugin Git │                         │ plugin LLM Wiki
                   ▼                         ▼
        ┌──────────────────┐      ┌──────────────────────────┐
        │ GitHub (repo      │      │ Gateway locale opzionale │
        │ PRIVATO)          │      │ (LiteLLM :4000)          │
        └──────────────────┘      └────────────┬──────────────┘
                                                 ▼
                                     ┌──────────────────────────┐
                                     │ Google AI Studio          │
                                     │ Gemini free tier          │
                                     │ (multi-modello, multi-    │
                                     │  progetto, fallback)      │
                                     └──────────────────────────┘
```

Il gateway locale (sezione 5.4) è la parte che trasforma "quota gratuita risicata" in "quota
gratuita utilizzabile": il plugin vede **un solo endpoint OpenAI-compatibile**, il gateway dietro
fa routing, fallback e rotazione fra modelli e chiavi.

---

## 1. Prerequisiti

| Componente      | Versione / requisito | Note                                                                          |
| --------------- | --------------------- | ------------------------------------------------------------------------------ |
| Obsidian        | **1.11.4+**            | requisito minimo del plugin LLM Wiki                                          |
| Git             | 2.30+ sul desktop      | il plugin Obsidian Git su mobile usa isomorphic-git, non serve git di sistema |
| Account GitHub  | qualsiasi piano        | repo **privato**                                                              |
| Account Google  | qualsiasi              | per AI Studio                                                                  |
| Python 3.10+    | opzionale              | solo se usi il gateway LiteLLM                                                |
| Ollama          | opzionale              | fallback locale quando la quota giornaliera è esaurita                        |

Non servono terminale, Python o app esterne per il funzionamento base: il plugin gira interamente
dentro Obsidian.

---

## 2. Vault Obsidian versionato su GitHub

### 2.1 Creare il repository

Crea su GitHub un repo **privato** (es. `llm-wiki-vault`). Privato non è un dettaglio estetico: il
vault conterrà `.obsidian/plugins/*/data.json`, e per alcuni plugin quel file può contenere segreti
(vedi 2.4 e 7).

```bash
# sul desktop
mkdir -p ~/Vaults/llm-wiki && cd ~/Vaults/llm-wiki
git init -b main
git remote add origin https://github.com/<utente>/llm-wiki-vault.git
```

Apri poi Obsidian → _Open folder as vault_ → seleziona `~/Vaults/llm-wiki`.

### 2.2 `.gitignore` del vault

Questo è il file che decide se il repo resta pulito o diventa ingestibile. Salvalo come
`.gitignore` nella root del vault:

```gitignore
# --- Stato locale di Obsidian: NON versionare ---
.obsidian/workspace.json
.obsidian/workspace-mobile.json
.obsidian/cache
.obsidian/graph.json

# --- Segreti e credenziali ---
# esclude i data.json dei plugin che possono contenere API key
.obsidian/plugins/*/data.json
.env
*.key
secrets/

# --- Cache e artefatti del plugin LLM Wiki ---
.obsidian/plugins/karpathy-llm-wiki/cache/
**/.pdf-cache/

# --- Sistema ---
.DS_Store
Thumbs.db
.trash/
```

> **Trade-off consapevole:** escludendo tutti i `data.json` perdi il versionamento delle
> impostazioni dei plugin. Se preferisci versionarle, escludi _solo_ quelle dei plugin che toccano
> credenziali:
>
> ```gitignore
> .obsidian/plugins/karpathy-llm-wiki/data.json
> .obsidian/plugins/obsidian-git/data.json
> ```
>
> e verifica con `git show :.obsidian/plugins/.../data.json | grep -i key` prima di ogni push
> iniziale.

Se ingerisci PDF pesanti, aggiungi Git LFS:

```bash
git lfs install
git lfs track "*.pdf"
git add .gitattributes
```

### 2.3 Primo commit

```bash
git add .
git commit -m "chore: bootstrap vault"
git push -u origin main
```

### 2.4 Plugin Obsidian Git

Settings → Community plugins → Browse → **Git** (autore _Vinzent03_) → Install → Enable.

Configurazione consigliata in Settings → Git:

| Impostazione                          | Valore                                | Perché                                                                       |
| -------------------------------------- | -------------------------------------- | ------------------------------------------------------------------------------ |
| Vault backup interval (minutes)        | `0`                                     | disattiva il commit automatico a tempo: con l'ingest LLM vuoi commit semantici, non ogni 10 minuti |
| Auto pull on startup                   | ✅                                      | evita divergenze quando lavori su più macchine                                |
| Auto push on commit-and-sync           | ✅                                      |                                                                                 |
| Commit message                         | `wiki: {{date}} — {{numFiles}} file`    | rende leggibile la history                                                     |
| Pull before push                       | ✅                                      | riduce i reject                                                                |
| Disable notifications                  | ❌                                      | vuoi vedere gli errori                                                         |
| Line authoring / Source control view   | abilitato                              | comodo per review dei diff generati dall'LLM                                  |

Comandi utili (`Ctrl/Cmd+P`): _Git: Commit-and-sync_, _Git: Open source control view_, _Git: Pull_,
_Git: Edit .gitignore_.

**Autenticazione.** Usa un **fine-grained personal access token** GitHub, non la password:

1. GitHub → Settings → Developer settings → Personal access tokens → **Fine-grained tokens** →
   Generate new token
2. Repository access: _Only select repositories_ → `llm-wiki-vault`
3. Permissions → Repository permissions → **Contents: Read and write** (è l'unico permesso
   necessario)
4. Scadenza: 90 giorni, con promemoria in calendario

Sul desktop, salva il token nel credential helper del sistema invece che in chiaro:

```bash
# Linux
git config --global credential.helper "libsecret"
# macOS
git config --global credential.helper osxkeychain
# Windows
git config --global credential.helper manager
```

Al primo push git chiede username e password: come password incolla il token.

### 2.5 Mobile (iOS / Android)

Il plugin Git funziona anche su mobile (backend isomorphic-git), con due limiti pratici da
conoscere in anticipo:

- **clone iniziale lento e pesante in RAM** su vault grandi → usa `depth` ridotta (shallow clone)
  nelle impostazioni del plugin, oppure copia il vault via file manager e poi lascia che il plugin
  riconosca il `.git` esistente;
- **niente merge a tre vie completo**: in caso di conflitto il plugin ti fa scegliere una versione.
  Regola operativa: **un solo dispositivo scrive alla volta**, e prima di aprire il vault su mobile
  fai _Pull_.

Su mobile l'ingest LLM è comunque sconsigliato (batch lunghi + app in background = richieste
interrotte). Mobile = lettura + query + note nuove; desktop = ingest.

### 2.6 Igiene della history

Il plugin LLM Wiki **non riscrive mai i file sorgente**, tocca solo `wiki/`. Questo rende i diff
leggibili: un ingest produce un commit in cui vedi esattamente quali entità e concetti sono nati o
cambiati.

Convenzione consigliata per i messaggi di commit, così la history resta grep-abile:

```
src:  aggiunta di note/PDF sorgente
wiki: output di un ingest
fix:  correzioni manuali su pagine wiki
lint: risultato di Lint wiki / Smart Fix All
cfg:  modifiche a .obsidian/
```

---

## 3. Installazione del plugin LLM Wiki

### 3.1 Installazione

Settings → Community plugins → Browse → cerca **"Karpathy LLM Wiki"** → Install → Enable.

(Se il plugin non compare nella marketplace della tua versione, l'alternativa è BRAT: installa
_Obsidian42 - BRAT_, poi "Add beta plugin" con `green-dalii/obsidian-llm-wiki`. In quel caso
aggiungi la cartella del plugin al `.gitignore` finché non si stabilizza.)

### 3.2 Cosa genera

L'output vive tutto sotto `wiki/`:

- `wiki/sources/` → un riassunto per fonte, con backlink alla nota originale
- `wiki/entities/` → una pagina per entità estratta (persone, organizzazioni, prodotti, eventi)
- `wiki/concepts/` → una pagina per concetto (teorie, metodi, termini di dominio)
- `wiki/index.md` → hub con tutte le pagine e i loro alias

Ogni pagina ha frontmatter con `type`, `tags`, `aliases` (varianti cross-lingua e sigle), `source`
e `reviewed`. **`reviewed: true` protegge la pagina dalla sovrascrittura**: è il tuo meccanismo per
congelare le pagine che hai curato a mano.

### 3.3 Comandi principali

| Comando                     | Quando usarlo                                                             |
| ---------------------------- | --------------------------------------------------------------------------- |
| 📥 Ingest single source      | test iniziale, note singole importanti                                     |
| 📁 Ingest from folder        | batch su una cartella, con skip dei già ingeriti                           |
| 📑 Ingest multiple files     | selezione puntuale con coda e cancel per file                              |
| 🔍 Query wiki                | Q&A groundato sul vault, risposte con `[[wiki-link]]`                      |
| 🛠️ Lint wiki                 | duplicati, link morti, pagine vuote, orfane, contraddizioni                |
| ⚡ Smart Fix All             | riparazione automatica in ordine causale                                   |
| 🔄 Regenerate index          | ricostruisce `wiki/index.md`                                               |
| ⏹️ Cancel                    | ferma al prossimo batch boundary (**fondamentale con le quote free**)      |
| 📜 Ingestion history         | storico ricercabile delle operazioni                                       |

### 3.4 Impostazioni che pesano sulla quota

Nella sezione LLM/Wiki delle impostazioni del plugin, tre parametri determinano il consumo:

- **Extraction Granularity** → `Coarse` / `Minimal` / `Fine` / `Custom`. È la leva principale:
  `Fine` moltiplica entità e concetti estratti, quindi chiamate e token. **Parti da `Coarse`.**
- **Max Tokens** → tetto per richiesta. Tenerlo basso non risparmia quota RPD (che conta le
  richieste) ma protegge dal TPM.
- **Tag Vocabulary → Custom** → definire un vocabolario di tag proprio è _schema injection_:
  riduce la deriva dei modelli piccoli/veloci, quindi riduce i giri di lint e ri-ingest. Con Gemini
  Flash-Lite è quasi obbligatorio.

Il plugin **non usa embedding**: il retrieval è a cascata (match lessicale → generazione keyword
via LLM → scan substring → fallback semantico → Personalized PageRank sul grafo dei `[[link]]`).
Conseguenza pratica per il free tier: **una query costa 0–2 chiamate LLM, non una chiamata di
embedding per nota**. È il motivo per cui questo stack è sostenibile a quota gratuita.

### 3.5 I tre slot di modello dell'interfaccia

Le impostazioni del plugin non espongono un campo Model per comando, ma **tre slot indipendenti**
che raggruppano i comandi di 3.3 per natura dell'azione:

| Slot nell'interfaccia            | Comandi coperti                                                        |
| ---------------------------------- | -------------------------------------------------------------------------- |
| **Acquisizione**                  | 📥 Ingest single source, 📁 Ingest from folder, 📑 Ingest multiple files |
| **Manutenzione e riparazione**    | 🛠️ Lint wiki, ⚡ Smart Fix All, 🔄 Regenerate index                       |
| **Query**                         | 🔍 Query wiki                                                            |

Questo è il livello di granularità reale su cui puoi assegnare modelli diversi (§5.1): non per
singolo comando, ma per questi tre gruppi.

---

## 4. API key Google AI Studio

### 4.1 Creare la chiave

1. Vai su **aistudio.google.com** → _Get API key_
2. _Create API key_ → scegli **"Create API key in new project"** (importante: vedi 5.2)
3. Copia la chiave: `AIza...`

### 4.2 Due modi per collegarla al plugin

**Modo A — provider nativo (consigliato per iniziare)**

Settings → Karpathy LLM Wiki → Provider: **Google Gemini** → incolla API key → Model name:
`gemini-2.5-flash` → **Test Connection**.

**Modo B — endpoint OpenAI-compatibile (necessario per gateway e rotazione)**

Provider: **Custom OpenAI-Compatible**

```
Base URL:  https://generativelanguage.googleapis.com/v1beta/openai/
API Key:   AIza...
Model:     gemini-2.5-flash
```

Il modo B è quello che ti serve nella sezione 5.4, perché puoi sostituire il base URL con
`http://localhost:4000/v1` senza toccare altro.

Verifica rapida da terminale prima di configurare Obsidian:

```bash
curl -s https://generativelanguage.googleapis.com/v1beta/openai/chat/completions \
  -H "Authorization: Bearer $GEMINI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"model":"gemini-2.5-flash","messages":[{"role":"user","content":"ping"}]}' \
  | head -c 400
```

### 4.3 Avvertenza privacy (non opzionale, per il tuo profilo d'uso)

Sul **piano gratuito** Google può usare prompt e output per migliorare i propri prodotti e
modelli. Il piano a pagamento no. Conseguenze dirette:

- non ingerire nel vault materiale coperto da NDA, dati personali di terzi, o risultati di ricerca
  non ancora pubblicati che vuoi tenere riservati;
- se il vault mescola materiale pubblico e riservato, tieni **due vault separati** con provider
  diversi (free tier per il pubblico, Ollama locale per il riservato) invece di sperare nella
  disciplina;
- per il materiale sensibile: provider `Ollama` o `LM Studio` nel plugin, tutto resta sulla
  macchina.

### 4.4 I limiti (da verificare)

Tre metriche, tutte simultanee:

- **RPM** → richieste al minuto, finestra mobile di 60 s
- **TPM** → token al minuto, input + output sommati
- **RPD** → richieste al giorno, **reset a mezzanotte Pacific Time** = 09:00 ora italiana (ora
  legale) / 10:00 (ora solare)

**La pagina ufficiale `ai.google.dev/gemini-api/docs/rate-limits` non pubblica più tabelle
numeriche per modello**: dal 2026 rimanda esplicitamente alla dashboard live del tuo progetto
(`aistudio.google.com/rate-limit`). Qualunque tabella statica — inclusa quella qui sotto — è quindi
uno **snapshot**, non un contratto.

Snapshot verificato il 16/08/2026 su un progetto reale (finestra 28 giorni, picco di utilizzo vs
limite):

| Modello                                            | RPM | RPD    | Stato                                                                          |
| ---------------------------------------------------- | ---- | ------ | --------------------------------------------------------------------------------- |
| `gemini-3.5-flash-lite`                              | 15   | **500** | disponibile, best bucket per volume                                            |
| `gemini-3.1-flash-lite`                              | 15   | **500** | disponibile, bucket indipendente dal precedente                                |
| `gemini-3.6-flash`                                   | 5    | 20      | disponibile, quota minuscola                                                    |
| `gemini-2.5-flash`                                   | 5    | 20      | quota residua bassissima (coerente con i tagli riportati da dicembre 2025)      |
| `gemini-2.5-flash-lite`                              | 10   | 20      | non più il "cavallo da tiro": oggi rende solo 20 richieste/giorno              |
| `gemini-2.0-flash` / `gemini-2.0-flash-lite`         | 0    | 0       | **ritirati**, non disponibili                                                  |
| `gemini-2.5-pro` / modelli Pro                       | —    | —       | **assente dalla lista**: nessun Pro gratuito su questo progetto                |

Il vincolo che conta davvero per questo workflow resta **RPD**, non RPM — ma con questi numeri il
divario fra i modelli Lite di ultima generazione (500 RPD) e tutto il resto (20 RPD) è enorme, e
determina quasi da solo la strategia del capitolo 5.

**Come leggere i limiti veri sul tuo progetto:**

1. `aistudio.google.com/rate-limit` → è oggi l'unica fonte affidabile, lo dice la stessa doc
   ufficiale;
2. in alternativa Google Cloud Console → _IAM & Admin → Quotas_, filtro su _Generative Language
   API_;
3. empiricamente, leggendo gli header di risposta e il corpo dell'errore 429.

---

## 5. Strategia per sfruttare al massimo il piano gratuito

Questa è la parte che fa la differenza fra "ingerisco 20 note e mi blocco" e "ingerisco un archivio
intero in una settimana".

### 5.1 Principio: modelli diversi hanno budget diversi e indipendenti

I limiti sono **per modello**. Esaurire `gemini-2.5-flash` non tocca la quota di
`gemini-2.5-flash-lite`. Il moltiplicatore gratuito è quindi assegnare un modello diverso a
ciascuno dei **tre slot reali dell'interfaccia** (§3.5), non un tiering ipotetico per fase:

| Slot                            | Modello (snapshot 16/08/2026)              | Razionale                                                                                                                          |
| --------------------------------- | --------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------- |
| **Acquisizione**                 | `gemini-3.5-flash-lite` (RPM 15 / RPD 500)    | è lo slot a volume più alto per definizione (§5.3): gli va riservato il bucket più capiente disponibile                            |
| **Manutenzione e riparazione**   | `gemini-3.6-flash` (RPM 5 / RPD 20)           | è lo slot con meno chiamate ma più impatto sulla qualità (lint, dedup, contraddizioni): merita il modello migliore, anche con RPD basso |
| **Query**                        | `gemini-3.1-flash-lite` (RPM 15 / RPD 500)    | **modello diverso da Acquisizione apposta**: essendo un modello diverso, il bucket RPD è indipendente anche restando sullo stesso progetto/chiave, quindi le query quotidiane non intaccano la quota dell'ingest |

Con questi tre modelli, Acquisizione e Query hanno **1000 RPD combinati** invece di condividere un
unico bucket, e Manutenzione resta isolata sul modello di qualità migliore disponibile — anche se
con soli 20 RPD (vedi §5.6 per come dimensionare il lavoro di lint su quel budget).

`gemini-2.5-pro`, indicato in versioni precedenti di questa guida come modello per l'arbitraggio
delle contraddizioni, **non è più un'opzione gratuita affidabile** (§4.4): non pianificarci sopra.
Se ti serve occasionalmente più capacità di ragionamento per lo slot Manutenzione, le alternative
sono: (a) un modello Flash come surrogato di qualità inferiore ma gratuito; (b) billing abilitato
solo su un progetto dedicato a questo slot — a volumi bassi la spesa resta trascurabile; (c)
fallback locale via Ollama (§5.4).

Operativamente: **cambia il campo Model dello slot nelle impostazioni del plugin quando serve**,
ma con un gateway LiteLLM (§5.4) non serve nemmeno più farlo manualmente: ogni slot punta a un
alias fisso e il routing avviene dietro.

### 5.2 Multi-progetto: le quote sono per progetto, non per chiave

Regola chiave, spesso fraintesa: **il rate limit si applica al progetto Google Cloud, non alla
singola API key**. Creare cinque chiavi nello stesso progetto non moltiplica nulla.

Creare **progetti distinti** invece dà quote distinte. In AI Studio, ogni volta che scegli
_"Create API key in new project"_ ottieni un budget separato.

> ⚠️ **Nota di merito, non di forma.** Moltiplicare i progetti per aggirare deliberatamente le
> quote è a rischio di violazione dei Termini di Servizio Google, e le sospensioni avvengono a
> livello di account, non di progetto. La lettura difendibile è: progetti separati per **contesti
> d'uso separati** (es. `wiki-personale`, `wiki-ricerca`, `sperimentazione`), che è esattamente ciò
> che Google suggerisce come pratica di isolamento. Tenerne 2–3 è normale amministrazione; tenerne
> 20 con rotazione automatica è un'altra cosa. Scegli consapevolmente.

### 5.3 Dimensionare la giornata di ingest

Stima grezza: **un ingest produce ~2–5 chiamate LLM per nota sorgente** (estrazione entità,
estrazione concetti, deduplica in stage, generazione pagina), fortemente dipendente da granularità
e densità della nota.

Con `flash-lite` a ~1000 RPD:

```
1000 RPD / 3.5 chiamate-per-nota  ≈  285 note/giorno   (granularità Coarse)
1000 RPD / 8   chiamate-per-nota  ≈  125 note/giorno   (granularità Fine)
```

Procedura per calibrare sul tuo materiale reale, senza bruciare la quota a indovinare:

1. **Ingest single source** su 3 note rappresentative;
2. apri **📜 Ingestion history** e leggi il numero di chiamate riportato;
3. dividi la tua RPD per quella media → è la dimensione del batch giornaliero;
4. programma i batch **subito dopo le 09:00–10:00 italiane** (reset RPD), così un eventuale
   sforamento ha 24 ore per rientrare.

### 5.4 Gateway locale con LiteLLM: fallback e rotazione automatici

Il salto di qualità. Invece di puntare il plugin a Google, lo punti a un proxy locale che espone
un'API OpenAI-compatibile e dietro gestisce: fallback automatico su 429, round-robin fra
modelli/progetti, e retry con backoff.

Installazione:

```bash
pip install "litellm[proxy]"
```

`~/.config/litellm/llm-wiki.yaml`:

```yaml
# Mappatura sui 3 slot reali del plugin (§3.5), basata sui limiti verificati
# su AI Studio > Limiti di frequenza il 16/08/2026. Riverifica periodicamente:
# aistudio.google.com/rate-limit (vedi §4.4).

model_list:
  # --- Slot "Acquisizione": RPM 15 / RPD 500 sul modello verificato ---
  - model_name: wiki-acquisizione
    litellm_params:
      model: gemini/gemini-3.5-flash-lite
      api_key: os.environ/GEMINI_KEY_PROJ_A
      rpm: 15   # coincide col limite reale: lascia che LiteLLM metta in coda
                # invece di far sforare il picco (429 osservati a 20/15)

  # --- Slot "Manutenzione e riparazione": RPM 5 / RPD 20 — quota minuscola,
  # secondo backend nello stesso alias per raddoppiare il budget (20+20=40 RPD)
  # quando il lint di giornata supera la prima quota ---
  - model_name: wiki-manutenzione
    litellm_params:
      model: gemini/gemini-3.6-flash
      api_key: os.environ/GEMINI_KEY_PROJ_A
      rpm: 5
  - model_name: wiki-manutenzione
    litellm_params:
      model: gemini/gemini-2.5-flash
      api_key: os.environ/GEMINI_KEY_PROJ_A
      rpm: 5

  # --- Slot "Query": modello DIVERSO da Acquisizione apposta — essendo un
  # modello diverso il bucket RPD è indipendente anche sullo stesso progetto ---
  - model_name: wiki-query
    litellm_params:
      model: gemini/gemini-3.1-flash-lite
      api_key: os.environ/GEMINI_KEY_PROJ_A
      rpm: 15

  # --- Rete di sicurezza locale: quota esaurita su tutti gli slot sopra ---
  - model_name: wiki-local
    litellm_params:
      model: ollama/qwen3:8b
      api_base: http://localhost:11434

router_settings:
  routing_strategy: usage-based-routing-v2
  num_retries: 3
  retry_after: 20
  allowed_fails: 2
  cooldown_time: 120
  fallbacks:
    - wiki-acquisizione: ["wiki-local"]
    - wiki-manutenzione: ["wiki-local"]
    - wiki-query: ["wiki-local"]

litellm_settings:
  drop_params: true
  request_timeout: 300
  cache: true
  cache_params:
    type: local
    ttl: 86400        # 24h: ri-ingest o ri-query dello stesso testo non consuma quota
```

Avvio:

```bash
export GEMINI_KEY_PROJ_A="AIza..."
litellm --config ~/.config/litellm/llm-wiki.yaml --port 4000
```

Configurazione nel plugin (modo B della sezione 4.2), **uno per ciascuno dei tre slot di §3.5**:

```
Provider:  Custom OpenAI-Compatible
Base URL:  http://localhost:4000/v1
API Key:   sk-anything            (o la master key che imposti in LiteLLM)

Slot Acquisizione       → Model: wiki-acquisizione
Slot Manutenzione       → Model: wiki-manutenzione
Slot Query              → Model: wiki-query
```

Cosa guadagni concretamente:

- **`cache: true`** → il beneficio più grosso e meno ovvio. Ri-ingest della stessa nota, retry dopo
  un crash, prove di granularità: tutto servito da cache, quota intatta.
- **`fallbacks`** → quando `flash-lite` risponde 429, la richiesta passa a `flash`, poi a Ollama.
  L'ingest **non si interrompe a metà** lasciando il grafo inconsistente.
- **`rpm` per deployment** → LiteLLM stacca le richieste prima che sia Google a farlo, evitando i
  429 invece di subirli.
- **cambio di modello senza toccare Obsidian** → modifichi il YAML, riavvii il proxy.

Per farlo partire al login (Linux, systemd user unit in
`~/.config/systemd/user/litellm.service`):

```ini
[Unit]
Description=LiteLLM gateway per Obsidian LLM Wiki
After=network-online.target

[Service]
Type=simple
EnvironmentFile=%h/.config/litellm/env
ExecStart=%h/.local/bin/litellm --config %h/.config/litellm/llm-wiki.yaml --port 4000
Restart=on-failure

[Install]
WantedBy=default.target
```

```bash
chmod 600 ~/.config/litellm/env     # contiene le API key: fuori dal vault, fuori da git
systemctl --user enable --now litellm
```

### 5.5 Altre leve, in ordine di rapporto beneficio/sforzo

1. **Granularità `Coarse` in prima passata, `Fine` solo sulle note che contano.** Il tiering vale
   anche per la granularità, non solo per il modello.
2. **Pre-filtra le fonti.** Ingerire tutto l'archivio "perché c'è" è il modo più veloce per
   bruciare la quota su rumore. Una cartella `sources/queue/` da cui promuovi manualmente è un
   filtro efficace.
3. **Converti i PDF a Markdown fuori banda.** L'ingest PDF nativo consuma molti più token
   dell'equivalente testuale. Con MinerU, Docling o Markitdown converti localmente, poi ingerisci
   `.md`. Su Apple Silicon, oMLX fa OCR locale.
4. **Ingest incrementale, non ricostruzioni.** _Ingest from folder_ salta i file già processati:
   sfruttalo invece di rilanciare tutto.
5. **Lint prima di ri-ingerire.** Molti problemi (duplicati, alias mancanti) si risolvono con
   _Smart Fix All_ a costo di poche chiamate, invece che con un ri-ingest completo.
6. **`reviewed: true` sulle pagine curate.** Impedisce che una passata successiva sprechi quota per
   riscrivere ciò che hai già sistemato.
7. **Ollama per le query, cloud per l'ingest.** Le query sono la parte ad alta frequenza e bassa
   difficoltà: un modello locale da 8B le regge, e ti lascia l'intera quota cloud per l'estrazione.

### 5.6 Un calendario settimanale che funziona

Con `wiki-acquisizione` a 500 RPD e `wiki-manutenzione` a soli 20–40 RPD (§5.1), il collo di
bottiglia della settimana non è più l'ingest ma il lint:

| Giorno   | Attività                                             | Slot                                       | Costo indicativo | Note                                                                                       |
| -------- | ------------------------------------------------------ | --------------------------------------------- | ------------------ | ----------------------------------------------------------------------------------------------- |
| Lun      | Ingest cartella A (~150 note)                          | `wiki-acquisizione`                            | ~500 richieste     | satura il bucket giornaliero, va bene così                                                 |
| Mar      | Ingest cartella B / note dense                         | `wiki-acquisizione`                            | ~500 richieste     |                                                                                                    |
| Mer      | Lint + Smart Fix All + Regenerate index                | `wiki-manutenzione`                            | ~40 richieste      | con soli 20–40 RPD, spezza il lint su più giorni se il vault è grande, invece di lanciarlo tutto in un colpo |
| Gio      | Prosegui lint se non concluso, altrimenti nuovo ingest | `wiki-manutenzione` / `wiki-acquisizione`      | variabile          |                                                                                                    |
| Ven      | Review manuale, `reviewed: true`, commit                | —                                              | 0                   |                                                                                                    |
| Sab–Dom  | Query e lettura                                         | `wiki-query` o `wiki-local`                    | trascurabile        | slot separato da Acquisizione: non consuma la stessa quota                                       |

Commit-and-sync a fine di ogni giornata: la history diventa un log dell'evoluzione della wiki.

---

## 6. Workflow operativo quotidiano

```
1. Obsidian → Git: Pull                       (allinea con GitHub)
2. Aggiungi fonti in sources/                 → commit "src: ..."
3. Verifica il modello nelle impostazioni     (bulk vs quality)
4. 📁 Ingest from folder                      (batch dimensionato su 5.3)
5. 📜 Ingestion history                       (controlla errori e chiamate usate)
6. 🛠️ Lint wiki → ⚡ Smart Fix All            (se emergono problemi)
7. 🔄 Regenerate index
8. Review dei diff in Source control view     (cosa ha scritto davvero l'LLM)
9. Git: Commit-and-sync                       → "wiki: ..."
```

Il passo 8 non è burocrazia: è il punto in cui il controllo umano entra in un processo altrimenti
generativo. Il diff git di una wiki generata da LLM è lo strumento di verifica migliore che hai.

---

## 7. Sicurezza e igiene del repository

**Le API key non devono mai entrare nel repo.** Il plugin dichiara di salvare le chiavi in
_Obsidian SecretStorage_, ma non fidarti per assunzione: verifica.

```bash
# 1) cosa contiene davvero il data.json del plugin
grep -ri "AIza\|sk-\|Bearer" .obsidian/plugins/ 2>/dev/null

# 2) cosa è già finito in git (anche in commit passati)
git log -p --all | grep -n "AIza[0-9A-Za-z_-]\{30,\}"
```

Aggiungi una scansione automatica pre-push con **gitleaks**:

```bash
# hook: .git/hooks/pre-push
#!/usr/bin/env bash
gitleaks protect --staged --redact --no-banner || {
  echo "gitleaks: possibile segreto nel commit — push bloccato"; exit 1;
}
```

```bash
chmod +x .git/hooks/pre-push
```

Se una chiave è già stata pushata: **revocala subito in AI Studio** e generane una nuova.
Riscrivere la history (`git filter-repo`) è utile per l'igiene ma non è una mitigazione: assumi che
la chiave sia compromessa dal momento del push.

Checklist finale:

- [ ] repo GitHub **privato**
- [ ] `.gitignore` include `data.json` dei plugin e `workspace.json`
- [ ] PAT fine-grained, un solo repo, solo _Contents: read/write_, con scadenza
- [ ] API key in variabili d'ambiente / file `600` fuori dal vault
- [ ] `gitleaks` in pre-push
- [ ] materiale riservato **non** ingerito con provider free tier
- [ ] backup del vault indipendente da GitHub (il repo è versionamento, non backup)

---

## 8. Troubleshooting

| Sintomo                                                  | Causa                                                             | Rimedio                                                                                                          |
| ----------------------------------------------------------- | -------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------- |
| `429 RESOURCE_EXHAUSTED` durante l'ingest                    | RPD o RPM esaurito                                                   | ⏹️ Cancel (si ferma pulito al batch boundary), passa a un altro modello o attendi il reset delle 09:00/10:00. Con LiteLLM il fallback è automatico |
| 429 immediato con quota apparentemente libera                | il progetto ha quota 0 su quel modello, o il modello non è nel free tier | controlla Cloud Console → Quotas; prova un modello diverso                                                          |
| `Test Connection` fallisce con base URL custom                | slash finale mancante                                                 | il base URL deve terminare con `/v1beta/openai/` (o `/v1` per LiteLLM)                                              |
| Estrazione incoerente, tag inventati                          | modello troppo piccolo per il task                                    | definisci il _Custom Tag Vocabulary_, oppure passa temporaneamente lo slot Acquisizione a un modello più capace (es. `gemini-3.6-flash`) per quella cartella |
| Ingest lentissimo o bloccato su vault grande                   | context window insufficiente                                          | usa modelli long-context per il batch; per vault oltre ~2000 pagine servono 200K+ token di contesto                 |
| Duplicati e pagine orfane dopo più ingest                      | deduplica parziale fra batch                                          | 🛠️ Lint wiki → ⚡ Smart Fix All → 🔄 Regenerate index                                                               |
| Conflitti git ricorrenti                                       | due dispositivi che scrivono                                          | Pull prima di ogni sessione; regola "un solo scrittore alla volta"; verifica che `workspace.json` sia ignorato       |
| Push mobile lentissimo                                          | isomorphic-git su history lunga                                       | shallow clone, oppure sincronizza solo da desktop                                                                    |
| Repo che cresce a dismisura                                     | PDF versionati                                                        | Git LFS, oppure tieni i PDF fuori dal vault e ingerisci il `.md` convertito                                          |

---

## 9. Checklist di avvio (30 minuti)

1. [ ] Repo GitHub privato creato
2. [ ] Vault locale con `.gitignore` della sezione 2.2, primo commit e push
3. [ ] Plugin **Git** installato, PAT configurato, commit-and-sync testato
4. [ ] Plugin **Karpathy LLM Wiki** installato (Obsidian ≥ 1.11.4)
5. [ ] API key Google AI Studio creata in un progetto dedicato
6. [ ] Provider configurato, _Test Connection_ verde
7. [ ] Granularità impostata su `Coarse`, Custom Tag Vocabulary abbozzato
8. [ ] **Ingest single source** su 3 note → lettura di _Ingestion history_ → calibrazione del
       batch
9. [ ] Prima **Query wiki** di verifica
10. [ ] `gitleaks` in pre-push, controllo che nessuna chiave sia nel repo
11. [ ] (opzionale) LiteLLM attivo con cache e fallback

---

## 10. Note di verifica

Punti che dipendono dalla versione e vanno confermati sulla tua installazione, non dati per
acquisiti:

- **nomi esatti delle voci di impostazione** del plugin (Granularity, Tag Vocabulary, Custom
  OpenAI-Compatible, e i tre slot di §3.5): l'interfaccia evolve rapidamente;
- **limiti di quota Gemini**: dal 2026 la doc ufficiale non pubblica più tabelle numeriche statiche
  e rimanda alla dashboard `aistudio.google.com/rate-limit` (§4.4) — è oggi l'unica fonte
  affidabile per il _tuo_ progetto, più affidabile di Cloud Console → Quotas per il dettaglio
  per-modello;
- **numero di chiamate LLM per nota**: la stima 2–5 è un ordine di grandezza; misurala su
  _Ingestion history_ prima di pianificare batch grandi;
- **disponibilità di `gemini-2.5-pro` e dei modelli Gemini 2.0**: allo snapshot del 16/08/2026,
  2.5-pro non compariva più come opzione gratuita e i modelli 2.0 Flash/Flash-Lite risultavano
  ritirati (§4.4) — verifica lo stato corrente prima di costruirci sopra un flusso, perché la
  generazione dei modelli Gemini continua a evolvere (a quella data era già in campo una
  generazione 3.x);
- **nomi dei modelli 3.x usati in questa revisione** (`gemini-3.5-flash-lite`,
  `gemini-3.1-flash-lite`, `gemini-3.6-flash`): riflettono lo snapshot del 16/08/2026 su un
  singolo progetto — è probabile che entro pochi mesi Google ne rilasci di più recenti (era già
  annunciato `gemini-3.5-pro` in test con partner selezionati a quella data).

---

## Fonti

- [green-dalii/obsidian-llm-wiki — repository e README del plugin](https://github.com/green-dalii/obsidian-llm-wiki)
- [Vinzent03/obsidian-git — plugin Git per Obsidian](https://github.com/Vinzent03/obsidian-git)
- [Getting Started — Obsidian Git (DeepWiki)](https://deepwiki.com/Vinzent03/obsidian-git/1.1-getting-started)
- [OpenAI compatibility — Gemini API, Google AI for Developers](https://ai.google.dev/gemini-api/docs/openai)
- [Gemini is now accessible from the OpenAI Library — Google Developers Blog](https://developers.googleblog.com/en/gemini-is-now-accessible-from-the-openai-library/)
- [Gemini API Free Tier Rate Limits 2026 — AI Prompts Hub](https://aipromptshub.co/blog/gemini-api-free-tier-rate-limits)
- [Gemini API Free Tier Rate Limits: Complete Guide for 2026 — AI Free API](https://www.aifreeapi.com/en/posts/gemini-api-free-tier-rate-limits)
- [Import OpenAI-Compatible Google Gemini API — Microsoft Learn](https://learn.microsoft.com/en-us/azure/api-management/openai-compatible-google-gemini-api)
