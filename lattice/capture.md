# Setări și moduri de captură

Captura din fila „Camere” este controlată de un buton roșu **Captură totală**și de un panou**Setări de captură** care determină ce rezultă la apăsarea acelui buton: ce camere participă, ce tipuri de export salvează fiecare cameră și dacă declanșarea se face o singură dată, continuu sau la intervale. Această pagină documentează întregul flux — configurarea, procesul de captură în sine, locația fișierelor pe disc și modul în care acestea pot fi reprocesate ulterior pentru a obține produse calibrate. Comenzile propriu-zise ale camerei și ale matricei se găsesc în [Setări cameră](camera-settings.md).

{% hint style="info" %}
**Capturile necesită un proiect deschis.** Opțiunile „Captură totală” și rotița de setări de captură sunt dezactivate până când se deschide un proiect („Creați sau deschideți un proiect pentru a salva capturile”). Fiecare captură este salvată în folderul proiectului din `captures/`.
{% endhint %}

## Panoul „Setări de captură”

Deschideți-l folosind **rotița de lângă „Captură totală”**din lista de camere din bara laterală sau cu ajutorul butonului**„Deschideți setările de captură…”** din partea de jos a oricărui panou de setări specifice unei camere. Antetul afișează „Setări de captură” și un buton ← de revenire.

<!-- SCREENSHOT-NEEDED: the full Capture Settings pane — Single/Continuous/Interval mode buttons at top, the bulk export-type toggle rows (All Raw … All Index), the orange Fastest Capture toggle, an array group card with the Aligned checkbox and Record buttons, and an expanded per-camera row showing per-type checkboxes. -->

Selecțiile dvs. de aici — camerele incluse, casetele de selectare pe tip și modul de captură — sunt salvate **pe proiect** și restabilite când îl redeschideți.

### Moduri de captură

Trei butoane de mod în partea de sus a panoului:

| Mod | Ce face | Subsetări (implicite) |
| --- | --- | --- |
| **Singur** *(implicit)* | O singură captură pentru fiecare cameră selectată. | — |
| **Continuu**| Capturi consecutive până la îndeplinirea unei condiții de oprire. | Oprire după**Număr de capturi** (implicit 1) *sau* **Durata capturii** (implicit 10 s; unități: secunde / minute / ore / zile). |
| **Interval**(timelapse) | Serii de capturi la un interval prestabilit. |**Capturi / interval**(implicit 1) ·**La fiecare**N unități (implicit 5 s) ·**Timp de** N unități (implicit 1 m). |

În modul Continuu sau Interval, butonul „Capturează tot” devine butonul **Oprire (N)** în timpul funcționării, numărând capturile pe măsură ce acestea sunt realizate.

<!-- SCREENSHOT-NEEDED: the capture-mode area of Capture Settings with Interval selected — showing the "Captures / interval", "Every N (unit)" and "For N (unit)" rows with their defaults (1, 5 s, 1 m). -->

### Alegerea camerelor și a tipurilor de export

Textul de ajutor al panoului rezumă totul: alegeți ce camere și tipuri de export generează funcția „Capture All” — totul este activat implicit, iar opțiunile sunt salvate odată cu acest proiect.

* Butoanele **Selectează tot / Deselectează tot** comută simultan casetele de selectare ale fiecărei camere.
* **Comutatoare pentru tipuri de export în bloc**(două rânduri de butoane):**Toate Raw / Toate debayered / Toate previzualizare / Toate radianță / Toate reflectanță / Toate index**. Fiecare buton are trei stări colorate: verde ✓ = activat pentru fiecare cameră care îl suportă, portocaliu – = activat pentru unele, gri = niciuna. Un comutator este dezactivat atunci când nicio cameră conectată nu suportă acel tip. Toate devin gri în timp ce opțiunea „Captură rapidă” este activată.
* **Rânduri pentru fiecare cameră**: o casetă de selectare „Include”, plus o listă extensibilă (▸/▾) a tipurilor de export aplicabile acelei camere, cu casete de selectare individuale. Rândul afișează un număr indicând câte sunt activate, de exemplu „4/6”.

### Tipuri de export și camerele care le suportă

Există șase tipuri de export: **Raw, Debayered, Radiance, Reflectance, Preview, Index**. Doar cele aplicabile apar în rândul fiecărei camere:

| Tip de export | Conținut | RGB (FRGB) | Multispectral Bayer (FRGN/FOCN/FNGB) | Mono (M3M) |
| --- | --- | --- | --- | --- |
| **Raw** | Mozaic Bayer (mono: o singură bandă) direct de la senzor | ✓ | ✓ | ✓ |
| **Debayered** | Demosaic liniar (mono: scală de gri cu 1 canal) | ✓ | ✓ | ✓ |
| **Previzualizare** | Lanț complet de afișare (balans de alb + gamma conform profilului camerei; multispectral: extindere în culori false) | ✓ | ✓ | ✓ |
| **Radianță** | float32 W/m²/sr/nm prin lanțul radiometric complet | — (nu este disponibil) | ✓ | ✓ |
| **Reflectanță** | uint16 ρ (32768 = 1,0) | — (nu este disponibil) | ✓ — afișat numai când camera are un senzor de lumină DAQ (propriu sau moștenit de la matricea sa) | la fel ca în cazul modului multispectral |
| **Indice** | Redare indice de vegetație (LUT) | — | ✓ — necesită o expresie de indice activată și ne goală pe cameră și nu este disponibilă pentru membrii matricei combinate (matricea deține un singur indice partajat) | — (un indice necesită ≥2 benzi; vezi [Camere mono și indici de vegetație](mono-indices.md)) |

Radianța și reflectanța nu sunt niciodată disponibile pentru camerele RGB — radianța pe pixel Bayer nu are sens pentru un senzor fotometric de bandă largă.

### Captură rapidă

Comutatorul **⚡ Captură rapidă — numai format raw**(portocaliu când este activat) anulează toate selecțiile de export, setându-le la**numai format raw** — plus o compoziție gratuită cu indice combinat pentru matrice — astfel încât cadrul să fie generat cât mai repede posibil: calculele privind radianța/reflectanța/afișarea sunt omise în totalitate în momentul captării.

{% hint style="info" %}
**Un `.daq` este totuși salvat.** Atunci când este alocat un senzor de lumină, funcția „Captură rapidă” înregistrează în continuare valoarea DAQ a radiației descendente lângă cadrele raw — astfel încât produsele de radianță, reflectanță și indice pot fi generate ulterior prin reprocesare (vezi [Reprocesarea capturilor](#re-processing-captures-into-calibrated-products)). De asemenea, Fastest Capture nu afectează selecțiile din casetele de selectare: dacă îl dezactivați, acestea revin la starea inițială.
{% endhint %}

### Comenzi pentru fiecare matrice

Fiecare matrice conectată are propria sa carte de grup în panou:

* **Caseta de selectare „Include”** (cu trei stări pentru membri) și numele matricei cu modul său de afișare: „(combinat | separat)”.
* Caseta de selectare **Aliniat**(implicit**activată**): aliniază exporturile elementelor la profilul de aliniere al matricei, astfel încât exporturile să fie înregistrate la nivel de pixeli între camere. Fișierele brute rămân ne-warpate, dar poartă transformarea în metadatele lor. (Profilul în sine este calculat în [panoul de setări al matricei](camera-settings.md#alignment-co-registration-combined-only).)
* Rândurile camerelor membre sunt imbricate în interiorul cardului.

Cardul matricei găzduiește, de asemenea, două dispozitive de înregistrare. Gândiți-vă la ele ca la **monitorizare vs. analiză**:

| Dispozitiv de înregistrare | Nivel | Ce înregistrează |
| --- | --- | --- |
| **● Înregistrează video index combinat / ■ Oprește înregistrarea** *(numai matrice combinate)* | **Monitorizare** | Compoziția live a indexului combinat în format video la 10 fps — 8 biți, rezoluție de previzualizare, LUT integrat. Necesită un proiect deschis și o vizualizare live în flux. Afișează cadrele și timpul scurs în timpul înregistrării. |
| **⦿ Înregistrare rafală brută / ■ Oprire serie brută** *(orice matrice)* | **Analiză**| Cadre Bayer brute la rata de captură live (fără procesare), plus un manifest per cadru și citirile `.daq`, în formatul `captures/bursts/`. După o serie de cadre, apare un buton**Creează videoclip**: acesta reprocesează seria de cadre offline într-un videoclip calibrat — indice combinat și/sau radianță / reflectanță / indice pentru fiecare cameră — plus fișiere TIFF opționale. Crearea videoclipului cu indice combinat pornește automat când opriți seria de cadre.

<!-- SCREENSHOT-NEEDED: an array group card in Capture Settings while a raw burst is recording — the ⦿/■ burst button in its recording state with frame count, and (in a second capture) the Build video button that appears after stopping. -->

|## Fluxul

<!-- SCREENSHOT-NEEDED: the sidebar during a capture — Capture All showing live "Capturing… 3/6" progress text, and (second capture) the result flash "Saved N files". -->

„Capture All” Apăsați **Capture All** în lista de camere din bara laterală:

1. Fiecare cameră inclusă, vizibilă și care nu este pusă pe pauză capturează cu tipurile de export selectate. **Grupurile de camere se declanșează ca un singur declanșator sincronizat** (un singur grup sincronizat pentru toți membrii — a se vedea [Grupuri de camere multiple](arrays.md)); camerele independente înregistrează individual.
2. Camerele ascunse (ochi) sau puse în pauză sunt omise. Un grup este blocat complet numai atunci când *toți* membrii săi sunt ascunși sau puși în pauză.
3. Ori de câte ori este alocat un senzor de lumină, valoarea corespunzătoare a citirii DAQ a radiației descendente este salvată ca fișier `.daq` alături de imagini — chiar și pentru capturi exclusiv în format brut — astfel încât produsele radiometrice să poată fi întotdeauna obținute ulterior.
4. Butonul afișează progresul în timp real — „Captură în curs… terminată/total” — iar în modul Continuu/Interval devine **Oprire (N)**. Fiecare element de captură are un timp de expirare de 300 s.
5. La finalizarea trecerii, un mesaj de rezultat afișează **„S-au salvat N fișiere”**sau**„S-au salvat N, F eșuat”**, plus „(S ascuns/în pauză/sărit)” atunci când camerele au fost omise.

## Unde sunt salvate capturile

Capturile sunt salvate în cadrul proiectului deschis la `<project>/captures/`. Fiecare tip de export este plasat în **propriul subfolder**, astfel încât o captură pe mai multe niveluri nu amestecă niciodată tipurile:

```
<project>/captures/
├── raw/           capture_<ts>_SN<serial>_raw.tif
├── debayered/     capture_<ts>_SN<serial>_debayered.tif
├── radiance/      capture_<ts>_SN<serial>_radiance.tif
├── reflectance/   capture_<ts>_SN<serial>_reflectance.tif
├── preview/       capture_<ts>_SN<serial>_display.tif
├── index/         per-camera vegetation-index (LUT) render, when Index is selected
├── composite/     array foreground/background live-view composite, when produced
├── bursts/        raw-burst recordings (frames + manifest + .daq per burst)
└── *.daq          the downwelling reading matched to the capture
```

* `<ts>` reprezintă marca temporală a capturii, iar `<serial>` reprezintă numărul de serie al camerei. Capturile independente sunt denumite `capture_<ts>_SN<serial>_<level>`; capturile matriciale de la un singur declanșator sincronizat sunt denumite `sync_<ts>_SN<serial>_<level>` și **au un singur timestamp comun pentru toate camerele din grup** (sufixul de nivel este omis atunci când o cameră salvează doar un singur nivel).
* **O asimetrie de reținut:** nivelul de afișare este stocat într-un dosar denumit `preview/`, în timp ce fișierele păstrează `_display` în denumire — dosarul și sufixul diferă doar pentru acel nivel.
* Nivelurile necunoscute sunt salvate într-un folder cu numele propriu; dacă nu se poate crea un subfolder, fișierul este scris în rădăcina capturilor, în loc să fie pierdut.
* Fișierele TIFF ale capturilor sunt comprimate fără pierderi (DEFLATE) în mod implicit și conțin toate metadatele de calibrare și procesare **în interiorul fișierului XMP** — capturile sunt autodescriptive, fără fișiere auxiliare în afară de cel cu numele `.daq`.

Aceasta este aceeași structură pe care `chloros-cli lattice capture` / `array-capture` o scriu în directorul lor `-o` — documentată în [Referința CLI § Cum arată un folder de capturi](../reference/cli-reference.md#what-a-captures-folder-looks-like).

<!-- SCREENSHOT-NEEDED: OS file explorer showing a real <project>/captures/ folder after a multi-level array capture — the raw/debayered/radiance/reflectance/preview subfolders, a .daq file at the root, and sync_<ts>_SN<serial>_<level>.tif filenames visible inside one subfolder. -->

## Reprocesarea capturilor în produse calibrate

Cadrele brute capturate, împreună cu fișierul `.daq` salvat, reprezintă tot ceea ce are nevoie fluxul de procesare — de aceea, opțiunea „Fastest Capture” este sigură pentru utilizarea în condiții reale.

* **GUI**: adăugați folderul de capturi la un proiect ([Adăugarea fișierelor la un proiect](../processing-images-gui/adding-files-to-a-project.md)) și procesați ca de obicei.
* **CLI**: indicați fișierul `process` către**rădăcina folderului de capturi**:

```bash
chloros-cli process "C:/ChlorosProjects/MyField/captures"
```

`process` importă în mod normal doar folderul pe care îl specificați, dar atunci când acel folder nu conține imagini și are subfoldere, acesta parcurge automat subfolderele — astfel încât subfolderele de nivel și fișierele din directorul rădăcină `.daq` sunt preluate dintr-o singură mișcare. Fiecare captură este importată ca o **singură imagine**, cu celelalte niveluri atașate ca moduri de vizualizare, nu ca o imagine pentru fiecare nivel.

Denumirea directă a unui subfolder de nivel (de exemplu, `…/captures/raw/`) funcționează, de asemenea, dar lasă în urmă fișierele rădăcină `.daq` — copiați-le împreună atunci când re-derivați un produs radiometric din `raw/`, altfel potrivirea marcajului temporal nu are cu ce să se raporteze.

{% hint style="warning" %}
**Prelucrarea începe întotdeauna de la `raw`.**În cadrul fiecărei capturi, cadrul brut reprezintă sursa procesului de prelucrare; `debayered`, `radiance`, `reflectance` și `preview` apar ca moduri de vizualizare, dar nu sunt niciodată reintroduse în procesul de prelucrare — reprocesarea unui produs derivat ar aplica din nou efectele de vignetare, culoare și radianță deja integrate în pixelii săi, astfel încât Chloros este respins în loc să fie procesat de două ori. Renderizările `index/` și `composite/` nu sunt procesate deloc (sunt ieșiri, nu capturi). Un folder „captures” salvat**fără** importuri raw se afișează normal, dar `process` îl omite și menționează acest lucru; `--input-level {raw,debayered,processed}` este „portul de scăpare” intenționat care forțează un punct de intrare. Consultați [Referința CLI](../reference/cli-reference.md#what-a-captures-folder-looks-like) pentru mesajele exacte de omitere.
{% endhint %}

Alte două comportamente care merită cunoscute atunci când se scrie un script de reprocesare:

* O execuție `chloros-cli process` care a solicitat produse, dar **nu a scris niciun produs de imagine**, eșuează cu un mesaj de eroare vizibil și se închide cu un cod de ieșire diferit de zero** — nu veți obține niciodată o execuție silențioasă fără rezultate. Execuțiile reușite raportează numărul de produse generate. (O execuție deliberată care generează doar metadate este totuși considerată reușită.)
* Exporturile procesate și reimportate nu ocupă niciodată slotul de date brute al unei capturi — datele brute originale rămân întotdeauna sursa procesului.

## Echivalente CLI

Tot ce se află pe această pagină poate fi controlat fără interfață grafică. Modurile de captură din GUI se corespund direct cu `chloros-cli lattice array-capture`:

| GUI | CLI |
| --- | --- |
| Singur | `chloros-cli lattice array-capture` |
| Continuu | `array-capture --continuous [--count N] [--duration S]` |
| Interval | `array-capture --interval S [--duration S]` |
| Captură maximă | `array-capture --fastest` |
| Casetă de selectare aliniată | `--aligned / --no-aligned` |
| Casete de selectare de tip export | `--processing LEVEL` sau `--levels L1,L2,…` (implicit `all`) |
| Înregistrare video index | `chloros-cli lattice array-record` |
| Înregistrare rafală neprelucrată / Creare video | `chloros-cli lattice array-burst` / `array-build-video` |

Tabelele complete de indicatori, opțiunea de captură stabilizată cu AE inteligent (`--smart`) și modelul cu rată susținută se găsesc în [CLI Referință § Moduri de captură, înregistratoare și reprocesare offline](../reference/cli-reference.md#capture-modes-recorders--offline-reprocess).
