# Straturi de imagine

**Meniul derulant pentru straturi** din colțul din dreapta sus al vizualizatorului de imagini permite comutarea între toate versiunile imaginii pe care o vizualizați — de la captura sursă, trecând prin fiecare produs procesat, până la imaginile index calculate — fără a părăsi vizualizatorul.

## Ce sunt straturile de imagine?

Un „strat” în Chloros este un **fișier de produs**înregistrat corespunzător unei imagini sursă. Importul vă oferă fișierele sursă; procesarea adaugă un strat pentru fiecare produs generat de execuție. Fișierele exportate păstrează numele fișierului sursă —**folderul** este cel care identifică produsul, iar numele stratului este eticheta atribuită de Chloros acelui folder.

<!-- SCREENSHOT-NEEDED: Image Viewer full screen with the layer dropdown open on a processed LATTICE multispectral image, showing the full list: TIFF base, RAW (Original), RAW (Debayered), RAW (Preview), RAW (Radiance), RAW (Reflectance), and one RAW (NDVI Index) entry. -->

***

## Lista straturilor

### Întotdeauna prezente

| Strat | Ce reprezintă |
| --- | --- |
| **JPG**(sau**PNG**/**TIFF**) | Fișierul de bază care a fost inclus odată cu captură. Survey3 importă un `.JPG` lângă fiecare `.RAW`; Capturile LATTICE aduc o previzualizare de afișare PNG sau TIFF. Etichetată pentru ceea ce a fost importat efectiv |
| **RAW (Original)** | Cadrul brut sursă, debayerat pentru afișare, fără corecții aplicate. Disponibilă din momentul importului — nu necesită procesare |

O captură LATTICE al cărei fișier de bază **este** cadrul său brut nu are o intrare de bază separată: `RAW (Original)` o acoperă deja.

### Produse de procesare Survey3

| Strat | Scris în | Există când |
| --- | --- | --- |
| **RAW (Țintă)** | — | Cadrul a fost identificat ca conținând o țintă de calibrare |
| **RAW (Reflectanță)** | `Reflectance_Calibrated_Images/` | Calibrarea reflectanței s-a efectuat cu succes pe acest cadru |
| **Corectare vignetă**| `Vignette_Corrected_Images/` | Cadrul nu a putut fi calibrat în ceea ce privește reflectanța**și** *corectarea vignetei* era activată |
| **Răspunsul senzorului**| `Sensor_Response_Images/` | Cadrul nu a putut fi calibrat în ceea ce privește reflectanța**și** *corecția de vignetare* era dezactivată |
| **Echilibru de alb** | `White_Balanced_Images/` | A fost generat un produs cu echilibru de alb |

{% hint style="info" %}
**Corecția vignetării și răspunsul senzorului sunt alternative, nu pot fi activate amândouă.** Există exact un singur produs de rezervă necalibrat pe fiecare rulare, pentru fiecare model de cameră, iar comutatorul *Corecție vignetă* alege care anume. Vezi [Setări proiect](../project-settings/project-settings.md).
{% endhint %}

### Niveluri LATTICE

LATTICE capturează fan-out-ul în aceste niveluri într-o singură etapă de procesare. Care dintre ele există depinde de opțiunile de export per produs din Setările proiectului și de ceea ce se aplică camerei respective.

| Strat | Se scrie în | Se aplică la |
| --- | --- | --- |
| **RAW (Debayered)** | `Debayered_Images/` | RGB și multispectral |
| **RAW (Previzualizare)** | `Preview_Images/` | Multispectral (extindere culori false) |
| **Echilibru de alb** | `Preview_Images/` | Camerele principale RGB — previzualizarea RGB este înregistrată sub acest nume, astfel încât să se alinieze cu stratul Survey3 cu același nume |
| **RAW (radianță)** | `Radiance_Images/` | Numai multispectral |
| **RAW (Reflectanță)** | `Reflectance_Calibrated_Images/` | Numai multispectral și numai atunci când o înregistrare corespunzătoare de tip „downwelling” `.daq` sau o țintă din cadru care a trecut testul de control al calității acoperă cadrul |

Camerele principale RGB nu dispun de radiometrie pe benzi, astfel încât radianța și reflectanța sunt omise pentru acestea ca fiind **inaplicabile** — jurnalul indică acest lucru, în loc să genereze o eroare silențioasă.

### Straturi de index, LUT și sandbox

| Tip de strat | Exemplu | De unde provine |
| --- | --- | --- |
| **RAW (`<INDEX>` Index)** | `RAW (NDVI Index)` | Câte unul pentru fiecare index configurat în Setările proiectului, calculat în timpul procesării |
| **`<INDEX>` LUT** | `NDVI LUT` | Versiunea cu mapare de culori a unui index |
| **Sandbox (`<Name>` `<Index\|LUT>` `<NNN>`)** | `Sandbox (NDVI LUT 003)` | Câte unul pentru fiecare execuție de export [Index/LUT Sandbox](index-lut-sandbox.md) |

Dacă același nume de index este configurat de mai multe ori cu setări diferite, al doilea și următoarele primesc un număr în nume (`RAW (NDVI2 Index)`), astfel încât straturile să rămână ușor de distins.

***

## Utilizarea selectorului de straturi

1. Deschideți o imagine pe ecran complet făcând clic pe o miniatură din grilă
2. Faceți clic pe **meniul derulant al straturilor** din colțul din dreapta sus al vizualizatorului
3. Alegeți un strat — imaginea se actualizează imediat

Meniul derulant afișează mai întâi **JPG, RAW (Original), RAW (Target), RAW (Reflectance)**, în această ordine, iar restul sunt listate după acestea, în ordinea în care au fost înregistrate produsele.

### Preferința stratului la navigare

Apăsarea tastelor **←**/**→** trece la imaginea următoare și încearcă să vă mențină pe același strat:

1. **Potrivire exactă mai întâi** — dacă imaginea următoare are un strat cu același nume, acesta este afișat. Aceasta este opțiunea care vă menține pe stratul `RAW (NDVI Index)` pe măsură ce parcurgeți un set întreg
2. **Apoi, potrivire după tip** — un strat index caută orice strat index, un LUT orice LUT, un strat de reflectanță orice strat de reflectanță, un strat țintă orice strat țintă, un strat original orice strat original, un strat de bază orice strat de bază
3. **Apoi, numai pentru straturile de export** — numele este păstrat chiar dacă lista de straturi nu s-a sincronizat încă, deoarece fișierul există deja pe disc. Acesta este motivul pentru care puteți revizui produsele în timp ce procesul de generare încă le scrie
4. **În caz contrar** — primul strat disponibil, care este de obicei imaginea de bază

Fișierele sidecar `.daq` și `.csv` din proiect sunt omise la navigarea cu tastele săgeată, astfel încât parcurgerea imaginilor nu ajunge niciodată la o înregistrare a senzorului de lumină.

Funcțiile de zoom și panoramare se aplică și între imagini, ceea ce facilitează compararea „înainte/după” a aceleiași poziții din câmp.

***

## Înțelegerea valorilor pixelilor pe straturi

[Panoul „Valori cursor”](opening-an-image-full-screen.md#cursor-values) afișează valoarea reală pe canal sub cursor, în unitatea în care este stocat stratul respectiv. Coloanele sale se modifică în funcție de strat:

| Strat | Unitate afișată | Note |
| --- | --- | --- |
| Base (JPG / PNG / Previzualizare TIFF) | DN, 0–255 | Valori de afișare, corectate gamma pe RGB. Numai pentru inspecție vizuală |
| RAW (Original) | DN | Valori digitale brute ale senzorului. Axa histogramei indică adâncimea: 255 (8 biți), 4095 (12 biți) sau 65535 (16 biți) |
| RAW (Debayered) | DN | Liniar, fără extindere a afișajului |
| RAW (Previzualizare) / Balans de alb | DN | Produs de afișare — extins sau cu corecție gamma. Nu este destinat măsurării |
| RAW (Radianță) | **W/m²/sr/nm** | Radianță fizică Float32. Fără coloană DN |
| RAW (Reflectanță) | DN **și %** | Procentul calculat pe baza scalei proprii a fișierului — vezi mai jos |
| Exporturi index / LUT / sandbox | Valoarea indexului sau componentele RGB | Un fișier index monocanal raportează valoarea indexului; un fișier LUT cu mapare de culori raportează componentele Red/Green/Blue |

### Reflectanță: scara este specifică fiecărui fișier

{% hint style="warning" %}
**„Împărțirea la 65.535” este corectă doar pentru Survey3.** Reflectanța LATTICE este stocată la o scară diferită, iar amestecarea celor doi divizori este cea mai comună modalitate de a obține valori ale reflectanței care sunt exact jumătate din ceea ce ar trebui să fie.
{% endhint %}

| Sursă | DN care corespunde reflectanței 1,0 | Identificat prin |
| --- | --- | --- |
| **LATTICE**(M3C / M3M) |**32768** | Eticheta XMP `Chloros:PixelScale=32768` inclusă în fiecare export de reflectanță LATTICE. Marja de 2× înseamnă că ρ peste 1,0 poate fi reprezentat, în loc să fie decupat |
| **Survey3**|**65535** | Fără eticheta de scală XMP Chloros — calibrarea Survey3 scrie ρ × dtype-max și decupează la 1,0 |

Pentru GIS și scripturi: citiți `Chloros:PixelScale` din fișier și împărțiți la această valoare. Dacă eticheta lipsește, fișierul este la scara Survey3 (65535). Vizualizatorul, mediul de testare index/LUT și exportul indexului rezolvă toți scara în același mod, astfel încât numărul pe care îl vedeți la cursor este numărul folosit în calculele indexului.

Stocare specifică formatului pe lângă această scală:

* **TIFF (32 de biți, procent)** stochează DN / 65535 ca număr cu virgulă mobilă
* **PNG (8 biți)**și**JPG (8 biți)** stochează DN × 255 / 65535
* O **exportare TIFF pe 8 biți a unei capturi cu sursă pe 8 biți** este limitată la intervalul 0–255 în loc să fie redimensionată și, în mod deliberat, nu conține nicio etichetă de scală. Panoul afișează doar DN pentru aceste fișiere, fără coloana de procente

### Intervale de valori ale indicelui

| Familie de indici | Interval tipic | Citire |
| --- | --- | --- |
| Diferența normalizată (NDVI, GNDVI, NDRE, ENDVI…) | −1 până la +1 | Vegetația sănătoasă are de obicei valori între 0,4 și 0,9; solul gol este aproape de 0; apa are valori negative |
| Ajustat la sol (SAVI, OSAVI, MSAVI2…) | aproximativ de la −1 la +1,5 | Valoare similară cu NDVI, cu fondul solului suprimat |
| Raport (GRVI, GCI, MSR, CIRE…) | nelimitat în sus | Rapoartele cresc fără limită pe măsură ce banda numitorului tinde spre zero |
| EVI / LAI | de la 0 la ~1, de la 0 la ~3,5 | Norii și alți pixeli saturați îi împing pe amândoi în afara intervalului — mascați-i mai întâi |

Consultați [Formulele indicilor multispectrali](../project-settings/multispectral-index-formulas.md) pentru formula exactă din spatele fiecărei presetări.

***

## Fluxuri de lucru obișnuite

### Comparație înainte/după

1. Selectați **RAW (Original)** și observați vignetarea și valorile necalibrate
2. Treceți la **RAW (Reflectanță)**

3. Comparați — vignetarea a fost eliminată, valorile au fost calibrate. Zoomul și panoramarea rămân fixe, astfel încât priviți aceeași zonă de la sol

### Verificați un indice pe întregul set

1. Deschideți prima imagine procesată și selectați stratul de indice
2. Apăsați **→** în mod repetat — stratul de indice vă urmează de la o imagine la alta
3. Observați histograma din bara laterală pe măsură ce parcurgeți imaginile: un cadru a cărui distribuție prezintă salturi merită o analiză mai atentă

### Verificați țintele de calibrare

1. Selectați **RAW (Țintă)** pe un cadru țintă
2. Confirmați că ținta este clar vizibilă și detectată
3. Treceți la următorul cadru țintă — stratul țintă vă urmează

### Verificați acuratețea valorilor de reflectanță

1. Selectați **RAW (Reflectanță)**

2. Citiți coloana**%** din panoul „Valori cursor” — aceasta este deja scalată corect pentru fișierul respectiv
3. Verificați corectitudinea valorilor în raport cu materialele cunoscute din cadru: vegetația sănătoasă are valori ridicate în NIR și valori scăzute în roșu; o țintă de calibrare ar trebui să prezinte valori apropiate de reflectanța publicată

***

## Depanare

### Un strat pe care îl așteptam nu se află în meniul derulant

**Cauze posibile**

* Imaginea nu a fost procesată niciodată — există doar stratul de bază și `RAW (Original)`
* Opțiunea de export a produsului nu este bifată în Setările proiectului
* Produsul nu se aplică acelei camere (radianță și reflectanță pe o cameră master RGB; orice indice pe o cameră monocromă M3M cu o singură bandă)
* Calibrarea reflectanței nu a avut date pe care să se bazeze — nu există acoperire descendentă `.daq` și nici țintă în cadru care să fi trecut de controlul de calitate — astfel încât cadrul a revenit la „Vignette Corrected” (Corectare vignetă) sau „Sensor Response” (Răspunsul senzorului)

**Ce trebuie făcut**

1. Verificați jurnalul execuției: Chloros indică momentul în care un produs de export solicitat a fost imposibil de obținut și motivul
2. Verificați opțiunile de export pentru fiecare produs în [Setări proiect](../project-settings/project-settings.md)
3. Confirmați că folderul produsului există în arborele de ieșire al proiectului
4. Reprocesați cu produsul activat

### Lista straturilor pare învechită

Chloros rescanifică folderele de produse ale proiectului în timp ce o execuție este în curs și corectează înregistrările de straturi lipsă pe baza conținutului real de pe disc; astfel, un strat a cărui export s-a finalizat în mod normal apare de la sine în cadrul unui sondaj. Trecerea la o altă imagine și revenirea la aceasta forțează o nouă rezolvare.

### Valorile de reflexie par a fi jumătate din ceea ce ar trebui să fie

Este aproape sigur că împărțiți un fișier LATTICE la 65535. Utilizați `Chloros:PixelScale` (32768) sau citiți coloana **%**, care a aplicat deja această valoare.

### Stratul index există, dar imaginea este goală

Indexul necesită benzi pe care stratul dumneavoastră nu le are — de exemplu, un index care citește un al treilea canal aplicat unui fișier cu unul sau două canale. Treceți la un strat multibandă (reflectanță sau debayered) sau alegeți un index care se potrivește cu filtrul camerei.

***

## Pași următori

* [**Deschiderea unei imagini pe ecran complet**](opening-an-image-full-screen.md) — afișarea cursorului, histograma și controlul GSD
* [**Index/LUT Sandbox**](index-lut-sandbox.md) — vizualizare interactivă a indexului și export
* [**Formule pentru indici multispectrali**](../project-settings/multispectral-index-formulas.md) — referința pentru indici
* [**Finalizarea procesării**](../processing-images-gui/finishing-the-processing.md) — arborele folderelor de ieșire către care indică aceste straturi
