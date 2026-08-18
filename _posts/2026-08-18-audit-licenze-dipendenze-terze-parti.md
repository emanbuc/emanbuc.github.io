---
layout: post
title: "Audit delle licenze nei progetti con componenti di terze parti"
date: 2026-08-18 08:00:00
description: "Perché la license compliance non è un adempimento formale, come si analizzano le licenze delle dipendenze di un progetto Python e come automatizzare il controllo in CI/CD con licence-audit-py."
categories: software-engineering
tags: license-compliance open-source python SBOM cyber-resilience-act
toc:
  sidebar: left
---

Nessuno scrive più software da zero. Un progetto Python di dimensioni ordinarie porta con sé
qualche decina di dipendenze dirette e, dietro di esse, un centinaio di dipendenze transitive che
nessuno ha scelto consapevolmente. Ognuna arriva con un contratto: la licenza. E il punto che
sfugge più spesso è che una licenza open source violata non è un problema "di forma" — è una
concessione che decade, e con essa il diritto di usare quel componente.

Ho raccolto la metodologia che uso per questo tipo di analisi, insieme a uno strumento che la
automatizza, nel repository **[emanbuc/licence-audit-py](https://github.com/emanbuc/licence-audit-py)**.
Questo post ne è la sintesi ragionata: quando serve un audit, quali domande deve davvero
rispondere, e dove si nascondono i problemi.

> **Disclaimer.** Quanto segue descrive un processo tecnico e ingegneristico, non è consulenza
> legale. La classificazione automatica delle licenze produce *indizi*, non pareri: i casi dubbi e
> le strategie di remediation vanno validati dall'ufficio legale.

## Quando l'audit smette di essere facoltativo

Ci sono momenti precisi in cui la domanda "che licenze stiamo usando?" arriva dall'esterno, e a
quel punto rispondere "non lo so" costa parecchio:

- **rilascio di un prodotto** — on-prem, SDK, container, dispositivo embedded, app mobile;
- **due diligence M&A o round di investimento** — è quasi sempre il primo documento richiesto;
- **cliente enterprise** che impone in contratto una clausola di indennizzo sulla proprietà
  intellettuale;
- **conformità normativa** — nell'UE il *Cyber Resilience Act* impone un SBOM in formato
  machine-readable per i prodotti con elementi digitali, con obblighi di segnalazione delle
  vulnerabilità sfruttate attivamente dall'**11 settembre 2026**, piena applicazione dall'**11
  dicembre 2027** e conservazione della documentazione tecnica per **dieci anni**;
- **cambio di modello di business** — il passaggio da software distribuito a SaaS, o da progetto
  interno a prodotto commerciale, riscrive completamente il verdetto su componenti che fino a ieri
  erano innocui.

Quest'ultimo punto è il più insidioso, perché non richiede che nessuno tocchi il codice: cambia il
contesto, e lo stesso identico albero di dipendenze diventa un problema.

## Le due domande che contano davvero

Un audit di license compliance può diventare un esercizio infinito. Nella pratica, per un prodotto
commerciale, quasi tutte le decisioni si riducono a due domande:

1. **Ci sono componenti che ci obbligano a pubblicare il nostro codice sorgente?**
2. **Ci sono componenti che limitano l'uso commerciale?**

Sono domande diverse, che rispondono a meccanismi giuridici diversi, e confonderle è l'errore più
comune in circolazione.

### Domanda 1 — l'obbligo di disclosure e il suo *trigger*

Non basta sapere che una licenza è "copyleft". Bisogna sapere **cosa** va pubblicato e **quando**
scatta l'obbligo:

| Classe                                            | Cosa va pubblicato                                 | Trigger                                                | Impatto su un prodotto proprietario                                 |
| ------------------------------------------------- | -------------------------------------------------- | ------------------------------------------------------ | ------------------------------------------------------------------- |
| **Copyleft di rete** (AGPL, OSL, RPL, EUPL, SSPL) | l'intera opera derivata, il vostro codice compreso | distribuzione **oppure** semplice interazione via rete | bloccante **anche per un SaaS puro**                                |
| **Copyleft forte** (GPL, CeCILL)                  | l'intera opera derivata                            | distribuzione a terzi                                  | bloccante se distribuite, tollerabile se usate solo internamente    |
| **Copyleft debole** (LGPL, MPL, EPL, CDDL)        | solo il componente e le modifiche ad esso          | distribuzione del componente                           | gestibile: pubblicare le vostre patch e garantire la sostituibilità |

La distinzione fra copyleft forte e copyleft di rete è quella che discrimina il maggior numero di
decisioni reali. Un'azienda che vende esclusivamente SaaS può convivere con una GPL-3.0 lato
server — nessuna distribuzione, nessun obbligo — ma **non** con una AGPL-3.0, la cui §13 impone di
offrire il *Corresponding Source* a chiunque interagisca con il software attraverso una rete.

Un caveat specifico per Python: la LGPL è scritta pensando al linking C. Un `import` Python è
dinamico e ricade in generale nel caso "uso della libreria non modificata". Ma se **modificate** il
pacchetto, o lo congelate in un binario (PyInstaller, Nuitka, un container distroless in cui
l'utente non può sostituire la libreria), il quadro cambia.

### Domanda 2 — il limite all'uso commerciale

Qui va detta chiaramente una cosa: **le licenze open source approvate OSI non limitano mai l'uso
commerciale**. GPL, LGPL e AGPL permettono esplicitamente di vendere il software; impongono
obblighi di *reciprocità*, non divieti. Trattare "copyleft" come sinonimo di "vietato in ambito
commerciale" produce blocchi ingiustificati e fa perdere credibilità all'intero processo.

Le licenze che limitano davvero l'uso commerciale sono in gran parte **non-OSI**, e agiscono con
meccanismi diversi fra loro:

| Meccanismo                           | Esempi                                                            | Effetto                                                                         |
| ------------------------------------ | ----------------------------------------------------------------- | ------------------------------------------------------------------------------- |
| Divieto esplicito di uso commerciale | `CC-BY-NC-*`, `PolyForm-Noncommercial`, licenze "research only"   | vietato in qualsiasi prodotto o servizio a pagamento                            |
| Divieto di rivendita / hosting       | `Commons-Clause`, `Elastic-2.0`                                   | uso interno sì, offerta come servizio no                                        |
| Licenza a scadenza                   | `BUSL-1.1`                                                        | uso in produzione limitato fino alla *Change Date*                              |
| Onere sproporzionato                 | `SSPL-1.0`                                                        | non un divieto, ma rilasciare l'intero stack di servizio lo rende impraticabile |
| Restrizione di campo d'uso           | licenza `JSON` ("Good, not Evil"), RAIL/OpenRAIL per i modelli ML | vincolo non verificabile automaticamente                                        |
| No-derivatives                       | `CC-BY-ND-*`                                                      | vieta patch e porting: incompatibile con l'uso in un prodotto                   |

E naturalmente un componente può essere problematico sia per i requisiti di pubblicazione dei sorgenti che per l'utilizzo in un prodotto/servizio commerciale. Ad esempio AGPL e SSPL stanno rientrano in entrambe le categorie.

## Il problema tecnico: dove vivono i dati di licenza

Prima di poter decidere qualcosa bisogna sapere *cosa dichiara* ogni pacchetto — e in Python
questa è la parte sorprendentemente fragile. **PEP 639** (ormai *Final*, Core Metadata 2.4) ha
introdotto il campo `License-Expression` con un'espressione SPDX validata, deprecando sia il vecchio
campo libero `License` sia i classifier `License :: OSI Approved :: ...`. Nella realtà di un
`site-packages` convivono ancora tutte e quattro le generazioni di metadati, con affidabilità molto
diversa:

1. **`License-Expression`** nel `METADATA` — espressione SPDX validata da PyPI. Fonte autoritativa.
2. **Classifier `License :: ...`** — vocabolario chiuso ma grossolano: `BSD License` non dice
   *quale* BSD (la 4-Clause, con la sua *advertising clause*, è incompatibile con la GPL).
3. **Campo `License`** libero — può contenere un identificatore, una frase o l'intero testo.
4. **File `LICENSE` / `COPYING` / `NOTICE`** e header dei singoli sorgenti — l'unico modo per
   scoprire codice di terze parti incorporato con licenza diversa da quella dichiarata.

Da qui la regola pratica: **ogni licenza non ricavata da `License-Expression` va marcata con
confidenza inferiore**, e verificata a mano se il componente è critico.

## Lo strumento: `licence-audit-py`

Da questa esigenza è nato
**[licence-audit-py](https://github.com/emanbuc/licence-audit-py)**: uno script a riga di comando
che legge le distribuzioni installate in un ambiente virtuale, risolve la licenza di ciascuna,
applica una policy versionata e produce un report verificabile — con un exit code utilizzabile in
CI.

Il repository contiene tre file:

| File                                 | Contenuto                                                                                     |
| ------------------------------------ | --------------------------------------------------------------------------------------------- |
| `license_audit.py`                   | lo script di audit — **solo standard library**, Python ≥ 3.10                                 |
| `policy.json`                        | policy di esempio: categorie consentite, vietate, da revisionare, eccezioni                   |
| `guida_license_compliance_python.md` | la guida operativa completa: metodologia, tassonomia, matrice decisionale, CI/CD, remediation |

Il vincolo "solo standard library" non è un vezzo minimalista: lo script deve girare **con
l'interprete dell'ambiente da analizzare**, e installarci dentro delle dipendenze significherebbe
inquinare l'inventario con i componenti dello strumento di audit stesso.

La logica applicata è in quattro passaggi:

- **risoluzione della licenza** per priorità decrescente di affidabilità (PEP 639 → classifier →
  campo libero → file di licenza), tracciando *da dove* viene l'informazione;
- **normalizzazione SPDX**, mappando i formati legacy (`"Apache Software License"`, `"BSD
  License"`, i classifier completi) sugli identificativi canonici;
- **classificazione** in cinque categorie di rischio: `non_commercial`, `network_copyleft`,
  `strong_copyleft`, `weak_copyleft`, `permissive`;
- **applicazione della policy**, con esito `allow` / `review` / `deny` / `unknown`, motivazione ed
  eccezioni tracciate.

### Uso

Il primo passo è il più importante e viene saltato quasi sempre: **ricostruire l'ambiente di
produzione a partire dal lockfile**, non fare l'audit sul venv di sviluppo. `pytest`, `mypy`, `ruff`
e i loro alberi transitivi non finiranno mai nel prodotto, e inquinano il risultato con decine di
falsi allarmi.

```bash
python -m venv .audit
.audit/bin/pip install -r requirements.lock
# oppure: uv pip install --python .audit/bin/python -r requirements.lock
```

Poi si esegue l'audit con l'interprete di quell'ambiente:

```bash
.audit/bin/python license_audit.py --policy policy.json --format markdown -o license-report.md
```

Il report è un riepilogo per esito seguito dal dettaglio componente per componente:

| Pacchetto          | Versione  | Licenza            | Fonte         | Categoria                         | Esito      | Note                                                 |
| ------------------ | --------- | ------------------ | ------------- | --------------------------------- | ---------- | ---------------------------------------------------- |
| some-lib           | 2.1.0     | `AGPL-3.0-only`    | expression    | network_copyleft, strong_copyleft | **deny**   | categoria vietata: network_copyleft, strong_copyleft |
| python-Levenshtein | 0.27.4    | `GPL-2.0-or-later` | license-field | strong_copyleft                   | **deny**   | categoria vietata: strong_copyleft                   |
| certifi            | 2026.7.22 | `MPL-2.0`          | classifier    | weak_copyleft                     | **review** | valutare in base a linking e distribuzione           |
| pandas             | 3.0.5     | `BSD-3-Clause`     | classifier    | permissive                        | **review** | classifier ambiguo: confermare la variante           |

Nota l'ultima riga: `pandas` è BSD-3-Clause e non crea alcun problema, ma l'informazione arriva da
un classifier ambiguo e non da un'espressione SPDX. Lo strumento non finge una certezza che non ha
— la degrada a `review`. È una scelta di progetto: un audit che dichiara `allow` su dati deboli è
peggio di uno che chiede una verifica.

### La policy è codice, non un documento Word

Il file `policy.json` è la traduzione del **contesto di distribuzione** in criteri verificabili.
Lo stesso componente può essere innocuo per un uso interno e inutilizzabile in SaaS: la policy è il
punto in cui questa differenza viene dichiarata una volta sola, versionata insieme al codice e resa
disponibile sia al gate automatico sia alla revisione umana.

```json
{
  "deny_categories": ["non_commercial", "network_copyleft", "strong_copyleft"],
  "review_categories": ["weak_copyleft"],
  "allow_categories": ["permissive"],
  "deny_licenses": ["SSPL-1.0", "BUSL-1.1", "Elastic-2.0", "Commons-Clause", "BSD-4-Clause"],
  "exceptions": {
    "chardet": "LGPL-2.1+ importata dinamicamente e non modificata - LEGAL-142 - rev. 2027-01-31"
  },
  "unknown_verdict": "review"
}
```

Due dettagli che fanno la differenza fra una policy che sopravvive e una che viene aggirata:

- **le eccezioni esistono e sono tracciate.** Un gate senza meccanismo di deroga viene disattivato
  entro due settimane. Ogni eccezione deve avere motivazione tecnica, contesto approvato,
  approvatore, ticket e **data di scadenza** — altrimenti sopravvive a un major upgrade che ne
  cambia i presupposti.
- **`unknown` non è `allow`.** Un pacchetto senza licenza dichiarata non è di pubblico dominio: per
  default è "tutti i diritti riservati". Giuridicamente un `UNKNOWN` è il caso *peggiore*, più grave
  di una GPL correttamente identificata.

### Il gate in CI/CD

```bash
python license_audit.py --policy policy.json --fail-on deny,unknown
```

Exit code 1 in presenza di violazioni. Una severità sensata differenzia per contesto: `deny` blocca
sempre; `unknown` avvisa sulle PR di feature e blocca sul merge in `main`; `review` blocca solo alla
release, se non coperto da un'eccezione valida.

Un accorgimento che vale la pena adottare: far girare il job **anche a schedule**, tipicamente
settimanale sul branch principale. Una licenza upstream può cambiare senza che il vostro repository
venga toccato — è successo a `mongomock`, Redis, Elasticsearch, Terraform, HashiCorp Vault. Se le
dipendenze non sono bloccate con un lockfile, la build di domani può introdurre una licenza vietata
a codice invariato.

## Cinque trappole che i tool non vedono

L'automazione copre forse l'80% del lavoro. Il restante 20% è dove si concentrano i problemi seri:

**1. Le dipendenze transitive.** Il grosso delle violazioni reali sta lì, in componenti che nessuno
ha mai scelto. Prima di rimuovere qualcosa, conviene capire chi lo tira dentro:
`uv pip tree --package Levenshtein --invert`. Spesso la GPL non è una scelta architetturale ma un
extra opzionale di una libreria: disattivarlo risolve il problema in cinque minuti.

**2. Il dual licensing letto come congiunzione.** `MIT OR GPL-3.0-only` è un regalo — potete
prendere MIT. Uno strumento che tratta l'espressione come stringa opaca la marca come GPL e produce
un falso positivo. Gli operatori SPDX `AND`, `OR` e `WITH` vanno interpretati, non cercati con
`grep`.

**3. La divergenza fra dichiarato e reale.** Un pacchetto dichiarato MIT può contenere codice
*vendorizzato* di terzi con altra licenza, oppure impacchettare librerie native precompilate
(`libssl`, `ffmpeg`, driver di database) la cui licenza non compare da nessuna parte nei metadati.
Vale la pena ispezionare le wheel dei componenti critici:

```bash
python -m zipfile -l dist/pacchetto-1.0-cp311-*.whl | grep -Ei '\.(so|dll|dylib)$'
```

**4. Modelli e dataset scaricati a runtime.** Un pacchetto Python MIT può scaricare all'avvio un
modello con pesi `CC-BY-NC`, una *responsible AI license* o un dataset con restrizioni d'uso.
**Nessuno strumento di license scanning per Python rileva questo caso.** Va gestito con una
checklist manuale sui componenti che scaricano artefatti esterni, ed è il rischio più sottovalutato
nei progetti di machine learning.

**5. I fork e i pacchetti "mirror".** Un fork su PyPI può avere licenza diversa dall'upstream. Il
caso da manuale: `python-Levenshtein` è GPL-2.0-or-later, mentre `RapidFuzz` — funzionalità
sovrapponibili — è MIT. Sostituzione a costo quasi nullo che elimina un blocco copyleft.

## Quando trovi un problema

In ordine di preferenza, dalla soluzione più pulita alla più costosa:

1. **Sostituzione con un equivalente permissivo** — quasi sempre la strada giusta:
   `python-Levenshtein` → `rapidfuzz`, `mysql-connector-python` → `PyMySQL`, librerie PDF GPL →
   `pypdf` o `pdfminer.six`.
2. **Rimozione dell'extra** che tira dentro la dipendenza.
3. **Isolamento a livello di processo** — eseguire il componente copyleft come processo separato con
   interfaccia CLI o IPC, distribuendolo con il proprio sorgente. La "mera aggregazione" di
   programmi indipendenti non crea un'opera derivata. È una strategia legittima ma **non un
   trucco**: va progettata seriamente e validata dal legale, e non funziona con l'AGPL se il
   processo separato serve gli stessi utenti sulla stessa rete.
4. **Conformarsi all'obbligo** — per il copyleft debole spesso è banale: sorgente del componente,
   testo della licenza, sostituibilità garantita.
5. **Licenza commerciale** — molti progetti copyleft sono in dual licensing (Qt, MySQL, iText).
6. **Riscrittura interna** in *clean room* — ultima risorsa.

Quello che non va mai fatto: cambiare l'intestazione di licenza di codice altrui, rimuovere un file
`LICENSE` da un pacchetto ridistribuito, o considerare accettabile un `UNKNOWN`.

## L'output finale: SBOM e artefatti di release

L'audit non è finito quando il report è verde. Ogni release dovrebbe portarsi dietro un set di
artefatti immutabili e legati al commit: l'**SBOM** (CycloneDX o SPDX), il report di compliance nelle
due varianti leggibile e machine-readable, il file `THIRD-PARTY-NOTICES` con i testi integrali delle
licenze, e il registro delle eccezioni approvate.

Una nota pratica che fa risparmiare parecchi grattacapi: l'SBOM prodotto per il Cyber Resilience Act
e quello prodotto per la license compliance **devono essere lo stesso artefatto**. Mantenerne due è
una fonte garantita di divergenza.

## Per approfondire

Il repository è pubblico:
**[github.com/emanbuc/licence-audit-py](https://github.com/emanbuc/licence-audit-py)**.

Oltre allo script e alla policy di esempio, contiene la
[guida operativa completa](https://github.com/emanbuc/licence-audit-py/blob/main/guida_license_compliance_python.md):
la pipeline in sette fasi, la toolchain di riferimento (`pip-licenses`, `licensecheck`,
`cyclonedx-py`, `syft`, ScanCode, ORT, FOSSology), la tassonomia operativa delle licenze, la matrice
decisionale per contesto di distribuzione, il workflow GitHub Actions pronto all'uso e il triage dei
falsi positivi più frequenti.

Lo strumento è rilasciato sotto GPL-3.0 — il che, con una certa ironia, lo colloca nella categoria
che etichetta come `strong_copyleft`. Nessun conflitto: si usa come eseguibile autonomo per
analizzare un ambiente, non come libreria da importare nel vostro prodotto. Che poi è esattamente la
distinzione fra linking e mera aggregazione di cui sopra — un buon promemoria del fatto che, in
questa materia, la domanda giusta non è mai "che licenza ha?" ma "che licenza ha, e *come* lo sto
usando?".
