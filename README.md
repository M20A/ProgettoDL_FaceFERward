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

La cartella `data/processed/`, l'ambiente virtuale `.venv/` e i modelli salvati in `models/` sono esclusi dal push tramite `.gitignore`.

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

### Training

Il training e' stato ristrutturato per usare direttamente:

```text
data/original/train
data/original/test
```

La validation viene creata da `data/original/train` con:

```python
validation_split=0.2
```

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
- salvataggio del miglior modello in `models/best_cnn.keras`

Parametri del modello attuale:

```text
Total params: 305,639
Trainable params: 304,743
Non-trainable params: 896
```

Il numero di parametri e' piu' basso soprattutto perche' `GlobalAveragePooling2D` sostituisce `Flatten()`. Questo riduce molto i parametri nella parte finale del classificatore e puo' aiutare a limitare l'overfitting.

## Risultati ottenuti

Ultima run del notebook `Training.ipynb`:

```text
Train images:      22,968
Validation images:  5,741
Test images:        7,178
```

Training completato in 42 epoche su 50, fermato dai callback.

Risultato finale sul test set:

```text
Test loss:     1.0841
Test accuracy: 0.5846
Test AUC:      0.9009
```

Rispetto al risultato precedente riportato di circa:

```text
Test accuracy: 0.44
```

il nuovo risultato e' un miglioramento netto:

```text
0.44 -> 0.5846
```

cioe' circa +14 punti percentuali di accuracy.

## Analisi dei risultati

Il miglioramento sembra reale dal punto di vista pratico: il modello generalizza meglio sul test set rispetto alla CNN iniziale.

Le classi migliori sono:

- `happy`, con F1-score circa 0.83
- `surprise`, con F1-score circa 0.72
- `neutral`, con F1-score circa 0.57

Le classi piu' problematiche sono:

- `fear`, con recall basso
- `disgust`, molto sbilanciata e con pochi esempi
- `sad`, spesso confusa con `neutral`

Questo e' coerente con FER-2013: il dataset e' rumoroso, alcune emozioni sono visivamente simili e le classi non sono perfettamente bilanciate.

## Data leakage

La struttura attuale riduce il rischio di data leakage:

- training e validation derivano solo da `data/original/train`
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

Su Windows TensorFlow non usa la GPU nativa con le versioni moderne. Per training piu' veloce conviene usare Google Colab con GPU oppure WSL2.

## Possibili miglioramenti

Prossimi step consigliati:

- usare transfer learning con un modello preaddestrato
- provare architetture piu' robuste come MobileNet/EfficientNet adattate a FER-2013
- usare callback e metriche per salvare e confrontare piu' esperimenti
- aggiungere controllo hash per escludere duplicati tra train e test
- salvare grafici e confusion matrix in una cartella `reports/`
- provare tuning di augmentation, learning rate e batch size
- usare Colab/GPU per testare piu' configurazioni in meno tempo

## Conclusione

La modifica e' un miglioramento sia strutturale sia prestazionale.

Strutturale, perche' il progetto e' piu' portabile, meno dipendente da Colab e piu' ordinato nei percorsi.

Prestazionale, perche' l'accuracy sul test set passa da circa 0.44 a 0.5846 nella run attuale.

Il modello non e' ancora allo stato dell'arte, ma ora ha una base piu' solida su cui continuare.
