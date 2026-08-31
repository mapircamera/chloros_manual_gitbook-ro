---
description: This page lists some multispectral indices that Chloros uses
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/multispectral-index-formulas
---

# Formule pentru indici multispectrali

Formulele de mai jos pentru indici utilizează o combinație a intervalelor medii de transmisie ale filtrului Survey3:

<table><thead><tr><th align="center">Culoarea filtrului Survey3</th><th width="196.199951171875" align="center">Survey3 Denumirea filtrului</th><th width="159.800048828125" align="center">Interval de transmisie (FWHM)</th><th align="center">Transmisie medie</th></tr></thead><tbody><tr><td align="center">Blue</td><td align="center">NGB - Blue</td><td align="center">468–483 nm</td><td align="center">475 nm</td></tr><tr><td align="center">Cyan</td><td align="center">OCN - Cyan</td><td align="center">476–512 nm</td><td align="center">494 nm</td></tr><tr><td align="center">Green</td><td align="center">RGN | NGB - Green</td><td align="center">543–558 nm</td><td align="center">547 nm</td></tr><tr><td align="center">Orange</td><td align="center">OCN - Orange</td><td align="center">598–640 nm</td><td align="center">619 nm</td></tr><tr><td align="center">Red</td><td align="center">RGN - Red</td><td align="center">653–668 nm</td><td align="center">661 nm</td></tr><tr><td align="center">RedEdge</td><td align="center">Re - RedEdge</td><td align="center">712–735 nm</td><td align="center">724 nm</td></tr><tr><td align="center">NIR1</td><td align="center">OCN - NIR1</td><td align="center">798–848 nm</td><td align="center">823 nm</td></tr><tr><td align="center">NIR2</td><td align="center">RGN | NGB | NIR - NIR2</td><td align="center">835–865 nm</td><td align="center">850 nm</td></tr></tbody></table>Atunci când se utilizează aceste formule, denumirea se poate termina cu „\_1” sau „\_2”, ceea ce indică faptul că a fost utilizat filtrul NIR, fie NIR1, fie NIR2.

Pentru camerele LATTICE M3C (Bayer cu trei benzi de trecere), același motor de indexare utilizează benzile de filtrare M3C:

| Filtru M3C | Banda 1 (centru/FWHM) | Banda 2 (centru/FWHM) | Banda 3 (centru/FWHM) |
| --- | --- | --- | --- |
| FRGB | Blue 475 nm / 30 nm | Green 550 nm / 30 nm | Red 625 nm / 30 nm |
| FRGN | Red 660 nm / 21 nm | Green 550 nm / 30 nm | NIR 850 nm / 30 nm |
| FOCN | Orange 615 nm / 21 nm | Cyan 490 nm / 38 nm | NIR 808 nm / 14 nm |
| FNGB | Blue 475 nm / 30 nm | Green 550 nm / 30 nm | NIR 850 nm / 30 nm |

Camerele LATTICE M3M sunt monobandă (un filtru de bandă îngustă per cameră), astfel încât indicii multibandă nu sunt calculați pentru o singură imagine M3M. Pentru a calcula indicii cu M3M, combinați două sau mai multe camere într-o stivă multibandă aliniată și utilizați motorul de indici LATTICE (`chloros-cli lattice index` sau Calculatorul de indici în timp real din interfața grafică).

***

## Unde funcționează fiecare nume de indice

Chloros are **trei** suprafețe de indice, iar listele lor prestabilite nu sunt identice. Utilizați această secțiune pentru a verifica dacă un nume va funcționa acolo unde intenționați să îl utilizați.

| Unde vă aflați | Ce listă se aplică | Număr |
| --- | --- | --- |
| Setări proiect → Index → Adăugare index (interfață grafică) | Suprafață 1 | 27 |
| Vizualizator de imagini [Index/LUT Sandbox](../image-viewer-gui/index-lut-sandbox.md) (GUI) | Suprafața 1 | 27 |
| `chloros-cli process --indices NDVI,NDRE` | Suprafața 2 | 22 |
| SDK `process_folder(indices=[...])` | Suprafața 2 | 22 |
| `chloros-cli lattice index --preset` | Suprafața 3 | 22 (un alt 22) |
| Fila „Camere” – Calculator index live | Surface 3 | 22 (un alt 22) |

Surface 1 și 2 funcționează cu **o singură imagine la un moment dat, provenită de la o singură cameră**, folosind sloturile de simboluri `x`/`y`/`z`(/`a`) asociate canalelor de filtru ale camerei respective. Suprafața 3 funcționează pe un**stiv de imagini multibandă aliniat** — mai multe camere LATTICE co-înregistrate într-un singur cub — și se referă la canale prin nume scrise cu litere mici.

### 1. Setări proiect GUI / Meniul derulant al zonei de testare a vizualizatorului de imagini — 27 de formule

Meniul derulant le listează în această ordine (este ordinea de inserare, nu cea alfabetică):

`NDVI, GNDVI, CVI, ENDVI, EVI, MSR, OSAVI, TDVI, LAI, FCI1, FCI2, GARI, GCI, GEMI, GLI, GOSAVI, GRVI, GSAVI, LCI, MNLI, MSAVI2, NDRE, NLI, RDVI, SAVI, VARI, WDRVI`

În GUI, trageți canalele de filtru ale camerei dvs. pe sloturile de bandă ale formulei, astfel încât orice formulă să poată fi utilizată cu orice alocare de bandă pe care o suportă camera dvs. Formulele personalizate pe care le-ați salvat sunt adăugate sub această listă.

Cele **cinci formule disponibile doar în GUI** — cele pe care lista CLI/SDK `--indices` nu le acceptă — sunt implementate astfel:

| Presetare exclusiv pentru GUI | Formulă (așa cum este implementată) | Sloturi |
| --- | --- | --- |
| FCI1 | `x*y` | x, y |
| FCI2 | `x*y` | x, y |
| GARI | `(y-(x-1.7*(z-a)))/(y+(x-1.7*(z-a)))` | x, y, z, a (patru sloturi) |
| GEMI | `((2*(y*y-x*x)+1.5*y+0.5*x)/(y+x+0.5))*(1-0.25*((2*(y*y-x*x)+1.5*y+0.5*x)/(y+x+0.5)))-((x-0.125)/(1-x))` | x, y |
| LCI | `(y-x)/(y+z)` | x, y, z |

Corelarea prevăzută pentru fiecare dintre acestea este prezentată într-o secțiune separată mai jos pe această pagină (de exemplu, GARI se așteaptă la x=Green, y=NIR, z=Blue, a=Red). GARI este singura formulă din Chloros care utilizează un al patrulea slot.

### 2. Extinderea numelui CLI / SDK `--indices` — 22 de presetări

Opțiunea `chloros-cli process --indices` (și parametrul SDK `indices`) acceptă următoarele nume de presetări:

`NDVI, GNDVI, NDRE, OSAVI, SAVI, MSAVI2, EVI, MSR, TDVI, LAI, GCI, GRVI, GSAVI, GOSAVI, NLI, MNLI, RDVI, WDRVI, CVI, ENDVI, GLI, VARI`

{% hint style="warning" %}
**Numele de index necunoscute sunt omise fără avertisment.** Un nume care nu se regăsește în această listă (inclusiv cele cinci formule disponibile doar în interfața grafică: `FCI1`, `FCI2`, `GARI`, `GEMI`, `LCI` și orice formulă personalizată pe care ați salvat-o în interfața grafică) este omis cu doar o notificare în jurnal — execuția continuă fără acel index, iar execuția în sine raportează în continuare succesul. Notificarea este afișată astfel:

```
[INDEX_EXPAND] skipping unknown preset 'LCI'; known: ['CVI', 'ENDVI', 'EVI', ...]
```

Numele sunt comparate fără a se ține cont de majuscule și minuscule după eliminarea spațiilor, astfel încât `ndvi`, `NDVI` și ` NDVI ` reprezintă aceeași presetare. O presetare este, de asemenea, omisă dacă necesită o bandă pe care filtrul camerei dvs. nu o oferă.
{% endhint %}

Formulele exacte așa cum sunt implementate (simbolurile `x`/`y`/`z` reprezintă sloturi de bandă; maparea implicită este afișată pentru fiecare preset):

| Setare prestabilită | Formulă (așa cum este implementată) | Filtru implicit | Sloturi (x, y, z) |
| --- | --- | --- | --- |
| NDVI | `(y-x)/(y+x)` | RGN | Red, NIR |
| GNDVI | `(y-x)/(y+x)` | RGN | Green, NIR |
| NDRE | `(y-x)/(y+x)` | RE | RE, NIR |
| OSAVI | `(y-x)/(y+x+0.16)` | RGN | Red, NIR |
| SAVI | `1.5*(y-x)/(y+x+0.5)` | RGN | Red, NIR |
| MSAVI2 | `(2*y+1-sqrt((2*y+1)*(2*y+1)-8*(y-x)))/2` | RGN | Red, NIR |
| EVI | `2.5*(y-x)/(y+6*x-7.5*z+1)` | RGN | Red, NIR, Blue |
| MSR | `((y/x)-1)/(sqrt(y/x)+1)` | RGN | Red, NIR |
| TDVI | `1.5*(y-x)/sqrt(y*y+x+0.5)` | RGN | Red, NIR |
| LAI | `3.618*(2.5*(y-x)/(y+6*x-7.5*z+1))-0.118` | RGN | Red, NIR, Blue |
| GCI | `(y/x)-1` | RGN | Green, NIR |
| GRVI | `y/x` | RGN | Green, NIR |
| GSAVI | `1.5*(y-x)/(y+x+0.5)` | RGN | Green, NIR |
| GOSAVI | `(y-x)/(y+x+0.16)` | RGN | Green, NIR |
| NLI | `((y*y)-x)/((y*y)+x)` | RGN | Red, NIR |
| MNLI | `((y*y-x)*(1+0.5))/((y*y)+x+0.5)` | RGN | Red, NIR |
| RDVI | `(y-x)/sqrt(y+x)` | RGN | Red, NIR |
| WDRVI | `(0.2*y-x)/(0.2*y+x)` | RGN | Red, NIR |
| CVI | `(z/y)/(x/y)` | RGB | Red, Green, Blue |
| ENDVI | `((x+y)-(2*z))/((x+y)+(2*z))` | RGB | Red, Green, Blue |
| GLI | `((y-x)+(y-z))/((2*y)+x+z)` | RGB | Red, Green, Blue |
| VARI | `(y-x)/(y+x-z)` | RGB | Red, Green, Blue |

#### Cum se transformă numele unui preset în poziții de bandă

Când se introduce un nume simplu, cum ar fi `NDVI`, Chloros trebuie să decidă din ce canal al cărui fișier citește fiecare simbol. Se utilizează acest tabel, care asociază un cod de filtru cu poziția în matrice a fiecărui canal:

| Cod filtru | Canal → index matrice |
| --- | --- |
| OCN | Orange 0, Cyan 1, NIR 2 (`Red` este acceptat ca alias pentru Orange, de asemenea 0) |
| RGN | Red 0, Green 1, NIR 2 |
| NGB | NIR 0, Green 1, Blue 2 |
| RGB | Red 0, Green 1, Blue 2 |
| RE | RE 0 |
| NIR | NIR 0 |

**Filtrul implicit** al presetării (coloana „Filtru implicit” de mai sus) este utilizat atunci când proiectul conține imagini cu acel filtru. Dacă nu conține, Chloros scanează filtrele prezente efectiv în proiect în ordinea `RGN, OCN, NGB, RGB, RE, NIR` și îl alege pe primul care poate furniza fiecare canal de care are nevoie presetarea. Dacă niciunul nu poate, presetarea este omisă pentru acea execuție. Acesta este motivul pentru care `NDVI` solicitat pe un set de date care conține doar OCN produce totuși un rezultat rezonabil — se leagă de pozițiile Orange și NIR din OCN.

Șirurile modelului LATTICE M3C conțin filtrul cu prefixul `F` (`LATT-M3C-L41-FRGN`), dar prefixul este omis atunci când codul filtrului este citit din imagine, astfel încât o cameră FRGN rezolvă prin rândul `RGN` de mai sus și nu necesită o tratare specială.

### 3. Motorul de indexare LATTICE (`lattice index --preset`, Calculator de indexare în timp real) — 22 de presetări

Motorul LATTICE funcționează pe stive multibandă aliniate (matrice live sau fișiere TIFF multibandă exportate) și utilizează nume de canale cu litere mici (`red`, `green`, `blue`, `red_edge`, `nir`). Lista sa de presetări diferă de cele două de mai sus:

| Presetare | Formulă | Canale |
| --- | --- | --- |
| NDVI | `(nir - red) / (nir + red)` | roșu, nir |
| GNDVI | `(nir - green) / (nir + green)` | verde, NIR |
| BNDVI | `(nir - blue) / (nir + blue)` | albastru, nir |
| NDRE | `(nir - red_edge) / (nir + red_edge)` | roșu\_margine, nir |
| ENDVI | `((nir + green) - 2*blue) / ((nir + green) + 2*blue)` | albastru, verde, nir |
| SAVI | `1.5 * (nir - red) / (nir + red + 0.5)` | roșu, nir |
| OSAVI | `1.5 * (nir - red) / (nir + red + 0.16)` | roșu, nir |
| MSAVI | `(2*nir + 1 - sqrt((2*nir + 1)**2 - 8*(nir - red))) / 2` | roșu, infraroșu |
| EVI | `2.5 * (nir - red) / (nir + 6*red - 7.5*blue + 1)` | albastru, roșu, infraroșu |
| EVI2 | `2.5 * (nir - red) / (nir + 2.4*red + 1)` | roșu, infraroșu |
| CVI | `(nir / green) - (red / green)` | roșu, verde, infraroșu |
| MSR | `((nir/red) - 1) / (sqrt(nir/red) + 1)` | roșu, infraroșu |
| TDVI | `sqrt((nir - red) / (nir + red) + 0.5)` | roșu, infraroșu |
| LAI | `3.618 * ((nir - red) / (nir + 6*red - 7.5*green + 1)) - 0.118` | roșu, verde, infraroșu |
| GLI | `(2*green - red - blue) / (2*green + red + blue)` | roșu, verde, albastru |
| NGRDI | `(green - red) / (green + red)` | roșu, verde |
| VARI | `(green - red) / (green + red - blue)` | roșu, verde, albastru |
| TGI | `green - 0.39*red - 0.61*blue` | roșu, verde, albastru |
| EXG | `2*green - red - blue` | roșu, verde, albastru |
| CIRE | `(nir / red_edge) - 1` | roșu\_margine, nir |
| CIGREEN | `(nir / green) - 1` | verde, nir |
| NDWI | `(green - nir) / (green + nir)` | verde, nir |

Rulați `chloros-cli lattice index --list-presets` pentru a imprima acest tabel din versiunea instalată, iar `--list-gradients` pentru gradientele de culoare disponibile. Simbolurile canalelor sunt sensibile la majuscule și minuscule și trebuie să corespundă cu numele presetărilor scrise cu litere mici (de exemplu, `--channel red=Red_660 --channel nir=NIR_850`).

***

## CVI

Așa cum este implementat în interfața grafică și în lista de presetări CLI/SDK, CVI reprezintă formula raportului dintre rapoarte:

$$
CVI = {(z / y) \over (x / y)}
$$

cu maparea implicită a canalelor RGB: x=Red, y=Green, z=Blue. În interfața grafică puteți trage oricare dintre canalele camerei dvs. în sloturile x/y/z. Rețineți că presetarea `CVI` a motorului de indexare LATTICE utilizează o formulă diferită, `(NIR / Green) - (Red / Green)` — consultați tabelele de mai sus pentru suprafața pe care o utilizați.

***

## ENDVI - Indicele îmbunătățit al diferenței normalizate a vegetației

Acest indice utilizează canalul albastru pe lângă NIR și cel verde și este popular în cazul camerelor cu filtru NGB, unde banda albastră înlocuiește cea roșie.

$$
ENDVI = {(NIR + Green) - (2 * Blue) \over (NIR + Green) + (2 * Blue)}
$$

Implementarea se face prin formula simbolică `((x+y)-(2*z))/((x+y)+(2*z))` — atribuiți canalele NIR și Green ale camerei dvs. sloturilor x/y și canalul Blue la z (pentru o cameră NGB: x=NIR, y=Green, z=Blue).

***

## EVI - Indicele îmbunătățit al vegetației

Acest indice a fost dezvoltat inițial pentru a fi utilizat cu datele MODIS, ca o îmbunătățire față de NDVI, prin optimizarea semnalului de vegetație în zonele cu indice ridicat al suprafeței foliare (LAI). Este cel mai util în regiunile cu valori ridicate ale LAI, unde NDVI poate ajunge la saturație. Utilizează regiunea de reflectanță albastră pentru a corecta semnalele de fond ale solului și pentru a reduce influențele atmosferice, inclusiv împrăștierea aerosolilor.

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

Valorile EVI ar trebui să varieze între 0 și 1 pentru pixelii de vegetație. Elementele luminoase, cum ar fi norii și clădirile albe, împreună cu elementele întunecate, cum ar fi apa, pot genera valori anomale ale pixelilor într-o imagine EVI. Înainte de a crea o imagine EVI, ar trebui să mascați norii și elementele luminoase din imaginea de reflectanță și, opțional, să stabiliți un prag pentru valorile pixelilor de la 0 la 1.

_Referință: Huete, A., et al. „Prezentare generală a performanței radiometrice și biofizice a indicilor de vegetație MODIS.” Remote Sensing of Environment 83 (2002):195–213._

***

## FCI1 - Indicele de acoperire forestieră 1

_Disponibil numai în GUI — nu este disponibil ca presetare CLI/SDK `--indices`._

Acest indice distinge coronamentul forestier de alte tipuri de vegetație folosind imagini multispectrale de reflectanță care includ o bandă „red edge”.

$$
FCI1 = Red * RedEdge
$$

Zonele împădurite vor avea valori FCI1 mai mici, datorită reflectanței mai reduse a copacilor și prezenței umbrelor în interiorul coronamentului.

_Referință: Becker, Sarah J., Craig S.T. Daughtry și Andrew L. Russ. „Indici robusti ai acoperirii forestiere pentru imagini multispectrale.” Photogrammetric Engineering &amp; Remote Sensing 84.8 (2018): 505-512._

***

## FCI2 - Indicele acoperirii forestiere 2

_Disponibil numai în interfața grafică (GUI) — nu este disponibil ca presetare CLI/SDK `--indices`._

Acest indice distinge coronamentul forestier de alte tipuri de vegetație utilizând imagini multispectrale de reflectanță care nu includ o bandă de margine roșie.

$$
FCI2 = Red * NIR
$$

Zonele împădurite vor avea valori FCI2 mai scăzute datorită reflectanței mai reduse a copacilor și prezenței umbrelor în interiorul coronamentului.

_Referință: Becker, Sarah J., Craig S.T. Daughtry și Andrew L. Russ. „Indici robusti ai acoperirii forestiere pentru imagini multispectrale.” Photogrammetric Engineering &amp; Remote Sensing 84.8 (2018): 505-512._

***

## GEMI - Indicele de monitorizare globală a mediului

_Disponibil numai în interfața grafică (GUI) — nu este disponibil ca presetare CLI/SDK `--indices`._

Acest indice de vegetație neliniar este utilizat pentru monitorizarea globală a mediului pe baza imaginilor satelitare și încearcă să corecteze efectele atmosferice. Este similar cu NDVI, dar este mai puțin sensibil la efectele atmosferice. Este influențat de solul gol; prin urmare, nu se recomandă utilizarea sa în zone cu vegetație rară sau moderat de densă.

$$
GEMI = eta (1 - 0.25 * eta) - {Red - 0.125 \over 1 - Red}
$$

Unde:

$$
eta = {2(NIR^{2}-Red^{2}) + 1.5 * NIR + 0.5 *  Red \over NIR + Red + 0.5}
$$

_Referință: Pinty, B. și M. Verstraete. GEMI: un indice neliniar pentru monitorizarea vegetației globale prin sateliți. Vegetation 101 (1992): 15-20._

***

## GARI - Green Indicator rezistent la influențele atmosferice

_Disponibil numai în interfața grafică (GUI) — nu este disponibil ca presetare CLI/SDK `--indices`._

Acest indice este mai sensibil la o gamă largă de concentrații de clorofilă și mai puțin sensibil la efectele atmosferice decât NDVI.

$$
GARI = {NIR - [Green - \gamma(Blue - Red)] \over NIR + [Green - \gamma(Blue - Red)]   }
$$

Constanta gamma este o funcție de ponderare care depinde de condițiile aerosolilor din atmosferă. ENVI utilizează o valoare de 1,7, care este valoarea recomandată de Gitelson, Kaufman și Merzylak (1996, pagina 296).

_Referință: Gitelson, A., Y. Kaufman și M. Merzylak. „Utilizarea unui canal Green în teledetecția vegetației globale de la EOS-MODIS.” Remote Sensing of Environment 58 (1996): 289-298._

***

## GCI - Green Indicele clorofilei

Acest indice este utilizat pentru estimarea conținutului de clorofilă din frunze la o gamă largă de specii de plante.

$$
GCI = {NIR \over Green} - 1
$$

Utilizarea unor lungimi de undă largi, precum NIR și cele din spectrul verde, asigură o predicție mai bună a conținutului de clorofilă, oferind în același timp o sensibilitate mai mare și un raport semnal-zgomot mai ridicat.

_Referință: Gitelson, A., Y. Gritz și M. Merzlyak. „Relațiile dintre conținutul de clorofilă din frunze și reflectanța spectrală, precum și algoritmi pentru evaluarea nedistructivă a clorofilei în frunzele plantelor superioare.” Journal of Plant Physiology 160 (2003): 271-282._

***

## GLI - Green Indicele frunzelor

Acest indice a fost conceput inițial pentru a fi utilizat împreună cu o cameră digitală RGB pentru măsurarea acoperirii cu grâu, unde valorile numerice digitale (DN) pentru roșu, verde și albastru variază între 0 și 255.

$$
GLI = {(Green - Red) + (Green - Blue)  \over (2 * Green) + Red + Blue }
$$

Valorile GLI variază între -1 și +1. Valorile negative reprezintă solul și elementele nevii, în timp ce valorile pozitive reprezintă frunzele verzi și tulpinile.

_Referință: Louhaichi, M., M. Borman și D. Johnson. „Platformă cu localizare spațială și fotografie aeriană pentru documentarea impactului pășunatului asupra grâului”. Geocarto International 16, nr. 1 (2001): 65-70._

***

## GNDVI – Green Indicele de vegetație diferențial normalizat

Acest indice este similar cu NDVI, cu excepția faptului că măsoară spectrul verde de la 540 la 570 nm în loc de spectrul roșu. Acest indice este mai sensibil la concentrația de clorofilă decât NDVI.

$$
GNDVI = {(NIR - Green) \over (NIR + Green)  }
$$

_Referință: Gitelson, A. și M. Merzlyak. „Teledetecția concentrației de clorofilă în frunzele plantelor superioare”. Advances in Space Research 22 (1998): 689-692._

***

## GOSAVI - Green Indicele de vegetație optimizat și ajustat în funcție de sol

Acest indice a fost conceput inițial cu ajutorul fotografiei color-infraroșu pentru a prezice necesarul de azot al porumbului. Este similar cu OSAVI, dar înlocuiește banda verde cu cea roșie.

$$
GOSAVI = {NIR - Green \over NIR + Green + 0.16)  }
$$

_Referință: Sripada, R., et al. „Determinarea necesarului de azot pe durata sezonului pentru porumb folosind fotografia aeriană color-infraroșu”. Teză de doctorat, Universitatea de Stat din Carolina de Nord, 2005._

***

## Raportul GRVI - Green al indicelui de vegetație

Acest indice este sensibil la ratele de fotosinteză din coronamentul pădurilor, deoarece reflectanțele verde și roșu sunt puternic influențate de modificările pigmentelor frunzelor.

$$
GRVI = {NIR \over Green }
$$

_Referință: Sripada, R., et al. „Fotografia aeriană color-infraroșu pentru determinarea necesarului de azot la începutul sezonului în cazul porumbului.” Agronomy Journal 98 (2006): 968-977._

***

## GSAVI - Green Indicele de vegetație ajustat în funcție de sol

Acest indice a fost conceput inițial cu ajutorul fotografiei color-infraroșu pentru a prezice necesarul de azot al porumbului. Este similar cu SAVI, dar înlocuiește banda verde cu cea roșie.

$$
GSAVI = 1.5 * {(NIR - Green) \over (NIR + Green + 0.5)  }
$$

_Referință: Sripada, R., et al. „Determinarea necesarului de azot pe durata sezonului pentru porumb folosind fotografia aeriană color-infraroșu”. Teză de doctorat, Universitatea de Stat din Carolina de Nord, 2005._

***

## LAI – Indicele suprafeței foliare

Acest indice este utilizat pentru estimarea acoperirii cu frunziș și pentru prognozarea creșterii și a randamentului culturilor. ENVI calculează LAI verde folosind următoarea formulă empirică din Boegh et al (2002):

$$
LAI = 3.618 * EVI - 0.118
$$

Unde EVI este:

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

Valorile ridicate ale LAI variază de obicei între aproximativ 0 și 3,5. Cu toate acestea, atunci când scena conține nori și alte elemente luminoase care produc pixeli saturați, valorile LAI pot depăși 3,5. În mod ideal, ar trebui să mascați norii și elementele luminoase din scenă înainte de a crea o imagine LAI.

_Referință: Boegh, E., H. Soegaard, N. Broge, C. Hasager, N. Jensen, K. Schelde și A. Thomsen. „Date multispectrale aeriene pentru cuantificarea indicelui suprafeței foliare, a concentrației de azot și a eficienței fotosintetice în agricultură.” Remote Sensing of Environment 81, nr. 2-3 (2002): 179-193._

***

## LCI - Indicele de clorofilă al frunzelor

_Numai în GUI — nu este disponibil ca presetare CLI/SDK `--indices`._

Acest indice este utilizat pentru estimarea conținutului de clorofilă din plantele superioare, fiind sensibil la variațiile de reflectanță cauzate de absorbția clorofilei.

$$
LCI = {NIR2 - RedEdge \over NIR2 + Red}
$$

_Referință: Datt, B. „Teledetecția conținutului de apă din frunzele de eucalipt”. Journal of Plant Physiology 154, nr. 1 (1999): 30-36._

***

## MNLI – Indicele neliniar modificat

Acest indice reprezintă o îmbunătățire a indicelui neliniar (NLI), care încorporează indicele de vegetație ajustat în funcție de sol (SAVI) pentru a ține cont de fondul solului. ENVI utilizează o valoare a factorului de ajustare a fundalului coronamentului (_L_) de 0,5.

$$
MNLI = {(NIR^{2} - Red) * (1 + L) \over (NIR^{2} + Red + L)  }
$$

_Referință: Yang, Z., P. Willis și R. Mueller. „Impactul imaginii AWIFS îmbunătățite prin raportul de benzi asupra preciziei clasificării culturilor”. Lucrările Simpozionului Pecora 17 privind teledetecția (2008), Denver, CO._

***

## MSAVI2 – Indicele de vegetație ajustat în funcție de sol modificat 2

Acest indice este o versiune simplificată a indicelui MSAVI propus de Qi et al. (1994), care reprezintă o îmbunătățire a indicelui de vegetație ajustat în funcție de sol (SAVI). Acesta reduce zgomotul de sol și mărește gama dinamică a semnalului de vegetație. MSAVI2 se bazează pe o metodă inductivă care nu utilizează o valoare constantă a lui _L_ (ca în cazul SAVI) pentru a evidenția vegetația sănătoasă.

$$
MSAVI2 = {2 * NIR + 1 - \sqrt{(2 * NIR + 1)^{2} - 8(NIR - Red)} \over 2}
$$

_Referință: Qi, J., A. Chehbouni, A. Huete, Y. Kerr și S. Sorooshian. „Un indice modificat al vegetației ajustat la sol.” Remote Sensing of Environment 48 (1994): 119-126._

***

## MSR – Raport simplu modificat

Acest indice este o modificare a raportului simplu NIR/Red, conceput pentru a liniariza relația sa cu parametrii biofizici, și este mai sensibil decât NDVI la densități mai mari de vegetație.

$$
MSR = {(NIR / Red) - 1 \over \sqrt{NIR / Red} + 1}
$$

_Referință: Chen, J. „Evaluarea indicilor de vegetație și a unui raport simplu modificat pentru aplicații boreale”. Canadian Journal of Remote Sensing 22 (1996): 229-242._

***

## NDRE – Diferența normalizată RedEdge

Acest indice este similar cu NDVI, dar compară contrastul dintre NIR și RedEdge în loc de Red, care detectează adesea mai repede stresul vegetației.

$$
NDRE = {NIR - RedEdge \over NIR + RedEdge  }
$$

***

## NDVI – Indicele de vegetație prin diferență normalizată

Acest indice măsoară vegetația verde și sănătoasă. Combinația dintre formularea sa bazată pe diferența normalizată și utilizarea regiunilor cu cea mai mare absorbție și reflectanță ale clorofilei îl face robust într-o gamă largă de condiții. Cu toate acestea, se poate satura în condiții de vegetație densă, când valoarea LAI devine ridicată.

$$
NDVI = {NIR - Red \over NIR + Red  }
$$

Valoarea acestui indice variază între -1 și 1. Intervalul obișnuit pentru vegetația verde este cuprins între 0,2 și 0,8.

_Referință: Rouse, J., R. Haas, J. Schell și D. Deering. Monitorizarea sistemelor de vegetație în Marile Câmpii cu ERTS. Al treilea simpozion ERTS, NASA (1973): 309-317._

***

## NLI – Indicele neliniar

Acest indice pornește de la premisa că relația dintre mulți indici de vegetație și parametrii biofizici de suprafață este neliniară. El liniarizează relațiile cu parametrii de suprafață care tind să fie neliniari.

$$
NLI = {NIR^{2} - Red \over NIR^{2} + Red  }
$$

_Referință: Goel, N. și W. Qin. „Influențele arhitecturii coronamentului asupra relațiilor dintre diverși indici de vegetație și LAI și Fpar: o simulare pe calculator.” Remote Sensing Reviews 10 (1994): 309-347._

***

## OSAVI – Indicele de vegetație optimizat, ajustat în funcție de sol

Acest indice se bazează pe Indicele de vegetație ajustat în funcție de sol (SAVI). Acesta utilizează o valoare standard de 0,16 pentru factorul de ajustare a fundalului coronamentului. Rondeaux (1996) a stabilit că această valoare oferă o variație mai mare a solului decât SAVI în cazul acoperirii vegetale reduse, demonstrând în același timp o sensibilitate crescută la acoperirea vegetală mai mare de 50%. Acest indice se utilizează cel mai bine în zone cu vegetație relativ rară, unde solul este vizibil prin coronament.

$$
OSAVI = {(NIR - Red) \over (NIR + Red + 0.16)  }
$$

_Referință: Rondeaux, G., M. Steven și F. Baret. „Optimizarea indicilor de vegetație ajustați în funcție de sol”. Remote Sensing of Environment 55 (1996): 95-107._

***

## RDVI – Indicele de vegetație al diferenței renormalizate

Acest indice utilizează diferența dintre lungimile de undă din infraroșu apropiat și roșu, împreună cu NDVI, pentru a evidenția vegetația sănătoasă. Este insensibil la efectele solului și la geometria de observare a soarelui.

$$
RDVI = {(NIR- Red) \over \sqrt{(NIR + Red)}  }
$$

_Referință: Roujean, J. și F. Breon. „Estimarea PAR-ului absorbit de vegetație pe baza măsurătorilor de reflectanță bidirecțională”. Remote Sensing of Environment 51 (1995): 375-384._

***

## SAVI - Indicele de vegetație ajustat în funcție de sol

Acest indice este similar cu NDVI, dar suprimă efectele pixelilor de sol. Acesta utilizează un factor de ajustare a fundalului coronamentului, _L_, care este o funcție a densității vegetației și necesită adesea cunoașterea prealabilă a cantităților de vegetație. Huete (1988) sugerează o valoare optimă a lui _L_=0,5 pentru a ține cont de variațiile de fond ale solului de ordinul întâi. Acest indice se utilizează cel mai bine în zone cu vegetație relativ rară, unde solul este vizibil prin coronament.

$$
SAVI = {1.5 * (NIR- Red) \over (NIR + Red + 0.5)  }
$$

_Referință: Huete, A. „A Soil-Adjusted Vegetation Index (SAVI)”. Remote Sensing of Environment 25 (1988): 295-309._

***

## TDVI – Indicele de vegetație al diferenței transformate

Acest indice este util pentru monitorizarea acoperirii vegetale în mediile urbane. El nu se saturează, spre deosebire de NDVI și SAVI.

$$
TDVI = 1.5 * {(NIR- Red) \over \sqrt{NIR^{2} + Red + 0.5}  }
$$

_Referință: Bannari, A., H. Asalhi și P. Teillet. „Indicele de vegetație prin diferență transformată (TDVI) pentru cartografierea acoperirii vegetale” în Lucrările Simpozionului de Geostiințe și Teledetecție, IGARSS &#x27;02, IEEE International, volumul 5 (2002)._

***

## VARI - Indicele vizibil rezistent la efectele atmosferice

Acest indice se bazează pe ARVI și este utilizat pentru a estima fracția de vegetație dintr-o scenă, cu o sensibilitate redusă la efectele atmosferice.

$$
VARI = {Green - Red \over Green + Red - Blue  }
$$

_Referință: Gitelson, A., et al. „Vegetația și liniile de sol în spațiul spectral vizibil: un concept și o tehnică pentru estimarea de la distanță a fracției de vegetație”. International Journal of Remote Sensing 23 (2002): 2537−2562._

***

## WDRVI – Indicele de vegetație cu gamă dinamică largă

Acest indice este similar cu NDVI, dar utilizează un coeficient de ponderare (_a_) pentru a reduce disparitatea dintre contribuțiile semnalelor din spectrulinfraroșu apropiat și roșu la NDVI. WDRVI este deosebit de eficient în scenele cu densitate a vegetației moderată până la ridicată, atunci când NDVI depășește 0,6. NDVI tinde să se stabilizeze odată cu creșterea fracției de vegetație și a indicelui de suprafață foliară (LAI) cresc, în timp ce WDRVI este mai sensibil la un interval mai larg de fracții de vegetație și la modificările valorii LAI.

$$
WDRVI = {(\alpha * NIR- Red) \over (\alpha * NIR + Red)}
$$

Coeficientul de ponderare (_a_) poate varia între 0,1 și 0,2. O valoare de 0,2 este recomandată de Henebry, Viña și Gitelson (2004).

_Referințe_

_Gitelson, A. „Indicele de vegetație cu interval dinamic larg pentru cuantificarea de la distanță a caracteristicilor biofizice ale vegetației”. Journal of Plant Physiology 161, nr. 2 (2004): 165-173._

_Henebry, G., A. Viña și A. Gitelson. „Indicele de vegetație cu gamă dinamică largă și utilitatea sa potențială pentru analiza lacunelor.” Gap Analysis Bulletin 12: 50-56._
