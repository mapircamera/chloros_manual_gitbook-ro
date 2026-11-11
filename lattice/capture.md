# Setări și moduri de captură

Captura din fila „Camere” este controlată de un buton roșu **Captură totală**și de un panou**Setări de captură** care determină ce rezultă la apăsarea acelui buton: ce camere participă, ce tipuri de fișiere de export salvează fiecare cameră și dacă declanșatorul se activează o singură dată, continuu sau la un anumit interval. Această pagină documentează întregul flux — configurarea, procesul de captură în sine, locația fișierelor pe disc și modul în care acestea pot fi reprocesate ulterior pentru a obține produse calibrate. Comenzile propriu-zise ale camerei și ale matricei se găsesc în [Setări cameră](camera-settings.md).

{% hint style="info" %}
**Capturile necesită un proiect deschis.** Opțiunea „Captură totală” și rotița de setări de captură sunt dezactivate până când se deschide un proiect („Creați sau deschideți un proiect pentru a salva capturile”). Fiecare captură este salvată în folderul proiectului din `captures/`.
{% endhint %}

## Panoul „Setări de captură”

Deschideți-l folosind **rotița de lângă „Captură totală”**din lista de camere din bara laterală sau cu butonul**„Deschideți setările de captură…”** din partea de jos a oricărui panou de setări specifice camerei. Antetul afișează „Setări de captură” și un buton ← de revenire.

<!-- SCREENSHOT-NEEDED: the full Capture Settings pane — Single/Continuous/Interval mode buttons at top, the bulk export-type toggle rows (All Raw … All Index), the orange Fastest Capture toggle, an array group card with the Aligned checkbox and Record buttons, and an expanded per-camera row showing per-type checkboxes. -->



Selecțiile dvs. de aici — camerele incluse, casetele de selectare pe tip și modul de captură — sunt salvate **pe proiect** și restabilite când îl redeschideți.

### Moduri de captură

Trei butoane de mod în partea de sus a panoului:

| Mod | Ce face | Subsetări (implicite) |
| --- | --- | --- |
| **Unic** *(implicit)* | O singură captură pentru fiecare cameră selectată. | — |
| **Continuu**| Capturi consecutive până la îndeplinirea unei condiții de oprire. | Oprire după**Numărul de capturi** (implicit 1) *sau* **Durata capturii** (implicit 10 s; unități: secunde / minute / ore / zile). |
| **Interval**(timelapse) | Serii de capturi la un interval prestabilit. |**Capturi / interval**(implicit 1) ·**La fiecare**N unități (implicit 5 s) ·**Timp de** N unități (implicit 1 m). |

În modul Continuu sau Interval, butonul Capturează tot devine un buton **Oprire (N)** în timpul rulării, numărând capturile pe măsură ce acestea sunt înregistrate. O unitate plină oprește rularea (vezi [Fluxul „Capture All”](#the-capture-all-flow)).**Modul Continuu pe matrice rulează în backend (nou în 1.3.0).** Când toate camerele din listă aparțin unei matrice (camerele independente fiind ascunse sau excluse), modul „Continuous” nu mai trimite o cerere pentru fiecare captură: backend-ul Chloros

rulează singur bucla de captură, scriind un grup sincronizat în timp ce îl selectează pe următorul, iar butonul interoghează numărul acestora. Cu camere independente în listă, Chloros

capturează pas cu pas, ca și până acum.

{% hint style="warning" %}
**Captură rapidă pe Windows

: mențineți backend-ul Chloros

exclus din Microsoft Defender.** Scanarea în timp real a fiecărui fișier scris de o captură de mare viteză costă aproximativ 30 % din rata de captură în testele MAPIR

(sute de fișiere pe secundă la o zonă de interes (ROI) mică). Programul de instalare pentru Windows

oferă opțiunea **„Excludeți backend-ul Chloros

din scanarea Microsoft Defender (captură mult mai rapidă cu camera; recomandat)”**, bifată implicit, iar la dezinstalare elimină această excludere. Dacă ați debifat această opțiune, rulați din nou programul de instalare cu opțiunea bifată sau adăugați o excludere a procesului Defender pentru `chloros-backend.exe` (în folderul `backend` din folderul de instalare Chloros

). Atunci când Defender utilizează un nucleu de procesor sau mai multe în timpul capturii continue a unei matrice, backend-ul Chloros

notează acest lucru o singură dată în jurnalul său, cu aceleași două soluții.
{% endhint %}

<!-- SCREENSHOT-NEEDED: the capture-mode area of Capture Settings with Interval selected — showing the "Captures / interval", "Every N (unit)" and "For N (unit)" rows with their defaults (1, 5 s, 1 m). -->




### Alegerea camerelor și a tipurilor de export

Textul de ajutor al panoului rezumă totul: alegeți ce camere și tipuri de export generează opțiunea „Capture All” — totul este activat implicit, iar opțiunile sunt salvate odată cu acest proiect.

* Butoanele **Selectează tot / Nu selecta nimic** comută simultan casetele de selectare ale fiecărei camere.
* **Comutatoare pentru tipuri de export în bloc**(două rânduri de butoane):**Toate Raw / Toate Debayered / Toate Previzualizare / Toate Radianță / Toate Reflectanță / Toate Index**. Fiecare are trei stări de culoare: verde ✓ = activat pentru fiecare cameră care îl suportă, portocaliu – = activat pentru unele, gri = niciuna. Un buton de comutare este dezactivat atunci când nicio cameră conectată nu acceptă acel tip. Toate devin gri atunci când opțiunea „Captură rapidă” este activată.
* **Rânduri pentru fiecare cameră**: o casetă de selectare pentru includere, plus o listă extensibilă (▸/▾) cu tipurile de export aplicabile pentru camera respectivă, cu casete de selectare individuale. Rândul afișează un număr indicând câte sunt activate, de exemplu „4/6”.

### Tipuri de export și camerele care le suportă

Există șase tipuri de export: **Raw, Debayered, Radiance, Reflectance, Preview, Index**. Doar cele aplicabile apar în rândul fiecărei camere:

| Tip de export | Conținut | RGB

(FRGB) | Multispectral Bayer (FRGN/FOCN/FNGB) | Mono (M3M) |
| --- | --- | --- | --- | --- |
| **Raw** | Mozaic Bayer (mono: o singură bandă) direct de la senzor | ✓ | ✓ | ✓ |
| **Debayered** | Demosaic liniar: offsetul de întuneric (DSNU) din pachetul de calibrare este eliminat din fișierul raw, urmat de un demosaic sensibil la margini — fără corecție de liniaritate, câmp plat sau separare de culori. Salvate ca fișiere RGB

TIFF

cu 3 canale (mono: 1 canal în tonuri de gri) | ✓ | ✓ | ✓ |
| **Previzualizare** | Lanț complet de afișare (balans de alb + gamma conform profilului camerei, plus setarea „Highlights” a camerei RGB

; multispectral: extindere în culori false) | ✓ | ✓ | ✓ |
| **Radianță** | float32 W/m²/sr/nm prin lanțul radiometric complet | — (nu este disponibil) | ✓ | ✓ |
| **Reflectanță** | uint16 ρ (32768 = 1,0) | — (nu este disponibilă) | ✓ — afișată doar când camera are un senzor de lumină DAQ (propriu sau moștenit de la matricea sa) | la fel ca la multispectral |
| **Indice** | Redare indice de vegetație (LUT) | — | ✓ — necesită o expresie de indice activată și ne goală pe cameră și nu este disponibilă pentru membrii matricei combinate (matricea deține un singur indice partajat) | — (un indice necesită ≥2 benzi; vezi [Camere mono și indici de vegetație](mono-indices.md)) |

Radianța și reflectanța nu sunt niciodată disponibile pentru camerele RGB

— radianța pe pixel Bayer nu are sens pentru un senzor fotometric de bandă largă.

### Captură rapidă

Comutatorul **⚡ Captură cea mai rapidă — numai raw**(portocaliu când este activat) anulează toate selecțiile de export, setându-le la**numai raw** — plus o compoziție gratuită cu indice combinat pentru matrice — astfel încât cadrul să fie generat cât mai repede posibil: calculele de radianță/reflectanță/afișare sunt omise în totalitate în momentul capturii.

{% hint style="info" %}
**Un fișier `.daq` este totuși salvat.** Atunci când este alocat un senzor de lumină, opțiunea „Captură rapidă” înregistrează în continuare valoarea DAQ a radiației descendente (în folderul `daq/` al capturilor) — astfel încât produsele de radianță, reflectanță și indice pot fi generate ulterior prin reprocesare (vezi [Reprocesarea capturilor](#re-processing-captures-into-calibrated-products)). De asemenea, Fastest Capture nu afectează selecțiile din casetele de selectare: dacă îl dezactivați, acestea revin la starea inițială.
{% endhint %}

### Comenzi specifice fiecărei matrice

Fiecare matrice conectată are propria sa fișă de grup în panou:

* **Caseta de selectare „Include”** (cu trei stări pentru membri) și numele matricei împreună cu modul său de afișare: „(combinat | separat)”.
* Caseta de selectare **Aliniat**(implicit**activată**): exporturile elementelor de warp sunt aliniate la profilul de aliniere al matricei, astfel încât exporturile să fie înregistrate la nivel de pixeli între camere. Fișierele Raw rămân ne-warpate, dar conțin transformarea în metadatele lor. (Profilul în sine este calculat în [panoul de setări al matricei](camera-settings.md#alignment-co-registration).) Fișierele membrilor matricei sunt salvate în orientarea nativă a fiecărei camere, orice setare de oglindire/rotație fiind înregistrată în fișier, nu aplicată.
* Rândurile camerelor membre sunt imbricate în interiorul cardului.

Cardul matricei găzduiește, de asemenea, două dispozitive de înregistrare. Gândiți-vă la ele ca la **monitorizare vs. analiză**:

| Dispozitiv de înregistrare | Nivel | Ce înregistrează |
| --- | --- | --- |
| **● Înregistrează video index combinat / ■ Oprește înregistrarea** *(numai matrice combinate)* | **Monitorizare** | Compoziția live a indexului combinat în format video la 10 fps — 8 biți, rezoluție de previzualizare, LUT integrat. Necesită un proiect deschis și o vizualizare live în flux. Afișează cadrele și timpul scurs în timpul înregistrării. |
| **⦿ Înregistrare rafală brută / ■ Oprire serie brută** *(orice matrice)* | **Analiză**| Cadre Bayer brute la rata de captură live (fără procesare), plus un manifest per cadru și valorile `.daq`, în format `captures/bursts/`. După o serie de fotografii, apare butonul**Creează videoclip**: acesta reprocesează seria offline într-un videoclip calibrat — indice combinat și/sau radianță / reflectanță / indice per cameră — plus fișiere TIFF opționale. Crearea videoclipului cu indice combinat pornește automat când opriți seria de fotografii. |

<!-- SCREENSHOT-NEEDED: an array group card in Capture Settings while a raw burst is recording — the ⦿/■ burst button in its recording state with frame count, and (in a second capture) the Build video button that appears after stopping. -->

## Fluxul

<!-- SCREENSHOT-NEEDED: the sidebar during a capture — Capture All showing live "Capturing… 3/6" progress text, and (second capture) the result flash "Saved N files". -->

„Capture All”

Apăsați **Capture All** în lista de camere din bara laterală:

1. Fiecare cameră inclusă, vizibilă și care nu este pusă pe pauză capturează cu tipurile de export selectate. **Grupurile de camere se declanșează ca un singur declanșator sincronizat** (un singur grup sincronizat între toți membrii — vezi [Grupuri de camere multiple](arrays.md)); camerele independente capturează individual.
2. Camerele ascunse (cu simbolul ochiului) sau puse în pauză sunt omise. O matrice este blocată complet doar atunci când *toți* membrii săi sunt ascunși sau puși în pauză.
3. Ori de câte ori este alocat un senzor de lumină, valoarea corespunzătoare a citirii DAQ a radiației descendente este salvată ca fișier `.daq` în folderul `daq/` al capturilor — chiar și pentru capturile exclusiv în format brut — astfel încât produsele radiometrice să poată fi întotdeauna obținute ulterior.
4. Butonul afișează progresul în timp real — „Capturare… terminată/total” — iar în modul Continuu/Interval devine **Oprire (N)**. Fiecare element de captură are un timp de așteptare de 300 s.
5. Când trecerea se termină, un mesaj de rezultat afișează **„S-au salvat N fișiere”**sau**„S-au salvat N, F eșuat”**, plus „(S ascuns/în pauză/omise)” atunci când camerele au fost omise. Dacă unitatea de stocare se umple, procesul se oprește și afișează mesajul**„Oprit: unitate plină. S-au salvat N, F eșuate”** — fiecare fișier care nu a putut fi scris este considerat eșuat.

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
├── clamp_masks/   radiance/…_clampmask.tif, reflectance/…, index/… — unmix clamp masks,
│                  one subfolder per product folder, only when a band was clamped
├── bursts/        raw-burst recordings (frames + manifest + .daq per burst)
└── daq/           the downwelling reading(s) matched to each capture (.daq), each with a
                   .daq.cameras.json naming the cameras it describes
```

* `<ts>` reprezintă marca temporală a capturii, iar `<serial>` numărul de serie al camerei. Capturile independente sunt denumite `capture_<ts>_SN<serial>_<level>`; capturile matriciale provenite de la un singur declanșator sincronizat sunt denumite `sync_<ts>_SN<serial>_<level>` și **împărtășesc o singură marcă temporală pentru toate camerele din grup** (sufixul de nivel este omis atunci când o cameră salvează un singur nivel).
* **O asimetrie de reținut:** nivelul de afișare este stocat într-un folder numit `preview/`, în timp ce fișierele păstrează `_display` în nume — folderul și sufixul diferă doar pentru acel nivel.
* Nivelurile necunoscute sunt stocate într-un dosar cu numele propriu; dacă nu se poate crea un subdosar, fișierul este scris în directorul rădăcină al capturilor, în loc să se piardă.
* Fișierele TIFF ale capturilor sunt comprimate fără pierderi (DEFLATE) în mod implicit și conțin metadatele complete de calibrare și procesare **în interiorul fișierului XMP** — capturile sunt autodescriptive, fără fișiere auxiliare în afară de fișierul de citire `.daq` (și fișierul său `.daq.cameras.json`).
* **Măști de limitare (nou în versiunea 1.3.0).**În cazul în care separarea spectrală 3×3 a trebuit să limiteze o bandă la 0 pe cel puțin un pixel, produsele de radianță, reflectanță și indice primesc o mască per pixel, `clamp_masks/<product folder>/<product name>_clampmask.tif` (bitul 0 = banda din canalul R al produsului, bitul 1 = G, bitul 2 = B); eticheta XMP a produsului, `Chloros:UnmixClampMask`, indică această mască. Măștile se află**lângă** folderele produselor, niciodată în interiorul acestora, astfel încât un folder de produs precum `reflectance/` poate fi transmis software-ului de fotogrammetrie așa cum este. Importul propriu al Chloros

omite măștile.
* **Orientare (1.3.0).** Produsele „Preview”, „Radiance”, „Reflectance” și „Index” ale unei camere independente sunt rotite conform setării „Mirror/Rotation”; fișierele „Raw” și „Debayered” rămân așa cum au fost citite de senzor. Toate fișierele membre ale matricei rămân în formatul lor nativ. Fiecare fișier înregistrează setarea în fișierul XMP, astfel încât reprocesarea o aplică în același mod.
* **Cărei camere îi aparține un fișier `.daq` (1.3.0).** Un `.daq` al unei capturi este asociat, în momentul procesării, numai cu imaginile provenite de la camerele menționate de `.daq.cameras.json`-ul său. Prin urmare, o cameră autonomă fără senzor de lumină propriu nu mai preia citirea unei alte camere — asociați-i un senzor sau înregistrați unul în fila Senzori de lumină. O înregistrare a senzorului de lumină (și orice `.daq` fără acel fișier, inclusiv capturile mai vechi) servește în continuare fiecare cameră.

Aceasta este aceeași structură pe care fișierele `chloros-cli lattice capture` / `array-capture` o scriu în directorul lor `-o` — documentată în [Referința CLI

§ Cum arată un folder de capturi](../reference/cli-reference.md#what-a-captures-folder-looks-like).

<!-- SCREENSHOT-NEEDED: OS file explorer showing a real <project>/captures/ folder after a multi-level array capture — the raw/debayered/radiance/reflectance/preview subfolders, the daq/ and clamp_masks/ folders, and sync_<ts>_SN<serial>_<level>.tif filenames visible inside one subfolder. -->

## Reprocesarea capturilor în produse calibrate

Cadrele brute capturate, împreună cu fișierul `.daq` salvat, reprezintă tot ceea ce are nevoie fluxul de procesare — de aceea, „Fastest Capture” este o opțiune sigură pentru lucrări reale.

* **GUI**: adăugați folderul cu capturi la un proiect ([Adăugarea fișierelor la un proiect](../processing-images-gui/adding-files-to-a-project.md)) și procesați ca de obicei.
* **CLI**: indicați `process` către **rădăcina capturilor**:

```bash
chloros-cli process "C:/ChlorosProjects/MyField/captures"
```

`process` importă în mod normal doar folderul pe care îl specificați, dar atunci când acel folder nu conține imagini și are subfoldere, acesta parcurge automat ierarhia — astfel încât subfolderele de nivel și folderul `daq/` sunt preluate dintr-o singură mișcare. Fiecare captură se importă ca o **singură imagine**, cu celelalte niveluri atașate ca moduri de vizualizare, nu ca o imagine pentru fiecare nivel.

Denumirea directă a unui subfolder de nivel (de exemplu, `…/captures/raw/`) funcționează, de asemenea, dar lasă în urmă folderul `daq/` — copiați-l alături atunci cândderivați un produs radiometric din `raw/`; în caz contrar, potrivirea marcajului temporal nu va avea cu ce să se raporteze.

{% hint style="warning" %}
**Prelucrarea începe întotdeauna de la `raw`.** În cadrul fiecărei capturi, cadrul brut este sursa procesului de prelucrare; `debayered`, `radiance`, `reflectance`, iar `preview` apar ca moduri vizibile, dar nu sunt niciodată retrimise prin pipeline — reprocesarea unui produs derivat ar reaplica calculele de vignetare, culoare și radianță deja integrate în pixelii săi, așa că Chloros

renunță la aceasta în loc să efectueze o procesare dublă. Renderizările `index/` și `composite/` nu sunt procesate deloc (sunt ieșiri, nu capturi). Un folder „captures” salvat **fără** importuri raw se afișează normal, dar `process` îl omite și indică acest lucru; `--input-level {raw,debayered,processed}` este o cale de ieșire deliberată care forțează un punct de intrare. Consultați [Referința CLI

](../reference/cli-reference.md#what-a-captures-folder-looks-like) pentru mesajele exacte de omitere.
{% endhint %}

Încă două comportamente care merită cunoscute atunci când scrieți scripturi de reprocesare:

* O execuție `chloros-cli process` care a solicitat produse, dar **nu a scris niciun produs imagine**, eșuează cu un mesaj de eroare vizibil și se închide cu un cod de ieșire diferit de zero** — nu veți obține niciodată o execuție silențioasă fără rezultate. Execuțiile reușite raportează numărul de produse generate. (O execuție deliberată care generează doar metadate este totuși considerată reușită.)
* Exporturile procesate și reimportate nu ocupă niciodată slotul de date brute al unei capturi — datele brute originale rămân întotdeauna sursa pipeline-ului.

## Echivalente CLI



Tot ce se află pe această pagină poate fi controlat fără interfață grafică. Modurile de captură din GUI corespund direct cu `chloros-cli lattice array-capture`:

| GUI | CLI

|
| --- | --- |
| Unică | `chloros-cli lattice array-capture` |
| Continuă | `array-capture --continuous [--count N] [--duration S]` |
| Interval | `array-capture --interval S [--duration S]` |
| Captură rapidă | `array-capture --fastest` |
| Casetă de selectare „Aliniat” | `--aligned / --no-aligned` |
| Casete de selectare de tip export | `--processing LEVEL` sau `--levels L1,L2,…` (implicit `all`) |
| Înregistrare video index | `chloros-cli lattice array-record` |
| Înregistrare rafală brută / Creare video | `chloros-cli lattice array-burst` / `array-build-video` |

Tabelele complete de indicatori, opțiunea de captură stabilizată cu AE inteligent (`--smart`) și modelul cu rată susținută sunt descrise în [Referința CLI

§ Moduri de captură, înregistratoare și reprocesare offline](../reference/cli-reference.md#capture-modes-recorders--offline-reprocess).
