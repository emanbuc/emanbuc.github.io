---
layout: post
title: "OpenWorker: un agente AI open source che gira dove decidi tu"
date: 2026-08-18 15:00:00
description: "OpenWorker è un agente AI desktop open source (MIT) che tiene loop di esecuzione, conversazioni e credenziali sulla macchina dell'utente, e che può parlare sia con endpoint pubblici sia con modelli privati in self-hosting. È la combinazione che rende un agente adottabile in un contesto aziendale."
categories: ai software-engineering
tags: openworker agenti-ai open-source privacy self-hosting llm-locale
toc:
  sidebar: left
---

Gli agenti AI hanno superato da un pezzo la fase della demo. Il problema che ne blocca l'adozione
in azienda, però, non è quasi mai la capacità del modello: è il **perimetro del dato**. Un agente
diventa utile esattamente nel momento in cui gli si dà accesso al materiale che conta  come ad esempio il
repository proprietario, la casella di posta, i ticket, i contratti, i fogli di calcolo con i
numeri veri. Cioè esattamente il materiale che, in molte organizzazioni, non può attraversare la
rete verso un fornitore terzo.

Da qui nascono le tre domande che un ufficio IT pone prima di autorizzare qualunque strumento di
questo tipo: **dove gira il loop dell'agente**, **dove stanno le credenziali**, e **cosa esce
davvero dalla macchina**. La maggior parte degli agenti commerciali risponde "sul nostro cloud, da
noi, tutto". E' una risposta legittima, ma che sposta la decisione dal piano tecnico a quello
contrattuale.

**OpenWorker** risponde in modo diverso, e per questo merita attenzione: è open source con licenza
MIT, il loop gira sul desktop dell'utente, e il modello (che è il componente che vede
effettivamente i dati) è **sostituibile con un endpoint privato**.

> **Nota di riferimento.** Quanto segue si basa sul repository `andrewyng/openworker` (branch
> `main`) e sul sito `openworker.com`, consultati il **18 agosto 2026**. Il progetto è in **open
> beta** e si auto-aggiorna: nomi dei modelli, elenco dei connettori e dettagli di interfaccia
> cambiano con frequenza. Prima di prendere decisioni di adozione, i termini di servizio dei
> provider e i limiti di quota vanno riverificati alla fonte.

## Cos'è OpenWorker

È un **agente AI open source che gira sul desktop** e che, invece di rispondere in chat, produce
deliverable finiti: un documento sul disco, una risposta Slack con i numeri reali dentro, un
calendario riorganizzato, una inbox triagiata. È stato rilasciato nel luglio 2026 dal gruppo di
Andrew Ng ed è costruito sopra **aisuite**, la libreria che astrae i provider LLM.

L'architettura è volutamente semplice:

```text
┌────────────────────────────────────────────────┐
│              OpenWorker desktop app            │  shell nativa (Tauri) + GUI React
├────────────────────────────────────────────────┤
│           local agent server (Python)          │  engine · tools · connettori — su aisuite
├───────────────┬────────────────┬───────────────┤
│  i tuoi file  │   i tuoi tool  │ il tuo modello│  tutto gira con le tue chiavi,
│  e terminale  │ 25+ connettori │ ogni provider │  sulla tua macchina
└───────────────┴────────────────┴───────────────┘
```

Il backend Python espone un server FastAPI in ascolto su `127.0.0.1:8765`; la shell Tauri lo
supervisiona e serve l'interfaccia React. Lo stesso server è raggiungibile da CLI (`openworker`,
una TUI Textual) e via HTTP con un token di lancio. Sul piano funzionale copre file e terminale
locali, oltre 25 connettori (Slack, Gmail, Outlook, Google Calendar, GitHub, GitLab, Jira, Linear,
Notion, Confluence, Salesforce, Stripe, Zendesk, Datadog…), qualunque server **MCP**, trigger in
ingresso (menzione in un canale Slack, email, eventi di calendario) e automazioni ricorrenti con
schedulazione cron.

Fin qui, la descrizione somiglia a quella di molti altri prodotti. Le differenze rilevanti sono
altre tre, e sono quelle che cambiano il discorso in un contesto professionale.

## 1. Open source non è un'etichetta: è ciò che ti permette di fare

La licenza è **MIT**, e questo ha conseguenze molto concrete che vale la pena rendere esplicite —
soprattutto se il vostro processo di adozione prevede un
[audit delle licenze dei componenti di terze parti]({% link _posts/2026-08-18-audit-licenze-dipendenze-terze-parti.md %}).

**Nessun obbligo di reciprocità.** MIT è permissiva: potete forkare il progetto, modificarlo,
distribuire una build interna con le vostre patch, integrarlo in un vostro processo, senza alcun
obbligo di pubblicare il codice risultante. È la differenza fra "possiamo provarlo" e "possiamo
costruirci sopra".

**Verificabilità.** Il motore dei permessi, il router dei provider e la gestione dei segreti sono
leggibili. Per uno strumento che ha accesso a shell, file e caselle di posta, la differenza fra
"il fornitore dichiara che i dati non escono" e "abbiamo letto il codice e verificato cosa apre
una connessione" non è retorica: è la differenza fra una due diligence documentale e una tecnica.

**Continuità.** Se il progetto cambia rotta, cambia licenza o viene abbandonato, il codice resta e
una build interna rimane possibile. È l'argomento di *exit strategy* che qualunque procurement
serio chiede, ed è l'unico caso in cui la risposta può essere sostanziale invece che contrattuale.

Detto questo, va aggiunta la parte scomoda: **open source non significa sicuro**. Un agente MIT
con accesso alla shell e ai connettori aziendali è, per costruzione, una superficie di attacco. La
licenza permessiva riguarda i vostri obblighi legali, non la qualità del codice né la sua supply
chain — che va sottoposta allo stesso audit di dipendenze che applichereste a qualunque altro
componente in ingresso.

## 2. Il perimetro del dato: cosa resta sulla macchina

OpenWorker si definisce **local-first**, e l'affermazione regge alla verifica. Il loop dell'agente,
le conversazioni, i token dei connettori e le chiavi dei modelli stanno tutti sulla macchina
dell'utente, in una **state dir** risolta in quest'ordine:

1. `$COWORKER_STATE_DIR` — override esplicito
2. Windows: `%APPDATA%\coworker`
3. macOS / Linux: `~/.config/coworker`

| File                   | Contenuto                                                                  |
| ---------------------- | -------------------------------------------------------------------------- |
| `config.toml`          | Configurazione globale (modello di default, modalità permessi, porte…)     |
| `secrets.json`         | Profili provider e token connettori, con permessi ristretti al solo utente |
| `.env`                 | Variabili risolvibili come `${VAR}` dentro `secrets.json`                  |
| `mcp.json`             | Server MCP globali (formato standard `mcpServers`)                         |
| `coworker.db`          | Conversazioni e memoria (SQLite)                                           |
| `memory-settings.json` | Switch della memoria e regole utente                                       |

**L'unico componente cloud del progetto** è un servizio che fa da broker per gli handshake OAuth
dei connettori ed è aggirabile creando le credenziali a mano. Questo è un dettaglio che in
azienda pesa parecchio: significa che è possibile un deployment in cui **nessun servizio gestito
dal progetto viene mai contattato**, e in cui il sign-in su un servizio di terze parti non è mai
richiesto.

Sui segreti, due scelte implementative fatte bene: `secrets.json` e `.env` sono ristretti al solo
utente (0600 su POSIX, ACL esplicite su Windows), e il **valore delle chiavi non entra mai nel contesto del modello** — configura soltanto il client SDK.

Resta il punto onesto: i dati escono comunque, ma **solo attraverso due canali che si scelgono
esplicitamente** — il modello e le integrazioni attivate. Il primo è quello che si governa con la
sezione seguente; il secondo con il motore dei permessi.

## 3. Endpoint pubblici o endpoint privati: il punto decisivo

Il componente che vede i vostri dati è il modello. OpenWorker non ha alcun lock-in su questo fronte: si porta la propria chiave, e sono supportati out of the box OpenAI, Anthropic, Google Gemini, Vertex AI, AWS Bedrock, Z.ai (GLM), DeepSeek, Kimi (Moonshot), Qwen, MiniMax, Mistral, xAI
(Grok), Meta, i reseller Together / Fireworks / OpenRouter, più **Ollama** e — il punto che qui interessa di più — **qualunque endpoint OpenAI-compatible**: LM Studio, llama.cpp, vLLM, Azure OpenAI.

Il meccanismo è il **ProviderRouter**: un singolo `ProviderClient` che smista in base al prefisso
della stringa di modello. `gemini:gemini-3.6-flash` va al client Gemini nativo,
`ollama:qwen3-coder:30b` al client Ollama, una stringa senza prefisso noto cade sul provider di
default (`openai`). I client vengono costruiti in modo lazy dal profilo salvato e messi in cache;
un cambio di configurazione chiama `invalidate()`, e le sessioni attive prendono la modifica senza
riavvio.

Il dettaglio implementativo che rende praticabile il self-hosting è nello slot OpenAI: quando il
campo `base_url` è **vuoto**, OpenWorker usa la **Responses API** (`/v1/responses`); quando
`base_url` è **valorizzato**, passa alla **Chat Completions API** (`/v1/chat/completions`). Il passaggio è automatico: non serve
configurare nulla oltre all'URL.

Il risultato è che la scelta dell'endpoint diventa una decisione di *policy*, non di prodotto:

| Opzione                      | Dove gira il modello      | Dove finiscono i prompt         | Quando ha senso                                                   |
| ---------------------------- | ------------------------- | ------------------------------- | ----------------------------------------------------------------- |
| Gemini free tier (AI Studio) | Google                    | Google, con riuso dichiarato    | Valutazione, prototipi, materiale pubblico. **Mai su dati reali** |
| API commerciale a consumo    | Il provider               | Il provider, senza riuso        | Task che richiedono il modello migliore, con DPA firmato          |
| Vertex AI                    | Il vostro progetto GCP    | Nel vostro perimetro cloud      | Aziende già su GCP, con billing e controlli IAM in essere         |
| LM Studio / llama.cpp locale | La macchina dell'utente   | Non escono dalla macchina       | Dati riservati, codice proprietario, lavoro offline               |
| vLLM su GPU aziendale        | Server interno            | Non escono dalla rete aziendale | Uso condiviso da più utenti, controllo centralizzato              |
| Ollama                       | Macchina o server interno | Non escono dal perimetro scelto | Setup rapido in locale, senza gestione di chiavi                  |

### Il requisito non negoziabile: il tool calling

Prima di collegare qualunque modello privato, va verificata una cosa sola ma decisiva: **il
modello e il runtime devono supportare il function calling nativo**. OpenWorker è un agente; senza
tool non può leggere file, eseguire comandi o usare connettori, e la sessione degenera in una chat
che *descrive* cosa farebbe.

Modelli con tool calling solido e ragionevolmente eseguibili in locale: la famiglia **Qwen3 /
Qwen3-Coder**, **GLM**, **Mistral**, **Devstral**, i **Llama 3.3/4** instruct. I modelli piccoli
(≤ 8B) tendono ad allucinare chiamate malformate: vanno bene per una demo, non per lavoro reale.

Regola pratica di dimensionamento: per un uso agentico serio servono almeno **~30B parametri
quantizzati a 4 bit** (≈ 18–20 GB fra VRAM e memoria unificata) e una context window di almeno
**32k token**, perché fra system prompt, definizioni dei tool e output dei comandi il contesto si
riempie molto in fretta. Se il tema è nuovo, ho scritto una guida dedicata su
[come eseguire un assistente basato su LLM in locale]({% link _posts/2026-01-18-esecuzione-locale-llm.md %}).

### Collegare LM Studio

Server locale da avviare dal tab *Developer* (o via CLI `lms`), endpoint di default
`http://localhost:1234/v1`:

```bash
lms server start
lms load qwen/qwen3-coder-30b --context-length 32768 --gpu max
lms ps                                  # identifier dei modelli caricati
curl http://localhost:1234/v1/models    # verifica prima di configurare l'app
```

In OpenWorker, *Settings ▸ Models ▸ card OpenAI*: come API key un valore qualsiasi non vuoto
(`lm-studio` va bene — l'SDK lo richiede, il server locale lo ignora), come *custom endpoint*
`http://localhost:1234/v1`. Poi si imposta come stringa di modello l'identifier restituito da
`lms ps`.

> **Trappola dei due punti.** Il router interpreta `prefisso:resto` come `provider:modello` **solo
> se il prefisso è un provider noto**. Un id come `qwen2.5-coder:32b` resta intatto, perché `32b`
> è un tag di versione; ma un id che iniziasse letteralmente con `qwen:`, `mistral:` o `kimi:`
> verrebbe dirottato sul provider cloud omonimo — cioè fuori dal vostro perimetro. In caso di
> ambiguità, qualificare esplicitamente: `openai:nome-modello`.

### Collegare llama.cpp

```bash
llama-server \
  -hf Qwen/Qwen3-Coder-30B-A3B-Instruct-GGUF:Q4_K_M \
  --jinja \
  --host 127.0.0.1 --port 8080 \
  --ctx-size 32768 \
  -ngl 99
```

Il flag `--jinja` è **indispensabile**: abilita il template della chat con il formato dei tool del
modello. Senza, il server espone `/v1/chat/completions` ma ignora il campo `tools`: OpenWorker si
collega, il modello risponde, e nessun tool viene mai invocato. È il sintomo più comune di un
setup locale che "funziona ma non fa niente". Configurazione nella card OpenAI: chiave placeholder
qualsiasi, endpoint `http://127.0.0.1:8080/v1`.

**vLLM** si configura allo stesso modo (endpoint `http://host:8000/v1`, chiave arbitraria salvo
`--api-key` impostata), ed è la strada corretta quando il modello va servito a più utenti su una
GPU dedicata invece che replicato su ogni portatile.

### Il percorso pubblico, per confronto

Sul versante opposto, il setup più rapido per una valutazione a costo zero è **Gemini via Google
AI Studio**: chiave creata in un minuto (inizia con `AIza…`), free tier senza carta di credito, e
Gemini è uno dei tre provider **nativi** di OpenWorker insieme a OpenAI e Anthropic — quindi con
supporto pieno per tool calling, vision e PDF inline. Il modello raccomandato dal repository è
`gemini:gemini-3.6-flash`, che sul free tier è anche la scelta pragmatica: un turno agentico
consuma molte richieste, perché ogni iterazione tool → modello è una request a sé.

Due avvertenze che in un contesto aziendale sono dirimenti:

- **Le quote sono per progetto, non per chiave.** Creare più chiavi nello stesso progetto Google
  Cloud non aumenta la quota. La fonte autorevole sui limiti è la pagina *Rate limit* del proprio
  progetto in AI Studio, non le tabelle di terze parti.
- **Sul free tier i contenuti scambiati possono essere usati per migliorare i prodotti del
  fornitore**; sui servizi a pagamento no. Per un agente che legge file locali e casella email
  questa distinzione non è teorica. Su materiale aziendale reale, il free tier **non è
  un'opzione**: o si attiva il billing, o si usa un endpoint privato.

## 4. Il pattern realistico: routing per sensibilità del task

L'errore da evitare è impostare la scelta come "tutto cloud" contro "tutto locale". OpenWorker
permette di cambiare modello **per sessione**, e questo abilita il compromesso che in pratica
funziona meglio: instradare in base alla classificazione del dato.

| Classe di dato                                 | Endpoint                      | Modalità permessi consigliata |
| ---------------------------------------------- | ----------------------------- | ----------------------------- |
| Codice proprietario, contratti, dati personali | Modello locale o vLLM interno | `interactive`                 |
| Documentazione interna non riservata           | API commerciale con DPA       | `interactive`                 |
| Ricerca su fonti pubbliche, redazione, bozze   | Free tier o API a consumo     | `plan` o `interactive`        |
| Task ripetitivi su workspace sacrificabile     | Indifferente                  | `custom` con `auto_allow`     |

Il costo di questa strategia va detto: il modello locale, più debole, sbaglia più spesso le catene
di tool lunghe. Si compensa tenendo la modalità `interactive` — che chiede conferma su ogni
scrittura — e un `max_iterations` più basso, così un loop degenere si ferma prima di produrre
danni o consumare un'ora di GPU.

## 5. Permessi tipizzati, non un dialogo "sei sicuro?"

È la terza caratteristica che rende lo strumento discutibile in una riunione di sicurezza senza
imbarazzo. Ogni chiamata a tool è classificata per **classe di rischio** — `read`, `write_local`,
`exec`, `external` — e filtrata da un motore di permessi con cinque modalità:

| Modalità      | Comportamento                                                                  |
| ------------- | ------------------------------------------------------------------------------ |
| `discuss`     | Sola lettura e conversazione: nessuna modifica, nessuna pianificazione         |
| `plan`        | Sola lettura più contratto di pianificazione: esplora, propone, esegue dopo ok |
| `interactive` | **Default.** Letture automatiche, approvazione su scritture e comandi          |
| `auto`        | Accesso pieno, nessun prompt                                                   |
| `custom`      | Come `interactive`, ma i tool elencati in `auto_allow` passano senza chiedere  |

Tre dettagli che rivelano un design pensato, e non una casella spuntata:

**L'allowlist dei comandi shell è severa, e vuota di default.** Un'entry in `allowed_commands` fa
eseguire il comando **senza prompt**, quindi il match sul prefisso non può essere ingenuo: `git
status` non deve autorizzare `git status && rm -rf ~`. Il motore rifiuta le composizioni e chiede
approvazione. Coerentemente, la lista built-in è vuota per scelta: non esiste un eseguibile
universalmente sicuro, ed è l'utente che si assume esplicitamente quell'autorità.

**Il workspace trust è un confine, non una comodità organizzativa.** La configurazione esiste su
due livelli — globale nella state dir, e per-progetto in `<progetto>/.coworker/`. Ma i campi
`allowed_commands` e `auto_allow`, e i server MCP definiti nel workspace, **non vengono mai letti
dal progetto** finché l'utente non ha marcato quel path come *trusted*. Tradotto: clonare il
repository di un fornitore non basta a fargli autorizzare comandi o caricare server MCP. È
esattamente la protezione che serve quando si lavora su codice che arriva dall'esterno.

**Le automazioni non agiscono da sole.** I task ricorrenti (morning brief, report settimanale,
sorveglianza di un canale) girano non presidiati, ma le richieste di approvazione vengono
**parcheggiate in una inbox** e restano lì finché qualcuno le esamina. Esistono regole standing con
ambito di task — tool più target esatto, di proprietà dell'automazione — per evitare di riapprovare
la stessa identica azione a ogni esecuzione, ma **mai per azioni di classe `exec`**. Ogni run
conserva il transcript completo: il che, incidentalmente, risolve buona parte del problema di
tracciabilità che un revisore porrebbe.

Sul fronte MCP, la stessa logica: transport `stdio` e `streamable-http`, OAuth per i server che lo
richiedono con i token nel secret store (mai in `mcp.json`, che resta un file in chiaro e
condivisibile), e i tool MCP **fuori dal catalogo interno delle capability** — restano una
superficie separata con permessi per-tool.

## 6. Configurazione minima, versionabile

Per un rollout su più macchine conviene evitare la GUI e versionare la configurazione. Il file
globale sta in `~/.config/coworker/config.toml` (o `%APPDATA%\coworker\config.toml`):

```toml
model = "gemini:gemini-3.6-flash"   # oppure l'identifier del modello locale
mode = "interactive"                 # plan | interactive | auto | custom
max_iterations = 150                 # iterazioni modello<->tool per turno prima dello stop

# Comandi auto-approvati senza prompt (match sul prefisso).
# Il default built-in è VUOTO, di proposito.
allowed_commands = ["ls", "cat", "pwd", "grep", "git status", "git diff", "git log"]

# In modalità "custom": tool auto-approvati (tutto il resto continua a chiedere).
auto_allow = ["write_file", "replace_in_file", "apply_patch"]

host = "127.0.0.1"
port = 8765

web_search_provider = "duckduckgo"   # "duckduckgo" (senza chiave) | "tavily" | "brave"
```

Le chiavi si possono passare da variabile d'ambiente (`GEMINI_API_KEY`, `OPENAI_API_KEY`,
`ANTHROPIC_API_KEY`, …), e la risoluzione segue l'ordine **esplicito → env → SecretStore**.

> **Attenzione, è il problema che fa perdere più tempo.** L'app desktop lanciata da Tauri **non
> eredita l'ambiente della shell**: se si avvia OpenWorker dal Finder o dal menu Start, le `export`
> messe nel `.zshrc` non arrivano al sidecar. La soluzione pulita è tenere la chiave fuori dal file
> dei segreti ma disponibile anche all'app, usando un riferimento `${VAR}` risolto da un `.env`
> posto **accanto** a `secrets.json` nella state dir:

```jsonc
// ~/.config/coworker/secrets.json
{
  "provider:gemini": { "api_key": "${GEMINI_API_KEY}" }
}
```

```bash
# ~/.config/coworker/.env
GEMINI_API_KEY=AIza...
```

Da sorgente, il bootstrap è una riga (`bash packaging/setup_dev_env.sh`, con Python 3.10+, Node 20+
e la toolchain Rust solo per la shell desktop); il server standalone si avvia con
`.venv/bin/openworker-server --cwd ~/progetti/mio-progetto --port 8765` e scrive un token
per-lancio in `<state-dir>/sidecar-8765.token`, da passare nell'header `X-OpenWorker-Token` per le
chiamate dirette. L'app desktop usa invece un token in memoria, mai scritto su disco.

## 7. Cosa mettere in conto prima di un pilota

Tre rischi vanno nominati esplicitamente, perché nessuno dei tre si risolve con una configurazione.

**Prompt injection.** La persona operatore di default tratta i contenuti provenienti da tool, log,
pagine web, file e messaggi in arrivo come **dati non fidati, non come istruzioni**. È la
mitigazione corretta ed è dichiarata nel design, ma resta probabilistica: un agente con accesso a
shell, file e connettori che legge una email ostile è una superficie di attacco reale. Le modalità
`plan` e `interactive` sono la difesa in profondità che rende visibile un comportamento anomalo
prima che diventi un'azione.

**Il perimetro di `auto`.** La modalità `auto` disattiva ogni prompt. Ha senso in un workspace
sacrificabile — un container, una VM, una copia di lavoro — e su un modello di cui ci si fida; su
una home directory con chiavi SSH e credenziali cloud è una scelta che va fatta consapevolmente e
documentata.

**La maturità del progetto.** È in open beta, si auto-aggiorna, e le build Windows non sono ancora
firmate (SmartScreen mostra un avviso). Per un pilota controllato non è un ostacolo; per un
rollout su cento postazioni è un elemento da mettere sul tavolo, insieme alla politica di
aggiornamento automatico.

## 8. Checklist per una valutazione aziendale

```text
□  Fissare la policy prima dello strumento: quali classi di dato possono uscire, e verso chi
□  Percorso pubblico (valutazione):  chiave AI Studio → Settings ▸ Models ▸ Gemini → Test
   Percorso privato (dati reali):    LM Studio o llama.cpp --jinja → Settings ▸ Models ▸ OpenAI
                                     key placeholder + endpoint locale → Test
□  Verificare che il modello scelto faccia davvero tool calling, prima di giudicare l'agente
□  Lasciare mode = "interactive" per tutte le prime sessioni
□  Aprire un workspace sacrificabile e assegnare un task reale ma a basso rischio
□  Collegare un solo connettore in sola lettura e osservare come vengono chieste le approvazioni
□  Rivedere il contenuto della state dir: chi ha accesso al file dei segreti sulla postazione?
□  Sottoporre il repository allo stesso audit di dipendenze e licenze degli altri componenti
□  Solo dopo: allowed_commands, automazioni, modalità custom, workspace trusted
```

## In sintesi

Il valore di OpenWorker, per un contesto professionale, non sta nell'essere un agente più bravo
degli altri: sta nel fatto che **le tre decisioni che contano restano vostre**. Il codice è
ispezionabile e la licenza MIT non impone nulla; il loop, le conversazioni e le credenziali stanno
sulla macchina; il modello — l'unico componente che vede davvero i dati — può essere un endpoint
pubblico quando il materiale lo consente e un server nella vostra rete quando non lo consente,
scegliendo per singola sessione.

È esattamente la granularità che serve per far coesistere due esigenze che di solito vengono
presentate come alternative: usare il modello migliore disponibile dove serve capacità, e non far
uscire un byte dove serve riservatezza.

## Riferimenti

- Sito ufficiale — <https://openworker.com>
- Repository (MIT) — <https://github.com/andrewyng/openworker>
- aisuite, la libreria su cui è costruito l'engine — <https://github.com/andrewyng/aisuite>
- Model Context Protocol — <https://modelcontextprotocol.io/>
- Google AI Studio — <https://aistudio.google.com/> · Rate limit del progetto — <https://aistudio.google.com/rate-limit>
- Gemini API, termini aggiuntivi — <https://ai.google.dev/gemini-api/terms>
- LM Studio, endpoint OpenAI-compatible — <https://lmstudio.ai/docs/developer/openai-compat> · Tool use — <https://lmstudio.ai/docs/developer/openai-compat/tools>
- llama.cpp, function calling — <https://github.com/ggml-org/llama.cpp/blob/master/docs/function-calling.md>
