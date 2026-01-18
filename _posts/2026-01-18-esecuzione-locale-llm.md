---
layout: post
title: Eseguire un assistente AI basato su LLM in locale
date: 2026-01-18 12:30:00
description: L'esecuzione di modelli di linguaggio di grandi dimensioni (LLM) in locale è diventata oggi una realtà pratica per utenti privati grazie alla maturazione di software intuitivi e a tecniche di compressione avanzate
tags: LLM AI
categories: AI 
---

L'esecuzione di modelli di linguaggio di grandi dimensioni (LLM) in locale è diventata oggi una realtà pratica per utenti privati grazie alla maturazione di software intuitivi e a tecniche di compressione avanzate. Gestire un'IA sul proprio PC o notebook offre vantaggi determinanti come la privacy assoluta dei dati, l'accesso offline, l'assenza di costi di abbonamento e la possibilità di personalizzazione avanzata.

Questa transizione è stata resa possibile dalla convergenza di tre fattori critici: la disponibilità di modelli open-weight di classe enterprise, come le serie Llama, Mistral e Qwen; lo sviluppo di tecniche di compressione sofisticate note come quantizzazione; e la democratizzazione dell'hardware ad alte prestazioni, incluse le GPU consumer e i chip con memoria unificata.

L'esecuzione locale di LLM è diventata un'imperativo strategico per le organizzazioni che intendono mantenere la sovranità sui propri dati, ottimizzare i costi operativi e garantire la continuità del servizio in assenza di connettività internet.  Per supportare carichi di lavoro elevati in contesti aziendali (molti utenti e numero richieste di inferenza anche contemporanee) è ncessario disporre di compenti hardware specifiche, dove la larghezza di banda della memoria e la VRAM della GPU risultano cruciali per garantire prestazioni accettabili. In questo articolo ci occupermo però solo dello scenario di utilizzo "personale" (singolo utente) e per un utilizzo personale un notebook recente con 16/32GB di RAM o un Mac Mini possono esserre già sufficienti.

## 1. Installazione ed Esecuzione in Locale di un LLM
I modelli LLM non sono eseguibili autonomi ma vengono gestiti da un motore di inferenza. Tipicamente i modelli sono sviluppati con framework come TensorFlow, PyTorch o altri, che includono anche gli strumenti per il caricamento di un modello pre-addestrato e permettono di utilizzarlo per per operazioni di inferenze. Il motore di esecuzione gestisce l’allocazione della memoria, e le operazioni di calcolo distribuito su CPU o GPU. Per utilizzare un LLM (ma anche un altro qualsiasi altro modello di machine learning) in genere servono quattro elementi:

1. Un modello (cioè la struttura del modello) 
2. I "pesi" generati dal processo di addestramento. In pratica si tratta dei valori numerici che alla fine del processo di addestramento sono stati assegnati ai parametri del modello. Un modello (architettura) dotato di 70B (Bilions= migliardi) di parametri prevede 70B di "pesi" (valori da assegnare ai suoi parametri). Quando si *scarica* un modello in pratica si scarica il file con pesi (valori) da assegnare a tutti i parametri oltre che la struttura del modello con i parametri non valorizzati.
3. Un motore di inferenza. A volte può essere lo stesso usato per l'addestramento, ma può essere anche diverso e specializzato solo per l'inferenza come ad esempio Llama.cpp
4. Hardware con risorse di calcolo sufficienti per il caricamento e l'esecuzione del modello

Ad esempio è possibile eseguire modelli pre-addestrati usanto da Libreria ([Transfromer ](https://huggingface.co/docs/transformers/model_doc/llama3)) di HugginFace in Python 

```python
import transformers
import torch

model_id = "meta-llama/Meta-Llama-3-8B"

pipeline = transformers.pipeline("text-generation", model=model_id, model_kwargs={"torch_dtype": torch.bfloat16}, device_map="auto")
pipeline("Hey how are you doing today?")

```

Per un utilizzo al difuori dell'ambiente di sviluppo è solitamente preferibile utilizzare utilizzare dei motori di inferenza dedicati che permettono una maggiore efficienza nell'utilizzo delle risosrse di calcolo.

I modelli (architettura e pesi) sono disponibili in vari formati. Negli ultimi anni si è affermato il formato [GUFF](https://huggingface.co/docs/hub/gguf) per la distribuzione di modelli complessi. La cosa importante è che il formato sia compatibile con il modete di inferenza utilizzato! 

I motori di inferemza dispongono solitamente di una libreria di modelli già preparata nel formato ottimale per lo specifico motore. La confersione è possibile, ma si tratta di un processo che può nascondere delle insidie e sicuramente da evitare durante i primi esperimenti con gli LLM.


## 2. Requisiti Hardware
Sebbene gli LLM siano nati per girare su server potenti, l'hardware consumer attuale (2025/2026) può gestire modelli di dimensioni contenute con prestazioni soddisfacenti.

*   **Memoria RAM/VRAM:** È il fattore più critico. Se il modello risiede interamente nella memoria video (**VRAM**) della scheda grafica (GPU), l'esecuzione è da 5 a 20 volte più rapida rispetto alla RAM di sistema.
    *   **Configurazione Base (8GB RAM):** Può far girare piccoli modelli da 1B-3B parametri.
    *   **Configurazione Consigliata (16GB+ RAM):** Il punto di equilibrio per modelli da **7B-8B** parametri, come Llama 3 o Mistral.
    *   **Apple Silicon (M1/M2/M3/M4):** Questi chip offrono un vantaggio unico grazie alla **Memoria Unificata**, che permette alla GPU di accedere a tutta la RAM di sistema come se fosse VRAM.
*   **Scheda Video (GPU):** Le schede **NVIDIA RTX** (serie 3000/4000/5000) sono lo standard grazie al supporto CUDA. Una GPU con **8GB-12GB di VRAM** (come la RTX 3060 o 4060) è ideale per un'esperienza fluida con modelli medi.
*   **Processore (CPU):** Se non si dispone di una GPU dedicata, è necessaria una CPU moderna (almeno 11ª gen Intel o AMD Zen 4) con supporto alle istruzioni **AVX-512** per accelerare i calcoli.

## 3. L'importanza della Quantizzazione
Senza compressione, un modello da 7 miliardi di parametri richiederebbe circa 14-16 GB di VRAM, risultando inaccessibile per hardware standard. La **quantizzazione** riduce la precisione numerica dei pesi del modello (es. da 16-bit a 4-bit) senza comprometterne drasticamente l'intelligenza.

Qualche esempio per capire l'effetto della quantizzazione:

- FP32 precision: 4 byte per parameter. ==> A 30B model = 120 GB just for weights (before KV cache) and without metadata!.
- FP16: 2 bytes/param → 60 GB for 30B.
- Q8 (8-bit quant): ~1 byte/param → 30 GB for 30B.
- Q4 (4-bit quant): ~0.5 bytes/param  → 15GB for 30B

Oltre al file dei pesi, nel calcolo complessivo della memoria devono essere inclusi anche la cache KV ed i metadati che solitamente aggungono qualche altro GB al valore totale.

Il formato **Q4_K_M** (4-bit) è considerato il **"gold standard"** per l'uso locale: riduce le dimensioni del modello del 75%, permettendo a un modello 7B di girare su soli 5-6 GB di VRAM con una perdita di qualità trascurabile.

Il tema dei formati dei modelli e della quantizzazione è ampio e lo approfondiremo in un altra occasione. 

## 4. Strumenti Software Principali
Esistono diverse piattaforme che riducono il processo di installazione e configuraizone a pochi click. La piattaforma di esecuzione più performante per un utilizzo personale è sicuramente [Llama.cpp](https://github.com/ggml-org/llama.cpp).

Llama.cpp è un'implementazione in puro C/C++ creata da Georgi Gerganov. La sua importanza risiede nel fatto che è stata la prima a dimostrare che i modelli linguistici di grandi dimensioni potevano girare in modo efficiente su hardware consumer (sia CPU che GPU) grazie a tecniche avanzate di quantizzazione e gestione della memoria. Supporta una vasta gamma di accelerazioni, tra cui CUDA (NVIDIA), Metal (Apple Silicon), ROCm (AMD) e Vulkan. 

Llama.cpp dispone di una sua interfaccia a riga di comando, di una interfaccia web in stile ChatGPT e di un server che ne permette l'utilizzo anche da parte di altri client. 

Ad esempio 

    llama-server -hf ggml-org/gpt-oss-20b-GGUF --n-cpu-moe 12 -c 32768 --jinja --no-mmap

Avvia carica il modello GPT-OSS 20B ed avvia il server e l'interfaccia web.

{% include figure.liquid 
    path="assets/img/llama-gpt-oss.png" 
    title="Optional hover title" 
    caption="Figure 1: llama-server in esecuzione." 
    zoomable=true 
%}

Ovviamene il "peso" dell'esecuzione del modello si fa sentire e non permette di eseguire contemporanemante altre attività pesanti però il sistema è utilizzabile. 

{% include figure.liquid 
    path="assets/img/gpt-oss-web-interface.png" 
    title="Optional hover title" 
    caption="Figure 2: Interfaccia web in stile Chat esposta da llama-server." 
    zoomable=true 
%}

Altri strumenti molto popolari, che aggiungono un interfaccia utente più elaborata sopra Llama.cpp sacrificando però le prestazioni compessive: la differenza tra Llama.cpp ed altri strumenti come Ollama è in alcuni casi è piuttosto evidente con Llama.cpp che è arriva ad essere anche un 30-40% più veloce degli altri strumenti. Ad esempio, sul mio Mac Mini M2 16GB con Llama.cpp il modello GPT-OSS 20B è utilizzabile, mentre con altri strumenti riesce a malapena ad avviarsi e poi è troppo lento per un qualsiasi utilizzo pratico interattivo. 
 
Altri strumenti popolari per eseguire LLM in locale sono:

*   **LM Studio:** Forse la soluzione più accessibile per utenti non tecnici. Offre un'interfaccia grafica completa, un motore di ricerca integrato per scaricare modelli da Hugging Face e permette di visualizzare l'uso delle risorse in tempo reale.
*   **Ollama:** Estremamente popolare tra gli utenti più esperti e gli sviluppatori. Funziona tramite riga di comando (CLI) ma può essere abbinato a interfacce web grafiche come **OpenWebUI** per un'esperienza simile a ChatGPT.
*   **Jan.ai:** Un'alternativa **open-source** focalizzata sulla privacy che funziona interamente offline. Ha un'interfaccia minimalista e supporta l'estensione tramite plugin.
*   **GPT4All:** Ottimizzato per girare su hardware modesto (anche solo CPU). Include la funzione **LocalDocs**, che permette di "chattare" con i propri documenti locali (PDF, Word) in modo semplice.


## 5. Modelli Consigliati per Hardware Standard
Per un PC o notebook standard, suggerisco di iniziare con le versioni più piccole e quantizzate di queste famiglie di modelli:
*   **Llama 3.2 (1B / 3B):** Ultra-leggeri, ideali per notebook con poca RAM.
*   **Mistral (7B) / Llama 3.1 (8B):** I modelli più versatili per compiti generali e scrittura.
*   **Qwen 2.5/3 (0.5B - 7B):** Eccellenti per il coding e compiti multilingue.
*   **Phi-3/4 (Mini):** Modelli piccoli di Microsoft con capacità di ragionamento sorprendenti per la loro dimensione.

Una volta testato il funzionamento di un modello di piccole dimensioni, vi consiglio di testare anche modelli più grandi fino a trovare il compromesso ottimale tra utilizzo delle risorse di calcolo disponibili e prestazioni.

### OpenAI GPT-OSS 
OpenAI GPT-OSS (Agosto 2025) è il primo rilascio open-weight significativo di OpenAI dopo anni. Il modello è disponibile principalmente nelle versioni 20B e 120B. La variante 120B, in particolare, offre prestazioni di livello GPT-4 ed eccelle in compiti di ragionamento avanzato, tool calling e workflow agentici complessi.

- La versione 20B è quella più indicata per hardware locale avanzato, ma richiede comunque una configurazione di fascia alta con almeno 32GB di RAM/VRAM per girare correttamente. *Sul mio Mac Mini Apple Silicon M2 con 16GB di RAM condivisa* si riesce ad eseguire usando LLAMA-CPP [seguendo la guida dedicata a GPT-OSS](https://github.com/ggml-org/llama.cpp/discussions/15396)

-  La versione 120B rimane fuori dalla portata dei PC standard, richiedendo solitamente infrastrutture di classe enterprise.

**In sintesi:** Per iniziare è sufficiente un notebook moderno con **16GB di RAM** (o acora meglio un Mackbook o un MacMini), scaricare **LM Studio** e provare un modello come **Mistral** o **Llama 3.1 8B** in versione quantizzata a 4-bit. Se la memoria video non è sufficiente, questi strumenti sposteranno automaticamente parte del lavoro sulla RAM di sistema, sacrificando la velocità per garantire l'esecuzione.

Con un Mac Mini, equipaggiato con processore AppleSilicon serie M, si ottiene un rapporto prezzo/prestazioni sicuramente interessante e la possibilità di eseguire modelli di dimensione fino a 20B.