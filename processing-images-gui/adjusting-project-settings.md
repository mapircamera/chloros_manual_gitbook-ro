# Configurarea setărilor proiectului

Înainte de a procesa imaginile, este important să configurați setările proiectului astfel încât să corespundă cerințelor fluxului dvs. de lucru. Panoul „Setări proiect” <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> oferă un control complet asupra calibrării, opțiunilor de procesare, indicilor multispectrali și formatelor de export.

## Accesarea setărilor proiectului

1. Deschideți proiectul în Chloros
2. Faceți clic pe pictograma **Setări proiect** <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> din bara laterală din stânga
3. Panoul Setări proiect afișează toate opțiunile de configurare

{% hint style="info" %}
**Setările sunt salvate automat** împreună cu proiectul. Când redeschideți un proiect, toate setările sunt restaurate.
{% endhint %}

***

## Configurare rapidă pentru fluxuri de lucru obișnuite

### Setări implicite (recomandate pentru majoritatea utilizatorilor)

Pentru fluxurile de lucru tipice ale camerelor MAPIR și Survey3, setările implicite funcționează bine:

* ✅ **Corecție vignetă**: Activată
* ✅ **Calibrare reflectanță**: Activată (necesită imagini ale țintelor MAPIR)
* ✅ **Metoda Debayer**: Standard (Rapidă, Calitate medie)
* ✅ **Format de export**: TIFF (16 biți)

Pur și simplu importați imaginile și începeți procesarea cu aceste setări implicite.

***

## Prezentare generală a setărilor proiectului

Panoul Setări proiect este organizat în mai multe categorii. Mai jos găsiți un rezumat al fiecărei secțiuni. Pentru documentația completă, consultați [Setări proiect](../project-settings/project-settings.md).

### Detectarea țintelor

Controlează modul în care Chloros identifică țintele de calibrare din imaginile dvs.

**Setări cheie:*** **Suprafață minimă a eșantionului de calibrare**: Prag de dimensiune pentru detectarea țintelor (implicit: 25 pixeli)
* **Grupare minimă a țintelor**: Prag de similitudine pentru gruparea regiunilor țintă (implicit: 60)**Când să ajustați:**

* Măriți suprafața eșantionului dacă obțineți detectări false
* Reduceți-o dacă țintele nu sunt detectate
* Reglați gruparea dacă țintele sunt împărțite în mai multe detectări

### Procesare

Opțiuni principale de procesare a imaginii și de calibrare.

**Setări cheie:*** **Corecție vignetă**: Compensează întunecarea lentilelor la margini ✅ Recomandat
* **Calibrare reflectanță**: Normalizează valorile folosind ținte de calibrare ✅ Recomandat
* **Metoda Debayer**: Algoritm pentru conversia RAW în multispectral cu 3 canale
* **Interval minim de recalibrare**: Timpul dintre utilizarea țintelor de calibrare (0 = utilizează toate)**Setări avansate:*** **Decalaj fus orar senzor de lumină**: Pentru sincronizarea orară PPK (implicit: 0)
* **Aplicare corecții PPK**: Utilizează datele GPS/pin de expunere din fișierele .daq
* **Pin de expunere 1/2**: Alocă camerele la pinii de expunere pentru configurații cu două camere

### Metoda de debayering

În prezent, oferim 2 metode de debayering în Chloros:

#### Standard (Rapid, Calitate medie)

Debayeringul standard procesează rapid, dar prezintă zgomot de culoare, ceea ce duce la imagini mai puțin precise și mai zgomotoase.

#### Texture Aware (lent, calitate maximă) \[Numai Chloros+]

Texture Aware utilizează un debayering de înaltă calitate, sensibil la margini, combinat cu un model de denoising AI/ML care elimină aproape tot zgomotul de debayering. Modelul Texture Aware necesită memorie GPU (VRAM) pentru a rula. Vă recomandăm să îl utilizați atunci când aveți &gt;4 GB VRAM disponibil pentru o procesare mai rapidă.

### Index (Indici multispectrali)

Configurați ce indici de vegetație să calculați și să exportați.

**Cum se adaugă indici:**

1. Faceți clic pe butonul**„Adăugați index”**

2. Selectați un indice din meniul derulant (NDVI, NDRE, GNDVI etc.)
3. Configurați setările de vizualizare (culori LUT, intervale de valori)
4. Adăugați mai mulți indici, după cum este necesar

**Indici populari:*** **NDVI**: Starea generală de sănătate a vegetației (cel mai frecvent)
* **NDRE**: Detectarea timpurie a stresului împreună cu RedEdge
* **GNDVI**: Sensibil la concentrația de clorofilă
* **OSAVI**: Funcționează bine cu solul vizibil
* **EVI**: Regiuni cu indice ridicat al suprafeței foliare (LAI)**Formule personalizate (numai Chloros+):**

* Creați formule personalizate de indici multispectrali
* Utilizați operații matematice pe benzi cu toate canalele imaginii
* Salvați formulele personalizate pentru reutilizare

Pentru toți indicii și formulele disponibile, consultați [Formule de indici multispectrali](../project-settings/multispectral-index-formulas.md).

### Export

Controlează formatul și calitatea fișierului de ieșire.

**Formate disponibile:*** **TIFF (16 biți)**: Recomandat pentru GIS și analize științifice (interval 0-65.535)
* **TIFF (32 biți, procent)**: Valori de reflectanță în virgulă mobilă (interval 0,0-1,0)
* **PNG (8 biți)**: Compresie fără pierderi pentru vizualizare (interval 0-255)
* **JPG (8 biți)**: Fișiere de dimensiuni reduse, compresie cu pierderi (interval 0-255)***

## Salvarea și încărcarea setărilor

### Salvarea șablonului de proiect

Creați șabloane reutilizabile pentru fluxuri de lucru consecvente:

1. Configurați toate setările dorite în panoul Setări proiect
2. Derulați până la secțiunea **„Salvare șablon proiect”** din partea de jos
3. Introduceți un nume descriptiv pentru șablon (de ex., „Survey3N\_RGN\_Agriculture”)
4. Faceți clic pe pictograma de salvare

**Avantaje:**

* Aplicați setări identice în mai multe proiecte
* Partajați configurațiile cu membrii echipei
* Mențineți consecvența pentru sondaje repetate

### Încărcarea șablonului într-un proiect nou

Când creați un proiect nou:

1. Selectați **„Proiect nou”** din meniul principal
2. Alegeți opțiunea **„Încărcare din șablon”**

3. Selectați șablonul salvat
4. Toate setările sunt aplicate automat

### Director de lucru

Setarea **„Salvare folder proiect”** specifică locația în care sunt create implicit proiectele noi:

* **Locație implicită**: `C:\Users\[Username]\Chloros Projects`
* **Modificare locație**: Faceți clic pe pictograma de editare și selectați un folder nou
* **Când să schimbați**:
  * Unitate de rețea pentru colaborare în echipă
  * Unitate diferită cu mai mult spațiu de stocare
  * Structură de foldere organizată pe ani/clienți

***

## Configurare PPK (Post-Processed Kinematic)

Dacă utilizați înregistratoare DAQ MAPIR cu GPS pentru geolocalizare precisă:

### Cerințe preliminare

* MAPIR DAQ cu modul GPS (GNSS)
* Fișier jurnal .daq cu intrări pentru pinii de expunere
* Cameră conectată la pinii de expunere DAQ în timpul sesiunii de captură

### Pași de configurare

1. Plasați fișierul jurnal .daq în folderul proiectului
2. În Setări proiect, bifați caseta **„Aplică corecții PPK”**

3. Setați**„Decalajul fusului orar al senzorului de lumină”** dacă este necesar (implicit: 0 pentru UTC)
4. Alocați camerele la pinii de expunere:
   * **Cameră unică**: Alocată automat la Pinul 1
   * **Camere duble**: Alocați manual fiecare cameră la pinul corespunzător**Alocarea pinilor de expunere:*** **Pin de expunere 1**: Selectați modelul camerei din meniul derulant
* **Pin de expunere 2**: Selectați a doua cameră sau „Nu utilizați”
* Aceeași cameră nu poate fi alocată ambelor pini

{% hint style="warning" %}
**Important**: Pinii de expunere trebuie alocați corect camerelor respective. O alocare incorectă va duce la date de geolocalizare eronate.
{% endhint %}

***

## Scenarii avansate

### Proiecte cu mai multe camere

Când se procesează imagini de la mai multe camere MAPIR într-un singur proiect:

1. Chloros detectează automat fiecare model de cameră
2. Fiecare cameră primește profilul de procesare corespunzător
3. PPK: Alocați manual fiecare cameră la pinul de expunere corect
4. Toate camerele utilizează același format de export și aceiași indici

**Exemplu**: Survey3W RGN + Survey3N OCN platformă cu două camere

### Studii time-lapse sau cu date multiple

Pentru studii repetate ale aceleiași zone de-a lungul timpului:

1. Creați un șablon cu setările dvs. standard
2. Utilizați o configurare consistentă a țintei de calibrare la fiecare sesiune
3. Prelucrați fiecare dată ca un proiect separat
4. Utilizați setări identice pentru rezultate comparabile
5. Exportați în același format pentru analiza temporală

### Seturi de date mari

Pentru proiecte cu multe imagini (peste 500):

* Luați în considerare împărțirea în proiecte mai mici, după dată sau zonă
* Utilizați procesarea paralelă Chloros+ pentru rezultate mai rapide
* Luați în considerare CLI sau API pentru automatizarea procesării în lot
* Reglați intervalul minim de recalibrare pentru a reduce timpul de detectare a țintelor

***

## Verificarea setărilor

Înainte de a începe procesarea, verificați aceste setări cheie:

* [ ] Modelul camerei detectat corect în File Browser
* [ ] Corecția vignetării activată
* [ ] Calibrarea reflectanței activată
* [ ] Cel puțin o imagine țintă de calibrare importată
* [ ] Indicii multispectrali doriți adăugați
* [ ] Format de export adecvat pentru fluxul dvs. de lucru
* [ ] Setări PPK configurate (dacă utilizați .daq cu evenimente de expunere)

***

## Pași următori

Odată ce setările sunt configurate:

1. **Marcați imaginile țintă de calibrare** - Consultați [Alegerea imaginilor țintă](choosing-target-images.md)
2. **Începeți procesarea** - Consultați [Începerea procesării](starting-the-processing.md)
3. **Monitorizați progresul** - Consultați [Monitorizarea procesării](monitoring-the-processing.md)

Pentru detalii complete despre toate setările disponibile, consultați documentația de referință [Setări proiect](../project-settings/project-settings.md).
