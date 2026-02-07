# Image Captioning Evaluation für Barrierefreiheit

Code zur Masterarbeit "Automatische Bildunterschriften-Generierung mit Vision-Language-Modellen zur Förderung barrierefreier Medieninhalte"


### Hardware
- Google Colab Pro (oder eine GPU mit ≥16 GB VRAM)
- Etwa 6-8 Stunden Laufzeit für alle 1100 Bilder
- ~20 GB freien Drive-Speicher für Checkpoints

### Accounts
- Google Drive (für Datensätze und Checkpoints)
- OpenAI API Key (für GPT-4V, kostet ca. $15-20 für 400 Bilder)
- Optional: Kaggle Account für Flickr30k-Download

### Python-Pakete
```bash
torch torchvision torchaudio
transformers accelerate pillow
pycocoevalcap datasets
openai anthropic
nltk scipy scikit-image
salesforce-lavis
```

Alles wird im Notebook automatisch installiert

## Wie läuft das ab?

Das Notebook ist in 9 Abschnitte aufgeteilt:

**1. Setup**: GPU-Check, Package-Installation (dauert ~2 Min)

**2. Konfiguration**: Pfade zu Google Drive, API-Keys, Sample-Größen  
→ Hier musst du deine Ordnerstruktur anpassen

**3. Datensätze laden**: COCO (lokal), Flickr30k (Kaggle), VizWiz (Hugging Face)  
→ COCO muss vorher manuell in Drive hochgeladen werden

**4. Modelle laden**: BLIP-1, BLIP-2, GPT-4V  
→ Dauert ~5 Min, lädt ~4 GB Modellgewichte

**5. Caption-Generierung**: Das Herzstück  
→ Läuft in Batches, speichert alle 3 Bilder einen Checkpoint (wegen Colab-Timeouts)  
→ Dauert 6-8 Stunden für alle Datensätze

**6. Metriken**: BLEU-1 bis BLEU-4, CIDEr  
→ METEOR und SPICE sind auskommentiert (haben nicht funktioniert)

**7. WCAG-Bewertung**: Manuelle Annotation von 150 Bildern  
→ Verständlichkeit, Informationsgehalt, Kontextadäquanz

**8. Visualisierung**: Beispielbilder mit generierten Captions

**9. Export**: Speichert CSVs mit allen Ergebnissen

## Wichtige Hinweise

### Colab-Timeouts
Colab wirft dich nach 90 Min Inaktivität raus, auch mit Pro. Deswegen gibt's ein Keep-Alive-Script (Zelle 1) und das Checkpoint-System. Wenn Colab abstürzt: Einfach Zelle 5 nochmal starten, die verarbeiteten Bilder werden übersprungen.

### Drive-Struktur
Das Notebook erwartet diese Ordner in deinem Google Drive:
```
/content/drive/MyDrive/
├── data/
│   ├── coco2017/
│   │   ├── val2017/              # COCO-Bilder
│   │   └── annotations/
│   │       └── captions_val2017.json
│   └── flickr30k/                # Wird automatisch runtergeladen
└── caption_generation_workspace/
    ├── checkpoints/              # Zwischenstände
    └── results/                  # Finale CSVs
```

Wenn deine Struktur anders aussieht: Zelle 2 (CONFIG) anpassen.

### API-Kosten (GPT-4V)
GPT-4V kostet aktuell ~$0.01-0.03 pro Bild (je nach Auflösung). Für 400 COCO + 300 Flickr30k + 400 VizWiz = 1100 Bilder kommst du auf etwa $15-25. Wenn dir das zu viel ist: In der CONFIG die Sample-Größe reduzieren.

### Was hat nicht funktioniert
- **METEOR**: Java-Dependencies spinnen, gibt inkonsistente Ergebnisse → auskommentiert
- **SPICE**: Braucht Stanford Scene Graph Parser, den ich nicht zum Laufen bekommen habe → auskommentiert
- **CLIPCap**: Ursprünglich geplant, aber das Modell hat nur Müll generiert. Daher BLIP-1 als Ersatz.

## Ergebnisse

Die fertigen CSVs landen in `/content/drive/MyDrive/caption_generation_workspace/results/`:

- `caption_results_FINAL.csv`: Alle generierten Captions (1100 Zeilen)
- `automatic_metrics.csv`: BLEU/CIDEr-Scores pro Modell und Datensatz
- `wcag_summary.csv`: WCAG-Erfüllungsraten (Verständlichkeit, Informationsgehalt, Kontextadäquanz)

## Reproduzierbarkeit

Random Seed ist auf 42 gesetzt. Für BLIP-1 und BLIP-2 solltest du identische Ergebnisse bekommen. Bei GPT-4V kann es leichte Abweichungen geben (die API ist nicht 100% deterministisch, auch mit temperature=0).

## Wenn was schiefgeht

**"CUDA out of memory"**: Batch-Size in CONFIG runtersetzen (von 8 auf 4)

**"Drive not mounted"**: Zelle mit `drive.mount()` nochmal ausführen

**"Image not found"**: COCO-Pfad in CONFIG prüfen

**"OpenAI API error"**: API-Key abgelaufen oder Rate Limit. Warte 1 Min und starte neu.

**Colab disconnected**: Keep-Alive-Script in Zelle 1 laufen lassen, dann läuft Colab Pro ca. 12h durch

