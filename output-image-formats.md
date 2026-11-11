---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/output-image-formats
---

# Formate de imagine de ieșire

Chloros exportă produsele procesate în patru formate de fișiere. Selectați formatul în Setările proiectului (GUI), cu `--format` (CLI) sau cu `export_format` (SDK). CLI și SDK acceptă exact șirurile de caractere de mai jos.

| Șir de format | Extensie | Tip de pixel | Interval de pixeli | Note |
| --- | --- | --- | --- | --- |
| `TIFF (16-bit)` *(implicit)* | `.tif` | număr digital uint16 | 0 – 65535 | Recomandat pentru fotogrammetrie / GIS. |
| `TIFF (8-bit)` | `.tif` | număr digital uint8 | 0 – 255 | Pentru toate capturile de 8 biți, care oferă doar formatele de 8 biți: Chloros nu mărește adâncimea de biți. |
| `TIFF (32-bit, Percent)` | `.tif` | float32 | 0,0 – 1,0 | 1,0 = 100% reflectanță. Unele aplicații nu pot citi fișiere TIFF cu virgulă mobilă; fișierele sunt mai mari. |
| `PNG (8-bit)` | `.png` | număr digital uint8 | 0 – 255 | Compresie fără pierderi, potrivită pentru vizualizare pe web și afișare. |
| `JPG (8-bit)` | `.jpg` | număr digital uint8 | 0 – 255 | Compresie cu pierderi, fișiere de dimensiuni minime. |

## Unde sunt salvate fișierele de ieșire

Fișierele sunt salvate în folderul proiectului, grupate pe cameră și apoi pe format de fișier:

```
<project>/
└── LATT-M3M-L41-F550/                  # one folder per camera (model+lens+filter)
    ├── tiff16/                          # follows --format: tiff16, tiff8, png8, jpg8, or tiff32
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── NDVI_Index_Images/           # one <INDEX>_Index_Images/ folder per requested index
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

Folderul camerei este `LATT-<sensor>-<lens>-F<filter>` pentru LATTICE și `<model>_<filter>` (de exemplu, `Survey3N_RGN`) pentru Survey3. **Fiecare produs exportat păstrează numele fișierului sursă — folderul identifică produsul, nu sufixul numelui de fișier.** Consultați [Unde sunt salvate fișierele de ieșire](reference/cli-reference.md) în Referința CLI pentru regulile complete.

## Produse LATTICE (niveluri de captură și export)

Un cadru brut LATTICE se ramifică în fiecare produs solicitat într-o singură trecere. Fiecare tip de produs are propriul comutator (casete de selectare în GUI sau comenzi CLI `--debayered` / `--preview` / `--radiance` / `--reflectance`, toate activate implicit):

| Nivel | Conținut | Tip de date |
| --- | --- | --- |
| `raw` | Date Bayer direct de la senzor (camere monocrome: banda unică). Prelucrarea începe întotdeauna de la datele brute. | Așa cum au fost capturate |
| `debayered` | Demosaic liniar — 3 canale pentru M3C, 1 canal în tonuri de gri pentru M3M: offsetul de întuneric din pachetul de calibrare este eliminat, urmat de un demosaic sensibil la margini, fără corecție de liniaritate, câmp plat sau separare de culori. O exportare `process` scrie același fișier pe care îl salvează o captură la acest nivel, în orice mod de procesare. | DN liniar |
| `radiance` | Radianța spectrală absolută din lanțul radiometric complet, în **W/m²/sr/nm**. Se scrie întotdeauna ca fișier TIFF pe 32 de biți (`tiff32/Radiance_Images/`), indiferent de formatul de export selectat. | float32 |
| `reflectance` | Reflectanța ρ, unde **DN 32768 = ρ 1,0 (100%)**, cu marjă de rezervă până la ρ 2,0. Compatibil cu Pix4D. | uint16 |
| `preview` | Redare gata de afișare: RGB = balans de alb + gamma; multispectral = extindere în culori false. | afișare pe 8 biți |

### Orientare

Fișierele `raw` și `debayered` rămân întotdeauna în orientarea **nativă** a senzorului; setarea de răsucire/rotire a camerei se aplică numai celorlalte produse (previzualizare, radianță, reflectanță, indice). Capturile de matrice păstrează fișierele fiecărui element în orientarea nativă, cu excepția cazului în care se solicită o orientare specifică (`--orient` în CLI, `orient=True` în SDK). Fiecare fișier înregistrează orientarea corespunzătoare pixelilor săi în XMP — `ChlorosRotation`, `ChlorosFlipHorizontal`, `ChlorosFlipVertical` — iar `ChlorosOrientationApplied` indică dacă proprii săi pixeli conțin această informație (`false` în fișierele raw și debayered). Chloros Image Viewer modifică fișierele raw și cele debayered conform acestui înregistrare doar pentru afișare; fișierele în sine nu sunt niciodată rescrise. Un fișier fără acest înregistrare (creat înainte de versiunea 1.3.0 sau cu un alt instrument) este afișat așa cum a fost stocat.

## Măști de limitare

Produsele de radianță și reflectanță ale unei camere multispectrale M3C sunt supuse unei separări de benzi 3×3. În cazul în care separarea produce o valoare negativă într-o bandă, acea valoare este limitată la zero (vizualizările în timp real sunt limitate exact în același mod). Noutate în versiunea 1.3.0: un produs în care cel puțin un pixel a fost limitat primește o **mască de limitare** asociată, care indică exact care sunt pixelii și benzile respective:

* **Unde:**într-un folder `clamp_masks/`**lângă** folderele produselor, niciodată în interiorul unuia: `tiff16/Reflectance_Calibrated_Images/x.tif` → `tiff16/clamp_masks/Reflectance_Calibrated_Images/x_clampmask.tif` și `tiff32/Radiance_Images/x.tif` → `tiff32/clamp_masks/Radiance_Images/x_clampmask.tif`.
* **Ce:** un fișier TIFF cu o singură bandă `uint8` pe grila de pixeli proprie a produsului (inclusiv orientarea, ROI, binning și alinierea matricei). Bitul 0 (`mask & 1`) este setat acolo unde banda din canalul R al produsului a fost decupată, bitul 1 (`& 2`) pe canalul G, iar bitul 2 (`& 4`) pe canalul B. Proporția de pixeli decupați pe bandă este, de asemenea, marcată pe produs ca `Chloros:BandRUnmixClampFraction`, `Chloros:BandGUnmixClampFraction` și `Chloros:BandBUnmixClampFraction`.
* **Cum este indicat acest lucru în produs:** eticheta XMP `Chloros:UnmixClampMask` conține calea către mască relativă la folderul produsului (`../clamp_masks/<product folder>/<name>_clampmask.tif`, întotdeauna separată de `/`), `none` când nu a fost decupat niciun pixel (nu se scrie nicio mască) sau `unavailable` când nu s-a putut scrie o mască. Un produs fără etichetă nu a trecut niciodată prin procesul de separare a culorilor (camere RGB și monocrome, niveluri raw și debayered).

Deoarece niciun dosar de produs nu conține o mască, puteți transmite un dosar de produs, cum ar fi `Reflectance_Calibrated_Images/`, către Pix4D, Metashape sau alt software de fotogrammetrie așa cum este, inclusiv subdosarele — transmiteți dosarul de produs, nu dosarul de format de deasupra acestuia. Procesul de import propriu al Chloros omite măștile de clampare. CLI-ul le numără separat: `Image products written: N (plus M clamp masks)`.

## Produse index

Fiecare index este plasat într-un dosar propriu de tip `<INDEX>_Index_Images/` (de exemplu, `NDVI_Index_Images/`).

* **Formatele întregi**(`TIFF (16-bit)`, `TIFF (8-bit)`, `PNG (8-bit)`, `JPG (8-bit)`) stochează o**extindere liniară per imagine**, nu o mapare −1…1: DN 0 este cea mai mică valoare a indexului acelui cadru, iar scala completă este cea mai mare. Fișierul conține `Chloros:IndexScaling=linear_minmax`, `Chloros:IndexMin` și `Chloros:IndexMax`, deci și `value = IndexMin + DN / full_scale × (IndexMax − IndexMin)`. Deoarece fiecare cadru are propriile sale valori limită, comparați cadrele ca valori, nu ca DN.
* **`TIFF (32-bit, Percent)`** stochează valorile indexului în sine ca float32, fără rescalare — cea mai bună alegere pentru lucrări cantitative pe durata unui zbor.

Noutate în versiunea 1.3.0: un produs indexat se descrie **pe sine**, în loc să repete etichetele sursei brute. Etichetele sale GPS și ale camerei sunt copiate din produsul pe baza căruia a fost calculat (produsul de reflectanță, dacă există), iar apoi:

| Etichetă | Valoare |
| --- | --- |
| `Chloros:ProcessingLevel` | `index` (astfel încât fișierul nu este niciodată reimportat ca captură) |
| `Chloros:IndexApplied` / `Chloros:IndexExpression` | `True` / formula utilizată |
| `Chloros:IndexLevel` | `reflectance` sau `radiance` — elementul pe baza căruia a fost calculat indicele (lipsă atunci când nu este vorba de niciunul dintre ele) |
| `Chloros:IlluminationInvariant` | `True` pentru un indice calculat pe baza reflectanței, `False` pe baza radianței sau a valorilor senzorului |
| `Chloros:ChlorosPipelineVersion` | versiunea Chloros care a generat indicele |

`Chloros:PixelScale` este eliminat din produsele indexate — utilizați `IndexMin` / `IndexMax` de mai sus. Rețeta completă de citire se găsește la secțiunea „O raster cu index întreg reprezintă o extindere a contrastului pe imagine” din [Referința CLI](reference/cli-reference.md).

## Citirea valorilor pixelilor de reflectanță

Reflectanța este stocată sub forma unui număr digital întreg, iar **DN-ul care corespunde valorii ρ = 1,0 (reflectanță de 100%) depinde de camera sursă**:

| Cameră sursă | ρ = 1,0 corespunde la DN | Cum se determină |
| --- | --- | --- |
| LATTICE (M3C / M3M) | `32768` (marjă de până la ρ 2,0) | Eticheta XMP `Chloros:PixelScale=32768` este înscrisă în fișier. |
| Survey3 | `65535` (tăiat la ρ 1,0) | Nu există eticheta `Chloros:PixelScale` — această absență este semnalul. (Produsele Survey3 conțin alte etichete de procesare `Chloros:*`, cum ar fi `Chloros:CalibrationApplied`, în fiecare mod de procesare începând cu versiunea 1.3.0.) |

**Citiți eticheta XMP `Chloros:PixelScale` și împărțiți la valoarea acesteia**, în loc să presupuneți o constantă. Eticheta este definită în domeniul uint16, deci rămâne `32768` în toate formatele de ieșire care redimensionează — normalizați mai întâi tipul de date stocat înapoi la uint16 (×257 de la 8 biți, ×65535 de la float32).

{% hint style="warning" %}
**Capturi pe 8 biți.** Exporturile unei capturi pe 8 biți sunt pe 8 biți. Reflectanța acesteia este redimensionată și marcată cu `Chloros:PixelScale=32768`, astfel încât scara se menține; produsele DN pe 8 biți și fișierele de reflectanță scrise înainte de versiunea 1.3.0 sunt limitate la intervalul 0–255 și nu conțin nicio etichetă de scală. Dacă eticheta lipsește dintr-un fișier de reflectanță LATTICE, nu presupuneți existența unei scale; reexportați în schimb la 16 biți sau 32 de biți.
{% endhint %}

Pentru regulile complete (inclusiv etichetele compatibile cu MicaSense), consultați **„Citirea pixelilor de reflectanță”** în [Referința CLI](reference/cli-reference.md).
