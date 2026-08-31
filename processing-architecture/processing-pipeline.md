# Fluxul de procesare

Chloros1.2.0 utilizează un flux de procesare cu 4 fire de execuție care funcționează ca o linie de asamblare pe etape. Fiecare fir de execuție gestionează o fază distinctă a fluxului de lucru, astfel încât mai multe imagini pot fi în curs de procesare în diferite etape simultan.

<figure><img src="../.gitbook/assets/image (39).png" alt=""><figcaption></figcaption></figure>

***

## Arhitectura fluxului de procesare

```

Images In → [Thread 1: Detection] → [Thread 2: Calibration] → [Thread 3: Processing] → [Thread 4: Export] → Files Out
```

Fiecare imagine trece prin toate cele patru fire de execuție în ordine. Cu procesarea multi-thread a Chloros+, mai multe imagini ocupă simultan fire de execuție diferite — în timp ce firul 3 procesează o imagine, firul 1 poate detecta următoarea, firul 2 poate calibra o altă imagine, iar firul 4 poate scrie pe disc o imagine finalizată.

Progresul este raportat pentru fiecare fir de execuție și transmis prin Server-Sent Events (backend-ul le publică pe `/api/events`). În afișajul live al progresului din CLI, cele patru etape sunt etichetate **Detectare, Analizare, Prelucrare, Export**.***

## Detalii despre firele de execuție

### Firul 1: Detectare

**Scop**: Încărcarea imaginilor și detectarea țintelor de calibrare.

* Citește fișierele imagine de pe disc — perechile Survey3 `.raw`+`.jpg`, capturi LATTICE `.tif`/`.tiff` și `.dng`
* Extrage metadatele EXIF (GPS, modelul camerei, marcaje temporale, expunere)
* Detectează țintele de calibrare: geometrii ale țintelor marcate cu ArUco pentru capturile LATTICE și detectorul clasic de tip panou pentru fotografiile cu ținte de calibrare dSurvey3
* Rezultate: date de imagine + metadate + rezultate ale detectării țintelor

În principal, un thread limitat de I/O și de CPU.

### Thread 2: Calibrare

**Scop**: Calculează parametrii de calibrare pe baza țintelor detectate.

* Calculează coeficienții de calibrare a reflectanței din imaginile țintelor
* Calculează parametrii de corecție a vignetării
* Determină curbele de calibrare pentru fiecare bandă
* Rezultate: parametrii de calibrare pentru fiecare imagine

Un thread de calcul limitat de CPU. Threadul 3 așteaptă finalizarea acestuia atunci când calibrarea reflectanței este activată, astfel încât coeficienții să fie gata înainte ca orice imagine să fie procesată.

### Firul 3: Procesare (GPU)

**Scop**: Aplicarea corecțiilor și calcularea indicilor de vegetație.**Acesta este firul cu cea mai mare intensitate de calcul.*** **Debayering**: convertește datele RAW Bayer în imagini multicanal
  * Standard (rapid, calitate medie) — implicit, `--debayer standard`
  * Texture Aware (lent, calitate maximă) — numai pentru Chloros+, `--debayer texture-aware`, utilizează un model de eliminare a zgomotului bazat pe AI/ML
  * Capturile LATTICE mono (M3M) sunt monobandă: etapele de demosaic și de echilibrare a albului sunt omise pentru acestea (cu un mesaj de jurnal de o singură linie), în timp ce orice imagini M3C/Bayer din aceeași serie beneficiază în continuare de acestea
* **Corecția vignetării**: aplică corecția vignetării obiectivului pe întreaga imagine
* **Calibrarea reflectanței**: aplică coeficienți de calibrare pentru a converti valorile în valori de reflectanță
* **Calculul indicilor**: calculează indicii de vegetație (NDVI, NDRE, GNDVI, …)
* Rezultate: date de imagine procesate, gata pentru export

Acest fir beneficiază cel mai mult de accelerarea GPU și este firul pe care îl reglează [Adaptarea dinamică a calculului](dynamic-compute-adaptation.md).

### Firul 4: Export

**Scop**: Scrierea imaginilor procesate pe disc.

* Scrie fișierele de ieșire în formatul selectat — `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)`
* Încorporează metadate în fișierele de ieșire (GPS, marcaje temporale, parametri de procesare)
* Organizează fișierele de ieșire în folderul proiectului sub forma `<camera>/<format>/<Product>_Images/` — de exemplu, `LATT-M3M-L41-F550/tiff16/Reflectance_Calibrated_Images/`. **Fișierele exportate păstrează numele fișierului sursă; folderul identifică produsul.**
* Pentru capturile LATTICE, un cadru sursă poate genera mai multe produse (Debayered, Preview, Radiance, Reflectance, Index), fiecare în propriul folder de produse
* Rezultate: fișiere finale pe disc

În principal, un thread limitat de I/O — stocarea pe SSD îmbunătățește semnificativ performanța.

***

## În detaliu: Executori

În cadrul Thread 3, procesarea fiecărei imagini este paralelizată cu `concurrent.futures`-ul standard dPython:

* **Strategiile GPU**(`GPU_SINGLE`, `GPU_PARALLEL`) utilizează un `ProcessPoolExecutor` cu metoda**spawn** — fiecare procesor este un proces separat, cu propriul context CUDA (`fork` ar moșteni starea CUDA inițializată a părintelui și ar corupe procesele fiice)
* **`CPU_PARALLEL`** utilizează un `ThreadPoolExecutor` — NumPy și OpenCV eliberează GIL-ul, astfel încât thread-urile sunt suficiente
* Dispozitivele Jetson cu 8 GB sau mai puțină memorie RAM partajată omit complet executorul și procesează în cadrul procesului, secvențial
* Texture Aware pe un GPU cu sub 7 GB VRAM rulează, de asemenea, secvențial — modelul de denoizare nu se poate încadra decât o singură dată

Chlorosnu utilizează niciun cadru distribuit de la terți (cum ar fi Ray). Consultați [Adaptarea dinamică a calculului](dynamic-compute-adaptation.md) pentru a afla cum sunt alese strategia și numărul de lucrători.

***

## Procesare secvențială vs. procesare în pipeline

### Mod liber (secvențial)

În versiunea gratuită a Chloros, imaginile sunt procesate **câte una pe rând**, secvențial, prin toate cele patru etape:

```

Image 1: [Detect] → [Calibrate] → [Process] → [Export]
                                                         Image 2: [Detect] → [Calibrate] → [Process] → [Export]
```

Interfața grafică afișează o bară de progres simplificată în modul gratuit; fazele sale seriale sunt indicate ca **Detectarea țintei**și apoi**Procesare**.

### Modul „Chloros” (în serie)

Cu o licență „Chloros”, toate cele patru fire de execuție funcționează **concomitent** pe imagini diferite:

```

Thread 1: [Image 1] [Image 2] [Image 3] [Image 4] ...
Thread 2:           [Image 1] [Image 2] [Image 3] ...
Thread 3:                     [Image 1] [Image 2] ...
Thread 4:                               [Image 1] ...
```

Bara de progres din interfața grafică (GUI) afișează cele patru etape; treceți cu mouse-ul peste ea pentru a vedea progresul fiecărui thread. În interfața de linie de comandă (CLI), aceleași patru etape sunt afișate în timp real ca **Detectare, Analizare, Procesare, Export**.

{% hint style="info" %}
**O singură etichetă, două denumiri.** CLI numește etapa 3 _Procesare_. Fluxul de progres din modul premium al backend-ului — cel redat de bara de progres din interfața grafică — etichetează aceeași etapă ca _Calibrare_. Este vorba despre același thread care efectuează aceeași operațiune (Thread 3: debayer, corecții, indici).
{% endhint %}

{% hint style="success" %}
**Prelucrarea în serie cu „Chloros”+** poate fi de 3-5 ori mai rapidă decât prelucrarea secvențială, în funcție de hardware și de dimensiunea setului de date. Creșterea de viteză este maximă pe sistemele cu GPU-uri rapide și SSD-uri.
{% endhint %}

***

## Progresul firului 4 de export

Firul de export are propriul sistem de urmărire a progresului, pe care îl puteți interoga separat:**CLI:**

```bash
chloros-cli export-status
```

**SDK:**

```python
status = chloros.get_status()
print(f"Export: {status['export']['percent']}% - Phase: {status['export']['phase']}")
```

Prelucrarea este finalizată când firul 4 ajunge la 100%.

{% hint style="info" %}
**O execuție care nu scrie nicio imagine este considerată eșuată.**În cazul unei execuții reușite, `chloros-cli process` raportează câte produse imagine a scris (`Image products written: N`). Dacă au fost solicitate produse și**niciunul**nu a fost scris — doar `project.json` și `calibration_data.json` — CLI afișează `Processing finished but wrote no image products.` și**iese cu valoare diferită de zero**, menționând numele folderului proiectului și cauzele obișnuite (folderul de intrare nu a fost recunoscut ca o captură — verificați structura și `--input-level` — sau fiecare produs solicitat era inaplicabil pentru acele camere). Scripturile se pot baza pe codul de ieșire.
{% endhint %}

***

## Relația cu Adaptarea dinamică a calculului

[Adaptarea dinamică a calculului](dynamic-compute-adaptation.md) afectează în principal **Firul 3 (Prelucrare)**:

* **`GPU_PARALLEL`**: Thread 3 procesează simultan mai multe imagini prin GPU folosind pipeline-ul `fused_gpu`
* **`GPU_SINGLE`**: Thread 3 serializează accesul la GPU cu ajutorul unui semafor, în timp ce procesele de lucru suprapun operațiunile de I/O, utilizând `fused_gpu` sau pipeline-ul `tiled_gpu`, eficient din punct de vedere al memoriei
* **`CPU_PARALLEL`**: Firul 3 utilizează procesarea bazată pe CPU cu paralelism multithread

Alocarea memoriei GPU a firului 3 crește, de asemenea, pe măsură ce firele 1 și 2 își finalizează execuția — consultați [Alocarea dinamică a memoriei GPU](dynamic-compute-adaptation.md#dynamic-gpu-memory-allocation).

***

## Pași următori

* [Adaptarea dinamică a calculului](dynamic-compute-adaptation.md) — Cum selectează Chloros strategia optimă pentru hardware-ul dvs.
* [Ghidul NVIDIA Jetson](../linux/nvidia-jetson-guide.md) — Comportamentul specific platformei al pipeline-ului pe Jetson
* [Monitorizarea procesării](../processing-images-gui/monitoring-the-processing.md) — Monitorizarea progresului prin interfața grafică
* [Referință CLI](../reference/cli-reference.md) — `process`, `export-status`, coduri de ieșire și structura ieșirii
