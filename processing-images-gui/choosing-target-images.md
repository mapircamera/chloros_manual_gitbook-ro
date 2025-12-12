# Alegerea imaginilor țintă

Marcarea imaginilor care conțin ținte de calibrare este un pas crucial care accelerează semnificativ procesul de prelucrare Chloros. Prin preselectarea imaginilor țintă, eliminați necesitatea ca Chloros să scaneze fiecare imagine din setul de date pentru a găsi ținte de calibrare.

## De ce să marcați imaginile țintă?

### Viteza de procesare

Fără marcarea imaginilor țintă, Chloros trebuie să:

* Scaneze fiecare imagine din proiectul dvs.
* Ruleze algoritmi de detectare a țintelor pe fiecare imagine
* Verifice inutil sute sau mii de imagini

**Rezultat**: Procesarea poate dura mult mai mult, în special pentru seturi de date mari.

### Cu imagini țintă marcate

Când bifați coloana Țintă pentru imagini specifice:

* Chloros scanează numai imaginile bifate pentru ținte
* Detectarea țintelor se finalizează mult mai repede
* Timpul total de procesare este redus considerabil

{% hint style=&quot;success&quot; %}
**Îmbunătățirea vitezei**: Marcarea a 2-3 imagini țintă într-un set de date de 500 de imagini poate reduce timpul de detectare a țintelor de la peste 30 de minute la mai puțin de 1 minut.
{% endhint %}

***

## Cum să marcați imaginile țintă

### Pasul 1: Identificați imaginile țintă

Răsfoiți imaginile importate în File Browser și identificați imaginile care conțin ținte de calibrare.

**Scenarii comune:**

* **Țintă pre-captură**: capturată înainte de începerea sesiunii
* **Țintă post-captură**: capturată după finalizarea sesiunii
* **Ținte în câmp**: ținte plasate în zona de captură
* **Ținte multiple**: 2-3 imagini țintă per sesiune (recomandat)

### Pasul 2: Verificați coloana Țintă

Pentru fiecare imagine care conține o țintă de calibrare:

1. Localizați imaginea în tabelul File Browser (Browser fișiere)
2. Găsiți coloana **Țintă** (coloana din extrema dreaptă)
3. Bifați caseta de selectare din coloana Țintă pentru imaginea respectivă
4. Repetați pentru toate imaginile care conțin ținte

### Pasul 3: Verificați selecția

Înainte de procesare, verificați din nou:

* [ ] Toate imaginile cu ținte de calibrare sunt bifate
* [ ] Nicio imagine care nu este țintă nu este bifată accidental
* [ ] Țintele sunt clar vizibile în imaginile bifate

***

## Cele mai bune practici pentru imaginile țintă

### Instrucțiuni de captare a țintelor

**Momentul:**

* Captarea imaginilor țintă imediat înainte și pe parcursul sesiunii de captare
* În aceleași condiții de iluminare ca senzorul de lumină DAQ
* În mod ideal, capturați imagini țintă cât mai des posibil pentru a obține cele mai bune rezultate. În caz contrar, datele senzorului de lumină vor fi utilizate pentru a ajusta calibrarea în timp.

**Poziția camerei:**

* Țineți camera deasupra țintei, astfel încât aceasta să fie centrată și să ocupe aproximativ 40-60% din centrul imaginii.
* Mențineți camera paralelă/nadir cu suprafața țintei

**Iluminare:**

* Aceeași iluminare ambientală ca senzorul de lumină DAQ
* Evitați umbrele pe suprafețele țintei
* Nu blocați sursa de lumină cu corpul, vehiculul sau vegetația
* Condițiile de cer înnorat oferă cele mai consistente rezultate

**Condiția țintei:**

* Mențineți panourile țintei curate și uscate
* Toate cele 4 panouri trebuie să fie clar vizibile și neobstrucționate
* Ținte perpendiculare/nadir față de sursa de lumină, dacă este posibil

### Câte imagini țintă?

**Minim:** 1 imagine țintă per sesiune. **Recomandat:** 3-5 imagini țintă per sesiune.

**Programul de bune practici:**

* 3-5 imagini capturate la scurt timp după ce senzorul de lumină începe înregistrarea
* Rotiți camera între capturi pentru rezultate optime
* Opțional: periodic, la jumătatea sesiunii, dacă condițiile de iluminare se schimbă constant

***

## Lucrul cu mai multe camere

### Configurații cu două camere

Dacă utilizați simultan două camere MAPIR (de exemplu, Survey3W RGN + Survey3N OCN):

1. Capturați imaginile țintă cu **ambele camere** în același timp.
2. Utilizați **aceeași țintă fizică** pentru ambele camere.
3. Marcați imaginile țintă pentru **ambele tipuri de camere** în browserul de fișiere.
4. Chloros va utiliza ținte adecvate pentru calibrarea fiecărei camere.

### Coloana Model cameră

Coloana **Model cameră** ajută la identificarea imaginilor provenite de la fiecare cameră:

* Survey3W\_RGN
* Survey3N\_OCN
* Survey3W\_RGB
* etc.

Utilizați această coloană pentru a verifica dacă ați marcat ținte pentru fiecare tip de cameră din proiectul dvs.

***

## Setări de detectare a țintelor

### Reglarea sensibilității de detectare

Dacă Chloros nu detectează corect țintele, reglați aceste setări în [Setări proiect](adjusting-project-settings.md):

**Zona minimă de eșantionare pentru calibrare:**

* **Implicit**: 25 pixeli
* **Creșteți** dacă obțineți detectări false pe artefacte mici
* **Reduceți** dacă țintele nu sunt detectate

**Gruparea minimă a țintelor:**

* **Implicit**: 60
* **Creșteți** dacă țintele sunt împărțite în mai multe detectări
* **Reduceți** dacă țintele cu variații de culoare nu sunt detectate complet

***

## Probleme comune cu imaginile țintelor

### Problemă: Nu sunt detectate ținte

**Cauze posibile:**

* Imaginile țintelor nu sunt marcate în File Browser (Browser fișiere)
* Ținta este prea mică în cadru (&lt; 30% din imagine)
* Iluminare slabă (umbre, strălucire)
* Setări de detectare a țintelor prea stricte

**Soluții:**

1. Verificați dacă coloana Țintă este bifată pentru imaginile corecte
2. Verificați calitatea imaginii țintei în previzualizare
3. Recapturați țintele dacă calitatea este slabă
4. Reglați setările de detectare a țintelor, dacă este necesar

### Problemă: Detectări false ale țintelor

**Cauze posibile:**

* Clădiri albe, vehicule sau acoperirea solului confundate cu ținte
* Pete luminoase în vegetație
* Sensibilitate de detectare prea scăzută

**Soluții:**

1. Marcați numai imaginile țintă reale pentru a limita domeniul de detectare
2. Măriți zona minimă de eșantionare pentru calibrare
3. Măriți valoarea minimă de grupare a țintelor
4. Asigurați-vă că imaginile țintă afișează numai ținta (dezordine minimă în fundal)

***

## Listă de verificare

Înainte de a începe procesarea, verificați selecția imaginilor țintă:

* [ ] Cel puțin o imagine țintă marcată per sesiune
* [ ] Casetele de selectare din coloana Țintă sunt bifate pentru toate imaginile țintă
* [ ] Imaginile țintă capturate în același interval de timp ca și sondajul
* [ ] Țintele sunt clar vizibile în previzualizare când se face clic pe ele
* [ ] Toate cele 4 panouri de calibrare sunt vizibile în fiecare imagine țintă
* [ ] Nu există umbre sau obstacole pe ținte
* [ ] Pentru cameră duală: ținte marcate pentru ambele tipuri de cameră

***

## Procesare fără ținte

### Procesare fără ținte de calibrare

Deși nu este recomandat pentru lucrări științifice, puteți procesa fără ținte:

1. Lăsați toate casetele de selectare din coloana Țintă debifate
2. **Dezactivați** „Calibrarea reflectanței” în Setările proiectului
3. Corecția vignetării va fi în continuare aplicată
4. Rezultatul nu va fi calibrat pentru reflectanța absolută

{% hint style=&quot;warning&quot; %}
**Nu este recomandat**: Fără calibrarea reflectanței, valorile pixelilor reprezintă doar luminozitatea relativă, nu măsurători științifice ale reflectanței. Utilizați ținte de calibrare pentru rezultate precise și repetabile.
{% endhint %}

***

## Pași următori

După ce ați marcat imaginile țintă:

1. **Verificați setările** - Consultați [Ajustarea setărilor proiectului](adjusting-project-settings.md)
2. **Începeți procesarea** - Consultați [Pornirea procesării](starting-the-processing.md)
3. **Monitorizați progresul** - Consultați [Monitorizarea procesării](monitoring-the-processing.md)

Pentru mai multe informații despre țintele de calibrare, consultați [Ținte de calibrare](../calibration-targets.md).
