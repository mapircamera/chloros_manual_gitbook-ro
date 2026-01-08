# Adăugarea fișierelor la un proiect

După ce ați creat sau deschis un proiect în Chloros, următorul pas este să adăugați imaginile multispectrale pentru a începe procesarea. Fila File Browser<img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> facilitează importul imaginilor și gestionarea setului de date.

## Accesarea browserului de fișiere

1. Deschideți sau creați un proiect în Chloros
2. Faceți clic pe pictograma **Browser de fișiere** <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> din bara laterală din stânga
3. Panoul File Browser va afișa lista de fișiere a proiectului dvs.

{% hint style="info" %}
**Tipuri de fișiere acceptate**: Chloros acceptă fișiere imagine RAW+JPG și JPG de la camerele MAPIR Survey3W și Survey3N. Se recomandă numai RAW+JPG.
{% endhint %}

***

## Adăugarea de imagini la proiectul dvs.

Există două modalități principale de a adăuga imagini la proiectul dvs.:

### Metoda 1: Adăugarea de fișiere

Utilizați această opțiune pentru a importa fișiere imagine individuale sau o selecție mică de fișiere.

1. Faceți clic pe butonul **„Adăugare fișiere”** <img src="../.gitbook/assets/image.png" alt="" data-size="line"> din partea de sus a panoului File Browser (Browser fișiere)
2. Navigați la folderul care conține imaginile dvs.
3. Selectați unul sau mai multe fișiere imagine (țineți apăsat **Ctrl** pentru a selecta mai multe fișiere)
4. Faceți clic pe **„Open”** (Deschidere) pentru a importa fișierele selectate

### Metoda 2: Adăugarea unui folder

Utilizați această opțiune pentru a importa toate imaginile dintr-un folder simultan.

1. Faceți clic pe butonul **„Adăugați folder”** <img src="../.gitbook/assets/image (1).png" alt="" data-size="line"> din partea de sus a panoului File Browser (Browser fișiere)
2. Navigați la și selectați folderul care conține imaginile sesiunii de captură
3. Faceți clic pe **„Select Folder”** (Selectați folder) pentru a importa toate imaginile acceptate din acel folder

***

## Înțelegerea tabelului File Browser (Browser fișiere)

Odată importate, imaginile apar într-un tabel cu următoarele coloane:

### File Name (Nume fișier)

* Numele original al fișierului din aparatul foto
* Păstrează convenția de denumire a aparatului foto (de exemplu, IMG\_0001.RAW)

### Marcaj temporal

* Data și ora la care a fost capturată imaginea
* Extras din metadatele EXIF ale imaginii
* Utilizat pentru sincronizarea PPK și detectarea țintei de calibrare

### Modelul aparatului foto

* Configurația camerei și a filtrului detectată automat
* Exemple: Survey3W\_RGN, Survey3N\_OCN, Survey3W\_RGB
* Utilizat pentru aplicarea profilurilor de procesare corecte

### Coloana țintă (casetă de selectare)

* Bifați această casetă pentru imaginile care conțin ținte de calibrare
* Accelerează considerabil detectarea țintelor în timpul procesării
* Consultați [Alegerea imaginilor țintă](choosing-target-images.md) pentru detalii

### Vizualizarea metadatelor imaginii

Dacă faceți clic pe butonul de comutare din colțul din dreapta sus deasupra tabelului, metadatele imaginii selectate sunt afișate în zona grilei de imagini.

<figure><img src="../.gitbook/assets/chloros_grid_meta.gif" alt=""><figcaption></figcaption></figure>

***

## Gestionarea fișierelor din proiectul dvs.

### Eliminarea fișierelor

Pentru a elimina imaginile nedorite din proiectul dvs.:

1. Selectați una sau mai multe imagini din tabelul File Browser (Browser fișiere)
2. Faceți clic pe butonul **„Remove Selected” (Eliminare selectate)** <img src="../.gitbook/assets/image (2).png" alt="" data-size="line"> .
3. Confirmați eliminarea (fișierele nu sunt șterse de pe disc, ci doar eliminate din proiect).

### Sortarea și filtrarea

* **Sortare după coloană**: faceți clic pe orice antet de coloană pentru a sorta imaginile.
* **Sortare după data și ora capturii**: utilă pentru organizarea secvențelor de captură în ordine cronologică.
* **Filtru după modelul camerei**: grupați imaginile după tipul camerei dacă utilizați mai multe camere.

***

## Previzualizare imagine

### Vizualizarea imaginii complete

Faceți clic pe orice miniatură a imaginii din browserul de fișiere pentru a o afișa în zona principală de previzualizare:

1. Imaginea apare în panoul central de previzualizare
2. Utilizați comenzile de zoom pentru a inspecta detaliile imaginii
3. Navigați între imagini utilizând tastele săgeată

### Navigare rapidă

* **Imaginea anterioară**: Faceți clic pe săgeata stânga sau apăsați tasta ←
* **Imaginea următoare**: faceți clic pe săgeata dreapta sau apăsați tasta →
* **Mărire/micșorare**: utilizați rotița mouse-ului sau butoanele de zoom
* **Panoramare**: faceți clic și trageți pe imagine când este mărită

***

## Gestionarea fișierelor duplicate

Chloros detectează și ignoră automat fișierele duplicate:

* Fișierele cu nume identice sunt omise
* Previne procesarea accidentală dublă
* Se afișează un mesaj de avertizare când sunt detectate duplicate

{% hint style="warning" %}
**Important**: Nu redenumiți și nu modificați fișierele imagine originale înainte de importare. Chloros se bazează pe numele fișierelor originale și metadatele pentru procesarea corectă.
{% endhint %}

***

## Seturi de date mixte ale camerei

Dacă proiectul dvs. conține imagini de la mai multe camere MAPIR:

1. Chloros detectează automat fiecare model de cameră
2. Fiecare tip de cameră este procesat cu profilul de calibrare corespunzător
3. Browserul de fișiere afișează modelul camerei în coloana Model cameră
4. Procesarea aplică setările corecte pentru fiecare tip de cameră

**Exemplu de scenariu**: Survey3W RGN + Survey3N OCN configurare cu două camere

***

## Cele mai bune practici

### Organizați înainte de import

* Păstrați imaginile țintă de calibrare în același folder cu imaginile de sondaj
* Păstrați structura originală a folderului de pe camera foto/cardul SD
* Nu amestecați seturi de date din sesiuni diferite într-un singur proiect

### Denumirea fișierelor

* Păstrați numele originale ale fișierelor camerei foto (IMG\_0001.RAW etc.)
* Nu redenumiți fișierele înainte de import
* Numele originale conțin metadate importante.

### Imagini țintă de calibrare

* Includeți întotdeauna 1-2 imagini țintă de calibrare per sesiune.
* Capturați țintele înainte și după sesiunea de captură.
* Plasați țintele în aceleași condiții de iluminare ca și zona de captură.
* Marcați imaginile țintă folosind caseta de selectare Țintă pentru a accelera procesarea.

***

## Probleme frecvente și soluții

### Imaginile nu apar după import

**Cauze posibile:**

* Format de fișier neacceptat (numai RAW+JPG și JPG de la camerele MAPIR)
* Imaginile provin de la camere care nu sunt MAPIR (consultați [Camere acceptate](../supported-cameras.md))
* Fișier corupt sau transfer incomplet de pe cardul SD

**Soluție**: Verificați compatibilitatea formatului fișierului și a modelului camerei foto.

### Modelul camerei foto nu este detectat

**Cauze posibile:**

* Metadate EXIF modificate
* Imagini editate în software extern
* Transfer de fișiere incomplet

**Soluție**: Reimportați fișierele originale, nemodificate, de pe camera foto/cardul SD.

### Lipsa marcajelor temporale

**Cauze posibile:**

* Ceasul camerei nu este setat corect
* Datele EXIF au fost șterse de un software extern

**Soluție**: Verificați dacă setările de timp ale camerei erau corecte în timpul capturii

***

## Pași următori

După importarea fișierelor:

1. **Verificați lista de fișiere** - Asigurați-vă că toate imaginile au fost încărcate corect
2. **Verificați modelele aparatului foto** - Verificați detectarea corectă a aparatului foto
3. **Marcați imaginile țintă** - Consultați [Alegerea imaginilor țintă](choosing-target-images.md)
4. **Reglați setările** - Configurați opțiunile de procesare în [Setări proiect](adjusting-project-settings.md)
5. **Porniți procesarea** - Consultați [Pornirea procesării](starting-the-processing.md)

Pentru informații detaliate despre configurarea proiectului, consultați [Ajustarea setărilor proiectului](adjusting-project-settings.md).
