# Adăugarea fișierelor într-un proiect

După ce ați creat sau deschis un proiect în Chloros, următorul pas este să adăugați imaginile multispectrale pentru a începe procesarea. Fila „File Browser”<img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> vă permite să importați cu ușurință imaginile și să vă gestionați setul de date.

## Accesarea „File Browser”

1. Deschideți sau creați un proiect în Chloros
2. Faceți clic pe pictograma **File Browser** <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> din bara laterală din stânga
3. Panoul „File Browser” va afișa lista de fișiere a proiectului dvs.

{% hint style="info" %}
**Tipuri de fișiere acceptate**:

* **Survey3W / Survey3N**: perechi RAW+JPG și imagini JPG (se recomandă RAW+JPG)
* **LATTICE**: capturi `.tif` / `.tiff` — înregistrate prin controlul camerei Chloros sau printr-un hub LATTICE
* **Date de la senzori de lumină**: înregistrări `.daq` (DAQ-U/M/E) și jurnale de radiație descendentă DAQ-M `.csv` — importate împreună cu imaginile pentru a realiza calibrarea reflectanței
{% endhint %}

***

## Adăugarea de imagini în proiectul dvs.

Există două modalități principale de a adăuga imagini în proiectul dvs.:

### Metoda 1: Adăugarea fișierelor

Utilizați această opțiune pentru a importa fișiere imagine individuale sau o selecție mică de fișiere.

1. Faceți clic pe butonul **„Adăugare fișiere”** <img src="../.gitbook/assets/image (3).png" alt="" data-size="line"> din partea de sus a panoului „File Browser”
2. Navigați la folderul care conține imaginile
3. Selectați unul sau mai multe fișiere imagine (țineți apăsat **Ctrl** pentru a selecta mai multe fișiere)
4. Faceți clic pe **„Deschidere”** pentru a importa fișierele selectate

### Metoda 2: Adăugare dosar

Utilizați această opțiune pentru a importa toate imaginile dintr-un dosar simultan. Puteți selecta **mai multe dosare** într-o singură fereastră de dialog.

1. Faceți clic pe butonul **„Adăugare folder”** <img src="../.gitbook/assets/image (1) (1).png" alt="" data-size="line"> din partea de sus a panoului „Browser de fișiere”
2. Navigați la și selectați folderul (folderele) care conține(conțin) imaginile sesiunii dvs. de captură
3. Faceți clic pe **„Selectare folder”** pentru a importa toate imaginile acceptate

{% hint style="info" %}
**Fișierele care nu se încarcă sunt semnalate.** Dacă un folder conține fișiere pe care Chloros le recunoaște, dar nu le poate încărca, veți primi un avertisment — imaginile nu dispar în tăcere din grilă.
{% endhint %}

***

## Importarea folderelor de captură LATTICE

Capturile LATTICE sunt salvate cu **un subfolder pentru fiecare nivel de export** — de exemplu `raw/`, `debayered/`, `radiance/`, `reflectance/`, `preview/` — cu fișierul corespunzător de tip „downwelling” `.daq` în directorul rădăcină:

```
output/
├── raw/           capture_<timestamp>_SN<serial>_raw.tif
├── debayered/     capture_<timestamp>_SN<serial>_debayered.tif
├── preview/       capture_<timestamp>_SN<serial>_display.tif
└── *.daq          the downwelling reading matched to the capture
```

**Selectați „Adăugați folder” la directorul rădăcină al capturilor** (`output/` de mai sus). Când folderul selectat nu conține imagini, dar are subfoldere, Chloros le accesează automat — subfolderele de nivel și folderul rădăcină `.daq` sunt preluate toate dintr-o singură mișcare.**Modul de importare a capturilor:*** Fiecare captură se importă ca o **singură imagine**, grupată pe captură (nu câte o intrare per nivel). Celelalte niveluri ale aceleiași capturi apar ca moduri de vizualizare ale acelei singure imagini.
* **Prelucrarea începe întotdeauna de la cadrul brut.** Celelalte niveluri sunt vizibile, dar numai `raw` este procesat prin fluxul de lucru — reprocesarea unui produs deja prelucrat ar aplica de două ori corecțiile, așa că Chloros refuză acest lucru. Un export reimportat nu poate ocupa niciodată slotul raw al unei capturi.
* Un dosar de capturi salvat **fără** fișiere raw se importă și se afișează normal, dar procesarea îl omite și menționează acest lucru în jurnal. (Indicatorul CLI `--input-level` poate forța un punct de intrare pentru acest caz — consultați [Referința CLI](../reference/cli-reference.md#what-a-captures-folder-looks-like).)
* Începând cu versiunea 1.3.0, un dosar de capturi poate conține și un dosar `clamp_masks/` (măști pe pixel pentru nivelurile de radianță și reflectanță — consultați [Formate de imagine de ieșire](../output-image-formats.md#clamp-masks)). Măștile nu sunt imagini ale scenei, iar Chloros le ignoră la import, indiferent unde se află.

**Sesiunile LATTICE hub** se importă în același mod: selectați „Add Folder” (Adăugare folder) în folderul sesiunii copiat din hub (acesta conține `raw/` plus `previews/`), împreună cu orice jurnal de date descendente DAQ-M `.csv`. Dacă calibrarea camerei sau a DAQ-ului nu este încă stocată în memoria cache a computerului dvs., Chloros o preia automat pe baza numărului de serie la import (necesită o singură conexiune la internet).***

## Înțelegerea tabelului din browserul de fișiere

Odată ce imaginile sunt importate, acestea apar într-un tabel cu următoarele coloane:

### Numele fișierului

* Numele original al fișierului din cameră
* Păstrează convenția de denumire a camerei (de exemplu, IMG\_0001.RAW sau capture\_20260816\_101500\_SN213800234\_raw.tif)

### Marcaj temporal

* Data și ora la care a fost capturată imaginea
* Extras din metadatele EXIF ale imaginii
* Utilizat pentru potrivirea senzorilor de lumină, sincronizarea PPK și programarea țintelor de calibrare

### Modelul camerei

* Configurația camerei și a filtrului detectată automat
* Exemple Survey3: Survey3W_RGN, Survey3N_OCN, Survey3W_RGB
* Exemple LATTICE: LATT-M3M-L41-F550, LATT-M3C-L87-FRGN
* Utilizate pentru aplicarea profilurilor de procesare corespunzătoare

### Coloana Țintă (casetă de selectare)

* Bifați această casetă pentru imaginile care conțin ținte de calibrare
* Când este bifată cel puțin o imagine, **numai imaginile bifate sunt scanate** pentru ținte
* Consultați [Alegerea imaginilor țintă](choosing-target-images.md) pentru detalii

### Vizualizarea metadatelor imaginii

Dacă faceți clic pe butonul de comutare din colțul din dreapta sus, deasupra tabelului, se afișează metadatele imaginii selectate în zona grilei de imagini.

<figure><img src="../.gitbook/assets/chloros_grid_meta.gif" alt=""><figcaption></figcaption></figure>

***

## Fișierele senzorului de lumină din proiectul dvs.

* Fișierele `.daq` și `.csv` apar în lista File Browser, dar nu sunt imagini pe care se poate face clic — acestea furnizează iradianța descendentă pentru calibrarea reflectanței.
* Fiecare fișier `.daq`/`.csv` importat este listat în **Setări proiect → Senzor de lumină DAQ**, unde puteți verifica corecția capacului difuzorului aplicată pentru fiecare fișier. Consultați [Reglarea setărilor proiectului](adjusting-project-settings.md).
* Înregistrările efectuate în fila **Senzori de lumină** sunt adăugate automat la proiectul deschis — nu este necesară importarea manuală.***

## Gestionarea fișierelor din proiectul dvs.

### Eliminarea fișierelor

Pentru a elimina imaginile nedorite din proiect:

1. Selectați una sau mai multe imagini din tabelul File Browser
2. Faceți clic pe butonul **„Remove Selected”** <img src="../.gitbook/assets/image (2) (1).png" alt="" data-size="line">
3. Confirmați eliminarea (fișierele nu sunt șterse de pe disc, ci doar eliminate din proiect)

### Sortarea și filtrarea

* **Sortare după coloană**: Faceți clic pe orice antet de coloană pentru a sorta imaginile
* **Sortare după data și ora**: Utilă pentru organizarea secvențelor de captură în ordine cronologică
* **Filtru după modelul camerei**: Grupați imaginile după tipul camerei dacă utilizați mai multe camere***

## Previzualizarea imaginii

### Vizualizarea imaginii complete

Faceți clic pe orice miniatură a imaginii din Browserul de fișiere pentru a o afișa în zona principală de previzualizare:

1. Imaginea apare în panoul central de previzualizare
2. Utilizați comenzile de zoom pentru a examina detaliile imaginii
3. Navigați între imagini folosind tastele săgeată

### Navigare rapidă

* **Imaginea anterioară**: Faceți clic pe săgeata stânga sau apăsați tasta ←
* **Imaginea următoare**: Faceți clic pe săgeata dreapta sau apăsați tasta →
* **Mărire/micșorare**: Utilizați rotița mouse-ului sau butoanele de zoom
* **Panoramare**: Faceți clic și trageți pe imagine când sunteți mărit***

## Gestionarea fișierelor duplicate

Chloros detectează și ignoră automat fișierele duplicate:

* Fișierele cu nume identice sunt omise
* Se previne procesarea accidentală dublă
* Se afișează un mesaj de avertizare atunci când sunt detectate duplicate

{% hint style="warning" %}
**Important**: Nu redenumiți și nu modificați fișierele imagine originale înainte de import. Chloros se bazează pe numele de fișiere originale și pe metadate pentru o procesare corectă.
{% endhint %}

***

## Seturi de date cu camere mixte

Dacă proiectul dvs. conține imagini de la mai multe camere MAPIR:

1. Chloros detectează automat fiecare model de cameră — Survey3, LATTICE sau o combinație a acestora
2. Fiecare tip de cameră este procesat cu profilul de calibrare corespunzător
3. Browserul de fișiere afișează modelul camerei în coloana „Model cameră”
4. Fiecare cameră primește propria structură de foldere de ieșire atunci când este procesată

**Exemple de scenarii**: configurație cu două camere Survey3W RGN + Survey3N OCN, sau o matrice LATTICE cu o cameră principală RGB și mai multe module de bandă îngustă***

## Cele mai bune practici

### Organizare înainte de import

* Păstrați imaginile țintă de calibrare în același folder cu imaginile de supraveghere
* Păstrați fișierele senzorului de lumină `.daq` / `.csv` ale fiecărei sesiuni de captură împreună cu imaginile respectivei sesiuni
* Păstrați structura originală a folderelor de pe cameră/cardul SD/hub
* Nu amestecați seturi de date din sesiuni diferite într-un singur proiect

### Denumirea fișierelor

* Păstrați numele originale ale fișierelor de pe cameră (IMG\_0001.RAW, capture\_..., etc.)
* Nu redenumiți fișierele înainte de import
* Numele originale conțin metadate importante

### Imagini țintă de calibrare

* Includeți întotdeauna 1-2 imagini țintă de calibrare per sesiune (Survey3; pentru LATTICE, o înregistrare DAQ poate înlocui aceste imagini — consultați [Alegerea imaginilor țintă](choosing-target-images.md))
* Capturați țintele înainte și după sesiunea de captură
* Așezați țintele în aceleași condiții de iluminare ca și zona de captură
* Marcați imaginile țintă folosind caseta de selectare „Target”

***

## Probleme frecvente și soluții

### Imaginile nu apar după import

**Cauze posibile:**

* Format de fișier neacceptat (consultați lista tipurilor acceptate din partea de sus a acestei pagini)
* Imaginile provin de la camere care nu sunt MAPIR (consultați [Camere acceptate](../supported-cameras.md))
* Fișier corupt sau transfer incomplet de pe cardul SD

**Soluție**: Verificați compatibilitatea formatului de fișier și a modelului de cameră și consultați avertismentul de încărcare a fișierelor pentru a identifica exact fișierele care au eșuat

### Modelul camerei nu a fost detectat

**Cauze posibile:**

* Metadate EXIF modificate
* Imagini editate în software extern
* Transfer incomplet al fișierelor

**Soluție**: Reimportați fișierele originale, nemodificate, de pe cameră/cardul SD

### Lipsește data și ora

**Cauze posibile:**

* Ceasul camerei nu este setat corect
* Datele EXIF au fost eliminate de un software extern

**Soluție**: Verificați dacă setările de timp ale aparatului foto erau corecte în timpul capturării

### Proiectul redeschis semnalează fișiere lipsă

Dacă fișierele sursă au fost mutate sau șterse de la ultima deschidere a proiectului, Chloros vă indică **care**fișiere lipsesc, în loc să se deschidă pe o grilă goală. Dacă folderul proiectului în sine a fost mutat, Chloros redirecționează căile sursă către folderul în care se află acum `project.json` și păstrează o copie de rezervă (`project.json.before-relink-<date>-<time>`). Când acestea nu sunt găsite, fereastra**Fișiere sursă neidentificate**oferă opțiunea**Răsfoire** către noua locație, urmată de opțiunile Proiect nou sau Ștergere proiect. Din CLI, utilizați `chloros-cli project open PATH --relink FOLDER`; din SDK, `open_project(path, relink=...)`.***

## Pași următori

Odată ce fișierele au fost importate:

1. **Verificați lista de fișiere** – Asigurați-vă că toate imaginile s-au încărcat corect
2. **Verificați modelele de camere** – Verificați dacă detectarea camerelor este corectă
3. **Marcați imaginile țintă** – Consultați [Alegerea imaginilor țintă](choosing-target-images.md)
4. **Reglați setările** – Configurați opțiunile de procesare în [Setările proiectului](adjusting-project-settings.md)
5. **Porniți procesarea** – Consultați [Pornirea procesării](starting-the-processing.md)

Pentru informații detaliate despre configurarea proiectului, consultați [Reglarea setărilor proiectului](adjusting-project-settings.md).
