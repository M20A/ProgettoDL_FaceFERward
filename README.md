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

Questo README mantiene due livelli di informazione:

- lo stato attuale del progetto, cioe' come va eseguito e dove salva ora i file;
- lo storico delle modifiche e dei risultati, cosi' resta chiaro come il progetto e' cambiato nel tempo.

Quando viene fatta una nuova modifica importante, aggiungere una nuova voce nello storico invece di cancellare le informazioni precedenti.

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
- Le metriche precedenti vengono mantenute nello storico risultati, cosi' e' possibile confrontare l'evoluzione del progetto.

### Aggiornamento del 28/06/2026 - struttura esperimenti

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

### Aggiornamento del 27/06/2026 - monitoraggio real-time

I notebook sono stati aggiornati per mostrare l'avanzamento delle fasi principali durante l'esecuzione, utile soprattutto su Google Colab.

In particolare:

- ogni fase stampa messaggi con orario di inizio/fine e durata;
- il preprocessing mostra barre di avanzamento durante split, copia immagini e augmentation;
- il training stampa numero di immagini, batch per epoca, distribuzione classi e `class_weight`;
- durante il training viene mostrata la durata di ogni epoca e le metriche principali;
- evaluation carica il modello salvato e mostra progressi durante valutazione, predizioni, report e confusion matrix.

Per le barre di avanzamento viene usata la libreria `tqdm`, inclusa in `requirements.txt`.

### Aggiornamento del 28/06/2026 12:44 - notebook MobileNetV2

E' stato aggiunto il notebook:

```text
notebooks/Training_MobileNetV2.ipynb
```

Questo notebook avvia la seconda linea sperimentale del progetto: transfer learning con MobileNetV2 preaddestrata su ImageNet.

La struttura resta coerente con quella gia' usata dalla CNN:

- il preprocessing rimane comune e viene eseguito una sola volta con `notebooks/Preprocessing.ipynb`;
- il notebook MobileNetV2 legge `data/processed/train`, `data/processed/validation` e `data/original/test`;
- ogni run MobileNetV2 viene salvata in una nuova cartella `experiments/<timestamp>_mobilenetv2_transfer/`;
- figure, tabelle e predizioni vengono esportate in `results/figures/`, `results/tables/` e `results/predictions/`;
- la tabella `results/tables/models_comparison.csv` viene aggiornata per facilitare il confronto tra CNN custom e modello preaddestrato.

Il notebook e' stato creato per documentare il confronto sperimentale tra CNN custom e transfer learning.

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

### Risultato corrente di riferimento

Run attualmente usata come riferimento nel repository:

Nota: questi risultati fanno riferimento alla run `experiments/20260628_120337_cnn_v1/`.
La documentazione di questa run e' stata aggiornata il 28/06/2026 alle 12:31 (+02:00).

```text
Train images:      32,155
Validation images:  5,741
Test images:        7,178
```

Tempi rilevati durante l'esecuzione completa locale:

```text
Preprocessing completo:              54.0s
Verifica split preprocessati:         1.2s
Training modello:                  1468.5s (~24.5 minuti)
Valutazione finale nel training:       7.7s
Evaluation separata modello salvato:   3.0s
Predizioni e report in evaluation:     3.6s
```

Training completato in 50 epoche su 50. La migliore `val_loss` e' stata:

```text
Best val_loss: 1.1268 all'epoca 44
Best val_accuracy: 0.5748 all'epoca 46
Best val_auc: 0.8929 all'epoca 46
```

Risultato finale sul test set:

```text
Test loss:     1.1314
Test accuracy: 0.5705
Test AUC:      0.8918
Macro F1:      0.52
Weighted F1:   0.56
```

Rispetto al risultato iniziale riportato di circa:

```text
Test accuracy: 0.44
```

il nuovo risultato e' un miglioramento netto:

```text
0.44 -> 0.5705
```

cioe' circa +13 punti percentuali di accuracy.

La nuova struttura di salvataggio ha funzionato correttamente. La run ha prodotto:

```text
experiments/20260628_120337_cnn_v1/config.json
experiments/20260628_120337_cnn_v1/model.keras
experiments/20260628_120337_cnn_v1/training_history.csv
experiments/20260628_120337_cnn_v1/training_log.txt
experiments/20260628_120337_cnn_v1/test_results.txt
results/tables/preprocessing_split_counts.csv
results/tables/20260628_120337_cnn_v1_classification_report.csv
results/tables/20260628_120337_cnn_v1_confusion_matrix.csv
results/predictions/20260628_120337_cnn_v1_test_predictions.csv
results/figures/20260628_120337_cnn_v1_training_curves.png
results/figures/20260628_120337_cnn_v1_confusion_matrix.png
```

### Storico risultati

Le metriche seguenti vengono mantenute come storico. Non vanno eliminate quando si aggiunge una nuova run: servono a capire l'evoluzione del progetto.

#### Baseline iniziale

La prima versione del progetto riportava una test accuracy di circa:

```text
Test accuracy: 0.44
```

Questa baseline e' utile come punto di partenza per mostrare il miglioramento ottenuto con le versioni successive.

#### Run `experiments/20260627_180349_cnn_v1/`

Questa run e' stata mantenuta nello storico dopo il passaggio alla struttura `experiments/`.

```text
Train images:      32,155
Validation images:  5,741
Test images:        7,178
Training: 50 epoche su 50

Best val_loss: 1.1576 all'epoca 48
Test loss:     1.1599
Test accuracy: 0.5592
Test AUC:      0.8854
Macro F1:      0.51
Weighted F1:   0.55
```

#### Run precedente documentata nel README

Prima del riordino della struttura in `experiments/`, il README riportava una run con:

```text
Train images:      22,968
Validation images:  5,741
Test images:        7,178
Training: 42 epoche su 50, fermato dai callback

Test loss:     1.0841
Test accuracy: 0.5846
Test AUC:      0.9009
```

Questa run resta nello storico per confronto. Se si vuole usarla nella presentazione finale, conviene collegarla a una cartella esperimento completa con modello, log, history e risultati.

#### Run `experiments/20260628_124858_mobilenetv2_transfer/`

Il 28/06/2026 alle 14:04 (+02:00) e' stato documentato il primo esperimento di transfer learning con MobileNetV2 preaddestrata su ImageNet.

Configurazione principale:

```text
Base model:          MobileNetV2
Pretrained weights:  ImageNet
Input size:          96x96x3
Feature extraction:  15 epoche
Fine tuning:         20 epoche
Fine-tuned layers:   ultimi 30 layer, con BatchNormalization congelata
```

Risultati sul test set:

```text
Test accuracy: 0.5153
Test AUC:      0.8580
Macro F1:      0.4615
Weighted F1:   0.5026
Training:      49.1 minuti
Inference:     1.53 ms/image
```

Confronto con la run CNN di riferimento `experiments/20260628_120337_cnn_v1/`:

```text
CNN custom accuracy:      0.5705
MobileNetV2 accuracy:    0.5153

CNN custom macro F1:     0.5246
MobileNetV2 macro F1:    0.4615

CNN custom weighted F1:  0.5601
MobileNetV2 weighted F1: 0.5026
```

La run MobileNetV2 e' quindi inferiore alla CNN custom in questa configurazione, anche se e' piu' veloce in inferenza. Il risultato resta importante per il progetto perche' mostra che il transfer learning da ImageNet non porta automaticamente un miglioramento su FER-2013: il dominio di partenza e' RGB/natural images, mentre FER-2013 contiene facce grayscale 48x48.

Dal classification report emergono buone prestazioni su `happy` e `surprise`, un miglioramento relativo su `sad` rispetto alla CNN, ma difficolta' marcate su `fear`, che ha recall basso. Questo sara' utile per l'analisi per classe e per la discussione degli errori.

I file salvati sono:

```text
experiments/20260628_124858_mobilenetv2_transfer/
results/figures/20260628_124858_mobilenetv2_transfer_training_curves.png
results/figures/20260628_124858_mobilenetv2_transfer_confusion_matrix.png
results/tables/20260628_124858_mobilenetv2_transfer_classification_report.csv
results/tables/20260628_124858_mobilenetv2_transfer_confusion_matrix.csv
results/predictions/20260628_124858_mobilenetv2_transfer_test_predictions.csv
results/tables/models_comparison.csv
```

## Analisi dei risultati

Il miglioramento sembra reale dal punto di vista pratico: il modello generalizza meglio sul test set rispetto alla CNN iniziale.

Le classi migliori sono:

- `happy`, con F1-score circa 0.81
- `surprise`, con F1-score circa 0.71
- `neutral`, con F1-score circa 0.56

Le classi piu' problematiche sono:

- `fear`, con recall basso
- `sad`, spesso confusa con `neutral`
- `disgust`, molto sbilanciata e con pochi esempi, anche se in questa run ha recall alto

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
2. `notebooks/Training.ipynb`, per la CNN custom
3. `notebooks/Training_MobileNetV2.ipynb`, per il transfer learning con ImageNet
4. `notebooks/Evaluation.ipynb`, se si vuole separare la fase di valutazione

Il preprocessing va eseguito prima dei training perche' crea `data/processed/train` e `data/processed/validation`, cioe' le cartelle che i notebook di training usano come sorgenti.

`Training.ipynb` e `Training_MobileNetV2.ipynb` sono due esperimenti alternativi: non serve creare due preprocessing separati. Dopo averli eseguiti entrambi, i risultati salvati in `experiments/` e `results/` permettono di confrontare CNN custom e MobileNetV2.

Su Windows TensorFlow non usa la GPU nativa con le versioni moderne. Per training piu' veloce conviene usare Google Colab con GPU oppure WSL2.

## Possibili miglioramenti

Prossimi step consigliati:

- usare `models_comparison.csv` come tabella base per relazione e presentazione
- provare una seconda configurazione MobileNetV2 o EfficientNet solo se serve un ulteriore confronto
- usare callback e metriche per salvare e confrontare piu' esperimenti
- aggiungere controllo hash per escludere duplicati tra train e test
- arricchire `results/` con grafici comparativi tra modelli e analisi degli errori
- provare tuning di augmentation, learning rate e batch size
- usare Colab/GPU per testare piu' configurazioni in meno tempo

## Conclusione

La modifica e' un miglioramento sia strutturale sia prestazionale.

Strutturale, perche' il progetto e' piu' portabile, meno dipendente da Colab e piu' ordinato nei percorsi.

Prestazionale, perche' l'accuracy sul test set passa da circa 0.44 a 0.5705 nella run attuale.

Il modello non e' ancora allo stato dell'arte, ma ora ha una base piu' solida su cui continuare.
