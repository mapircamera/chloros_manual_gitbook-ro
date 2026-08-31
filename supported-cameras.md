---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/supported-cameras
---

# Camere compatibile

Chloros prelucrează imagini de la două familii de camere MAPIR pe **toate platformele** (Windows, Linux amd64 și Linux arm64/Jetson):

* **Survey3** — camerele Survey3W (cu unghi larg) și Survey3N (cu unghi îngust). Intrare: `RAW+JPG`.
* **LATTICE**— module de camere multispectrale M3C și M3M. Intrare: capturi `.tif`/`.tiff`. Camerele LATTICE pot fi, de asemenea,**controlate în timp real** din Chloros — prin intermediul filei „Cameras” din interfața grafică (Windows) sau din `chloros-cli lattice` / Python SDK (Windows și Linux) — inclusiv matrice sincronizate cu mai multe camere. Consultați [ghidul LATTICE](lattice/).

Fluxul de procesare acceptă, de asemenea, fișiere de intrare de tip `.dng`.

## Survey3

<table data-header-hidden><thead><tr><th width="156">Producător</th><th width="250">Modelul camerei</th><th width="138">Model filtru</th><th width="187">Tip de imagine</th></tr></thead><tbody><tr><td><strong>Producător</strong></td><td><strong>Modelul camerei</strong></td><td><strong>Model filtru</strong></td><td><strong>Tip de imagine</strong></td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>RGB</td><td>RAW+JPG, JPG</td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>RGN</td><td>RAW+JPG, JPG</td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>OCN</td><td>RAW+JPG, JPG</td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>NGB</td><td>RAW+JPG, JPG</td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>RE</td><td>RAW+JPG, JPG</td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>NIR</td><td>RAW+JPG, JPG</td></tr></tbody></table>## LATTICE

Linia LATTICE este un sistem modular de camere multispectrale bazat pe senzorul Sony IMX265 cu obturator global (3,1 MP, pixeli de 3,45 µm). Fiecare cameră își stochează identitatea sub forma unui șir de caractere reprezentând modelul:

```
<sensor>-<lens>-F<filter>        e.g.  M3C-L41-FRGN,  M3M-L87-F550
```

Chloros o afișează cu prefixul `LATT-` (de exemplu, `LATT-M3M-L41-F550`), iar șirul de caractere al modelului controlează toate etapele ulterioare — profilul senzorului, dispunerea benzilor și calibrarea sunt stabilite automat; nu este necesară nicio configurare pentru fiecare cameră în parte. Numărul obiectivului reprezintă **câmpul vizual orizontal în grade**: `L41` = îngust 41°, `L87` = larg 87°.

Există două configurații ale senzorului:

| Configurație | Senzor      | Tipul filtrului                           | Benzi pe cameră                                                        |
| ------------- | ----------- | ------------------------------------- | ----------------------------------------------------------------------- |
| **M3C**       | Culoare Bayer | Triplă bandă de trecere                       | 3 benzi spectrale dintr-o singură expunere                                 |
| **M3M**       | Monocrom  | Filtru de interferență cu o singură bandă îngustă | 1 bandă calibrată — combinați mai multe camere M3M pentru indici de vegetație |

### Opțiuni de filtrare M3C (Bayer)

| Filtru | Benzi (denumire @ nm central / FWHM nm)       |
| ------ | ---------------------------------------- |
| `FRGB` | Blue 475/30 · Green 550/30 · Red 625/30  |
| `FRGN` | Red 660/21 · Green 550/30 · NIR 850/30   |
| `FOCN` | Orange 615/21 · Cyan 490/38 · NIR 808/14 |
| `FNGB` | Blue 475/30 · Green 550/30 · NIR 850/30  |

### Catalog de filtre M3M (mono) — 23 de coduri de produs

Numărul F reprezintă codul de referință (SKU); banda măsurată (ștampilată pe fiecare produs calibrat destinat exportului) este scanarea filtrului pentru fiecare lot:

| SKU    | Centru (nm, măsurat) | FWHM margini (nm) | Lățime (nm) |
| ------ | --------------------- | --------------- | ---------- |
| F385   | 379,4                 | 367–392         | 25         |
| F405   | 403,9                 | 390–417         | 27         |
| F450   | 443,7                 | 430–458         | 28         |
| F485   | 489,7                 | 478–502         | 24         |
| F520   | 519,9                 | 504–536         | 32         |
| F550   | 548,4                 | 531–566         | 35         |
| F590   | 589,0                 | 570–608         | 38         |
| F615   | 623,8                 | 614–634         | 20         |
| F632   | 633,4                 | 616–651         | 35         |
| F650   | 651,1                 | 636–666         | 30         |
| F685   | 686,2                 | 675–698         | 23         |
| F715   | — (nominal)           | 706–724         | 18         |
| F725   | 725.2                 | 712–738         | 26         |
| F750   | 746.0                 | 729–763         | 34         |
| F780   | 775.1                 | 754–796         | 42         |
| F808   | 810.3                 | 789–832         | 43         |
| F832   | 826,1                 | 810–843         | 33         |
| F850   | 846,5                 | 828–865         | 37         |
| F880   | — (nominal)           | 867–893         | 26         |
| F905   | — (nominal)           | 892–920         | 28         |
| F940   | 940,6                 | 923–958         | 35         |
| F950   | 945,1                 | 929–961         | 32         |
| F988 † | 985,3                 | 968–1003        | 35         |

_„Limitele benzii sunt măsurate ca valori ale lățimii totale la jumătatea maximului din scanările cu filtru pe lot ale MAPIR — aceleași valori pe care Chloros le înscrie în fiecare export calibrat.”_ „— (nominal)” = nu s-a efectuat încă nicio scanare a lotului; pentru acele SKU-uri, centrul indicat este numărul SKU, iar lățimea este valoarea furnizată de producător.

† „Reflectanța F988 este calibrată folosind un panou de reflectanță în cadru: banda se află în afara intervalului calibrat al senzorului de lumină DAQ, astfel încât Chloros aplică cea mai recentă captură a panoului și o menține între observările panoului.” Consultați [Țintele de calibrare](calibration-targets.md).

Pentru controlul în timp real al camerei, matrice, configurarea rețelei și lanțul de procesare radiometrică, consultați [ghidul LATTICE](lattice/).
