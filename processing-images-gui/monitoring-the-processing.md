# Monitorizarea procesării

Odată ce procesarea a început, Chloros oferă mai multe modalități de a monitoriza progresul, de a verifica dacă există probleme și de a înțelege ce se întâmplă cu setul dvs. de date. Această pagină explică cum să urmăriți procesarea și să interpretați informațiile furnizate de Chloros.

## Prezentare generală a barei de progres

Bara de progres din antetul superior afișează starea procesării în timp real și procentul de finalizare.

### Bara de progres în modul gratuit

Pentru utilizatorii fără licență Chloros+:

**Afișare a progresului în 2 etape:**

1.**Detectarea țintelor** - Găsirea țintelor de calibrare în imagini
2. **Procesare** - Aplicarea corecțiilor și exportarea**Bara de progres afișează:**

* Procentul general de finalizare (0-100%)
* Numele etapei curente
* Vizualizare simplă sub formă de bară orizontală

### Bara de progres Chloros+

Pentru utilizatorii cu licență Chloros+:

**Afișare progres în 4 etape:**

1.**Detectare** - Identificarea țintelor de calibrare
2. **Analiză** - Examinarea imaginilor și pregătirea fluxului de lucru
3. **Calibrare** - Aplicarea corecțiilor de vignetă și reflectanță
4. **Export** - Salvarea fișierelor procesate**Funcții interactive:*** **Treceți cu mouse-ul peste** bara de progres pentru a vedea panoul extins cu 4 etape
* **Faceți clic** pe bara de progres pentru a îngheța/fixa panoul extins
* **Faceți clic din nou** pentru a dezgheța și a ascunde automat la îndepărtarea mouse-ului
* Fiecare etapă afișează progresul individual (0-100%)

***

## Înțelegerea fiecărei etape de procesare

{% hint style="info" %}
**Arhitectura pipeline**: Aceste 4 etape ale interfeței grafice corespund [pipeline-ului de procesare cu 4 thread-uri](../processing-architecture/processing-pipeline.md). Pe sistemele cu accelerare GPU, Thread 3 (Calibrare) beneficiază de [Adaptare dinamică a calculului](../processing-architecture/dynamic-compute-adaptation.md), care optimizează procesarea pentru hardware-ul dvs. specific.
{% endhint %}

### Etapa 1: Detectare (Detectarea țintelor)

**Ce se întâmplă:**

* Chloros scanează imaginile marcate cu caseta de selectare Țintă
* Algoritmii de viziune computerizată identifică cele 4 panouri de calibrare
* Valorile de reflectanță sunt extrase din fiecare panou
* Se înregistrează marcajele temporale ale țintelor pentru programarea corectă a calibrării

**Durată:**

* Cu ținte marcate: 10-60 secunde
* Fără ținte marcate: 5-30+ minute (scanează toate imaginile)

**Indicator de progres:**

* Detectare: 0% → 100%
* Numărul de imagini scanate
* Numărul de ținte găsite

**Ce trebuie urmărit:**

* Ar trebui să se finalizeze rapid dacă țintele sunt marcate corespunzător
* Dacă durează prea mult, este posibil ca țintele să nu fie marcate
* Verificați jurnalul de depanare pentru mesaje de tip „Țintă găsită”

### Etapa 2: Analizarea

**Ce se întâmplă:**

* Citirea metadatelor EXIF ale imaginii (timestamp-uri, setări de expunere)
* Determinarea strategiei de calibrare pe baza timestamp-urilor țintelor
* Organizarea cozii de procesare a imaginilor
* Pregătirea procesorilor de procesare paralelă (numai Chloros+)

**Durată:** 5-30 secunde**Indicator de progres:**

* Analizare: 0% → 100%
* Etapă rapidă, se finalizează de obicei repede

**Ce trebuie urmărit:**

* Ar trebui să avanseze constant, fără pauze
* Avertismentele privind metadatele lipsă vor apărea în Jurnalul de depanare

### Etapa 3: Calibrare

**Ce se întâmplă:*** **Debayering**: Conversia modelului RAW Bayer în 3 canale
* **Corecția vignetării**: Eliminarea întunecării marginilor lentilelor
* **Calibrarea reflectanței**: Normalizarea cu valorile țintă
* **Calculul indicilor**: Calcularea indicilor multispectrali
* Procesarea fiecărei imagini prin întregul flux de lucru

**Durată:** Majoritatea timpului total de procesare (60-80%)**Indicator de progres:**

* Calibrare: 0% → 100%
* Imaginea curentă în curs de procesare
* Imagini finalizate / Total imagini

**Comportament de procesare:*** **Mod liber**: Procesează o imagine pe rând, secvențial
* **Mod Chloros+**: Procesează până la 16 imagini simultan
* **Accelerare GPU**: Accelerează semnificativ această etapă**Ce trebuie urmărit:**

* Progres constant prin numărul de imagini
* Verificați jurnalul de depanare pentru mesaje de finalizare per imagine
* Avertismente privind calitatea imaginii sau probleme de calibrare

### Etapa 4: Exportare

**Ce se întâmplă:**

* Scrierea imaginilor calibrate pe disc în formatul selectat
* Exportarea imaginilor index multispectrale cu culori LUT
* Crearea subfolderelor pentru modelele de camere
* Păstrarea numelor de fișiere originale cu sufixele corespunzătoare

**Durată:** 10-20% din timpul total de procesare**Indicator de progres:**

* Exportare: 0% → 100%
* Fișiere în curs de scriere
* Formatul de export și destinația

**Ce trebuie urmărit:**

* Avertismente privind spațiul pe disc
* Erori de scriere a fișierelor
* Finalizarea tuturor ieșirilor configurate

***

## Fila Jurnal de depanare

Jurnalul de depanare oferă informații detaliate despre progresul procesării și orice probleme întâmpinate.

### Accesarea jurnalului de depanare

1. Faceți clic pe pictograma **Jurnal de depanare** <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> din bara laterală din stânga
2. Se deschide panoul de jurnal, afișând mesaje de procesare în timp real
3. Se derulează automat pentru a afișa cele mai recente mesaje

### Înțelegerea mesajelor din jurnal

#### Mesaje informative (alb/gri)

Actualizări normale de procesare:

```
[INFO] Processing started
[INFO] Target detected in IMG_0015.RAW - 4 panels found
[INFO] Calibrating IMG_0234.RAW
[INFO] Exported NDVI image: IMG_0234_NDVI.tif
[INFO] Processing complete
```

#### Mesaje de avertizare (galben)

Probleme necritice care nu opresc procesarea:

```
[WARN] No GPS data found in IMG_0145.RAW
[WARN] Target image timestamp gap > 30 minutes
[WARN] Low contrast in calibration panel - results may vary
```

**Acțiune:** Verificați avertismentele după procesare, dar nu întrerupeți

#### Mesaje de eroare (Red)

Probleme critice care pot cauza eșecul procesării:

```
[ERROR] Cannot write file - disk full
[ERROR] Corrupted image file: IMG_0299.RAW
[ERROR] No targets detected - enable reflectance calibration or mark target images
```

**Acțiune:** Opriți procesarea, remediați eroarea, reporniți

### Mesaje comune din jurnal

| Mesaj                          | Înțeles                                | Acțiune necesară                                         |
| -------------------------------- | -------------------------------------- | ----------------------------------------------------- |
| „Țintă detectată în \[numele fișierului]” | Ținta de calibrare găsită cu succes  | Niciuna - normal                                         |
| „Se procesează imaginea X din Y”        | Actualizare progres curent                | Niciuna - normal                                         |
| „Nu s-au găsit ținte”               | Nu s-au detectat ținte de calibrare        | Marcați imaginile țintă sau dezactivați calibrarea reflectanței |
| „Spațiu insuficient pe disc”        | Spațiu de stocare insuficient pentru ieșire          | Eliberați spațiu pe disc                                    |
| „Se omite fișierul corupt”        | Fișierul imagine este deteriorat                  | Copiați din nou fișierul de pe cardul SD                             |
| „Date PPK aplicate”               | Corecții GPS din fișierul .daq aplicate | Niciuna - normal                                         |

### Copierea datelor din jurnal

Pentru a copia jurnalul în scopul depanării sau al asistenței tehnice:

1. Deschideți panoul Jurnal de depanare
2. Faceți clic pe butonul **„Copiere jurnal”** (sau faceți clic dreapta → Selectați tot)
3. Lipiți într-un fișier text sau într-un e-mail
4. Trimiteți către asistența tehnică MAPIR, dacă este necesar

***

## Monitorizarea resurselor sistemului

### Utilizarea procesorului

**Modul gratuit:**

* 1 nucleu de procesor la ~100%
* Celelalte nuclee sunt inactive sau disponibile
* Sistemul rămâne receptiv

**Chloros+ Mod paralel:**

* Mai multe nuclee la 80-100% (până la 16 nuclee)
* Utilizare generală ridicată a procesorului
* Sistemul poate părea mai puțin receptiv

**Pentru monitorizare:**

* Windows Task Manager (Ctrl+Shift+Esc)
* Fila Performanță → secțiunea CPU
* Căutați procesele „Chloros” sau „chloros-backend”

### Utilizarea memoriei (RAM)

**Utilizare tipică:**

* Proiecte mici (&lt; 100 de imagini): 2-4 GB
* Proiecte medii (100-500 de imagini): 4-8 GB
* Proiecte mari (peste 500 de imagini): 8-16 GB
* Modul paralel Chloros+ utilizează mai multă memorie RAM

**Dacă memoria este insuficientă:**

* Procesați loturi mai mici
* Închideți alte aplicații
* Extindeți memoria RAM dacă procesați regulat seturi de date mari

### Utilizarea GPU (Chloros+ cu CUDA)

Când accelerarea GPU este activată:

* GPU-ul NVIDIA prezintă o utilizare ridicată (60-90%)
* Utilizarea VRAM crește (necesită 4 GB+ VRAM)
* Etapa de calibrare este semnificativ mai rapidă

**Pentru monitorizare:**

* Pictograma NVIDIA din bara de sistem
* Task Manager → Performanță → GPU
* GPU-Z sau un instrument de monitorizare similar

### I/O disc

**La ce să vă așteptați:**

* Citire intensă a discului în timpul etapei de analiză
* Scriere intensă pe disc în timpul etapei de export
* SSD-ul este semnificativ mai rapid decât HDD-ul

**Sfat de performanță:**

* Utilizați SSD pentru folderul proiectului, dacă este posibil
* Evitați unitățile de rețea pentru seturi de date mari
* Asigurați-vă că discul nu este aproape de capacitate maximă (afectează viteza de scriere)

***

## Detectarea problemelor în timpul procesării

### Semne de avertizare

**Progresul se blochează (nicio schimbare timp de peste 5 minute):**

* Verificați jurnalul de depanare pentru erori
* Verificați spațiul disponibil pe disc
* Verificați Task Manager pentru a vă asigura că Chloros rulează

**Mesajele de eroare apar frecvent:**

* Opriți procesarea și verificați erorile
* Cauze frecvente: spațiu pe disc, fișiere corupte, probleme de memorie
* Consultați secțiunea Depanare de mai jos

**Sistemul nu mai răspunde:**

* Modul paralel Chloros+ utilizează prea multe resurse
* Luați în considerare reducerea sarcinilor simultane sau actualizarea hardware-ului
* Modul liber consumă mai puține resurse

### Când să opriți procesarea

Opriți procesarea dacă observați:

* ❌ Erori de tip „Disk full” (Disc plin) sau „Cannot write file” (Nu se poate scrie fișierul)
* ❌ Erori repetate de corupere a fișierelor imagine
* ❌ Sistem complet blocat (nu răspunde)
* ❌ V-ați dat seama că au fost configurate setări greșite
* ❌ Imagini importate greșit

**Cum să opriți:**

1. Faceți clic pe**butonul Oprire/Anulare** (înlocuiește butonul Start)
2. Procesarea se oprește, progresul se pierde
3. Remediați problemele și reporniți de la început

***

## Depanare în timpul procesării

### Procesarea este foarte lentă

**Cauze posibile:**

* Imagini țintă nemarcate (se scanează toate imaginile)
* Stocare pe HDD în loc de SSD
* Resurse de sistem insuficiente
* Multe indexuri configurate
* Acces la unitate de rețea

**Soluții:**

1. Dacă tocmai ați început și vă aflați în etapa de detectare: Anulați, marcați țintele, reporniți
2. Pentru viitor: Utilizați SSD, reduceți indexurile, actualizați hardware-ul
3. Luați în considerare CLI pentru procesarea în lot a seturilor mari de date

### Avertismente privind „Spațiul pe disc”

**Soluții:**

1. Eliberați imediat spațiu pe disc
2. Mutați proiectul pe o unitate cu mai mult spațiu
3. Reduceți numărul de indici de exportat
4. Utilizați formatul JPG în loc de TIFF (fișiere mai mici)

### Mesaje frecvente de „Fișier corupt”

**Soluții:**

1. Copiați din nou imaginile de pe cardul SD pentru a asigura integritatea
2. Testați cardul SD pentru erori
3. Eliminați fișierele corupte din proiect
4. Continuați procesarea imaginilor rămase

### Supraîncălzirea sistemului / Limitarea performanței

**Soluții:**

1. Asigurați-vă că există o ventilație adecvată
2. Curățați praful din orificiile de ventilație ale computerului
3. Reduceți sarcina de procesare (utilizați modul Free în loc de Chloros+)
4. Efectuați procesarea în perioadele mai răcoroase ale zilei

***

## Notificare privind finalizarea procesării

Când procesarea se termină:

* Bara de progres ajunge la 100%
* Mesajul **„Procesare finalizată”** apare în jurnalul de depanare
* Butonul Start devine din nou activ
* Toate fișierele de ieșire se află în subfolderul modelului de cameră

***

## Pași următori

Odată ce procesarea se finalizează:

1. **Verificați rezultatele** - Consultați [Finalizarea procesării](finishing-the-processing.md)
2. **Verificați folderul de ieșire** - Verificați dacă toate fișierele au fost exportate corect
3. **Verificați jurnalul de depanare** - Verificați dacă există avertismente sau erori
4. **Previzualizați imaginile procesate** - Utilizați vizualizatorul de imagini sau un software extern

Pentru informații despre verificarea și utilizarea rezultatelor procesate, consultați [Finalizarea procesării](finishing-the-processing.md).
