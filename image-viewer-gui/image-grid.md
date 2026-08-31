# Grila de imagini

După importarea imaginilor într-un proiect, le veți vedea dispuse într-o grilă în zona principală. Grila este locul în care alegeți **ce versiune a fiecărei imagini doriți să vizualizați** — butoanele de deasupra acesteia comută simultan fiecare miniatură între fișierele sursă și fiecare produs procesat.

## Dimensiunea miniaturilor

Utilizați glisorul de zoom din colțul din dreapta sus pentru a regla dimensiunea miniaturilor imaginilor. Glisorul variază de la **64 px la 1200 px**.

* **Ctrl + rotița mouse-ului** permite, de asemenea, redimensionarea miniaturilor.
* **Ctrl + `+`**/**Ctrl + `=`**și**Ctrl + `−`** modifică dimensiunea cu 4 px la fiecare apăsare. Intervalul de dimensiuni accesibil prin tastatură se oprește la 64 px la dimensiunea minimă și, la dimensiunea maximă, la orice dimensiune care încap exact două miniaturi pe rând în fereastra curentă.
* Dimensiunea aleasă este salvată odată cu proiectul (`UI → Grid thumbnail size` în `project.json`, implicit `160`), astfel încât redeschiderea proiectului o restabilește.

<figure><img src="../.gitbook/assets/chloros_grid_zoom.gif" alt=""><figcaption></figcaption></figure>*Rezoluția* miniaturii este o setare separată de *dimensiunea* miniaturii: consultați **Afișare → Rezoluția miniaturii imaginii** în [Setări proiect](../project-settings/project-settings.md) (implicit 512 px pe latura lungă). Dimensiunea se referă la cât de mare este redată caseta; rezoluția se referă la cât de multe detalii sunt preluate pentru a o umple.***

## Bara de instrumente a grilei

Rândul de butoane de deasupra grilei are până la trei grupuri, de la stânga la dreapta:

1. **Per declanșator / Per cameră** — modul de grupare. Apare numai pentru proiectele care conțin capturi LATTICE.
2. **Butoane de filtrare a camerelor**— câte unul pentru fiecare cameră LATTICE. Apare numai în modul**Per cameră**.
3. **Butoane de export/mod de vizualizare** — produsul pe care îl afișează fiecare miniatură.

Când fereastra este prea îngustă pentru a le afișa pe toate, grupurile se reduc de la dreapta la stânga sub formă de meniuri derulante la trecerea cursorului: butoanele de export/vizualizare se ascund mai întâi, apoi butoanele camerelor. Grupul redus lasă în urmă un singur buton etichetat cu opțiunea activă în acel moment, iar trecerea cursorului peste acesta derulează setul complet în jos. **Modurile „Per Trigger” / „Per Camera” nu se pliază niciodată.

<!-- SCREENSHOT-NEEDED: Image grid toolbar of a LATTICE array project at full width, showing all three button groups inline: Per Trigger / Per Camera, three camera filter buttons labelled "LATT-M3M (serial)", and the export/view buttons including TIFF, RAW (Original), RAW (Radiance), RAW (Reflectance). -->

*****

## Butoane de export și vizualizare

Aceste butoane comută miniaturile din grilă între tipurile de imagini. **Un buton apare imediat ce produsul pe care îl denumește există** — ceea ce, în cazul fișierelor sursă, înseamnă imediat la import, nu după procesare. Chloros rescanează produsele proiectului în timp ce o execuție este în curs, astfel încât butoanele apar în timpul procesării pe măsură ce fiecare produs începe să fie salvat pe disc.

### Butonul de bază

Butoanele de export din extrema stângă sunt etichetate în funcție de **ceea ce ați importat efectiv**:

| Ce ați importat | Eticheta butonului |
| --- | --- |
| Survey3 RAW+JPG | `JPG` |
| Capturi LATTICE cu o previzualizare afișată alături de cadrul RAW | `PNG` sau `TIFF`, în funcție de previzualizări |
| Capturi LATTICE în care fișierul de bază **este** cadrul RAW | *fără buton* — `RAW (Original)` afișează deja acel fișier |

Într-un proiect mixt, eticheta urmează extensia folosită de cele mai multe imagini.

### Butoane de produs

| Buton | Afișează | Când apare |
| --- | --- | --- |
| **Ținte** | Imagini cu o țintă de calibrare detectată | După o execuție care a detectat ținte |
| **Reflectanță** | Imagini de reflectanță calibrate | Numai în proiectele Survey3 — proiectele LATTICE folosesc în schimb `RAW (Reflectance)`, astfel încât grila nu afișează niciodată două butoane de reflectanță |
| **Echilibru de alb** | Produsul cu echilibru de alb (camere RGB) | După procesare |
| **Corectare vignetare** | Opțiunea de rezervă necalibrată cu corectare a vignetării | După o sesiune în care calibrarea reflectanței nu a putut fi aplicată și *Corectarea vignetării* era activată |
| **Răspunsul senzorului** | Opțiunea de rezervă necalibrată cu răspunsul senzorului | La fel, dar cu *Corecția de vignetare* dezactivată |
| **`RAW (<INDEX> Index)`** | Un buton pentru fiecare indice calculat | După o execuție cu indici configurați |
| **`<INDEX> LUT`** | Un buton pentru fiecare indice cu mapare de culori | După o rulare cu o LUT configurată |
| **`<Index> <Index\|LUT> <NNN>`** | Un buton pentru fiecare rulare de export [Index/LUT Sandbox](index-lut-sandbox.md) | În momentul în care se finalizează un export din sandbox |

### Butoane la nivel de LATTICE

Proiectele care conțin capturi LATTICE adaugă aceste butoane, etichetate cu numele nivelului în loc de numele produsului:

| Buton | Nivel |
| --- | --- |
| **RAW (Original)** | Cadrul RAW sursă, așa cum a fost importat |
| **RAW (Radianță)** | Radianță spectrală Float32, W/m²/sr/nm |
| **RAW (Reflectanță)** | Reflectanță uint16, 32768 = ρ 1,0 |

`RAW (Original)` este disponibil încă din momentul importului — nu necesită nicio prelucrare. Atunci când un import LATTICE nu are deloc un buton de bază (fișierul de bază al fiecărei capturi este cadrul său brut), grila se deplasează automat către primul buton de nivel disponibil, astfel încât evidențierea din bara de instrumente să corespundă cu ceea ce vedeți.

Exporturile Chloros cu două niveluri **nu au un buton de grilă propriu**:

* **Debayered** — vizualizarea `RAW (Original)` este deja redată fără efectul „debayering”, astfel încât un al doilea buton pe o imagine identică din punct de vedere vizual ar fi inutil. Produsul `RAW (Debayered)` este totuși scris pe disc și poate fi selectat în continuare din meniul derulant al straturilor pe ecran complet.
* **Previzualizare** — pe camerele RGB, previzualizarea este înregistrată ca stratul `White Balanced`, care are un buton. Pe camerele multispectrale, acesta este înregistrat ca `RAW (Preview)` și este accesibil din meniul derulant al straturilor din modul ecran complet.

{% hint style="info" %}
Aceste butoane de nivel se afișează doar pentru proiectele care conțin efectiv cadre LATTICE. Proiectele Survey3 înregistrează unele dintre aceleași denumiri interne de straturi, iar butoanele sunt filtrate pentru acestea, astfel încât o grilă Survey3 își păstrează setul familiar `JPG / Targets / Reflectance`.
{% endhint %}

Dacă faceți clic pe o miniatură a grilei, se deschide [Vizualizatorul de imagini](opening-an-image-full-screen.md) pe ecran complet pentru **același produs pe care îl afișează grila** — dacă grila este setată la `Targets`, miniatura deschide imaginea țintă exportată.

<figure><img src="../.gitbook/assets/chloros_grid_mode.gif" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: This GIF predates the LATTICE level buttons and the toolbar group separators. Reshoot on a LATTICE project cycling base -> RAW (Original) -> RAW (Radiance) -> RAW (Reflectance) -> an index button, so the new button set and the level names are visible. -->

***

## Gruparea unui proiect LATTICE: pe declanșator vs. pe cameră

Capturile în matrice produc mai multe imagini ale aceluiași moment din diferite module de cameră. Gruparea decide modul în care grila le stivuiește. Ambele moduri afișează bare de antet pliabile pe toată lățimea; **fiecare grup pornește desfăcut**, iar Chloros reține cele pe care le închideți. Starea de pliere este urmărită separat pentru fiecare mod, astfel încât închiderea unui grup în modul „Per cameră” nu închide nimic în modul „Per declanșator”.

### Pe cameră (implicit)

Un grup pe modul de cameră. Antetul afișează modelul camerei și numărul de serie (`LATT-M3M — <serial>`), precum și numărul de fotografii. Imaginile din interiorul unui grup sunt ordonate cronologic în funcție de momentul capturării.

În acest mod, bara de instrumente dispune, de asemenea, de un **buton de filtrare a camerelor pentru fiecare cameră**, etichetat `MODEL (SERIAL)`. Toate camerele sunt selectate inițial; făcând clic pe un buton, camera respectivă este deselectată, iar grupul acesteia este eliminat din grilă. Aceasta este modalitatea rapidă de a analiza o singură bandă pe parcursul unui zbor întreg.

### Pe declanșare

Un grup pe eveniment de captură — setul de cadre realizate de toate modulele la aceeași declanșare. Antetul afișează ora capturii, numărul de camere care au contribuit și o insignă pentru fiecare model de cameră din grup. Miniaturile din interiorul unui grup sunt ordonate după numărul de serie al camerei, astfel încât aceeași bandă se află în aceeași coloană pentru fiecare declanșare.

<!-- SCREENSHOT-NEEDED: Image grid in Per Trigger mode for a 3-camera LATTICE array, showing two consecutive trigger groups with their header bars (chevron, capture timestamp, "3 cameras", and the three model badges) and one group collapsed to show the closed state. -->
Imaginile non-LATTICE dintr-un proiect mixt nu sunt grupate — acestea sunt afișate ca miniaturi simple după grupuri.

***

## Miniaturile din grilă respectă dimensiunea blocului GSD

Dacă ați setat o dimensiune a blocului **GSD (px)** în bara laterală a filei „Imagine”, miniaturile din grilă sunt afișate la aceeași rezoluție la sol — nu doar în vizualizarea pe ecran complet. O dimensiune a blocului de 8 înseamnă că fiecare pixel afișat reprezintă media unui bloc de 8 × 8 pixeli sursă, oriunde în aplicație unde este afișată imaginea.

Deoarece o plăcuță are, de la bun început, doar câteva sute de pixeli lățime, dimensiunile grosiere ale blocurilor încetează să mai facă o diferență vizibilă în grilă cu mult înainte de a o face în vizualizarea pe ecran complet: un cadru de 4000 px desenat într-o plăcuță de 160 px are deja aproximativ 25 de pixeli sursă pe pixel afișat. Consultați [Deschiderea unei imagini pe ecran complet](opening-an-image-full-screen.md#gsd-block-size) pentru controlul propriu-zis.

***

## Pagini conexe

* [**Deschiderea unei imagini pe ecran complet**](opening-an-image-full-screen.md) — vizualizatorul pe ecran complet, valorile cursorului și histograma
* [**Straturi de imagine**](image-layers.md) — meniul derulant al straturilor din vizualizatorul pe ecran complet
* [**Index/LUT Sandbox**](index-lut-sandbox.md) — crearea și exportarea vizualizărilor index
* [**Setări proiect**](../project-settings/project-settings.md) — opțiunile de export care determină ce produse sunt disponibile
