# 🎬 Azure Data Factory ETL Pipeline: Data Processing & Translation


## 📌 Panoramica del Progetto
Questa repository contiene la definizione (in formato JSON) di una pipeline **ETL** sviluppata su **Azure Data Factory (ADF)**. 

L'obiettivo principale del progetto è l'ingestione, la pulizia, la traduzione dinamica delle intestazioni tramite API REST e il filtraggio di grandi volumi di dati, integrando best practices di sicurezza come la gestione dei segreti tramite Key Vault e l'arricchimento finale con metadati custom.

---

## 🚀 Key Features & Highlights Tecnici

* **Data Cleaning & Wrangling (Power Query):** 
  * Configurazione iniziale di un gruppo di risorse, un account di archiviazione e due contenitori (`input` e `output`). 
  * Risoluzione di anomalie nei dati di input: conversione della colonna `YEAR` in formato numerico (da stringa), applicazione del valore assoluto per eliminare i segni negativi e ridenominazione di colonne che presentano un nome errato.
  * Gestione dei generi multipli (separati dal carattere `|`) mantenuti in un'unica colonna a causa delle limitazioni di Power Query su ADF rispetto a strumenti come Excel o Power BI.

* **Dynamic Column Translation (Azure Translator API):** 
  * Estrazione delle intestazioni di colonna tramite l'attività `Recupera metadati`.
  * Utilizzo di un costrutto `For Each` configurato in modalità **Sequenziale** per inserire i valori in una lista salvata come variabile, preservando l'ordine di arrivo fondamentale per il successivo Data Flow.
  * Recupero protetto della chiave API tramite un'attività Web con metodo GET da un Azure Key Vault dedicato.
  * Esecuzione della traduzione tramite un'attività Web con metodo POST sfruttando la risorsa Translator.

* **Dynamic Rule-Based Mapping & Expression Filtering:** 
  * Sviluppo di un Data Flow in cui la select sfrutta un mapping basato su regole tramite il parametro dinamico `$ColonneItaliano`.
  * Filtraggio avanzato dei record con valutazione superiore a 7 mediante l'espressione condizionale `toInteger(byName($ColonneItaliano[4])) > 7`.

* **Optimization & Data Enrichment:**
  * Esportazione del dataset pulito in un sink e successivo trasferimento nel contenitore di output tramite un'attività di copia dati.
  * Ottimizzazione del throughput tramite il tuning del grado di parallelismo della copia impostato a 2.
  * Arricchimento dei dati finali tramite l'aggiunta di metadati di sink (`IntestazionilnItaliano` e `IntestazionilnInglese`) convertiti in stringa.

---

## 💻 Tech Stack & Servizi Azure

* **Orchestratore:** Azure Data Factory (ADF)
* **Storage:** Azure Blob Storage / Storage Account
* **Security:** Azure Key Vault
* **Cognitive Services:** Azure Translator API

---

## 📂 Struttura della Repository

```text
.
├── pipelines/
│   └── Pipeline.json             # File JSON esportato da Azure Data Factory
├── docs/
│   └── documentazione.pdf            # Documentazione tecnica di progetto
└── README.md                         # Documentazione della repository