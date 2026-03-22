# Fluxul de procesare

Chloros 1.1.0 utilizează un flux de procesare cu 4 fire de execuție care funcționează ca o linie de asamblare etapizată. Fiecare fir de execuție gestionează o fază distinctă a fluxului de lucru de procesare, permițând procesarea simultană a mai multor imagini în diferite etape.

***

## Arhitectura fluxului de procesare

```

Images In → [Thread 1: Detection] → [Thread 2: Calibration] → [Thread 3: Processing] → [Thread 4: Export] → Files Out
```

Fiecare imagine trece prin toate cele patru fire în ordine. Cu procesarea multi-fire Chloros+, mai multe imagini pot fi în fire diferite simultan — în timp ce Firul 3 procesează o imagine, Firul 1 poate detecta următoarea, Firul 2 poate calibra o alta, iar Firul 4 poate scrie o imagine procesată anterior pe disc.

***

## Detalii despre fire

### Firul 1: Detectare

**Scop**: Încarcă imagini și detectează ținte de calibrare.

* Citește fișiere imagine de pe disc (RAW, JPG)
* Extrage metadatele EXIF (GPS, modelul camerei, timestamp-uri, expunere)
* Detectează țintele de calibrare ArUco în imaginile cu ținte marcate
* Rezultate: date de imagine + metadate + rezultate ale detectării țintelor

Acesta este în primul rând un thread legat de I/O și CPU.

### Thread 2: Calibrare

**Scop**: Calculează parametrii de calibrare din țintele detectate.

* Calculează coeficienții de calibrare a reflectanței din imaginile țintă
* Calculează parametrii de corecție a vignetării
* Determină curbele de calibrare pentru fiecare bandă
* Rezultate: parametrii de calibrare pentru fiecare imagine

Acesta este un thread de calcul dependent de CPU.

### Thread 3: Procesare (GPU)

**Scop**: Aplică corecții și calculează indicii de vegetație.**Acesta este thread-ul cu cea mai mare intensitate de calcul.*** **Debayering**: Convertește datele RAW în format Bayer în imagini multicanal
  * Standard (Rapid, Calitate medie) — implicit
  * Texture Aware (Lent, Calitate maximă) — numai Chloros+, utilizează denoizare AI/ML
* **Corecția vignetării**: Aplică corecția vignetării obiectivului pe întreaga imagine
* **Calibrarea reflectanței**: Aplică coeficienți de calibrare pentru a converti în valori de reflectanță
* **Calculul indicilor**: Calculează indicii de vegetație (NDVI, NDRE, GNDVI etc.)
* Rezultate: date de imagine procesate, gata pentru export

Acest thread beneficiază cel mai mult de accelerarea GPU. Sistemul [Dynamic Compute Adaptation](dynamic-compute-adaptation.md) optimizează în principal comportamentul acestui thread.

### Firul 4: Export

**Scop**: Scrierea imaginilor procesate pe disc.

* Scrie fișierele de ieșire în formatul selectat (TIFF 16 biți, TIFF 32 biți %, PNG, JPG)
* Încorporează metadate EXIF în fișierele de ieșire (GPS, timestamp-uri, parametri de procesare)
* Organizează ieșirea în subfoldere pentru fiecare model de cameră
* Ieșiri: fișiere finale pe disc

Acesta este în principal un thread legat de I/O. Stocarea pe SSD îmbunătățește semnificativ performanța Thread-ului 4.

***

## Procesare secvențială vs. procesare în pipeline

### Mod gratuit (secvențial)

În versiunea gratuită a Chloros, imaginile sunt procesate **una câte una**, secvențial, prin toate cele patru etape:

```

Image 1: [Detect] → [Calibrate] → [Process] → [Export]
                                                         Image 2: [Detect] → [Calibrate] → [Process] → [Export]
```

Bara de progres din interfața grafică afișează 2 etape: Detectarea țintei și Procesarea.

### Modul Chloros+ (în serie)

Cu o licență Chloros+, toate cele patru fire de execuție funcționează **concomitent** pe imagini diferite:

```

Thread 1: [Image 1] [Image 2] [Image 3] [Image 4] ...
Thread 2:           [Image 1] [Image 2] [Image 3] ...
Thread 3:                     [Image 1] [Image 2] ...
Thread 4:                               [Image 1] ...
```

Bara de progres a interfeței grafice (GUI) afișează 4 etape: Detectare, Analiză, Calibrare, Export. Treceți cu mouse-ul peste bara de progres pentru a vedea progresul fiecărui fir de execuție.

{% hint style="success" %}
**Procesarea în serie cu Chloros+** poate fi de 3-5 ori mai rapidă decât procesarea secvențială, în funcție de hardware și de dimensiunea setului de date. Creșterea de viteză este maximă pe sistemele cu GPU-uri rapide și SSD-uri.
{% endhint %}

***

## Progresul exportului firului 4

În Chloros 1.1.0, firul de export (Firul 4) are propria sa urmărire dedicată a progresului. Puteți monitoriza separat progresul exportului:**CLI:**
```bash
chloros-cli export-status
```

**SDK:**
```python
status = chloros.get_status()
print(f"Export: {status['export']['percent']}% - Phase: {status['export']['phase']}")
```

Procesarea este finalizată când Thread 4 ajunge la 100%.

***

## Relația cu Adaptarea dinamică a calculului

Sistemul [Adaptare dinamică a calculului](dynamic-compute-adaptation.md) afectează în principal **Thread 3 (Procesare)**:

* Strategia **`GPU_PARALLEL`**: Thread 3 rulează simultan mai multe imagini prin GPU folosind pipeline-ul `fused_gpu`
* Strategia **`GPU_SINGLE`**: Thread 3 procesează o singură imagine la un moment dat folosind pipeline-ul `tiled_gpu`, eficient din punct de vedere al memoriei
* Strategia **`CPU_PARALLEL`**: Thread 3 utilizează procesarea bazată pe CPU cu paralelism multi-thread

Alocarea memoriei GPU pentru Thread 3 se modifică, de asemenea, dinamic pe măsură ce Thread-urile 1 și 2 se finalizează — consultați [Alocarea dinamică a memoriei GPU](dynamic-compute-adaptation.md#dynamic-gpu-memory-allocation).

***

## Pași următori

* [Adaptarea dinamică a calculului](dynamic-compute-adaptation.md) — Cum selectează Chloros strategia optimă pentru hardware-ul dvs.
* [Ghidul NVIDIA Jetson](../linux/nvidia-jetson-guide.md) — Comportamentul pipeline-ului specific platformei pe Jetson
* [Monitorizarea procesării](../processing-images-gui/monitoring-the-processing.md) — Monitorizarea progresului prin GUI
