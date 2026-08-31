# Alegerea imaginilor țintă

Marcarea imaginilor care conțin ținte de calibrare indică programului „Chloros” exact unde să le caute. Atunci când cel puțin o imagine este bifată în coloana „Target”, „Chloros” scanează **numai imaginile bifate** — astfel, marcarea țintelor reprezintă atât o modalitate de a accelera procesarea, cât și o modalitate de a evita confundarea imaginilor de supraveghere cu o țintă.

<figure><img src="../.gitbook/assets/image (40).png" alt=""><figcaption></figcaption></figure>

## De ce să marcați imaginile țintă?

### Marcarea controlează procesul de scanare

Când bifați coloana „Țintă” pentru anumite imagini:

* Chloros scanează doar imaginile bifate în căutarea țintelor
* Detectarea țintelor se finalizează mult mai rapid
* Imaginile de supraveghere nu pot genera detectări eronate ale țintelor

Dacă **nu** sunt bifate imagini, Chloros revine la scanarea fiecărei imagini din proiect:

* Algoritmii de detectare a țintelor rulează pe fiecare imagine
* Sute sau mii de imagini sunt verificate inutil
* Procesarea durează semnificativ mai mult, în special pentru seturi de date mari

{% hint style="success" %}
**Îmbunătățirea vitezei**: Marcarea a 2-3 imagini țintă dintr-un set de date de 500 de imagini poate reduce timpul de detectare a țintelor de la peste 30 de minute la sub 1 minut.
{% endhint %}

***

## Cum se marchează imaginile țintă

### Pasul 1: Identificați imaginile țintă

Parcurgeți imaginile importate în File Browser și identificați imaginile care conțin ținte de calibrare.

**Situații frecvente:*** **Țintă pre-captură**: Capturată înainte de începerea sesiunii
* **Țintă post-captură**: capturată după finalizarea sesiunii
* **Ținte în teren**: ținte amplasate în zona de captură
* **Ținte multiple**: 2-3 imagini cu ținte pe sesiune (recomandat)

### Pasul 2: Verificați coloana „Target” d<img src="../.gitbook/assets/image (33).png" alt="" data-size="original">

Pentru fiecare imagine care conține o țintă de calibrare:

1. Localizați imaginea în tabelul din File Browser
2. Găsiți coloana **Target** (coloana din extrema dreaptă)
3. Bifați caseta de selectare din coloana „Target” pentru imaginea respectivă
4. Repetați procedura pentru toate imaginile care conțin ținte

### Pasul 3: Verificați selecția

Înainte de procesare, verificați încă o dată:

* [ ] Toate imaginile cu ținte de calibrare sunt bifate
* [ ] Nicio imagine care nu conține ținte nu este bifată accidental
* [ ] Țintele sunt clar vizibile în imaginile bifate

***

## LATTICE: Țintele sunt opționale atunci când un DAQ înregistrează

Pentru camerele multispectrale LATTICE, o țintă de calibrare în cadru este **una dintre cele două** referințe de reflectanță posibile:

* **Țintă în cadru**: atunci când o imagine cu țintă marcată trece de criteriile de calitate (QA) ale sistemului de calitate automată (Chloros), ținta devine**referința absolută de reflectanță** pentru imaginile din jurul ei.
* **Radiație descendentă DAQ**: atunci când nu este prezentă nicio țintă (sau verificarea calității eșuează), Chloros calculează reflectanța pe baza iradianței descendente a senzorului de lumină al DAQ (ρ = π·L/E). Dacă o înregistrare `.daq` sau DAQ-M `.csv` acoperă capturile dvs., veți obține reflectanța calibrată**fără nicio imagine țintă**.

Acest comportament automat este setarea implicită. În „CLI” / „SDK” aceasta corespunde opțiunii `--reflectance-source auto`; puteți forța, de asemenea, `target` (strict — fără substituire DAQ) sau `daq` (autoritate DAQ). Consultați [Referința CLI](../reference/cli-reference.md#per-product-export-toggles-lattice-multispectral).

**Geometrii țintă LATTICE**: pe lângă detectarea clasică a panourilor utilizată pentru Survey3, procesarea LATTICE acceptă**ținte marcate cu ArUco**,**ținte cu ROI fix**și**ținte tip bandă**, configurate pentru fiecare proiect. Scanările**măsurate** ale reflectanței țintei pe unitate pot fi furnizate pe baza numărului de serie (CLI: `--target-reflectance-dir`, câte un `<serial>.csv` pentru fiecare unitate țintă), cu spectrele nominale T3/T4P ca soluție de rezervă.

{% hint style="info" %}
**Modulul F988**: reflectanța F988 este calibrată folosind un panou de reflectanță în scenă: banda se află dincolo de intervalul calibrat al senzorului de lumină DAQ, așChloros-ul utilizează cea mai recentă captură a panoului și o menține între observările panoului. Dacă un modul F988 este procesat exclusiv prin DAQ, Chloros respinge reflectanța bazată pe DAQ pentru banda respectivă (motiv de omitere `dls-uncalibrated-band-988`) — fluxul de lucru cu panoul este calea acceptată.
{% endhint %}

***

## Cele mai bune practici pentru imaginile țintă

### Linii directoare privind captarea țintei

**Sincronizare:**

* Capturați imaginile țintă imediat înainte și pe tot parcursul sesiunii de captare
* În aceleași condiții de iluminare ca și senzorul de lumină DAQ
* În mod ideal, capturați imagini ale țintei cât mai des posibil pentru a obține cele mai bune rezultate. În caz contrar, datele senzorului de lumină vor fi utilizate pentru a ajusta calibrarea în timp.

**Poziția camerei:**

* Țineți camera deasupra țintei astfel încât aceasta să fie centrată și să ocupe aproximativ 40-60% din centrul imaginii.
* Mențineți camera paralelă/nadir față de suprafața țintei

**Iluminare:**

* Aceeași iluminare ambientală ca cea a senzorului de lumină DAQ
* Evitați umbrele pe suprafețele țintei
* Nu blocați sursa de lumină cu corpul, vehiculul sau vegetația
* Condițiile de cer înnorat oferă cele mai consistente rezultate

**Starea țintei:**

* Mențineți panourile țintei curate și uscate
* Toate panourile țintei (de exemplu, toate cele 4 ale unui T4) trebuie să fie clar vizibile și fără obstacole
* Dacă este posibil, orientați ținta perpendicular/nadir față de sursa de lumină

### Câte imagini ale țintei?

**Minim:**1 imagine a țintei pe sesiune.**Recomandat:** 3-5 imagini ale țintei pe sesiune.**Program recomandat:**

* 3-5 imagini capturate la scurt timp după ce senzorul de lumină începe înregistrarea
* Rotiți camera între capturi pentru rezultate optime
* Opțional: periodic, în timpul sesiunii, dacă condițiile de iluminare se schimbă constant

***

## Lucrul cu mai multe camere

### Configurații cu două camere

Dacă utilizați simultan două camere MAPIR (de exemplu, Survey3W RGN + Survey3N OCN):

1. Capturați imagini țintă cu **ambele camere** în același timp
2. Utilizați **aceeași țintă fizică** pentru ambele camere
3. Marcați imaginile țintelor pentru **ambele tipuri de camere** în File Browser
4. Chloros va utiliza țintele corespunzătoare pentru calibrarea fiecărei camere

### Coloana „Model cameră”

Coloana **„Model cameră”** ajută la identificarea sursei imaginilor:

* Survey3W\_RGN
* Survey3N\_OCN
* LATT-M3M-L41-F550
* LATT-M3C-L87-FRGN
* etc.

Utilizați această coloană pentru a verifica dacă ați marcat țintele pentru fiecare tip de cameră din proiectul dvs.

***

## Setări de detectare a țintelor

### Reglarea sensibilității de detectare

Dacă Chloros nu detectează corect țintele, reglați aceste setări în [Setări proiect](adjusting-project-settings.md):**Suprafață minimă a eșantionului de calibrare (px):*** **Implicit**: 25 de pixeli
* **Măriți** valoarea dacă apar detectări eronate la artefacte mici
* **Reduceți** valoarea dacă țintele nu sunt detectate**Gruparea minimă a țintelor (0-100):*** **Implicit**: 60
* **Măriți** dacă țintele sunt împărțite în mai multe detectări
* **Reduceți** dacă țintele cu variații de culoare nu sunt detectate complet

{% hint style="info" %}
**Sfat pentru CLI**: `chloros-cli process` acceptă aceleași parametri (`--min-target-size`, `--target-clustering`), iar indicatorul său `--target`/`--targets` marchează un întreg folder de intrare ca fiind destinat exclusiv panoului de ținte. Consultați [Referința CLI](../reference/cli-reference.md).
{% endhint %}

***

## Probleme frecvente legate de imaginile țintă

### Problemă: Nu s-au detectat ținte

**Cauze posibile:**

* Imaginile țintă nu sunt marcate în File Browser
* Ținta este prea mică în cadru (&lt; 30% din imagine)
* Iluminare deficitară (umbre, strălucire)
* Setări de detectare a țintelor prea stricte

**Soluții:**

1. Verificați dacă coloana „Țintă” este bifată pentru imaginile corecte
2. Verificați calitatea imaginii țintei în previzualizare
3. Recapturați țintele dacă calitatea este slabă
4. Reglați setările de detectare a țintelor, dacă este necesar

### Problemă: Detectări false ale țintelor

**Cauze posibile:**

* Clădiri, vehicule sau acoperirea solului de culoare albă confundate cu ținte
* Pete luminoase în vegetație
* Sensibilitate de detectare prea scăzută

**Soluții:**

1. Marcați numai imaginile cu ținte reale — numai imaginile bifate sunt scanate
2. Măriți suprafața minimă a eșantionului de calibrare
3. Măriți valoarea minimă de grupare a țintelor
4. Asigurați-vă că imaginile cu ținte prezintă numai ținta (zgomot de fundal minim)

***

## Listă de verificare

Înainte de a începe procesarea, verificați selecția imaginilor țintă:

* [ ] Cel puțin o imagine țintă marcată pe sesiune (sau, pentru LATTICE, o înregistrare `.daq`/`.csv` care acoperă sesiunea)
* [ ] Casetele de selectare din coloana „Țintă” sunt bifate pentru toate imaginile țintă
* [ ] Imaginile țintă au fost capturate în același interval de timp cu sondajul
* [ ] Țintele sunt clar vizibile în previzualizare atunci când sunt selectate
* [ ] Toate panourile de calibrare sunt vizibile în fiecare imagine țintă
* [ ] Nu există umbre sau obstacole pe ținte
* [ ] Pentru configurația cu două camere: țintele sunt marcate pentru ambele tipuri de camere

***

## Prelucrare fără ținte

### LATTICE: Cu o înregistrare DAQ

Dacă un senzor de lumină DAQ a înregistrat iradianța descendentă în timpul capturilor LATTICE, nu este necesară nicio țintă:

1. Importați fișierul `.daq` (sau DAQ-M `.csv`) împreună cu imaginile
2. Lăsați coloana „Țintă” debifată
3. Reflectanța este calculată automat pe baza referinței de radiație descendentă înregistrate de DAQ
4. Radianța nu necesită niciodată o țintă sau un DAQ — aceasta provine exclusiv din calibrarea radiometrică din fabrică a camerei

### Prelucrare fără nicio referință

Puteți, de asemenea, să efectuați prelucrarea fără ținte și fără un DAQ:

1. Lăsați toate casetele de selectare din coloana „Țintă” debifate
2. **Dezactivați** opțiunea „Calibrare reflectanță / balans de alb” din Setările proiectului — detectarea țintelor va fi atunci omisă în totalitate
3. Corecția de vignetare va fi totuși aplicată
4. Rezultatul nu va fi calibrat pentru reflectanța absolută (LATTICE multispectral exportă în continuare produse debayered, de previzualizare și de radianță)

{% hint style="warning" %}
**Nu este recomandat pentru activități științifice Survey3**: Fără calibrarea reflectanței, valorile pixelilor dSurvey3reprezintă doar luminozitatea relativă, nu măsurători științifice ale reflectanței. Utilizați ținte de calibrare (sau, pentru LATTICE, un senzor de lumină DAQ) pentru rezultate precise și repetabile.
{% endhint %}

***

## Pașii următori

După ce ați marcat imaginile țintă:

1. **Verificați setările** – Consultați [Reglarea setărilor proiectului](adjusting-project-settings.md)
2. **Începeți procesarea** - Consultați [Începerea procesării](starting-the-processing.md)
3. **Monitorizați progresul** — Consultați [Monitorizarea procesării](monitoring-the-processing.md)

Pentru mai multe informații despre țintele de calibrare în sine, consultați [Ținte de calibrare](../calibration-targets.md).
