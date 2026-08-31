# Deschiderea unei imagini pe ecran complet

<figure><img src="../.gitbook/assets/image (34).png" alt=""><figcaption><p>O imagine deschisă pe ecran complet, cu selectorul de straturi în colțul din dreapta sus</p></figcaption></figure>

Vizualizatorul de imagini Chloros este interfața pe ecran complet pentru vizualizarea, inspectarea și măsurarea imaginilor dumneavoastră. Aici puteți citi **valorile reale ale pixelilor** — DN pe canal, procentul de reflectanță sau radianța în W/m²/sr/nm — în loc de previzualizarea întinsă afișată pe ecran.

## Accesarea vizualizatorului de imagini

### Din browserul de fișiere

1. Deschideți fila **File Browser** d<img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line">
2. Faceți clic pe orice **miniatură** din [grila de imagini](image-grid.md)
3. Imaginea se deschide pe ecran complet în fila **Vizualizator de imagini**

Imaginea se deschide pe produsul afișat în grilă. Dacă grila este setată pe `RAW (Reflectance)`, acesta este stratul pe care veți ajunge.

### Deschiderea barei laterale a Vizualizatorului de imagini

Faceți clic pe pictograma **Vizualizator de imagini** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> din bara laterală din stânga pentru a afișa panoul de analiză. Acesta conține, de sus în jos:

* numele imaginii și modelul camerei foto
* butonul **Export/Salvare imagini** (doar dacă este activ un index sau o LUT)
* casetele de selectare **Index**și**LUT** și panoul de configurare a indexului — consultați [Index/LUT Sandbox](index-lut-sandbox.md)
* panoul **Valori cursor**: citire pe canale, histograma stratului și controlul GSD***

## Navigare și mărire/micșorare

### Răsfoirea imaginilor

* **Imaginea următoare**: butonul → sau tasta**→** (săgeată dreapta)
* **Imaginea anterioară**: butonul ← sau tasta**←** (săgeată stânga)
* **Salt la o imagine specifică**: reveniți la grilă și faceți clic pe miniatura acesteia

Mărirea și panoramarea se mențin pe măsură ce vă deplasați între imagini, astfel încât puteți parcurge un set rămânând în aceeași parte a cadrului.

### Mărire

Zoomul se controlează cu **rotița mouse-ului**, în pași de 15%, ancorat pe cursor — punctul de sub indicator rămâne sub indicator. Intervalul este limitat de dimensiunea imaginii și a ferestrei: nu puteți mări imaginea dincolo de „potrivire la fereastră”, iar limita superioară este stabilită de rezoluția nativă a imaginii.

Nu există taste dedicate pentru zoom în vizualizatorul pe ecran complet. (În grilă, **Ctrl + `+` / `−`** redimensionează miniaturile — o funcție diferită.)

### Deplasare în timpul zoomului

Faceți clic și țineți apăsat butonul stâng al mouse-ului peste imagine și trageți. Deplasarea este limitată, astfel încât imaginea nu poate fi trasă în afara ecranului.

### Inspecție pixel cu pixel la zoom ridicat

Odată ce mărirea efectivă depășește **60×**, Chloros afișează un chenar de evidențiere în jurul pixelului afișat sub cursor și o valoare flotantă lângă acesta.

Mărirea „efectivă” ține cont de dimensiunea blocului GSD: cu o dimensiune a blocului de 8, evidențierea apare la o mărire de 7,5×, nu de 60×, deoarece un pixel afișat corespunde deja la 8 × 8 pixeli sursă. Dacă reduceți mărirea sub pragul respectiv, evidențierea dispare.

### Comenzi rapide de la tastatură

| Tastă                             | Unde       | Acțiune                              |
| ------------------------------- | ----------- | ----------------------------------- |
| **→**                           | Ecran complet | Imaginea următoare                          |
| **←**                           | Ecran complet | Imaginea anterioară                      |
| **Ctrl + R**                    | Ecran complet | Resetează indexul/sandbox-ul LUT         |
| **Ctrl + `+`**/**Ctrl + `=`** | Grilă        | Miniaturi mai mari (4 px la fiecare apăsare)  |
| **Ctrl + `−`**                  | Grilă        | Miniaturi mai mici (4 px la fiecare apăsare) |***

## Valori ale cursorului

Deplasați cursorul peste imagine, iar panoul **Valori ale cursorului** afișează valoarea fiecărui canal de sub acesta.

{% hint style="success" %}
**Acestea sunt valorile reale ale fișierului.** Pânza de pe ecran este o previzualizare extinsă pe 8 biți și nu le poate furniza, așa că Chloros prelevează eșantioane din fișierul produsului real pentru afișare. De aceea, un cadru brut pe 12 biți afișează valori peste 255, iar un strat de radianță de tip float32 afișează unități fizice.
{% endhint %}

### Ce înseamnă coloanele

Panoul se adaptează la stratul pe care îl vizualizați:

| Stratul pe care îl vizualizați              | Coloanele afișate    | Note                                                                                           |
| ---------------------------------- | ---------------- | ----------------------------------------------------------------------------------------------- |
| Reflectanță                        | **DN**și**%** | Procentul este calculat pe baza scalei proprii a fișierului — vezi mai jos                                      |
| Radianță                           | **W/m²/sr/nm**   | Valori fizice de tip float; nu există coloană DN, deoarece un DN nu are sens în acest context                           |
| Brut / Debayered / previzualizare / JPG    | **DN**           | Numere digitale întregi                                                                         |
| Exporturi de reflectanță procentuală pe 32 de biți | Numai **%**       | Valoarea în virgulă mobilă stocată nu este un DN, așa că rotunjirea ei la un număr întreg ar genera un rezultat fără sens, precum `0` sau `1` |

Fiecare rând este etichetat cu numele canalului pentru filtrul camerei dvs. — `Red / Green / NIR` pentru RGN, `Orange / Cyan / NIR` pentru OCN, `NIR / Green / Blue` pentru NGB, `Red / Green / Blue` pentru RGB, și denumirea benzii unice pentru camerele RE, NIR și mono M3M. Fiecare etichetă poartă un punct colorat care corespunde cercurilor de canal utilizate în editorul de formule de index.

Imaginile **de index și LUT** salvate reprezintă un caz special: acestea conțin componente ale hărții de culori în loc de benzi spectrale, astfel încât rândurile lor sunt etichetate cu `Red / Green / Blue` (sau `Index` pentru un fișier index cu un singur canal) în loc de numele filtrelor camerei.

Când un index este activ în zona de testare, apare un rând suplimentar sub canale, care afișează **valoarea indexului** la nivelul cursorului, împreună cu numele indexului și un punct alb care corespunde marcajului acestuia pe histogramă.

### Procentul de reflectanță utilizează scala specifică fiecărui fișier

{% hint style="warning" %}
**Nu presupuneți că 65535 = 100%.** Chloros stochează reflectanța la scări diferite, în funcție de camera care a generat-o, iar vizualizatorul determină scara corectă pentru fiecare fișier.
{% endhint %}

| Sursă                  | DN care corespunde reflectanței 1,0 | Cum este identificat                                                                                                                               |
| ----------------------- | ------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| **LATTICE**(M3C / M3M) |**32768**                      | Eticheta XMP `Chloros:PixelScale=32768` este inclusă în fiecare export de reflectanță LATTICE. Marja de 2× permite fișierului să conțină valori ale lui ρ mai mari de 1,0 fără a fi tăiate |
| **Survey3**|**65535**                      | Fără eticheta de scalare XMP Chloros — calibrarea Survey3 scrie ρ × dtype-max și decupează la 1,0                                                               |

Vizualizatorul, mediul de testare index/LUT și exportul indexului rezolvă toate scalarea prin aceeași implementare unică, astfel încât o valoare citită la cursor este aceeași valoare utilizată în calculele indexului.

Două consecințe demne de reținut:

* Un **procent pe 32 de biți**TIFF stochează DN/65535 ca număr cu virgulă mobilă, iar un export**pe 8 biți** PNG/JPG stochează DN × 255/65535 — vizualizatorul le convertește pe amândouă înapoi înainte de a afișa procentul.
* Un caz nu poate fi recuperat: un **export TIFF pe 8 biți al unei capturi cu sursă pe 8 biți** este decupat la intervalul 0–255 în loc să fie redimensionat și nu conține în mod deliberat nicio etichetă de scală. Pentru aceste fișiere, panoul afișează doar valoarea DN, fără coloana de procente. Acesta este răspunsul sincer, nu o eroare.***

## Histograma stratului

Sub rândurile cursorului se află o histogramă în timp real a stratului pe care îl vizualizați, în **256 de intervale**. În mod implicit, aceasta desenează o singură curbă combinată, ponderată**`(R + 2G + B) / 4`**— același spațiu de măsurare pe care îl utilizează histogramele camerei LATTICE. Activarea opțiunii**RGB**, aceasta este înlocuită cu curbe separate pentru fiecare canal, în culorile canalelor respective, amestecate aditiv astfel încât suprapunerile să rămână lizibile. Straturile monocrome afișează întotdeauna o singură curbă.

Axa orizontală este exprimată în unitatea specifică stratului:

| Strat       | Unitatea axei  | Valoarea maximă a axei                                               |
| ----------- | ---------- | ---------------------------------------------------------- |
| Reflectanță | procent    | 125% — marja produsului permite ρ peste 1,0           |
| Radianță    | W/m²/sr/nm | Valoarea maximă specifică cadrului, rotunjită la două cifre semnificative |
| Date pe 8 biți | DN         | 255                                                        |
| Date pe 12 biți | DN         | 4095                                                       |
| Date pe 16 biți | DN         | 65535                                                      |

Când axa este în DN și se oprește la una dintre aceste trei valori maxime, Chloros cunoaște, de asemenea, adâncimea de biți a ceea ce vizualizați.

Deasupra histogramei se află trei butoane:

| Buton     | Implicit | Efect                                                                                                                                                                                                                                                                                   |
| ---------- | ------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **CURSOR** | Activat      | Desenează linii de marcare pe histogramă la valorile exacte afișate în rândurile de mai sus, astfel încât să puteți vedea unde se află pixelul de sub cursor în distribuția cadrului. În modul RGB există un marcaj pe canal, în culoarea proprie; în caz contrar, apare un singur marcaj alb la valoarea combinată |
| **INDEX**| Activat      | Apare doar cât timp un indice este activ. Comută histograma de la benzile sursă la**distribuția valorilor indicelui**, cu cele două praguri de decupare trasate ca linii punctate portocalii și valoarea indicelui cursorului ca o linie albă                                                          |
| **RGB**| Dezactivat    | Comută de la curba combinată la curbele pe canale. Pe un senzor mono, acest buton afișează**MONO** și este dezactivat — există un singur canal de afișat                                                                                                                                  |

Histograma este calculată pe baza **blocurilor vizibile**, nu a pixelilor sursă din spatele acestora: modificați dimensiunea blocului GSD și distribuția se recalculează, astfel încât histograma, marcajul cursorului și imaginea afișată să corespundă întotdeauna.***

## Dimensiunea blocului GSD

În partea de jos a panoului se află controlul **GSD (px)**: o casetă numerică, un glisor de la**1 la 256**și un buton**RESET**.

Acesta reduce rezoluția imaginii _afișate_ prin calcularea mediei unui bloc N × N de pixeli sursă într-un singur pixel afișat. `1` reprezintă rezoluția nativă.

* Aceasta afectează **vizualizarea pe ecran complet, miniaturile din grilă, afișajul cursorului și ambele histograme** — tot ceea ce afișează imaginea are aceeași rezoluție la sol.
* Se aplică **doar afișării**. Procesarea și exportul nu sunt afectate. Singura excepție este una intenționată: un export [Index/LUT Sandbox](index-lut-sandbox.md) salvează ceea ce vedeți, deci păstrează dimensiunea curentă a blocului, iar panoul de export vă avertizează când dimensiunea blocului depășește 1.
* Valoarea este stocată **per proiect** ca `viewer_display.gsd_bin` în `project.json`, astfel încât se păstrează la închiderea și redeschiderea programului.
* Afișajul cursorului indică blocul, nu pixelul sursă, ori de câte ori dimensiunea blocului este mai mare de 1 — valoarea afișată este media blocului aflat sub cursor.

{% hint style="info" %}
**De ce „dimensiunea blocului” și nu centimetri pe pixel?** O valoare în cm/px necesită o înălțime față de sol. Datele EXIF ale unui singur cadru conțin altitudinea GPS față de nivelul mediu al mării, nu față de terenul către care a fost îndreptată camera, astfel încât Chloros nu va afișa o distanță față de sol pe care nu o poate calcula cu exactitate. Dimensiunea blocului în pixeli sursă este aceeași soluție de rezervă pe care o utilizează instrumentele de cloud MAPIR atunci când distanța de eșantionare la sol este necunoscută.
{% endhint %}

***

## Tipuri de imagini pe care le puteți vizualiza

Meniul derulant al straturilor din colțul din dreapta sus al vizualizatorului listează toate versiunile imaginii curente. Intrările afișate depind de cameră și de ceea ce a fost procesat — consultați [Straturi de imagine](image-layers.md) pentru lista completă și modul în care funcționează meniul derulant.

### Survey3

* **JPG** — fișierul de previzualizare propriu al camerei
* **RAW (Original)** — fișierul sursă `.RAW`, supus procesului de debayering pentru afișare, fără corecții
* **RAW (Țintă)** — un cadru identificat ca conținând o țintă de calibrare
* **RAW (Reflectanță)** — produsul de reflectanță calibrat (65535 = ρ 1,0)
* **Corectare vignetare**/**Răspuns senzor** — produsul de rezervă necalibrat
* **Echilibrat pentru alb** — produsul cu balansul de alb
* **RAW (`<INDEX>` Index)**și**`<INDEX>` LUT** — imagini index calculate

### LATTICE

Capturile LATTICE utilizează același meniu derulant, cu denumirile nivelurilor din fluxul de lucru:

| Strat                 | Ce conține                                                        |
| --------------------- | -------------------------------------------------------------------- |
| **RAW (Original)**    | Cadrul RAW sursă așa cum a fost capturat                                     |
| **RAW (Debayered)**   | Imaginea liniară debayered                                           |
| **RAW (Previzualizare)**     | Previzualizarea afișată — extindere în culori false pentru camerele multispectrale |
| **Echilibru de alb**    | Previzualizarea afișată pentru camerele principale RGB (echilibru de alb + gamma)   |
| **RAW (Radianță)**    | Radianță spectrală Float32 în W/m²/sr/nm                              |
| **RAW (Reflectanță)** | Reflectanță uint16, 32768 = ρ 1,0                                    |

Radianța și reflectanța sunt disponibile numai în modul multispectral: o cameră principală RGB nu dispune de radiometrie pe benzi, astfel încât aceste straturi nu sunt generate pentru aceasta.

***

## Aplicarea indicilor și a tabelelor de conversie (LUT)

Aplicați indicii multispectrali și tabele de consultare (LUT) pentru culori din bara laterală:

1. Deschideți **Image Viewer** d<img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line">a bara laterală
2. Bifați **Index**

3. Alegeți filtrul camerei și o formulă de indice, apoi trageți cercurile canalelor în sloturile formulei
4. Adăugați o LUT și alegeți un gradient, praguri și un mod de decupare
5. Citiți valorile la nivelul cursorului și salvați rezultatul cu **Export/Salvare imagine(i)**Consultați [Index/LUT Sandbox](index-lut-sandbox.md) pentru ghidul complet.***

## Depanare

### Imaginea nu se deschide

**Cauze posibile**: fișierul a fost mutat sau șters după import; produsul nu a fost niciodată salvat; memorie insuficientă pentru o imagine foarte mare.**Ce trebuie făcut**:

1. Verificați dacă fișierul stratului mai există în arborele de ieșire al proiectului
2. Deschideți fișierul într-un vizualizator extern pentru a confirma că este intact
3. Închideți celelalte aplicații pentru a elibera memorie

### Imaginea este neagră, albă sau are culori aberante

**Cauze posibile**: extinderea afișajului nu are cu ce să lucreze (un cadru aproape constant); un strat float32 cu valori neobișnuite; un index care nu a generat date valide.**Ce trebuie făcut**:

1. Citiți valorile cursorului — dacă fiecare canal este la zero sau aproape de zero, problema se află în date, nu în afișare
2. Verificați histograma: un singur vârf la un capăt indică faptul că cadrul este decupat sau gol
3. Verificați jurnalul de procesare pentru execuția care a generat stratul

### Valorile par incorecte

**Cauze posibile**: vă aflați pe un strat diferit de cel pe care îl credeți; comparați un procent cu un DN brut; comparați un fișier LATTICE cu un fișier Survey3 folosind același divizor.**Ce trebuie să faceți**:

1. Confirmați stratul selectat din meniul derulant — unitățile din panou se adaptează la strat
2. Pentru reflectanță, utilizați coloana **%** în loc să împărțiți manual valoarea DN; dacă trebuie să împărțiți, folosiți `Chloros:PixelScale` din acel fișier (32768 pentru LATTICE; dacă nu este specificat, înseamnă 65535 pentru Survey3)
3. Setați dimensiunea blocului GSD înapoi la 1 — peste 1 citiți o medie a blocului, nu un pixel
4. Verificați dacă calibrarea reflectanței s-a efectuat efectiv pentru acel cadru; un produs de rezervă necalibrat (Sensor Response / Vignette Corrected) nu reprezintă reflectanță

***

## Pași următori

* [**Straturi de imagine**](image-layers.md) — fiecare nume de strat, atunci când există, și semnificația valorilor sale
* [**Index/LUT Sandbox**](index-lut-sandbox.md) — creați, reglați și exportați vizualizări ale indexului
* [**Marcaje pe hartă**](map-markers.md) — același set de imagini pe o hartă
* [**Formule de indici multispectrali**](../project-settings/multispectral-index-formulas.md) — referința indicelui

Pentru fluxul de lucru de procesare, consultați [Procesarea imaginilor (GUI)](../processing-images-gui/adding-files-to-a-project.md).
