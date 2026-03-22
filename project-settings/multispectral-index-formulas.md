---
description: This page lists some multispectral indices that Chloros uses
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/multispectral-index-formulas
---

# Formule pentru indici multispectrali

Formulele de mai jos pentru indici utilizează o combinație a intervalelor medii de transmisie ale filtrului Survey3:

<table><thead><tr><th align="center">Culoarea filtrului Survey3</th><th width="196.199951171875" align="center">Survey3 Denumirea filtrului</th><th width="159.800048828125" align="center">Interval de transmisie (FWHM)</th><th align="center">Transmisie medie</th></tr></thead><tbody><tr><td align="center">Blue</td><td align="center">NGB - Blue</td><td align="center">468-483 nm</td><td align="center">475 nm</td></tr><tr><td align="center">Cyan</td><td align="center">OCN- Cyan</td><td align="center">476-512 nm</td><td align="center">494 nm</td></tr><tr><td align="center">Green</td><td align="center">RGN | NGB - Green</td><td align="center">543-558 nm</td><td align="center">547 nm</td></tr><tr><td align="center">Orange</td><td align="center">OCN - Orange</td><td align="center">598-640 nm</td><td align="center">619 nm</td></tr><tr><td align="center">Red</td><td align="center">RGN - Red</td><td align="center">653-668 nm</td><td align="center">661 nm</td></tr><tr><td align="center">RedEdge</td><td align="center">Re - RedEdge</td><td align="center">712-735 nm</td><td align="center">724 nm</td></tr><tr><td align="center">NIR1</td><td align="center">OCN - NIR1</td><td align="center">798-848 nm</td><td align="center">823 nm</td></tr><tr><td align="center">NIR2</td><td align="center">RGN | NGB | NIR - NIR2</td><td align="center">835-865 nm</td><td align="center">850 nm</td></tr></tbody></table>Când se utilizează aceste formule, numele poate termina în „\_1” sau „\_2”, ceea ce corespunde filtrului NIR, fie NIR1, fie NIR2.

***

## EVI - Indicele de vegetație îmbunătățit

Acest indice a fost dezvoltat inițial pentru a fi utilizat cu datele MODIS ca o îmbunătățire față de NDVI, prin optimizarea semnalului de vegetație în zonele cu indice de suprafață foliară ridicat (LAI). Este cel mai util în regiunile cu LAI ridicat, unde NDVI poate ajunge la saturație. Utilizează regiunea de reflectanță albastră pentru a corecta semnalele de fond ale solului și pentru a reduce influențele atmosferice, inclusiv împrăștierea aerosolilor.

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

Valorile EVI ar trebui să varieze între 0 și 1 pentru pixelii de vegetație. Elementele luminoase, cum ar fi norii și clădirile albe, împreună cu elementele întunecate, cum ar fi apa, pot duce la valori anomale ale pixelilor într-o imagine EVI. Înainte de a crea o imagine EVI, ar trebui să mascați norii și elementele luminoase din imaginea de reflectanță și, opțional, să stabiliți un prag pentru valorile pixelilor de la 0 la 1.

_Referință: Huete, A., et al. „Prezentare generală a performanței radiometrice și biofizice a indicilor de vegetație MODIS.” Remote Sensing of Environment 83 (2002):195–213._

***

## FCI1 - Indicele de acoperire forestieră 1

Acest indice distinge coronamentul forestier de alte tipuri de vegetație folosind imagini de reflectanță multispectrală care includ o bandă de margine roșie.

$$
FCI1 = Red * RedEdge
$$

Zonele împădurite vor avea valori FCI1 mai mici datorită reflectanței mai scăzute a copacilor și prezenței umbrelor în interiorul coronamentului.

_Referință: Becker, Sarah J., Craig S.T. Daughtry și Andrew L. Russ. „Indici robusti ai acoperirii forestiere pentru imagini multispectrale.” Photogrammetric Engineering &amp; Remote Sensing 84.8 (2018): 505-512._

***

## FCI2 - Indicele de acoperire forestieră 2

Acest indice distinge coronamentul forestier de alte tipuri de vegetație folosind imagini de reflectanță multispectrală care nu includ o bandă de margine roșie.

$$
FCI2 = Red * NIR
$$

Zonele împădurite vor avea valori FCI2 mai mici datorită reflectanței mai scăzute a copacilor și prezenței umbrelor în coronament.

_Referință: Becker, Sarah J., Craig S.T. Daughtry și Andrew L. Russ. „Indici robusti ai acoperirii forestiere pentru imagini multispectrale.” Photogrammetric Engineering &amp; Remote Sensing 84.8 (2018): 505-512._

***

## GEMI - Indicele de monitorizare globală a mediului

Acest indice de vegetație neliniar este utilizat pentru monitorizarea globală a mediului pe baza imaginilor satelitare și încearcă să corecteze efectele atmosferice. Este similar cu NDVI, dar este mai puțin sensibil la efectele atmosferice. Este afectat de solul gol; prin urmare, nu este recomandat pentru utilizare în zone cu vegetație rară sau moderat de densă.

$$
GEMI = eta (1 - 0.25 * eta) - {Red - 0.125 \over 1 - Red}
$$

Unde:

$$
eta = {2(NIR^{2}-Red^{2}) + 1.5 * NIR + 0.5 *  Red \over NIR + Red + 0.5}
$$

_Referință: Pinty, B. și M. Verstraete. GEMI: un indice neliniar pentru monitorizarea vegetației globale din sateliți. Vegetation 101 (1992): 15-20._

***

## GARI - Green Indicele rezistent la influențele atmosferice

Acest indice este mai sensibil la o gamă largă de concentrații de clorofilă și mai puțin sensibil la efectele atmosferice decât NDVI.

$$
GARI = {NIR - [Green - \gamma(Blue - Red)] \over NIR + [Green - \gamma(Blue - Red)]   }
$$

Constanta gamma este o funcție de ponderare care depinde de condițiile aerosolilor din atmosferă. ENVI utilizează o valoare de 1,7, care este valoarea recomandată de Gitelson, Kaufman și Merzylak (1996, pagina 296).

_Referință: Gitelson, A., Y. Kaufman și M. Merzylak. „Utilizarea unui canal Green în teledetecția vegetației globale de la EOS-MODIS.” Remote Sensing of Environment 58 (1996): 289-298._

***

## GCI - Green Indicele clorofilei

Acest indice este utilizat pentru a estima conținutul de clorofilă din frunze pentru o gamă largă de specii de plante.

$$
GCI = {NIR \over Green} - 1
$$

Utilizarea unor lungimi de undă largi NIR și verzi oferă o predicție mai bună a conținutului de clorofilă, permițând în același timp o sensibilitate mai mare și un raport semnal-zgomot mai ridicat.

_Referință: Gitelson, A., Y. Gritz și M. Merzlyak. „Relațiile dintre conținutul de clorofilă din frunze și reflectanța spectrală și algoritmi pentru evaluarea nedistructivă a clorofilei în frunzele plantelor superioare.” Journal of Plant Physiology 160 (2003): 271-282._

***

## GLI - Green Indicele frunzelor

Acest indice a fost conceput inițial pentru a fi utilizat cu o cameră digitală RGB pentru a măsura acoperirea cu grâu, unde valorile digitale (DN) pentru roșu, verde și albastru variază de la 0 la 255.

$$
GLI = {(Green - Red) + (Green - Blue)  \over (2 * Green) + Red + Blue }
$$

Valorile GLI variază între -1 și +1. Valorile negative reprezintă solul și elementele nevii, în timp ce valorile pozitive reprezintă frunzele verzi și tulpinile.

_Referință: Louhaichi, M., M. Borman și D. Johnson. „Platformă spațială și fotografie aeriană pentru documentarea impactului pășunatului asupra grâului.” Geocarto International 16, nr. 1 (2001): 65-70._

***

## GNDVI - Green Indicele de vegetație diferențiat normalizat

Acest indice este similar cu NDVI, cu excepția faptului că măsoară spectrul verde de la 540 la 570 nm în loc de spectrul roșu. Acest indice este mai sensibil la concentrația de clorofilă decât NDVI.

$$
GNDVI = {(NIR - Green) \over (NIR + Green)  }
$$

_Referință: Gitelson, A. și M. Merzlyak. „Remote Sensing of Chlorophyll Concentration in Higher Plant Leaves.” Advances in Space Research 22 (1998): 689-692._

***

## GOSAVI - Green Indicele de vegetație optimizat ajustat la sol

Acest indice a fost conceput inițial cu ajutorul fotografiei color-infraroșu pentru a prezice necesarul de azot pentru porumb. Este similar cu OSAVI, dar înlocuiește banda verde cu cea roșie.

$$
GOSAVI = {NIR - Green \over NIR + Green + 0.16)  }
$$

_Referință: Sripada, R., et al. „Determinarea necesarului de azot în timpul sezonului pentru porumb folosind fotografia aeriană color-infraroșu.” Teză de doctorat, Universitatea de Stat din Carolina de Nord, 2005._

***

## GRVI - Green Raportul indicelui de vegetație

Acest indice este sensibil la ratele de fotosinteză din coronamentul pădurilor, deoarece reflectanțele verde și roșu sunt puternic influențate de schimbările pigmentelor frunzelor.

$$
GRVI = {NIR \over Green }
$$

_Referință: Sripada, R., et al. „Fotografie aeriană în infraroșu color pentru determinarea necesarului de azot la începutul sezonului pentru porumb.” Agronomy Journal 98 (2006): 968-977._

***

## GSAVI - Green Indicele de vegetație ajustat la sol

Acest indice a fost conceput inițial cu ajutorul fotografiei color-infraroșu pentru a prezice necesarul de azot pentru porumb. Este similar cu SAVI, dar înlocuiește banda verde cu cea roșie.

$$
GSAVI = 1.5 * {(NIR - Green) \over (NIR + Green + 0.5)  }
$$

_Referință: Sripada, R., et al. „Determinarea necesarului de azot în timpul sezonului pentru porumb folosind fotografia aeriană color-infraroșu.” Teză de doctorat, Universitatea de Stat din Carolina de Nord, 2005._

***

## LAI - Indicele suprafeței foliare

Acest indice este utilizat pentru a estima acoperirea cu frunziș și pentru a prognoza creșterea și randamentul culturilor. ENVI calculează LAI verde folosind următoarea formulă empirică din Boegh et al (2002):

$$
LAI = 3.618 * EVI - 0.118
$$

Unde EVI este:

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

Valorile ridicate ale LAI variază de obicei între aproximativ 0 și 3,5. Cu toate acestea, atunci când scena conține nori și alte elemente luminoase care produc pixeli saturați, valorile LAI pot depăși 3,5. În mod ideal, ar trebui să mascați norii și elementele luminoase din scena dvs. înainte de a crea o imagine LAI.

_Referință: Boegh, E., H. Soegaard, N. Broge, C. Hasager, N. Jensen, K. Schelde și A. Thomsen. „Date multispectrale aeriene pentru cuantificarea indicelui de suprafață foliară, a concentrației de azot și a eficienței fotosintetice în agricultură.” Remote Sensing of Environment 81, nr. 2-3 (2002): 179-193._

***

## LCI - Indicele clorofilei frunzelor

Acest indice este utilizat pentru estimarea conținutului de clorofilă din plantele superioare, fiind sensibil la variațiile de reflectanță cauzate de absorbția clorofilei.

$$
LCI = {NIR2 - RedEdge \over NIR2 + Red}
$$

_Referință: Datt, B. „Teledetecția conținutului de apă în frunzele de eucalipt.” Journal of Plant Physiology 154, nr. 1 (1999): 30-36._

***

## MNLI - Indicele neliniar modificat

Acest indice este o îmbunătățire a indicelui neliniar (NLI) care încorporează indicele de vegetație ajustat la sol (SAVI) pentru a ține cont de fundalul solului. ENVI utilizează o valoare a factorului de ajustare a fundalului coronamentului (_L_) de 0,5.

$$
MNLI = {(NIR^{2} - Red) * (1 + L) \over (NIR^{2} + Red + L)  }
$$

_Referință: Yang, Z., P. Willis și R. Mueller. „Impactul imaginii AWIFS îmbunătățite prin raportul de benzi asupra preciziei clasificării culturilor.” Lucrările Simpozionului de teledetecție Pecora 17 (2008), Denver, CO._

***

## MSAVI2 - Indicele de vegetație ajustat la sol modificat 2

Acest indice este o versiune mai simplă a indicelui MSAVI propus de Qi et al. (1994), care îmbunătățește Indicele de vegetație ajustat la sol (SAVI). Acesta reduce zgomotul de sol și mărește gama dinamică a semnalului de vegetație. MSAVI2 se bazează pe o metodă inductivă care nu utilizează o valoare constantă _L_ (ca în cazul SAVI) pentru a evidenția vegetația sănătoasă.

$$
MSAVI2 = {2 * NIR + 1 - \sqrt{(2 * NIR + 1)^{2} - 8(NIR - Red)} \over 2}
$$

_Referință: Qi, J., A. Chehbouni, A. Huete, Y. Kerr și S. Sorooshian. „Un indice de vegetație ajustat în funcție de sol modificat.” Remote Sensing of Environment 48 (1994): 119-126._

***

## NDRE- Diferența normalizată RedEdge

Acest indice este similar cu NDVI, dar compară contrastul dintre NIR și RedEdge în loc de Red, care detectează adesea mai repede stresul vegetației.

$$
NDRE = {NIR - RedEdge \over NIR + RedEdge  }
$$

***

## NDVI - Indicele de vegetație diferențial normalizat

Acest indice este o măsură a vegetației verzi și sănătoase. Combinația dintre formularea sa diferențială normalizată și utilizarea regiunilor cu cea mai mare absorbție și reflectanță ale clorofilei îl face robust într-o gamă largă de condiții. Cu toate acestea, se poate satura în condiții de vegetație densă atunci când LAI devine ridicat.

$$
NDVI = {NIR - Red \over NIR + Red  }
$$

Valoarea acestui indice variază între -1 și 1. Intervalul obișnuit pentru vegetația verde este de la 0,2 la 0,8.

_Referință: Rouse, J., R. Haas, J. Schell și D. Deering. Monitorizarea sistemelor de vegetație în Marile Câmpii cu ERTS. Al treilea simpozion ERTS, NASA (1973): 309-317._

***

## NLI - Indicele neliniar

Acest indice presupune că relația dintre mulți indici de vegetație și parametrii biofizici de suprafață este neliniară. El liniarizează relațiile cu parametrii de suprafață care tind să fie neliniari.

$$
NLI = {NIR^{2} - Red \over NIR^{2} + Red  }
$$

_Referință: Goel, N. și W. Qin. „Influențele arhitecturii coronamentului asupra relațiilor dintre diverși indici de vegetație și LAI și Fpar: o simulare pe computer.” Remote Sensing Reviews 10 (1994): 309-347._

***

## OSAVI - Indicele de vegetație ajustat la sol optimizat

Acest indice se bazează pe Indicele de vegetație ajustat la sol (SAVI). Acesta utilizează o valoare standard de 0,16 pentru factorul de ajustare a fundalului coronamentului. Rondeaux (1996) a stabilit că această valoare oferă o variație mai mare a solului decât SAVI pentru acoperirea vegetală redusă, demonstrând în același timp o sensibilitate crescută la acoperirea vegetală mai mare de 50%. Acest indice este cel mai bine utilizat în zone cu vegetație relativ rară, unde solul este vizibil prin coronament.

$$
OSAVI = {(NIR - Red) \over (NIR + Red + 0.16)  }
$$

_Referință: Rondeaux, G., M. Steven și F. Baret. „Optimization of Soil-Adjusted Vegetation Indices.” Remote Sensing of Environment 55 (1996): 95-107._

***

## RDVI - Indicele de vegetație diferențial renormalizat

Acest indice utilizează diferența dintre lungimile de undă din infraroșu apropiat și roșu, împreună cu NDVI, pentru a evidenția vegetația sănătoasă. Este insensibil la efectele solului și la geometria de observare a soarelui.

$$
RDVI = {(NIR- Red) \over \sqrt{(NIR + Red)}  }
$$

_Referință: Roujean, J. și F. Breon. „Estimarea PAR absorbită de vegetație din măsurători bidirecționale ale reflectanței”. Remote Sensing of Environment 51 (1995): 375-384._

***

## SAVI - Indicele de vegetație ajustat în funcție de sol

Acest indice este similar cu NDVI, dar suprimă efectele pixelilor de sol. Acesta utilizează un factor de ajustare a fundalului coronamentului, _L_, care este o funcție a densității vegetației și necesită adesea cunoașterea prealabilă a cantităților de vegetație. Huete (1988) sugerează o valoare optimă de _L_=0,5 pentru a ține cont de variațiile de fundal ale solului de ordinul întâi. Acest indice este cel mai bine utilizat în zone cu vegetație relativ rară, unde solul este vizibil prin coronament.

$$
SAVI = {1.5 * (NIR- Red) \over (NIR + Red + 0.5)  }
$$

_Referință: Huete, A. „Un indice de vegetație ajustat la sol (SAVI).” Remote Sensing of Environment 25 (1988): 295-309._

***

## TDVI - Indicele de vegetație al diferenței transformate

Acest indice este util pentru monitorizarea acoperirii vegetale în mediile urbane. Nu se saturează ca NDVI și SAVI.

$$
TDVI = 1.5 * {(NIR- Red) \over \sqrt{NIR^{2} + Red + 0.5}  }
$$

_Referință: Bannari, A., H. Asalhi și P. Teillet. „Transformed Difference Vegetation Index (TDVI) for Vegetation Cover Mapping” În Proceedings of the Geoscience and Remote Sensing Symposium, IGARSS &#x27;02, IEEE International, Volumul 5 (2002)._

***

## VARI - Indicele vizibil rezistent la efectele atmosferice

Acest indice se bazează pe ARVI și este utilizat pentru a estima fracțiunea de vegetație dintr-o scenă cu sensibilitate redusă la efectele atmosferice.

$$
VARI = {Green - Red \over Green + Red - Blue  }
$$

_Referință: Gitelson, A., et al. „Vegetation and Soil Lines in Visible Spectral Space: A Concept and Technique for Remote Estimation of Vegetation Fraction. International Journal of Remote Sensing 23 (2002): 2537−2562._

***

## WDRVI - Indicele de vegetație cu gamă dinamică largă

Acest indice este similar cu NDVI, dar utilizează un coeficient de ponderare (_a_) pentru a reduce disparitatea dintre contribuțiile semnalelor din infraroșu apropiat și roșu la NDVI. WDRVI este deosebit de eficient în scenele cu densitate moderată până la ridicată a vegetației, atunci când NDVI depășește 0,6. NDVI tinde să se stabilizeze atunci când fracția de vegetație și indicele de suprafață foliară (LAI) cresc, în timp ce WDRVI este mai sensibil la o gamă mai largă de fracții de vegetație și la schimbările în LAI.

$$
WDRVI = {(\alpha * NIR- Red) \over (\alpha * NIR + Red)}
$$

Coeficientul de ponderare (_a_) poate varia între 0,1 și 0,2. O valoare de 0,2 este recomandată de Henebry, Viña și Gitelson (2004).

_Referințe_

_Gitelson, A. „Wide Dynamic Range Vegetation Index for Remote Quantification of Biophysical Characteristics of Vegetation.” Journal of Plant Physiology 161, nr. 2 (2004): 165-173._

_Henebry, G., A. Viña și A. Gitelson. „Indicele de vegetație cu gamă dinamică largă și utilitatea sa potențială pentru analiza decalajelor.” Gap Analysis Bulletin 12: 50-56._
