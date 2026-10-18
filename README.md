# Gestione di un dataset di film mediante orchestrazione di una pipeline

Pipeline di trasformazione dati costruita in **Azure Data Factory** per
processare grandi volumi di dati cinematografici.  
Il progetto prevede il
caricamento di un file CSV contenente informazioni su film, genere e
valutazioni, e la trasformazione di tali dati per renderli più rilevanti
e utilizzabili.

### Passi Chiave del Processo

1.  **Caricamento e Pulizia dei Dati**: si procede subito con la
    creazione di un gruppo di risorse, e successivamente con un account
    di archiviazione e due contenitori, **input** e **output**. Nel
    primo viene caricato il csv su cui effettuare le manipolazioni.\
    Aprendo il csv si notano subito dei dati sporchi, ad esempio le
    intestazioni di colonna("YEAR" scritto in maiuscolo), oppure alcuni
    valori della colonna "anno" negativi. Si decide di usare power query
    per eseguire le operazioni di pulizia. Nello specifico:

- si converte la colonna "YEAR" in numero, in quanto è stata
  riconosciuta come stringa, dopodichè si crea una colonna con il valore
  assoluto di YEAR per eliminare il segno negativo. Si procede poi ad
  eliminare la vecchia colonna
- viene rinominata la colonna "genresgenregenre" in "Genre".

Per quest'ultima colonna si nota inoltre che sono presenti più generi,
separati dal carattere \| .

La versione di Power Query su Azure Data Factory ha delle funzioni
limitate rispetto a quella presente in Excel/PowerBI, quindi non è
possibile separare la colonna in base ad un carattere delimitatore, in
modo da avere una colonna per ogni valore. Si decide quindi di mantenere
tutti i valori in una colonna.

2.  **Rimappatura dei Campi**: per la traduzione si sceglie di usare l'
    API della risorsa Translator, opportunamente importata nel gruppo di
    risorse.\
    Per dare in pasto all'attività Web della traduzione le intestazioni
    di colonna, si sceglie di usare l'attività "recupera metadati". Dato
    che l'output di questa attività non contiene solo i nomi di colonna,
    si sceglie di estrarli tramite un costrutto **For Each**, che
    inserisce i valori in una lista, salvata poi come variabile.\
    NB Nel For Each viene flaggata l'opzione "Sequenziale", per far sì
    che l'append metta le colonne nello stesso ordine in cui sono
    arrivate dall'attività precedente. Questo è importante per il
    DataFlow creato successivamente nella pipeline (vedi spiegazione al
    punto 3).\
    \
    Parallelamente al For Each, viene estratta, da un Vault creato
    appositamente per il progetto, la key necessaria per il
    funzionamento dell'API. Quest'operazione avviene tramite un'attività
    Web, con il meotodo GET.\
    \
    L'attività successiva è la traduzione vera e propria; viene quindi
    creata un'attività Web, con il metodo POST.\
    \
    Si può quindi procedere con la mappatura in italiano: viene creato
    un dataflow che prende in origine il dataset salvato dopo
    l'elaborazione in Power Query, e si effettua una select con un
    mapping basato su regole. Le colonne vengono selezionate tramite il
    parametro **name**, la traduzione delle colonne invece è passata al
    dataflow tramite il parametro \$ColonneItaliano, creato per
    permettere al dataflow di accedere alle intestazioni tradotte nella
    pipeline. Di seguito uno screenshot esplicativo:  
    ![screenshot](https://github.com/user-attachments/assets/9853d185-ca65-42f8-ac7b-e41b6c8a199f)  
    Si sfrutta in questo passaggio la possibilità di filtrare solo le
colonne necessarie, quindi Film, Genere e Valutazione.

3.  **Filtraggio dei Dati**: per tenere solo i record con valutazione
    superiore a 7, si usa un filtro nel dataflow:
    toInteger(byName(\$ColonneItaliano\[4\])) \> 7 .\
    Non è stato possibile usare direttamente il nome della colonna
    perché avendola assegnata nella select in modo
    dinamico(\$ColonneItaliano\[4\]), il generatore di espressioni non
    riesce a trovare le colonne inserendo direttamente il loro nome.

    \
    A questo punto possiamo esportare il dataset pulito in un sink. Si
    sceglie di non esportare direttamente nel contenitore "output"
    perché in questa fase non si riesce a selezionare dei metadati da
    conservare.

4.  ****Trasferimento e Parallelizzazione****: l'ultima attività della
    pipeline è un copia dati, grazie al quale si trasferisce il dataset
    pulito nel contenitore di output. La parallelizzazione avviene
    settando il grado di parallelismo della copia nelle impostazioni
    dell'attività, in questo caso valorizzato a 2, data la mole non
    esagerata di dati che l'attività sta gestendo.

5.  **Gestione dei Metadata**: nell'attività di copia dati, nelle
    impostazioni del sink, sono stati aggiunti due metadati:
    IntestazioniInItaliano e IntestazioniInInglese. Essi sono stati
    valorizzati con delle variabili usate precedentemente nella
    pipeline, e convertiti in stringa in quanto erano stati creati come
    matrici. È stato poi verificato nel contenitore di output che siano
    stati opportunamente esportati:  
     ![screenshot](https://github.com/user-attachments/assets/20b228b9-3024-4003-a85d-dc056629c021)


**Documentazione**

- Limiti di manipolazione di Power Query:
  <https://learn.microsoft.com/en-us/azure/data-factory/wrangling-functions>
- Ottenimento key sulla pipeline:
  <https://learn.microsoft.com/en-us/azure/data-factory/how-to-use-azure-key-vault-secrets-pipeline-activities>
- Mapping delle colonne nella select:
  <https://learn.microsoft.com/en-us/azure/data-factory/tutorial-data-flow-dynamic-columns>
- Grado di parallelismo della copia:
  <https://learn.microsoft.com/it-it/azure/data-factory/copy-activity-performance-features#parallel-copy>