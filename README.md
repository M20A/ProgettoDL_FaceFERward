# ProgettoDL FaceFERward

Progetto di Deep Learning per il riconoscimento delle emozioni facciali sul dataset FER-2013.

Il progetto usa immagini grayscale 48x48 divise in 7 classi:

- angry
- disgust
- fear
- happy
- neutral
- sad
- surprise

## Stato attuale

Riferimento Git usato per il confronto:

```text
1070df460f5ff158d3f9ff5b681dec9aff460b3a - Aggiunta commenti vari
```

Rispetto a questo commit, il lavoro attuale modifica soprattutto:

- `notebooks/Preprocessing.ipynb`
- `notebooks/Training.ipynb`
- aggiunge `requirements.txt`
- aggiunge `.gitignore`

Il dataset originale resta in:

```text
data/original/train
data/original/test
```

La cartella `data/processed/` e l'ambiente virtuale `.venv/` sono esclusi dal push tramite `.gitignore`.
Le run di training vengono salvate in `experiments/`, mentre figure, tabelle e predizioni riutilizzabili vengono esportate in `results/`.

## Storico modifiche

### Commit precedente di riferimento

Il commit:

```text
1070df460f5ff158d3f9ff5b681dec9aff460b3a - Aggiunta commenti vari
```

viene mantenuto come riferimento storico per confrontare la prima versione del progetto con quella attuale.

In quella versione il flusso era piu' legato a Colab/Google Drive e il training non usava ancora direttamente una cartella `data/processed` generata dal preprocessing.

### Commit del 27/06/2026

In data 27/06/2026 e' stato aggiornato il flusso dei notebook per rendere effettivo l'utilizzo del preprocessing nel training.

Modifiche principali:

- `notebooks/Preprocessing.ipynb` crea uno split preprocessato fisico in:

```text
data/processed/train
data/processed/validation
```

- `data/processed/train` contiene le immagini originali di training piu' una quota di immagini augmentate.
- `data/processed/validation` contiene immagini di validazione non augmentate.
- `notebooks/Training.ipynb` non usa piu' `data/original/train` direttamente per il training.
- `notebooks/Training.ipynb` usa ora:

```text
data/processed/train
data/processed/validation
data/original/test
```

- `data/original/test` resta separato e viene usato solo per la valutazione finale.
- Le metriche gia' riportate nel README appartengono alla run precedente e vanno aggiornate dopo un nuovo training con i dati preprocessati.

### Aggiornamento struttura esperimenti

Il progetto usa ora la struttura suggerita dalle linee guida:

```text
assets/
data/
experiments/
notebooks/
results/
```

Ogni nuovo training crea una sottocartella dedicata in `experiments/`, ad esempio:

```text
experiments/20260628_153012_cnn_v1/
```

La cartella dell'esperimento contiene almeno:

```text
config.json
model.keras
training_history.csv
training_log.txt
test_results.txt
```

Gli output pensati per documentazione, confronto e presentazione vengono salvati in:

```text
results/figures/
results/tables/
results/predictions/
```

### Aggiornamento monitoraggio real-time

I notebook sono stati aggiornati per mostrare l'avanzamento delle fasi principali durante l'esecuzione, utile soprattutto su Google Colab.

In particolare:

- ogni fase stampa messaggi con orario di inizio/fine e durata;
- il preprocessing mostra barre di avanzamento durante split, copia immagini e augmentation;
- il training stampa numero di immagini, batch per epoca, distribuzione classi e `class_weight`;
- durante il training viene mostrata la durata di ogni epoca e le metriche principali;
- evaluation carica il modello salvato e mostra progressi durante valutazione, predizioni, report e confusion matrix.

Per le barre di avanzamento viene usata la libreria `tqdm`, inclusa in `requirements.txt`.

## Differenze principali dal commit precedente

### Preprocessing

Prima erano presenti riferimenti a Colab e Google Drive, per esempio:

- `/content/FER2013`
- `/content/drive/MyDrive/FER2013_split`
- `google.colab`
- split fisico copiando immagini in nuove cartelle

Ora il preprocessing e' piu' adatto a girare da VS Code in locale:

- usa percorsi relativi al progetto
- evita dipendenze da Colab
- non richiede uno split fisico su Drive
- mantiene separato il dataset originale
- crea `data/processed/train` e `data/processed/validation` a partire dal training set originale
- aggiunge immagini augmentate solo in `data/processed/train`
- lascia intatto `data/original/test`, che resta riservato alla valutazione finale

### Training

Il training e' stato ristrutturato per usare:

```text
data/processed/train
data/processed/validation
data/original/test
```

La validation non viene piu' creata al volo con `validation_split`: viene letta direttamente da `data/processed/validation`, generata nel notebook di preprocessing.

Il test set rimane separato e viene usato solo alla fine per la valutazione.

## Modello attuale

Il modello precedente era una CNN piu' semplice, con:

- pochi blocchi convoluzionali
- `Flatten()`
- training a 15 epoche
- nessun bilanciamento esplicito delle classi

Il modello attuale usa:

- piu' blocchi convoluzionali
- `BatchNormalization`
- `Dropout`
- `GlobalAveragePooling2D`
- `class_weight` per gestire classi sbilanciate
- learning rate piu' basso: `3e-4`
- massimo 50 epoche con `EarlyStopping`
- `ReduceLROnPlateau`
- salvataggio del miglior modello in una cartella dedicata sotto `experiments/`
- esportazione di curve, confusion matrix, report e predizioni in `results/`

Parametri del modello attuale:

```text
Total params: 305,639
Trainable params: 304,743
Non-trainable params: 896
```

Il numero di parametri e' piu' basso soprattutto perche' `GlobalAveragePooling2D` sostituisce `Flatten()`. Questo riduce molto i parametri nella parte finale del classificatore e puo' aiutare a limitare l'overfitting.

## Risultati ottenuti

Ultima run del notebook `Training.ipynb`:

Nota: questi risultati fanno riferimento alla run `experiments/20260627_180349_cnn_v1/`.

```text
Train images:      32,155
Validation images:  5,741
Test images:        7,178
```

Training completato in 50 epoche su 50.

Risultato finale sul test set:

```text
Test loss:     1.1599
Test accuracy: 0.5592
Test AUC:      0.8854
```

Rispetto al risultato precedente riportato di circa:

```text
Test accuracy: 0.44
```

il nuovo risultato e' un miglioramento netto:

```text
0.44 -> 0.5592
```

cioe' circa +12 punti percentuali di accuracy.

## Analisi dei risultati

Il miglioramento sembra reale dal punto di vista pratico: il modello generalizza meglio sul test set rispetto alla CNN iniziale.

Le classi migliori sono:

- `happy`, con F1-score circa 0.83
- `surprise`, con F1-score circa 0.70
- `neutral`, con F1-score circa 0.55

Le classi piu' problematiche sono:

- `fear`, con recall basso
- `disgust`, molto sbilanciata e con pochi esempi
- `sad`, spesso confusa con `neutral`

Questo e' coerente con FER-2013: il dataset e' rumoroso, alcune emozioni sono visivamente simili e le classi non sono perfettamente bilanciate.

## Data leakage

La struttura attuale riduce il rischio di data leakage:

- training e validation derivano da `data/processed/train` e `data/processed/validation`, generati solo a partire da `data/original/train`
- le immagini augmentate vengono aggiunte solo al training preprocessato
- il test set usa solo `data/original/test`
- il test viene usato solo alla fine con `model.evaluate`

Per una verifica piu' forte si puo' aggiungere un controllo hash per cercare duplicati tra train e test.

## Come eseguire il progetto

Creare un ambiente Python da VS Code e installare le dipendenze:

```bash
pip install -r requirements.txt
```

Poi eseguire i notebook in questo ordine:

1. `notebooks/Preprocessing.ipynb`
2. `notebooks/Training.ipynb`
3. `notebooks/Evaluation.ipynb`, se si vuole separare la fase di valutazione

Il preprocessing va eseguito prima del training perche' crea `data/processed/train` e `data/processed/validation`, cioe' le cartelle che il notebook di training usa come sorgenti.

Su Windows TensorFlow non usa la GPU nativa con le versioni moderne. Per training piu' veloce conviene usare Google Colab con GPU oppure WSL2.

## Possibili miglioramenti

Prossimi step consigliati:

- usare transfer learning con un modello preaddestrato
- provare architetture piu' robuste come MobileNet/EfficientNet adattate a FER-2013
- usare callback e metriche per salvare e confrontare piu' esperimenti
- aggiungere controllo hash per escludere duplicati tra train e test
- arricchire `results/` con grafici comparativi tra modelli e analisi degli errori
- provare tuning di augmentation, learning rate e batch size
- usare Colab/GPU per testare piu' configurazioni in meno tempo

## Conclusione

La modifica e' un miglioramento sia strutturale sia prestazionale.

Strutturale, perche' il progetto e' piu' portabile, meno dipendente da Colab e piu' ordinato nei percorsi.

Prestazionale, perche' l'accuracy sul test set passa da circa 0.44 a 0.5592 nella run attuale.

Il modello non e' ancora allo stato dell'arte, ma ora ha una base piu' solida su cui continuare.
