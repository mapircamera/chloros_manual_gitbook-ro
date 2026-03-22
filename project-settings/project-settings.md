# Setări proiect

Bara laterală „Setări proiect” <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> din Chloros vă permit să configurați toate aspectele legate de procesarea imaginilor, detectarea țintelor de calibrare, calcularea indicilor multispectrali și opțiunile de export pentru proiectul dvs. Aceste setări sunt salvate împreună cu proiectul dvs. și pot fi salvate ca șabloane pentru a fi reutilizate în mai multe proiecte.

## Accesarea setărilor proiectului

Pentru a accesa setările proiectului:

1. Deschideți un proiect în Chloros
2. Faceți clic pe fila **Setări proiect**  <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> din bara laterală din stânga
3. Panoul de setări va afișa toate opțiunile de configurare disponibile, organizate pe categorii

***

## Detectarea țintelor

Aceste setări controlează modul în care Chloros detectează și procesează țintele de calibrare din imaginile dvs.

### Suprafața minimă a eșantionului de calibrare (px)

* **Tip**: Număr
* **Interval**: 0 până la 10.000 de pixeli
* **Implicit**: 25 de pixeli
* **Descriere**: Stabilește suprafața minimă (în pixeli) necesară pentru ca o regiune detectată să fie considerată o probă validă de țintă de calibrare. Valorile mai mici vor detecta ținte mai mici, dar pot crește numărul de rezultate fals pozitive. Valorile mai mari necesită regiuni țintă mai mari și mai clare pentru detectare.
* **Când să reglați**:
  * Măriți valoarea dacă obțineți detectări false pe artefacte mici din imagine
  * Reduceți valoarea dacă țintele de calibrare apar mici în imaginile dvs. și nu sunt detectate

### Gruparea minimă a țintelor (0-100)

* **Tip**: Număr
* **Interval**: 0 până la 100
* **Implicit**: 60
* **Descriere**: Controlează pragul de grupare pentru gruparea regiunilor de culori similare la detectarea țintelor de calibrare. Valorile mai mari necesită gruparea mai multor culori similare, ceea ce duce la o detectare mai conservatoare a țintelor. Valorile mai mici permit o variație mai mare de culori în cadrul unui grup de ținte.
* **Când să reglați**:
  * Măriți dacă țintele de calibrare sunt împărțite în mai multe detectări
  * Micșorați dacă țintele de calibrare cu variații de culoare nu sunt detectate complet

***

## Procesare

Aceste setări controlează modul în care Chloros procesează și calibrează imaginile dvs.

### Corecția vignetării

* **Tip**: Casetă de selectare
* **Implicit**: Activat (bifat)
* **Descriere**: Aplică corecția vignetării pentru a compensa întunecarea lentilelor la marginile imaginilor. Vignetarea este un fenomen optic comun în care colțurile și marginile unei imagini apar mai întunecate decât centrul, din cauza caracteristicilor lentilelor.
* **Când să dezactivați**: Dezactivați doar dacă combinația dvs. de cameră/obiectiv a aplicat deja corecția vignetării sau dacă doriți să corectați manual vignetarea în post-procesare.

### Calibrarea reflectanței / balansul de alb

* **Tip**: Casetă de selectare
* **Implicit**: Activat (bifat)
* **Descriere**: Activează calibrarea automată a reflectanței folosind țintele de calibrare detectate în imaginile dvs. Aceasta normalizează valorile de reflectanță în setul de date și asigură măsurători consistente, indiferent de condițiile de iluminare.
* **Când să dezactivați**: Dezactivați doar dacă doriți să procesați imagini brute, necalibrate sau dacă utilizați un flux de lucru de calibrare diferit.

### Metoda Debayer

* **Tip**: Meniu derulant
* **Opțiuni**:
  * Standard (Rapid, Calitate medie)
  * Sensibil la textură (lent, calitate maximă) \[Chloros+]
* **Implicit**: Standard (rapid, calitate medie)
* **Descriere**: Selectează algoritmul de demosaic utilizat pentru a converti datele brute ale senzorului cu model Bayer în imagini color. Metoda „Standard (rapid, calitate medie)” oferă un echilibru optim între viteza de procesare și calitatea imaginii. Metoda „Sensibil la textură (lent, calitate maximă)” \[Chloros+] utilizează un algoritm de demosaic de înaltă calitate, sensibil la margini, combinat cu un model de denoizare AI/ML care elimină aproape tot zgomotul de demosaic. Modelul Sensibil la textură necesită memorie GPU (VRAM) pentru a rula. Vă recomandăm să îl utilizați atunci când aveți la dispoziție &gt;4 GB VRAM pentru o procesare mai rapidă.
* **Notă**: Metode suplimentare de debayer pot fi adăugate în versiunile viitoare ale Chloros.

### Interval minim de recalibrare

* **Tip**: Număr
* **Interval**: 0 până la 3.600 secunde
* **Implicit**: 0 secunde
* **Descriere**: Setează intervalul de timp minim (în secunde) între utilizarea țintelor de calibrare. Când este setat la 0, Chloros va utiliza fiecare țintă de calibrare detectată. Când este setat la o valoare mai mare, Chloros va utiliza doar țintele de calibrare care sunt separate de cel puțin acest număr de secunde, reducând timpul de procesare pentru seturile de date cu capturi frecvente ale țintelor de calibrare.
* **Când se ajustează**:
  * Setați la 0 pentru o precizie maximă de calibrare atunci când condițiile de iluminare variază
  * Măriți (de exemplu, la 60-300 de secunde) pentru o procesare mai rapidă atunci când iluminarea este constantă și aveți imagini frecvente ale țintelor de calibrare

### Decalajul fusului orar al senzorului de lumină

* **Tip**: Număr
* **Interval**: -12 până la +12 ore
* **Implicit**: 0 ore
* **Descriere**: Specifică decalajul fusului orar (în ore față de UTC) pentru marcajele de timp ale datelor senzorului de lumină. Acest lucru este utilizat la procesarea fișierelor de date PPK (Post-Processed Kinematic) pentru a asigura sincronizarea corectă a timpului între capturile de imagini și datele GPS.
* **Când se ajustează**: Setați această opțiune la decalajul fusului orar local dacă datele PPK utilizează ora locală în loc de UTC. De exemplu:
  * Ora Pacificului: -8 sau -7 (în funcție de ora de vară)
  * Ora Estică: -5 sau -4 (în funcție de ora de vară)
  * Ora Europei Centrale: +1 sau +2 (în funcție de ora de vară)

### Aplicare corecții PPK

* **Tip**: Casetă de selectare
* **Implicit**: Dezactivat (debifat)
* **Descriere**: Activează utilizarea corecțiilor cinetice post-procesate (PPK) de la înregistratoarele DAQ MAPIR care conțin un GPS (GNSS). Când este activată, Chloros va utiliza orice fișiere jurnal .daq care conțin date de expunere în directorul proiectului dvs. și va aplica corecții precise de geolocalizare imaginilor dvs.
* **Cerință**: Fișierul jurnal .daq cu intrări de pin de expunere trebuie să fie prezent în directorul proiectului
* **Când să activați**: Se recomandă să activați întotdeauna corecția PPK dacă aveți intrări de feedback de expunere în fișierul jurnal .daq.

### Pin de expunere 1

* **Tip**: Selecție din meniu derulant
* **Vizibilitate**: Vizibilă numai când opțiunea „Aplică corecții PPK” este activată ȘI sunt disponibile date de expunere pentru Pin 1
* **Opțiuni**:
  * Denumirile modelelor de camere detectate în proiect
  * „Nu utiliza” – Ignoră acest pin de expunere
* **Implicit**: Selectat automat pe baza configurației proiectului
* **Descriere**: Alocă o cameră specifică la Pinul de expunere 1 pentru sincronizarea temporală PPK. Pinul de expunere înregistrează momentul exact în care se declanșează obturatorul camerei, ceea ce este esențial pentru o geolocalizare PPK precisă.
* **Comportament de selectare automată**:
  * O singură cameră + un singur pin: Selectează automat camera
  * O singură cameră + două pini: Pinul 1 este atribuit automat camerei
  * Mai multe camere: Este necesară selectarea manuală

### Pinul de expunere 2

* **Tip**: Selecție din meniu derulant
* **Vizibilitate**: Vizibil numai când opțiunea „Aplică corecții PPK” este activată ȘI sunt disponibile date de expunere pentru Pinul 2
* **Opțiuni**:
  * Numele modelelor de camere detectate în proiect
  * „Nu utiliza” - Ignoră acest pin de expunere
* **Implicit**: Selectat automat pe baza configurației proiectului
* **Descriere**: Alocă o cameră specifică la Pinul de expunere 2 pentru sincronizarea temporală PPK atunci când se utilizează o configurație cu două camere.
* **Comportament de selectare automată**:
  * O singură cameră + un singur pin: Pinul 2 este setat automat la „Nu utilizați”
  * O singură cameră + două pini: Pinul 2 este setat automat la „Nu utilizați”
  * Mai multe camere: Este necesară selectarea manuală
* **Notă**: Aceeași cameră nu poate fi atribuită simultan atât pinului 1, cât și pinului 2.***

## Index

Aceste setări vă permit să configurați indici multispectrali pentru analiză și vizualizare.

### Adăugare indice

* **Tip**: Panou special de configurare a indicilor
* **Descriere**: Deschide un panou interactiv în care puteți selecta și configura indici multispectrali de vegetație (NDVI, NDRE, EVI etc.) pentru a fi calculați în timpul procesării imaginii. Puteți adăuga mai mulți indici, fiecare cu propriile setări de vizualizare.
* **Indici disponibili**: Sistemul include peste 30 de indici multispectrali predefiniti, printre care:
  * NDVI (Indicele de vegetație diferențial normalizat)
  * NDRE (Diferența normalizată RedEdge)
  * EVI (Indicele de vegetație îmbunătățit)
  * GNDVI, SAVI, OSAVI, MSAVI2
  * Și multe altele (consultați [Formule de indici multispectrali](multispectral-index-formulas.md) pentru lista completă)
* **Caracteristici**:
  * Selectați din formule de indici predefinite
  * Configurați gradientele de culori pentru vizualizare (LUT - tabele de consultare)
  * Setați valori prag pentru analiză
  * Creați formule de indici personalizate

### Formule personalizate (Caracteristică Chloros+)

* **Tip**: Matrice de definiții de formule personalizate
* **Descriere**: Vă permite să creați și să salvați formule de indici multispectrali personalizate folosind operații matematice pe benzi. Formulele personalizate sunt salvate împreună cu setările proiectului și pot fi utilizate la fel ca indicii încorporați.
* **Cum se creează**:
  1. În panoul de configurare Index, căutați opțiunea de formulă personalizată
  2. Definiți formula utilizând identificatori de bandă (de exemplu, NIR, Red, Green, Blue)
  3. Salvați formula cu un nume descriptiv
* **Sintaxa formulei**: Sunt acceptate operațiile matematice standard, inclusiv:
  * Aritmetică: `+`, `-`, `*`, `/`
  * Paranteze pentru ordinea operațiilor
  * Referințe de bandă: NIR, Red, Green, Blue, RedEdge, Cyan, Orange, NIR1, NIR2

***

## Export

Aceste setări controlează formatul și calitatea imaginilor procesate exportate.

### Format imagine calibrată

* **Tip**: Selecție din meniu derulant
* **Opțiuni**:
  * **TIFF (16 biți)** - Format TIFF necomprimat de 16 biți
  * **TIFF (32 de biți, procent)** - TIFF de 32 de biți cu virgulă mobilă, cu valori de reflectanță exprimate în procente
  * **PNG (8 biți)** - Format PNG comprimat pe 8 biți
  * **JPG (8 biți)** - Format JPEG comprimat pe 8 biți
* **Implicit**: TIFF (16 biți)
* **Descriere**: Selectează formatul de fișier pentru salvarea imaginilor procesate și calibrate.
* **Recomandări privind formatul**:
  * **TIFF (16 biți)**: Recomandat pentru analize științifice și fluxuri de lucru profesionale. Păstrează calitatea maximă a datelor fără artefacte de compresie. Cel mai potrivit pentru analiza multispectrală și procesarea ulterioară în software-ul GIS.
  * **TIFF (32 de biți, procent)**: Cel mai potrivit pentru fluxurile de lucru care necesită valori de reflectanță exprimate în procente (0-100%). Oferă precizie maximă pentru măsurătorile radiometrice.
  * **PNG (8 biți)**: Bun pentru vizualizare web și vizualizare generală. Dimensiuni mai mici ale fișierelor cu compresie fără pierderi, dar cu o gamă dinamică redusă.
  * **JPG (8 biți)**: Cele mai mici dimensiuni de fișiere, ideal doar pentru previzualizări și afișare web. Utilizează compresie cu pierderi, care nu este potrivită pentru analize științifice.***

## Salvare șablon de proiect

Această funcție vă permite să salvați setările curente ale proiectului ca șablon reutilizabil.

* **Tip**: Introducere text + buton Salvare
* **Descriere**: Introduceți un nume descriptiv pentru șablonul de setări și faceți clic pe pictograma de salvare. Șablonul va stoca toate setările curente ale proiectului (detectarea țintei, opțiunile de procesare, indicii și formatul de export) pentru a fi reutilizate cu ușurință în proiectele viitoare.
* **Cazuri de utilizare**:
  * Creați șabloane pentru diferite sisteme de camere (RGB, multispectral, NIR)
  * Salvați configurații standard pentru tipuri specifice de culturi sau fluxuri de lucru de analiză
  * Partajați setări consistente în cadrul unei echipe
* **Mod de utilizare**:
  1. Configurați toate setările dorite pentru proiect
  2. Introduceți un nume de șablon (de ex., „RedEdge Survey3 NDVI Standard”)
  3. Faceți clic pe pictograma de salvare
  4. Șablonul poate fi acum încărcat la crearea de proiecte noi

***

## Salvați folderul proiectului

Această setare specifică locația în care sunt salvate implicit proiectele noi.

* **Tip**: Afișare cale director + buton Editare
* **Implicit (Windows)**: `C:\Users\[Username]\Chloros Projects`
* **Implicit (Linux)**: `~/.local/share/chloros/projects`
* **Descriere**: Afișează directorul implicit curent în care sunt create noile proiecte Chloros. Faceți clic pe pictograma de editare pentru a selecta un alt director.
* **Când să modificați**:
  * Setați o unitate de rețea pentru colaborarea în echipă
  * Schimbați într-o unitate cu mai mult spațiu de stocare pentru seturi de date mari
  * Organizați proiectele pe ani, clienți sau tipuri de proiect în dosare diferite
* **Notă**: Modificarea acestei setări afectează doar proiectele NOI. Proiectele existente rămân în locațiile lor originale.***

## Persistența setărilor

Toate setările proiectului sunt salvate automat împreună cu fișierul proiectului (format de proiect `.mapir`). Când redeschideți un proiect, toate setările sunt restaurate exact așa cum le-ați lăsat.

### Ierarhia setărilor

Setările sunt aplicate în următoarea ordine:

1. **Setări implicite de sistem** - Setări implicite încorporate definite de Chloros
2. **Setări șablon** - Dacă încărcați un șablon la crearea unui proiect
3. **Setări proiect salvate** - Setări salvate împreună cu fișierul de proiect
4. **Ajustări manuale** - Orice modificări pe care le faceți în timpul sesiunii curente

### Setări și procesarea imaginilor

Majoritatea modificărilor de setări (în special în categoriile Procesare și Export) vor declanșa reprocesarea imaginilor pentru a reflecta noile setări. Cu toate acestea, unele setări sunt „doar pentru export” și nu necesită reprocesare imediată:

* Salvare șablon de proiect
* Director de lucru
* Format de imagine calibrat (se aplică la export)

***

## Cele mai bune practici

1. **Începeți cu setările implicite**: Setările implicite funcționează bine pentru majoritatea sistemelor de camere MAPIR și pentru fluxurile de lucru tipice.
2. **Creați șabloane**: Odată ce ați optimizat setările pentru un flux de lucru sau o cameră specifică, salvați-le ca șablon pentru a asigura consecvența între proiecte.
3. **Testați înainte de procesarea completă**: Când experimentați cu setări noi, testați pe un subset mic de imagini înainte de a procesa întregul set de date.
4. **Documentați-vă setările**: Folosiți nume descriptive pentru șabloane care indică sistemul de camere, tipul de procesare și utilizarea prevăzută (de exemplu, „Survey3\_RGB\_NDVI\_Agriculture”).
5. **Selectarea formatului de export**: Alegeți formatul de export în funcție de utilizarea finală:
   * Analiză științifică → TIFF (16 biți sau 32 biți)
   * Prelucrare GIS → TIFF (16 biți)
   * Vizualizare rapidă → PNG (8 biți)
   * Partajare web → JPG (8 biți)

***

Pentru mai multe informații despre indicii multispectrali din Chloros, consultați pagina [Formule de indici multispectrali](multispectral-index-formulas.md).
