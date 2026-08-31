# Monitorizarea procesării

Odată ce procesarea a început, Chloros oferă mai multe modalități de a monitoriza progresul, de a verifica dacă există probleme și de a înțelege ce se întâmplă cu setul dvs. de date. Această pagină explică cum să urmăriți procesarea și să interpretați informațiile furnizate de Chloros.

## Prezentare generală a barei de progres

Bara de progres din antetul de sus afișează starea procesării în timp real și procentajul de finalizare. Progresul este transmis în direct din backend prin Server-Sent Events (SSE), astfel încât bara reflectă ceea ce face efectiv pipeline-ul.

### Bara de progres în modul gratuit

Pentru utilizatorii fără licență Chloros+:

**Afișare a progresului în 2 etape:**

1.**Detectarea țintelor** – Identificarea țintelor de calibrare în imagini
2. **Prelucrare** – Aplicarea corecțiilor și exportarea**Bara de progres afișează:**

* Procentajul general de finalizare (0-100%)
* Denumirea etapei curente
* Vizualizare simplă sub formă de bară orizontală

### Bara de progres Chloros+

Pentru utilizatorii cu licență Chloros+:

**Afișare a progresului în 4 etape:**

1.**Detectare** – Identificarea țintelor de calibrare
2. **Analiză** – Examinarea imaginilor și pregătirea fluxului de lucru
3. **Calibrare** – Aplicarea corecțiilor de vignetare și de reflectanță
4. **Export** – Salvarea fișierelor procesate**Funcții interactive:*** **Treceți cu mouse-ul peste** bara de progres pentru a vedea panoul extins cu cele 4 etape
* **Faceți clic** pe bara de progres pentru a fixa panoul extins
* **Faceți clic din nou** pentru a debloca panoul și a-l ascunde automat la îndepărtarea mouse-ului
* Fiecare etapă afișează progresul individual (0-100%)

{% hint style="info" %}
**Paritate CLI**: în timpul unei rulări `chloros-cli process`, aceleași patru fire de execuție raportează „Detectare”, „Analizare”, „Prelucrare” și „Export”, iar `chloros-cli export-status` afișează în timp real progresul de export al firului 4 de pe un alt terminal. Consultați [Referința CLI](../reference/cli-reference.md).
{% endhint %}

***

## Înțelegerea fiecărei etape de procesare

{% hint style="info" %}
**Arhitectura pipeline**: Aceste 4 etape ale interfeței grafice corespund [pipeline-ului de procesare cu 4 fire](../processing-architecture/processing-pipeline.md). Pe sistemele cu accelerare GPU, firul 3 (Calibrare) beneficiază de [Adaptarea dinamică a calculului](../processing-architecture/dynamic-compute-adaptation.md), care optimizează procesarea pentru hardware-ul dvs. specific.
{% endhint %}

### Etapa 1: Detectare (Detectarea țintelor)

**Ce se întâmplă:**

* Chloros scanează imaginile pe care le-ați selectat cu caseta de selectare „Țintă” (toate imaginile, numai dacă nu este bifată nicio opțiune)
* Algoritmii de viziune computerizată identifică panourile de calibrare
* Valorile de reflectanță sunt extrase din fiecare panou
* Se înregistrează marcajele temporale ale țintelor pentru programarea corectă a calibrării

**Durată:**

* Cu ținte marcate: 10–60 de secunde
* Fără ținte marcate: 5–30+ minute (scanează toate imaginile)

**Indicator de progres:**

* Detectare: 0% → 100%
* Numărul de imagini scanate (se numără doar imaginile care sunt efectiv scanate)
* Numărul țintelor găsite

**Ce trebuie urmărit:**

* Ar trebui să se finalizeze rapid dacă țintele sunt marcate corespunzător
* Dacă durează prea mult, este posibil ca țintele să nu fie marcate
* Verificați jurnalul de depanare pentru mesaje de tip „Țintă găsită”

### Etapa 2: Analiza

**Ce se întâmplă:**

* Citirea metadatelor EXIF ale imaginii (marcaje temporale, setări de expunere)
* Stabilirea strategiei de calibrare pe baza marcajelor temporale ale țintelor și a datelor DAQ disponibile privind fluxul descendent
* Organizarea cozii de procesare a imaginilor
* Pregătirea procesorilor de procesare paralelă (numai Chloros+)

**Durată:** 5–30 de secunde**Indicator de progres:**

* Analizare: 0% → 100%
* Etapă rapidă, se finalizează de obicei repede

**Ce trebuie urmărit:**

* Ar trebui să avanseze constant, fără pauze
* Avertismentele privind metadatele lipsă vor apărea în jurnalul de depanare

### Etapa 3: Calibrare

**Ce se întâmplă:*** **Debayering**: Conversia modelului RAW Bayer în 3 canale (etapă omisă pentru modulele mono LATTICE, cu o notă)
* **Corecția vignetării**: Eliminarea întunecării marginilor obiectivului
* **Calibrarea reflectanței**: Normalizare cu valorile țintă și/sau fluxul descendent DAQ
* **Calculul indicilor**: Calcularea indicilor multispectrali
* Procesarea fiecărei imagini prin întregul flux de procesare

**Durată:** Majoritatea timpului total de procesare (60-80%)**Indicator de progres:**

* Calibrare: 0% → 100%
* Imaginea curentă în curs de procesare
* Imagini finalizate / Total imagini

**Comportament de procesare:*** **Mod liber**: Procesează o singură imagine pe rând, secvențial
* **Modul Chloros+**: Rulează un grup de procesori adaptabil la hardware — 1-4 procesori simultani pe sistemele cu GPU (în funcție de VRAM), câte un procesor pe fiecare nucleu fizic (minus unul) pe sistemele numai cu CPU. Consultați [Adaptarea dinamică a calculului](../processing-architecture/dynamic-compute-adaptation.md)
* **Accelerare GPU**: Accelerează semnificativ această etapă**Ce trebuie urmărit:**

* Progres constant în funcție de numărul de imagini
* Verificați jurnalul de depanare pentru mesaje de finalizare pentru fiecare imagine
* Avertismente privind calitatea imaginii sau probleme de calibrare

### Etapa 4: Exportarea

**Ce se întâmplă:**

* Scrierea imaginilor procesate pe disc în formatul selectat, pe măsură ce sunt finalizate
* **LATTICE**: fiecare cadru se ramifică în fiecare produs activat (debayering / previzualizare / radianță / reflectanță)
* Exportarea imaginilor index multispectrale cu culori LUT
* Crearea arborelui de ieșire `<project>/<camera>/<format>/<Product>_Images/` — fișierele exportate păstrează numele fișierului sursă; folderul identifică produsul

**Durată:** 10-20% din timpul total de procesare**Indicator de progres:**

* Exportare: 0% → 100%
* Fișiere în curs de scriere
* Formatul de export și destinația

**Aspecte de urmărit:**

* Avertismente privind spațiul pe disc
* Erori la scrierea fișierelor
* Finalizarea tuturor ieșirilor configurate

***

## Fila Jurnal de depanare

Jurnalul de depanare oferă informații detaliate despre progresul procesării și despre eventualele probleme întâmpinate. Mesajele de pornire ale backend-ului sunt, de asemenea, redate în consola de jurnal, astfel încât jurnalul prezintă întreaga poveste chiar dacă îl deschideți mai târziu.

### Accesarea jurnalului de depanare

1. Faceți clic pe pictograma **Jurnal de depanare**<img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line">

din bara laterală din stânga
2. Se deschide panoul de jurnal, afișând mesaje de procesare în timp real
3. Se derulează automat pentru a afișa cele mai recente mesaje

<!-- SCREENSHOT-NEEDED: Debug Log tab open at the end of a completed run, showing real backend log lines including the [RUN-SUMMARY] lines (images / camera groups / targets / calibrated / files written) -->

### Înțelegerea mesajelor din jurnal

Liniile de jurnal Chloros sunt precedate de etichete între paranteze care denumesc subsistemul — de exemplu, `[PROCESSING]`, `[RUN-SUMMARY]`, `[LATTICE-EXPORT]`, `[EXPORT-CHECK]`, `[IMPORT-LEVEL]`. Cel mai important lucru de reținut este **rezumatul execuției**, afișat la sfârșitul fiecărei execuții (inclusiv a celor oprite):

```
[RUN-SUMMARY] 49 image(s) in 2 camera group(s); 4 target(s) detected; 45 image(s) calibrated; 180 file(s) written.
```

Urmăresc linii suplimentare cu indicații `[RUN-SUMMARY]` ori de câte ori este nevoie de o explicație — de exemplu, o execuție care nu a produs nimic sau o cameră al cărei produs solicitat a fost omis ca fiind inaplicabil. Liniile `[EXPORT-CHECK]` explică omisiunile pentru fiecare cameră în parte (de exemplu, de ce o cameră RGB nu a obținut niciun produs de radianță).

Nivelurile generale de gravitate ale mesajelor (exemplele de mai jos sunt ilustrative, nu sunt citate literal):

#### Mesaje informative (alb/gri)

Actualizări normale ale procesării: procesarea a început, ținte detectate (cu numărul de panouri), progresul calibrării pentru fiecare imagine, fișiere exportate, procesare finalizată.

#### Mesaje de avertizare (galben)

Probleme necritice care nu opresc procesarea — de exemplu, date GPS lipsă într-un cadru, un decalaj mare de timp între imaginile țintă sau contrast scăzut într-un panou de calibrare.

**Acțiune:** Verificați avertismentele după procesare, dar nu întrerupeți procesul

#### Mesaje de eroare (Red)

Probleme critice care pot duce la eșuarea procesării — de exemplu, disc plin, un fișier imagine corupt sau lipsa țintelor detectate în timp ce s-a solicitat calibrarea reflectanței.

**Acțiune:** Opriți procesarea, remediați eroarea, reporniți

### Situații frecvente din jurnal

| Situație                             | Semnificație                                       | Acțiune necesară                                         |
| ------------------------------------- | --------------------------------------------- | ----------------------------------------------------- |
| Țintă detectată în \[numele_fișierului]        | Țintă de calibrare găsită cu succes         | Niciuna — normal                                         |
| Linii de progres pentru fiecare imagine              | Actualizare progres curent                       | Niciuna — normal                                         |
| Nu s-au găsit ținte                      | Nu s-au detectat ținte de calibrare               | Marcați imaginile țintă sau dezactivați calibrarea prin reflectanță |
| Spațiu insuficient pe disc               | Spațiu de stocare insuficient pentru ieșire                 | Eliberați spațiu pe disc                                    |
| Se omite fișierul corupt               | Fișierul imagine este deteriorat                         | Copiați din nou fișierul de pe cardul SD                             |
| `[IMPORT-LEVEL] Skipping ... no raw source` | O captură fără un cadru raw nu poate fi procesată | Recapturați cu raw sau utilizați CLI `--input-level`  |
| `[RUN-SUMMARY] ... 0 file(s) written` | Rularea nu a generat produse imagine — raportată ca eșec cu indicii | Citiți liniile cu indicii; verificați ce a fost omis și de ce |

### Copierea datelor din jurnal

Pentru a copia jurnalul în scopul depanării sau al asistenței tehnice:

1. Deschideți panoul Jurnal de depanare
2. Faceți clic pe butonul **„Copiere jurnal”** (sau faceți clic dreapta → Selectați tot)
3. Lipiți conținutul într-un fișier text sau într-un e-mail
4. Trimiteți către serviciul de asistență MAPIR, dacă este necesar

***

## Monitorizarea resurselor de sistem

### Utilizarea procesorului

**Modul gratuit:**

* 1 nucleu al procesorului la ~100%
* Celelalte nuclee sunt inactive sau disponibile
* Sistemul rămâne receptiv

**Modul paralel Chloros+:**

* Mai multe nuclee cu grad ridicat de utilizare — numărul acestora depinde de strategia aleasă prin [Adaptarea dinamică a puterii de calcul](../processing-architecture/dynamic-compute-adaptation.md)
* Sistemul poate părea mai puțin receptiv

**Pentru monitorizare:**

* Windows Managerul de activități (Ctrl+Shift+Esc)
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
* Măriți memoria RAM dacă procesați regulat seturi mari de date

### Utilizarea GPU-ului (Chloros+ cu CUDA)

Când accelerarea GPU este activată:

* GPU-ul NVIDIA prezintă o utilizare ridicată (60–90%)
* Utilizarea VRAM crește (necesită 4 GB+ VRAM; 7 GB+ pentru debayering simultan cu Texture Aware)
* Etapa de calibrare este semnificativ mai rapidă

**Pentru monitorizare:**

* Pictograma NVIDIA din bara de sistem
* Task Manager → Performanță → GPU
* GPU-Z sau un instrument de monitorizare similar

### I/O pe disc

**La ce să vă așteptați:**

* Citire intensă de pe disc în etapa de analiză
* Scriere intensă pe disc în etapa de export
* SSD-ul este semnificativ mai rapid decât HDD-ul

**Sfat de performanță:**

* Utilizați un SSD pentru folderul proiectului, atunci când este posibil
* Evitați unitățile de rețea pentru seturi de date mari
* Asigurați-vă că discul nu este aproape de capacitatea maximă (afectează viteza de scriere)

***

## Detectarea problemelor în timpul procesării

### Semne de avertizare

**Progresul se blochează (nicio schimbare timp de peste 5 minute):**

* Verificați jurnalul de depanare pentru erori
* Verificați spațiul disponibil pe disc
* Verificați Task Manager pentru a vă asigura că Chloros rulează

**Mesajele de eroare apar frecvent:**

* Opriți procesarea și analizați erorile
* Cauze frecvente: spațiu pe disc, fișiere corupte, probleme de memorie
* Consultați secțiunea Depanare de mai jos

**Sistemul nu mai răspunde:**

* Modul paralel Chloros+ consumă prea multe resurse
* Luați în considerare reducerea numărului de sarcini simultane sau modernizarea hardware-ului
* Modul liber consumă mai puține resurse

### Când să opriți procesarea

Opriți procesarea dacă observați:

* ❌ Erori de tip „Disc plin” sau „Nu se poate scrie fișierul”
* ❌ Erori repetate de corupție a fișierelor imagine
* ❌ Sistemul s-a blocat complet (nu mai răspunde)
* ❌ Ați constatat că au fost configurate setări greșite
* ❌ Au fost importate imagini greșite

**Cum se oprește:**

1. Faceți clic pe**butonul Stop** (care înlocuiește butonul Start) — o singură dată este suficient
2. Bara afișează „Se oprește...” în timp ce imaginea în curs de procesare se finalizează, apoi procesul se încheie în stare oprită
3. Produsele deja exportate rămân pe disc; jurnalul afișează un raport detaliat `[RUN-SUMMARY]` cu privire la ceea ce s-a finalizat
4. Remediați problemele și reporniți — procesul începe de la început

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

1. Dacă procesul abia a început și se află în etapa de detectare: opriți-l, marcați țintele, reporniți
2. Pentru viitor: utilizați un SSD, reduceți numărul de indici, modernizați hardware-ul
3. Luați în considerare CLI pentru procesarea în lot a seturilor mari de date

### Avertismente privind „spațiul pe disc”

**Soluții:**

1. Eliberați imediat spațiu pe disc
2. Mutați proiectul pe o unitate cu mai mult spațiu
3. Reduceți numărul de indici de exportat
4. Dezactivați produsele de export LATTICE de care nu aveți nevoie (Setări proiect → Prelucrare)
5. Utilizați formatul JPG în loc de TIFF (fișiere mai mici)

### Mesaje frecvente privind „fișiere corupte”

**Soluții:**

1. Copiați din nou imaginile de pe cardul SD pentru a asigura integritatea acestora
2. Verificați cardul SD pentru a depista eventualele erori
3. Eliminați fișierele corupte din proiect
4. Continuați procesarea imaginilor rămase

### Supraîncălzirea sistemului / Limitarea performanței

**Soluții:**

1. Asigurați-vă că există o ventilație adecvată
2. Curățați praful din orificiile de ventilație ale computerului
3. Reduceți încărcarea de procesare (utilizați modul Free în loc de Chloros+)
4. Efectuați procesarea în momentele mai răcoroase ale zilei

***

## Notificare privind finalizarea procesării

Când procesarea se încheie:

* Bara de progres ajunge la 100%
* Liniile `[RUN-SUMMARY]` apar în jurnalul de depanare (Debug Log) împreună cu numărul final de fișiere
* Butonul „Start” devine din nou activ
* Toate fișierele de ieșire se află în arborele de ieșire al proiectului, organizat pe camere: `<project>/<camera>/<format>/<Product>_Images/`

***

## Pași următori

Odată ce procesarea se finalizează:

1. **Verificați rezultatele** – Consultați [Finalizarea procesării](finishing-the-processing.md)
2. **Verificați folderul de ieșire** – Asigurați-vă că toate fișierele au fost exportate corect
3. **Verificați jurnalul de depanare** – Verificați dacă există avertismente sau erori
4. **Previzualizați imaginile procesate** – Utilizați Image Viewer sau un software extern

Pentru informații despre verificarea și utilizarea rezultatelor procesate, consultați [Finalizarea procesării](finishing-the-processing.md).
