# Camere monocrome și indici de vegetație

## O cameră = o bandă

O cameră **M3M**este varianta monocromă a modelului Bayer**M3C**: un senzor monocrom IMX265 situat în spatele unui singur filtru de interferență de bandă îngustă. Șirul modelului denumește banda — `M3M-<lens>-F<wavelength>`, de exemplu `M3M-L87-F685` (afișat în Chloros ca `LATT-M3M-L87-F685`). Senzorul oferă o**singură bandă în tonuri de gri**, fără mozaic Bayer: nu există nimic de demosaic, nici o interferență între canale de separat și nici un echilibru de alb de setat.

Consecințe care merită cunoscute înainte de a planifica un sistem monocrom:

* **Radianța și reflectanța sunt definite complet pentru fiecare bandă.**Acestea sunt hărți radiometrice pe bandă, astfel încât o cameră M3M produce radianță calibrată float32 (W/m²/sr/nm) și reflectanță uint16 (`32768` = ρ 1,0) exact la fel ca o bandă M3C. Cadrele mono conțin o matrice**de identitate** a răspunsului senzorului — nu este necesară și nu se aplică nicio decompozitie 3×3.
* **O singură cameră mono nu poate produce un indice de vegetație.** NDVI, NDRE și alte indici similari necesită cel puțin două benzi. Pentru a calcula indici folosind hardware mono, se combină mai multe camere M3M — a se vedea mai jos.
* Camerele M3M transmit **Mono12** (12 biți, 2 octeți/pixel pe cablu), ceea ce este important pentru [planificarea lățimii de bandă a matricei](arrays.md#bandwidth-the-rules-of-thumb).

## Ce omite Chloros pentru modul mono — și cum vă informează

Etapele din fluxul de procesare a culorilor pur și simplu nu se aplică unui senzor cu o singură bandă. Chloros **le omite cu un mesaj de o singură linie** în loc să genereze o eroare și le execută în continuare în mod normal pentru orice cameră M3C (Bayer) din aceeași sesiune:

| Etapă | Comportament Mono (M3M) | Comportament M3C |
| --- | --- | --- |
| Demosaic / debayer | Omit — nivelul de export `debayered` este o imagine în tonuri de gri cu un singur canal. | Demosaic pe 3 canale. |
| Balansul de alb (`lattice white-balance`) | Omitut cu un mesaj de o singură linie. | Se execută normal. |
| Profilul de culoare (`lattice color-profile`) | Omitut cu un mesaj de o singură linie. | Se execută normal. |
| Saturație/contrast (`lattice color`) | Omitut cu un mesaj de o linie. | Se execută normal. |
| Separarea interferențelor spectrale | Identitate (fără matrice 3×3). | Se aplică o matrice 3×3 pentru fiecare cameră. |
| Radianță / reflectanță | **Funcționează** — pe bandă, complet calibrat. | Funcționează pe bandă. |

Interfața grafică aplică aceeași filtrare: pentru o cameră mono, panoul de setări pentru fiecare cameră ascunde rândurile specifice doar pentru RGB (Balans de alb, Gamma, Profil de culoare, Saturație, Contrast, divizări de canale), iar histograma în timp real este blocată pe o singură trasă **MONO**. Discriminatorul pe tot parcursul stivuirii este tokenul `M3M` din șirul modelului, afișat în GUI/SDK ca `is_mono`.

## Indicii necesită ≥ 2 benzi: aliniere → stivuire → indexare

Fluxul de lucru pentru indexarea mono constă întotdeauna în aceiași trei pași:

1. **Aliniere** — orientați mai multe camere M3M la lungimi de undă diferite (de exemplu, o F650 „Red” și o F850 „NIR”), conectați-le ca o [matrice multi-cameră](arrays.md) și lăsați Chloros să calculeze deformarea de co-înregistrare între camere.
2. **Stivă** — cadrele aliniate devin o singură imagine multibandă (fiecare cameră contribuie cu o bandă denumită).
3. **Index** — evaluați o formulă de index pe benzile stivei, redând-o opțional printr-o LUT.

În interfața grafică, acest lanț complet reprezintă modul de afișare **Combined Cameras**al matricei: compoziția în timp real este deja aliniată, iar Calculatorul de index al matricei (mai jos) definește formula pe care o redă. Exporturile capturate pot fi deformate pentru a obține aceeași aliniere cu opțiunea de captură**Aligned**.

## Calculatorul de indice

Calculatorul de indice creează expresia de indice utilizată de vizualizarea în timp real și de exporturile de indice pentru fiecare cameră. Este o singură suprafață comună, accesibilă din două locuri din bara laterală a filei „Cameras”:

* **Per cameră**— Previzualizare live → rotița**Index** (numai pentru camerele Bayer RGN/OCN/NGB; o cameră mono singură nu are control asupra indexului, deoarece o singură bandă nu poate forma un index).
* **Per matrice**— setări matrice → Previzualizare live → rotița**Index**. Aceasta este calea mono: lista de benzi acoperă**toate camerele membre**, astfel încât o pereche mono contribuie aici cu cele două benzi ale sale.

<!-- SCREENSHOT-NEEDED: Index Calculator pane opened for a combined array of two mono cameras (e.g. F650 + F850): band chips row showing the two bands with wavelength labels, the operator buttons, the expression textarea containing "(NIR - Red) / (NIR + Red)", the green "Valid expression" banner, the LUT controls (Apply LUT checked, Level 7-stop, Min 0.2 / Max 1), and the live histogram with p2/p98 percentile lines. -->

Comenzile sale, de sus în jos:

* **Chipuri de bandă** („Bande — faceți clic pentru a adăuga la expresie”) — un buton pentru fiecare bandă disponibilă, etichetat cu numele culorii + lungimea de undă în nm (numele de culori duplicate sunt diferențiate, de exemplu, „Culoare 850”). Dacă faceți clic, se inserează tokenul benzii la poziția cursorului. Benzile de la camerele care nu pot produce radianță pe bandă (RGB/FRGB) sunt filtrate.
* **Butoane pentru operatori și funcții** — `+ - * / ( ) ^ ,` plus `abs() sqrt() log() log10() exp() min() max() pow()`.
* **Câmpul de text pentru expresie** — formulă introdusă liber; marcajul de plasare afișează forma clasică NDVI: `(NIR - Red) / (NIR + Red)`. O previzualizare tokenizată, numai pentru citire, situată deasupra acesteia, redă chipurile de bandă, numerele și indicatorii ca tokeni necunoscuți.
* **Banner de validitate**— gri „Gol — nu se va aplica niciun index”; verde „Expresie validă”; roșu cu eroarea specifică de analiză (bandă necunoscută, bandă ambiguă detectată de mai multe camere, paranteză lipsă, …); sau galben când expresia este validă, dar**constantă** (de ex. `X/X`, sau un numitor NDVI introdus cu `−` în loc de `+`) — o constantă mapează întregul cadru la o singură culoare.
* Apare o avertizare separată de culoare chihlimbar dacă expresia aplicată este corectă, dar **cadru live este uniform** (scenă plată sau saturată) — colapsul histogramei este detectat automat.
* **Aplicare LUT**(activată implicit; dezactivată = extindere în tonuri de gri),**Nivel**2/3/5/7 stopuri (implicit 7 stopuri) și intrările**Min / Max**care flanchează bara de gradient. Valoarea implicită pentru Min este**0,2**— aceasta mărește rampa de culori în intervalul relevant pentru vegetație, în timp ce valorile sub acest nivel sunt redate ca tonuri de gri; setați Min la −1 pentru intervalul complet al indicelui (butonul**Reset** restabilește intervalul −1…+1). Valoarea implicită pentru Max este 1.
* **Histograma în timp real** a distribuției indicelui — bare scalate cu rădăcina pătrată, linii de percentilă portocalii p2/p98, o linie mediană albă și afișaje ale valorilor din afara intervalului („◀ N% &lt; lo” / „hi &lt; N% ▶”) care devin de culoare chihlimbar deasupra valorii de 1 %, indicând necesitatea lărgirii intervalului Min/Max.
* **Aplicare**aplică expresia la fluxul în timp real; ajustările LUT se aplică în timp real fără a apăsa pe Aplicare. Expresiile sunt în mod deliberat**valabile doar pentru sesiune** — ele nu sunt păstrate între sesiuni.

<!-- SCREENSHOT-NEEDED: Combined-array live tile rendering NDVI from a mono pair through the default 7-stop LUT, with the array name pill and fps readout visible — the result of applying the expression from the previous screenshot. -->

## Calea CLI

Același lanț aliniere → stivă → index, programabil de la un capăt la altul:

```bash
chloros-cli lattice array-connect --serials SN_RED,SN_NIR
chloros-cli lattice index --live --profile align.json \
  --preset NDVI --channel red=Red_660 --channel nir=NIR_850 \
  --save-multiband -o output/
```

`--channel` mapează simbolurile unui preset la numele de benzi din stivă. Două reguli vă scutesc de o execuție eșuată:

* **Simbolurile disting între majuscule și minuscule** și trebuie să corespundă exact cu numele canalelor din preset — preseturile folosesc litere mici (cele din NDVI sunt `red`,`nir`; verificați `--list-presets`). `--channel red=Red_660` funcționează; `--channel RED=660` eșuează cu o eroare `channel_map missing entries`.
* Partea de bandă trebuie să numească o bandă din stiva aliniată (`lattice align-info --profile align.json` le enumeră). Modul offline acceptă, de asemenea, indici de bandă cu bază 0, de exemplu `--channel red=0 --channel nir=1`.

`lattice index` funcționează, de asemenea, complet offline cu un fișier TIFF multibandă aliniat și salvat:

```bash
chloros-cli lattice index --input aligned.tif --preset NDVI \
  --output ndvi.tif --colorize --gradient RdYlGn
```

### Presetări de index

`lattice index --preset` (și [Index/LUT sandbox](../image-viewer-gui/index-lut-sandbox.md) din fila Imagine, care utilizează același motor) include aceste **22 de presetări**:

`NDVI, GNDVI, BNDVI, NDRE, ENDVI, SAVI, OSAVI, MSAVI, EVI, EVI2, CVI, MSR, TDVI, LAI, GLI, NGRDI, VARI, TGI, EXG, CIRE, CIGREEN, NDWI`

Rulați `chloros-cli lattice index --list-presets` pentru formula și simbolurile canalelor fiecărei presetări, iar `--list-gradients` pentru gradientele de culoare disponibile. Formulele personalizate utilizează `--formula EXPR` cu aceeași sintaxă ca și Calculatorul de index. Rețineți că această listă de presetări este specifică motorului de indici LATTICE — meniul derulant de procesare din Setările proiectului pentru imaginile importate conține o listă diferită (consultați [Formule de indici multispectrali](../project-settings/multispectral-index-formulas.md)).

Setul complet de indicatori (`--output-format`, `--vmin/--vmax/--percentile`, `--bg-mode`, butoanele de aliniere și deformare pentru `--live`, și altele) este documentat în [Referința CLI § Indici / Matematică pentru vegetație](../reference/cli-reference.md#index--vegetation-maths); echivalentele pentru SDK se găsesc în [Referința SDK](../reference/sdk-reference.md).

## Captarea produselor indexate dintr-un array mono

Cu un array conectat și o expresie de index aplicată, `array-capture` (sau opțiunea **Capture All** din interfața grafică) salvează nivelurile de export pentru fiecare cameră *și* randarea indexată — `--index`/`--no-index` activează/dezactivează această opțiune pe CLI, iar capturarea include implicit toate nivelurile aplicabile. Contribuția unei camere mono la fiecare grup de captură este banda sa unică la nivelurile raw/debayered (scală de gri)/radianță/reflectanță, plus compoziția cu index combinat partajat atunci când matricea funcționează în modul combinat. A se vedea [Matricea multi-cameră § Captare](arrays.md#capturing-monitoring-vs-analysis).
