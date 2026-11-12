# Configurarea setărilor proiectului

Înainte de a prelucra imaginile, este important să configurați setările proiectului astfel încât să corespundă cerințelor fluxului dvs. de lucru. Panoul „Setări proiect” <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> oferă un control complet asupra calibrării, opțiunilor de prelucrare, indicilor multispectrali și formatelor de export.

## Accesarea setărilor proiectului

1. Deschideți proiectul în Chloros
2. Faceți clic pe pictograma **Setări proiect** <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> din bara laterală din stânga
3. Panoul „Setări proiect” afișează toate opțiunile de configurare

<figure><img src="../.gitbook/assets/image (28).png" alt=""><figcaption><p>Panoul „Setări proiect” — Afișare, detectarea țintelor și procesare</p></figcaption></figure>

{% hint style="info" %}
**Setările sunt salvate automat** împreună cu proiectul. Când redeschideți un proiect, toate setările sunt restaurate.
{% endhint %}

***

## Configurare rapidă pentru fluxuri de lucru obișnuite

### Setări implicite (recomandate pentru majoritatea utilizatorilor)

Setările implicite funcționează bine pentru fluxurile de lucru tipice din Survey3 și LATTICE:

* ✅ **Corecția vignetării**: Activată
* ✅ **Calibrarea reflectanței / balansul de alb**: Activată (utilizează ținte MAPIR și/sau date de la senzorul de lumină DAQ)
* ✅ **Metoda de debayering**: Standard (Rapidă, Calitate medie)
* ✅ **Format de export**: TIFF (16 biți) (TIFF (8 biți) pentru un proiect integral de 8 biți)
* ✅ **Toate produsele de export**: Activat (LATTICE capturează automat imagini debayerizate, de previzualizare, de radianță și de reflectanță)

Pur și simplu importați imaginile și începeți procesarea cu aceste setări implicite.

***

## Prezentare generală a setărilor proiectului

Panoul Setări proiect este organizat în secțiunile de mai jos. Două secțiuni suplimentare — **Senzor de lumină DAQ**și**Alinierea matricei** — apar automat atunci când proiectul dvs. conține fișierele relevante. Pentru documentația completă, consultați [Setările proiectului](../project-settings/project-settings.md).

### Afișare

* **Rezoluția miniaturilor imaginilor**: Rezoluția miniaturilor din grila de imagini. Opțiuni:**Implicit (512 px)**,**1024 px**,**2048 px**,**Rezoluție maximă**. Doar pentru afișare — nu afectează niciodată procesarea. Valorile mai mari oferă o imagine mai clară la mărire, dar se încarcă mai lent.

### Detectarea țintelor

Controlează modul în care Chloros identifică țintele de calibrare din imaginile dvs.

**Setări cheie:**

* **Suprafața minimă a eșantionului de calibrare (px)**: Pragul de dimensiune pentru detectarea țintelor (implicit:**25**, interval 0–10000)
* **Gruparea minimă a țintelor (0-100)**: pragul de similitudine pentru gruparea regiunilor țintă (implicit:**60**)**Când să reglați:**

* Măriți suprafața eșantionului dacă apar detectări eronate
* Micșorați-o dacă țintele nu sunt detectate
* Reglați gruparea dacă țintele sunt împărțite în mai multe detectări

{% hint style="info" %}
Aceste setări sunt dezactivate atunci când opțiunea **Calibrare reflectanță / balans de alb** este dezactivată — în acest caz, detectarea țintelor nu se execută deloc.
{% endhint %}

### Procesare

Opțiuni principale de procesare a imaginii și de calibrare.

**Setări cheie:**

* **Corecția vignetării**: Compensează întunecarea produsă de obiectiv la margini ✅ Recomandat
* **Calibrarea reflectanței / balansul de alb**: Calibrează imaginile folosind țintele detectate (Survey3) și/sau datele senzorului de lumină DAQ (LATTICE) ✅ Recomandat
* **Metoda de debayerizare**: Algoritm pentru conversia formatului RAW în format multispectral cu 3 canale
* **Interval minim de recalibrare**: Timpul minim, în secunde, între utilizarea țintelor de calibrare (implicit:**0** = se utilizează toate, interval 0–3600)**Produse de rezervă necalibrate:**Când un cadru nu poate fi calibrat din punct de vedere al reflectanței (nu există țintă disponibilă sau calibrarea este dezactivată), acesta este exportat ca unul dintre cele două produse de rezervă —**există exact unul dintre cele două pe fiecare execuție**, ales prin comutatorul de corecție a vignetării:

* **Exportare răspuns senzor**: scrie `Sensor_Response_Images` — utilizat când corecția vignetării este**dezactivată**
* **Export cu corecție de vignetă**: scrie `Vignette_Corrected_Images` — utilizat când corecția de vignetă este**activată**Caseta de selectare care nu este activă este estompată. Debifarea celei active oprește complet scrierea acelui fișier.**Produse de export LATTICE** (afișate pentru fiecare proiect; se aplică capturilor LATTICE):

* **Export debayered**: imaginea debayered liniară (`Debayered_Images`): offsetul de întuneric al pachetului de calibrare este eliminat, apoi se efectuează demosaicarea, fără câmp plat sau demixare — același fișier pe care o captură îl salvează la nivelul său debayered — în orientarea nativă a senzorului. Se aplică modulelor RGB și multispectrale.
* **Export previzualizare**: previzualizarea afișată (`Preview_Images`). RGB = balans de alb (iluminant DAQ când este disponibil, altfel „gray-world”) + gamma; multispectral = extindere în culori false.
* **Export radianță**: radianță spectrală float32 (`Radiance_Images`, W/m²/sr/nm). Numai pentru modulele multispectrale — nu se aplică master-urilor RGB.
* **Export reflectanță**: reflectanță de tip uint16 (`Reflectance_Calibrated_Images`, DN 32768 = ρ 1,0) atunci când o citire descendentă `.daq` sau o țintă din cadru acoperă cadrul. Numai pentru modulele multispectrale.

Toate cele patru sunt **activate implicit**— un cadru brut LATTICE importat se distribuie către fiecare produs activat și aplicabil într-o singură etapă de procesare. Caseta de selectare**Export reflectanță** este dezactivată atunci când calibrarea reflectanței este dezactivată. Setările a căror opțiune părinte le face imposibile sunt întotdeauna dezactivate, cu un text de informație care indică opțiunea ce trebuie modificată.

**Setări avansate:**

* **Decalaj de fus orar al senzorului de lumină**: Ore față de UTC pentru sincronizarea orei senzorului de lumină (implicit: 0, interval de la −12 la +12)
* **Aplicare corecții PPK**: Utilizează datele GPS/pin de expunere din fișierele `.daq` (implicit: dezactivat)
* **Pin de expunere 1/2**: Alocă camerele la pinii de expunere pentru configurații cu două camere

{% hint style="info" %}
**Nivelul de intrare LATTICE este automat.** Capturile LATTICE conțin nivelul de procesare în metadatele XMP, iar procesarea intră întotdeauna în fluxul de lucru la nivelul cadrului brut — nu este necesară nicio configurare în interfața grafică. (Indicatorul CLI `--input-level` există ca opțiune de suprascriere pentru utilizatorii avansați, în cazul capturilor cu metadate pierdute; consultați [Referința CLI](../reference/cli-reference.md).)
{% endhint %}

### Metoda de debayering

În prezent, oferim 2 metode de debayering în Chloros:

#### Standard (Rapid, Calitate medie)

Metoda standard de debayering procesează rapid, dar prezintă zgomot de culoare generat de debayering, ceea ce duce la imagini mai puțin precise și mai zgomotoase.

#### Texture Aware (lent, calitate maximă) \[Numai Chloros+]

Texture Aware utilizează o metodă de debayering de înaltă calitate, sensibilă la margini, combinată cu un model de eliminare a zgomotului bazat pe AI/ML, care elimină aproape tot zgomotul generat de debayering. Modelul necesită memorie GPU (VRAM) pentru a rula: cu **7 GB sau mai mult de VRAM**, poate procesa mai multe imagini simultan; sub 7 GB, procesează o singură imagine la un moment dat (vizibil mai lent). Consultați [Adaptarea dinamică a calculului](../processing-architecture/dynamic-compute-adaptation.md).

{% hint style="info" %}
**Capturile LATTICE utilizează întotdeauna demosaic-ul standard.** Nu există un model Texture Aware antrenat pentru LATTICE, astfel încât opțiunea nu este disponibilă pentru imaginile LATTICE — imaginile Survey3 din același proiect o pot utiliza în continuare.
{% endhint %}

### Indici (indici multispectrali)

Configurați ce indici de vegetație să fie calculați și exportați. Meniul derulant al interfeței grafice oferă **27 de formule de indici predefinite**.**Cum se adaugă indici:**

1. Faceți clic pe butonul**„Adăugați indice”**

2. Selectați un indice din meniul derulant (NDVI, NDRE, GNDVI etc.)
3. Configurați setările de vizualizare (culori LUT, intervale de valori)
4. Adăugați mai mulți indici, după cum este necesar

**Indici populari:**

* **NDVI**: Starea generală de sănătate a vegetației (cel mai frecvent)
* **NDRE**: Detectarea timpurie a stresului cu RedEdge
* **GNDVI**: Sensibil la concentrația de clorofilă
* **OSAVI**: Funcționează bine cu solul vizibil
* **EVI**: Regiuni cu indice ridicat al suprafeței foliare (LAI)

**Formule personalizate:**

* Creați formule personalizate de indici multispectrali folosind operații matematice între benzi pe toate canalele imaginii
* Salvați formulele personalizate pentru a le reutiliza
* Formulele personalizate sunt o funcționalitate a Chloros+; disponibilitatea depinde de nivelul planului dvs.

Pentru toți indicii și formulele disponibile — inclusiv care denumiri sunt disponibile doar în interfața grafică (GUI) și care funcționează și în CLI/SDK — consultați [Formule de indici multispectrali](../project-settings/multispectral-index-formulas.md).

### Export

Controlează formatul fișierului de ieșire.

**Formate disponibile**(setare:**Format imagine calibrat**, implicit**TIFF (16 biți)**):

* **TIFF (16 biți)**: Recomandat pentru GIS și analize științifice
* **TIFF (8 biți)**: TIFF pe 8 biți* **TIFF (32 de biți, procent)**: Valori în virgulă mobilă
* **PNG (8 biți)**: Compresie fără pierderi pentru vizualizare
* **JPG (8 biți)**: Fișiere de dimensiuni reduse, compresie cu pierderi

Lista arată ce pot conține imaginile proiectului: Chloros nu mărește adâncimea de biți. Imaginile exclusiv pe 8 biți oferă doar formatele TIFF (8 biți), PNG (8 biți) și JPG (8 biți), iar setarea implicită este TIFF (8 biți). Într-un proiect mixt, fiecare captură se exportă la propria adâncime de biți sau la una mai mică. Radianța rămâne de tip float pe 32 de biți.

Fișierele de ieșire sunt salvate în folderul proiectului, grupate după cameră și format: `<project>/<camera>/<format>/<Product>_Images/`. Radianța este **întotdeauna** salvată ca float32 în folderul `tiff32`, indiferent de această setare. Fișierele exportate păstrează numele fișierului sursă — folderul identifică produsul. Exporturile multispectrale LATTICE pot scrie, de asemenea, măști de limitare pe pixel într-un folder `clamp_masks/` situat lângă folderele produselor. Consultați [Finalizarea procesării](finishing-the-processing.md) pentru arborele complet de ieșire.

{% hint style="warning" %}
**Citirea valorilor de reflectanță**: valoarea DN care corespunde lui ρ = 1,0 depinde de camera sursă — LATTICE utilizează 32768 (marcat ca XMP `Chloros:PixelScale`), iar Survey3 utilizează 65535. Citiți eticheta în loc să presupuneți o valoare constantă. Consultați [Formate de imagine de ieșire](../output-image-formats.md).
{% endhint %}

### Senzor de lumină DAQ

Această secțiune listează fiecare fișier DAQ de lumină descendentă (`.daq` / `.csv`) din proiectul dvs., câte un rând per fișier, afișând modelul senzorului, numele fișierului și **capacul** difuzorului aplicabilă pentru fișierul respectiv.

* **Suprascrierea capacului (toate fișierele)**: un singur meniu derulant valabil pentru întregul proiect.**Auto**(implicit) utilizează**cap**-ul înregistrat pentru fiecare fișier — se presupune prezența soarelui acolo unde nu s-a înregistrat nimic, dar DAQ-U, DAQ-M și DAQ-E sunt livrate fără**cap**, iar Chloros nu poate vedea ce a fost ajustat, așa că procesarea afișează o avertizare privind un **cap**presupus: alegeți**cap**-ul care a fost ajustat. Selectarea unei limite de maximă anulează setările din toate fișierele, cu excepția celor de tip DAQ-E-S, al căror difuzor face parte din calibrarea sa: înregistrările brute sunt corectate cu ajutorul acestuia, iar înregistrările care au deja o limită de maximă sunt re-referențiate (corecția înregistrată este anulată, iar limita de maximă selectată este aplicată).
* Rândurile afișează avertismente atunci când un capac înregistrat a fost considerat implicit de către hub, fără a fi confirmat de operator, și atunci când capacul selectat nu are un profil pentru acel model de dispozitiv (suprascrierea este refuzată pentru acel fișier).

Înregistrările DAQ realizate în fila „Light Sensors” sunt adăugate automat la proiectul deschis, iar fișierele importate `.daq` / `.csv` apar aici imediat ce sunt adăugate.

<figure><img src="../.gitbook/assets/image (32).png" alt=""><figcaption><p>Setări inferioare ale proiectului — Index, Format de export, secțiunea Senzori de lumină DAQ și comenzile pentru șablonul/folderul proiectului</p></figcaption></figure>### Alinierea matricei

Această secțiune apare **numai** atunci când cel puțin o imagine din proiect conține transformarea de aliniere modul-la-modul pe care matricele LATTICE o aplică la captură (`Chloros:Alignment*` XMP). Aceasta arată câte imagini conțin etichete și care cameră este de referință, cu următoarele comenzi:

* **Aplicare aliniere matrice** (implicit: activată): deformează fiecare produs procesat (debayering / previzualizare / radianță / reflectanță / indice) în geometria de referință comună a matricei. Dezactivată = export în geometria nativă a senzorului.
* **Decupare la suprapunerea comună** (implicit: activat): decupează exporturile aliniate la regiunea comună tuturor modulelor, astfel încât fiecare bandă să aibă aceeași amprentă. Dezactivat păstrează întreaga suprafață a senzorului (umplere neagră în afara sursei).
* **Reeșantionare**:**Biliniară (uniformă, implicită)**,**Cea mai apropiată (păstrează valorile exacte)**— fără amestecare între pixeli, pentru analize radiometrice stricte — sau**Cubică (cea mai clară)**.

***

## Salvarea și încărcarea setărilor

### Salvarea șablonului de proiect

Creați șabloane reutilizabile pentru fluxuri de lucru consecvente:

1. Configurați toate setările dorite în panoul Setări proiect
2. Derulați până la secțiunea **„Salvare șablon de proiect”** din partea de jos
3. Introduceți un nume descriptiv pentru șablon (de exemplu, „Survey3N\_RGN\_Agriculture”)
4. Faceți clic pe pictograma de salvare

**Avantaje:**

* Aplicați setări identice în mai multe proiecte
* Partajați configurațiile cu membrii echipei
* Mențineți consecvența pentru sondaje repetate

### Încărcarea șablonului într-un proiect nou

Când creați un proiect nou:

1. Selectați **„Proiect nou”** din meniul principal
2. Alegeți un șablon de proiect din selectorul opțional de șabloane
3. Toate setările din șablon sunt aplicate automat

### Director de lucru

Setarea **„Director de lucru”** specifică locația în care sunt create în mod implicit proiectele noi:

* **Locație implicită**: `Documents\MAPIR\Chloros Projects` în folderul „Documente” din Windows (de obicei `C:\Users\[Username]\Documents\MAPIR\Chloros Projects`; se mută odată cu folderul „Documente” dacă acesta este mutat de OneDrive sau de un profil de roaming)
* **Schimbare locație**: Faceți clic pe pictograma de editare și selectați un folder nou
* **Partajat cu CLI**: `chloros-cli` utilizează aceeași setare implicită pentru folderul de proiect
* **Păstrați numele scurt în Windows**: cu excepția cazului în care sunt activate căile lungi în Windows, un folder de proiect cu un nume mai lung de aproximativ 130 de caractere este respins înainte de procesare — consultați [Pornirea procesării](starting-the-processing.md#the-project-folder-is-too-deep-for-this-computer)
* **Când să modificați**:
  * Unitate de rețea pentru colaborarea în echipă
  * O altă unitate cu mai mult spațiu de stocare
  * Structură de foldere organizată pe ani/clienți

***

## Configurarea PPK (Post-Processed Kinematic)

Dacă utilizați înregistratoare DAQ MAPIR cu GPS pentru geolocalizare precisă:

### Cerințe preliminare

* Dispozitiv de înregistrare MAPIR DAQ cu modul GPS (GNSS)
* Fișier jurnal .daq cu intrări pentru pinii de expunere
* Cameră conectată la pinii de expunere ai dispozitivului DAQ în timpul sesiunii de captură

### Pași de configurare

1. Plasați fișierul jurnal .daq în folderul proiectului
2. În Setările proiectului, bifați caseta de selectare **„Aplică corecții PPK”**

3. Setați**„Decalajul fusului orar al senzorului de lumină”**, dacă este necesar (implicit: 0 pentru UTC)
4. Alocați camerele la pinii de expunere:
   * **O singură cameră**: Alocată automat la Pinul 1
   * **Două camere**: Alocați manual fiecare cameră la pinul corespunzător

**Alocarea pinilor de expunere:**

* **Pin de expunere 1**: Selectați modelul camerei din meniul derulant
* **Pin de expunere 2**: Selectați a doua cameră sau „Nu se utilizează”
* Aceeași cameră nu poate fi alocată ambilor pini

{% hint style="warning" %}
**Important**: Pinii de expunere trebuie alocați corect camerelor respective. O alocare incorectă va duce la date de geolocalizare eronate.
{% endhint %}

***

## Scenarii avansate

### Proiecte cu mai multe camere

Atunci când se prelucrează imagini de la mai multe camere MAPIR într-un singur proiect:

1. Chloros detectează automat fiecare model de cameră (atât Survey3, cât și LATTICE)
2. Fiecare cameră primește profilurile de procesare corespunzătoare și are propria sa structură de foldere de ieșire
3. PPK: Alocați manual fiecărei camere dSurvey3

pinul de expunere corect
4. Toate camerele utilizează același format de export și aceiași indici

**Exemple**: configurație cu două camere Survey3W RGN + Survey3N OCN, sau o matrice LATTICE care combină o cameră principală RGB cu module de bandă îngustă

### Studii time-lapse sau cu date multiple

Pentru studii repetate ale aceleiași zone de-a lungul timpului:

1. Creați un șablon cu setările dvs. standard
2. Utilizați o configurație consistentă a țintelor de calibrare la fiecare sesiune
3. Prelucrați fiecare dată ca un proiect separat
4. Utilizați setări identice pentru rezultate comparabile
5. Exportați în același format pentru analiza temporală

### Seturi mari de date

Pentru proiecte cu multe imagini (peste 500):

* Luați în considerare împărțirea în proiecte mai mici, după dată sau zonă
* Utilizați procesarea paralelă Chloros+ pentru rezultate mai rapide
* Luați în considerare utilizarea CLI-ului sau a API-ului pentru automatizarea procesării în lot
* Reglați intervalul minim de recalibrare pentru a reduce timpul de detectare a țintelor

***

## Verificarea setărilor

Înainte de a începe procesarea, verificați aceste setări cheie:

* [ ] Modelul camerei a fost detectat corect în File Browser
* [ ] Corecția vignetării este activată
* [ ] Calibrarea reflectanței este activată
* [ ] Pentru Survey3: cel puțin o imagine țintă de calibrare importată și verificată; pentru LATTICE: este prezentă o țintă și/sau o înregistrare `.daq` cu radiație descendentă
* [ ] Indicii multispectrali doriti au fost adaugati
* [ ] Formatul de export este adecvat pentru fluxul dvs. de lucru
* [ ] Setările PPK sunt configurate (dacă utilizați fișiere .daq cu evenimente de expunere)

***

## Pașii următori

Odată ce setările sunt configurate:

1. **Marcați imaginile țintă de calibrare** – Consultați [Alegerea imaginilor țintă](choosing-target-images.md)
2. **Porniți procesarea** – Consultați [Pornirea procesării](starting-the-processing.md)
3. **Monitorizați progresul** – Consultați [Monitorizarea procesării](monitoring-the-processing.md)

Pentru detalii complete privind toate setările disponibile, consultați documentația de referință [Setări proiect](../project-settings/project-settings.md).
