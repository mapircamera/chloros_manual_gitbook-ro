# Sandbox pentru indici/LUT

Sandbox-ul pentru indici/LUT este un spațiu de lucru interactiv din cadrul vizualizatorului de imagini Chloros, care vă permite să experimentați calcularea indicilor multispectrali și vizualizarea culorilor în timp real. Acest instrument puternic vă ajută să testați diferiți indici, să rafinați intervalele de valori și să creați vizualizări gata de publicare fără a fi nevoie să reprocesați întregul set de date.

## Ce este Index/LUT Sandbox?

### Scop

Sandbox oferă:

* **Calculul indicilor în timp real** - Aplicați instantaneu orice indice de vegetație
* **Reglarea interactivă a LUT** - Reglați cu precizie gradientele și intervalele de culori
* **Optimizarea fluxului de lucru** - Determinați cele mai bune setări înainte de procesarea în lot

### Sandbox vs. Procesare proiect

**Index/LUT Sandbox (Interactiv):**

* O singură imagine pe rând
* Feedback instantaneu
* Experimental și iterativ
* Fără modificări permanente ale fișierelor
* Perfect pentru explorare și testare

**Procesare proiect (în lot):**

* Întregul set de date simultan
* Setări preconfigurate
* Fișiere de ieșire permanente
* Consumă mult timp
* Ideal când setările sunt finalizate

{% hint style="success" %}
**Cel mai bun flux de lucru**: Utilizați Sandbox pentru a experimenta și a găsi setările optime pentru indice și LUT, apoi aplicați aceste setări în timpul procesării proiectului pentru întregul set de date.
{% endhint %}

***

## Lucrul cu Sandbox-ul pentru index/LUT

### Înțelegerea indexurilor precalculate

În Chloros, indexurile pot fi aplicate în timpul procesării proiectului. Pentru a determina ce setări de index și LUT doriți să aplicați exporturilor, cel mai simplu este să utilizați Sandbox-ul vizualizatorului de imagini.

Sandbox-ul vă permite să:

* **Aplicați indici noi și gradientele de culoare (LUT)** pentru a vizualiza datele
* **Reglați setările de vizualizare** în mod interactiv
* **Vizualizați** imaginile indexate deja calculate
* **Inspectați** valorile pixelilor la toate nivelurile de zoom

### Deschiderea Sandbox-ului

Sandbox-ul Index/LUT este accesat din fila **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> :

1. Faceți clic pe o imagine din grila de imagini a browserului de fișiere; aceasta se deschide în fila **Image Viewer**<img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> 2. Faceți clic pe fila**Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> pentru a deschide bara laterală pop-out din stânga, dacă aceasta nu este deja deschisă

### Selectarea unei imagini la care să se aplice un index/LUT

Pentru a lucra cu un index în <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> :

1. **Deschideți o imagine** din grila principală de imagini făcând clic pe ea
2. Se va deschide apoi fila **Vizualizator de imagini** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> se va deschide
3. Faceți clic pe **meniul derulant Layer** (în partea dreaptă sus a vizualizatorului)
4. Selectați stratul din meniul derulant:
   * RAW (Reflectanță)

### Aplicarea unui indice unei imagini

Odată ce imaginea este pe ecran complet și bara laterală a **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> este deschisă:

1. Bifați caseta Index din partea de sus a barei laterale
2. Alegeți filtrul camerei dvs. din meniul derulant din stânga
3. Alegeți formula de index dorită din meniul derulant din dreapta
4. Trageți cercurile de culoare ale canalului de filtru în locațiile din formula de index de mai jos
5. Odată ce formula este validă, imaginea se va actualiza și va afișa valorile indexului
6. Mișcați cursorul mouse-ului pentru a vedea valorile din locația cursorului
7. Măriți imaginea pentru a vedea pixelii individuali și valorile asociate acestora

Fiecare indice are un interval de valori și o semnificație specifice:

#### Exemplu NDVI

```

Formula: (NIR - Red) / (NIR + Red)

For Survey3W RGN camera:
NIR = 850nm band
Red = 661nm band

Result range: -1.0 to +1.0
Typical vegetation: 0.4 to 0.9
Stressed vegetation: 0.2 to 0.4
Bare soil: 0.0 to 0.2
Water: -0.1 to 0.1
```

Pentru documentația completă privind formulele de indice, consultați [Formule de indice multispectral](../project-settings/multispectral-index-formulas.md).

***

## Lucrul cu LUT-uri (tabele de consultare)

### Ce este un LUT?

O **tabelă de consultare (LUT)** mapează valorile numerice ale indexului la culori pentru vizualizare:

* **Intrare**: Valoarea pixelului indexului (de ex., NDVI 0,65)
* **Ieșire**: RGB culoare (de ex., verde strălucitor)
* **Scop**: Facilitează vizualizarea și interpretarea modelelor**LUT în tonuri de gri vs. LUT color:**

* Tonuri de gri: Științific și neutru, afișează datele brute
* LUT color: Intuitiv și de impact, evidențiază modelele și diferențele

{% hint style="success" %}
**Puterea vizualizării**: Aplicarea unei LUT color la o imagine index în tonuri de gri facilitează considerabil identificarea modelelor, anomaliilor și zonelor de interes dintr-o singură privire.
{% endhint %}

### Aplicarea unei LUT la o imagine index

Odată ce aveți o imagine index care afișează

1. Faceți clic pe butonul <img src="../.gitbook/assets/image (1) (1).png" alt="" data-size="line"> butonul „+Adăugați LUT”
2. Selectați gradientul de culoare
3. Reglați punctele finale min/max de decupare
4. Reglați modul de decupare
5. Bifați caseta Index din bara laterală a **Vizualizatorului de imagini** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> din bara laterală pentru a aplica LUT-ul

### Alegerea unui gradient de culori

**Selectarea unui gradient:**

1. În panoul LUT, localizați**bara de gradient colorat**

2. Treceți cu mouse-ul peste aceasta pentru a vizualiza presetările de gradient disponibile
3. Selectați gradientul dorit
4. Imaginea **se actualizează imediat** cu noile culori atunci când caseta Index este bifată

{% hint style="success" %}
**Cele mai bune practici**: Pentru indici de vegetație precum NDVI, gradientul Red-Galben-Green este cel mai intuitiv, deoarece se aliniază cu asocierile naturale de culori (verde = sănătos, galben = moderat, roșu = stresat).
{% endhint %}

### Reglarea claselor de culori

**Controlul Clase**determină câte trepte de culoare discrete apar în gradientul dvs.:**Opțiuni pentru numărul de clase:*** **2-5 clase**: Categorii foarte largi, zone distincte
* **6-10 clase**: Echilibrat, bun pentru clasificare
* **11-20 clase**: Gradiente line, aspect continuu
* **20+ clase**: Aproape continuu, netezime maximă**Cum se ajustează:**

1. În panoul LUT, localizați**pătratele cu mostre de culoare de sub bara de gradient**

2. Reglați numărul de clase adăugând cu butonul +
3. Eliminați numărul de clase făcând dublu clic pe o mostră de culoare
4. Gradientul se actualizează **în timp real** pe imagine**Efect asupra vizualizării:*** **Mai puține clase** (3-5): Creează zone distincte, clasificare simplificată, categorii mai ușor de distins
* **Clasă medie** (6-10): Abordare echilibrată, potrivită pentru majoritatea aplicațiilor
* **Mai multe clase** (15-20): Tranziții line, variație detaliată, aspect fotografic**Când se utilizează:*** **Puține clase (3-5)**: Slide-uri de prezentare, hărți de clasificare, rapoarte simple
* **Clasă medie (6-10)**: Analiză generală, detalii echilibrate, rapoarte standard
* **Multe clase (15-20)**: Analiză științifică, inspecție detaliată, rezultate de calitate pentru publicare

### Reglarea fină a intervalelor de valori

**Controalele intervalului de valori**determină ce valori ale indicelui sunt asociate cu ce culori din gradientul dvs.:**Controale ale intervalului în panoul LUT:*** **Valoare minimă**: Limita inferioară a scalei de culori
* **Valoare maximă**: Limita superioară a scalei de culori
* **Valori intermediare**: Distribuite automat între minim și maxim (în funcție de numărul de clase)

#### Reglarea valorilor minime/maxime

**Pentru a regla intervalele de valori:**

1. În panoul LUT, localizați câmpurile de introducere**Valoare minimă**și**Valoare maximă**

2. Faceți clic pe câmpul**Valoare minimă**

3. Introduceți valoarea minimă dorită (de exemplu, `0.2`)
4. Apăsați **Enter** sau faceți clic în afara câmpului
5. Repetați pentru câmpul **Valoare maximă** (de exemplu, `0.9`)
6. Vizualizarea **se actualizează imediat**{% hint style="info" %}**Scalare automată**: Când aplicați pentru prima dată o LUT, Chloros setează automat valorile min/max la intervalul real de date din imagine. Puteți apoi restrânge acest interval pentru a vă concentra pe anumite intervale de valori de interes.
{% endhint %}

**Exemplu de ajustări ale intervalului NDVI:*** **Interval complet**: `-1.0` până la `1.0` (afișează toate valorile posibile)
* **Concentrat pe vegetație**: de la `0.2` la `0.9` (exclude solul gol și apa)
* **Numai vegetație sănătoasă**: de la `0.5` la `0.9` (evidențiază numai plantele viguroase)
* **Detectarea stresului**: de la `0.2` la `0.5` (evidențiați zonele cu probleme)
* **Interval personalizat**: reglați în funcție de valorile pixelilor observați**De ce să reglați intervalele?*** **Creșteți contrastul** în zona de interes
* **Excludeți valorile irelevante** (de exemplu, corpuri de apă, sol gol)
* **Standardizați vizualizarea** pe mai multe imagini sau date
* **Evidențiați diferențele subtile** într-un interval de valori îngust

### Decuparea valorilor în afara intervalului

Când valorile pixelilor se află în afara intervalului minim/maxim definit, puteți controla modul în care sunt afișate folosind **modurile de decupare**.

#### **Opțiuni disponibile pentru modurile de decupare:**

#### 1. Minim și maxim

* Pixeli **sub minim**→ afișați folosind**prima culoare** din gradient (de ex., roșu)
* Pixeli **peste maxim**→ afișați folosind**ultima culoare** din gradient (de ex., verde)
* **Caz de utilizare**: Evidențiază valorile extreme, afișează intervalul complet de date cu culori saturate la limite
* **Exemplu**: Valorile NDVI sub 0,2 apar toate roșii, valorile peste 0,9 apar toate verzi

#### 2. Fundal transparent

* Pixelii **în afara intervalului**devin**complet transparenți*** Doar pixelii **din interval** afișează gradientul de culori
* **Caz de utilizare**: Suprapunere GIS, izolarea unor intervale specifice de valori, evidențierea doar a zonelor de interes
* **Exemplu**: Afișează doar NDVI 0,4-0,7 în culori, restul fiind transparent

{% hint style="warning" %}
**Limitarea transparenței**: Pixelii transparenți vor apărea ca culoarea fundalului în vizualizator. Când se exportă în timpul procesării, transparența este păstrată în formatul PNG, dar nu și în JPG.
{% endhint %}

#### 3. Fundal index

* Pixelii **în afara intervalului**se afișează în**tonuri de gri** (afișând valorile brute ale indexului)
* Pixelii **din interval**afișează**gradient de culoare*** **Caz de utilizare**: Evidențiere subtilă, menținerea contextului în timp ce se accentuează zonele de interes
* **Exemplu**: Evidențierea prin culoare a vegetației stresate (NDVI 0,3-0,5) în timp ce zonele sănătoase sunt afișate în gri

#### 4. Fundal original

* Pixelii **în afara intervalului**afișează**imaginea multispectrală originală*** Pixelii **în interiorul intervalului**prezintă**un gradient de culori*** **Caz de utilizare**: Cel mai intuitiv – combină contextul natural al imaginii cu o suprapunere analitică colorată
* **Exemplu**: Vedeți aspectul real al câmpului/culturii cu zonele de stres suprapuse prin coduri de culori

### Alegerea modului de decupare potrivit

| Mod de decupare              | Cel mai potrivit pentru                                   | Stil de vizualizare          |
| -------------------------- | ------------------------------------------ | ---------------------------- |
| **Minim și maxim**    | Afișare completă a datelor, analiză științifică     | Toți pixelii colorați           |
| **Fundal transparent** | Suprapuneri GIS, izolarea unor intervale specifice    | Culoare în interval, gol în afara acestuia |
| **Fundal index**       | Accentuare subtilă, menținerea contextului datelor  | Culoare în interval, gri în afara acestuia  |
| **Fundal original**    | Rapoarte, prezentări, analiză intuitivă | Culoare în interval, fotografie în afara acestuia |

### Crearea culorilor LUT personalizate

Pentru un control deplin asupra vizualizării, puteți crea **gradiente de culori personalizate** prin editarea punctelor de culoare individuale.**Pentru a crea un gradient personalizat:**

1. În panoul LUT, localizați**bara de previzualizare a gradientului**

2. Căutați**pătratele cu mostre de culoare** de sub gradient
3. **Faceți clic pe un punct de culoare** pentru a-l selecta
4. Se deschide un **selector de culori**

5. Alegeți o nouă culoare folosind:
   * **Roata de culori**: Selecție vizuală a culorii
   * **Glisoarele RGB/HSV**: Control precis al culorii
   * **Introducerea codului hexazecimal**: Specificație exactă a culorii (de ex., `#FF0000` pentru roșu)
6. Faceți clic în afara selectorului de culori **pentru a aplica noua culoare**

7. Gradientul**se actualizează imediat** pe imagine**Adăugarea sau eliminarea punctelor de culoare:*** **Adăugați un punct**: Faceți clic pe pictograma + pentru a adăuga o nouă mostră la sfârșit
* **Eliminați un punct**: Faceți dublu clic pe pătratul de culoare pentru a elimina mostra**Strategii de personalizare:*** **Inversare gradient**: Inversați ordinea culorilor pentru a schimba semnificația (de ex., verde = scăzut, roșu = ridicat)
* **Culori de brand**: Potriviți paleta de culori a organizației dvs. pentru rapoarte
* **Potrivit pentru daltonism**: Utilizați combinații portocaliu-albastru sau violet-galben
* **Optimizare pentru imprimare**: Alegeți culori care funcționează atât la imprimarea color, cât și la cea în tonuri de gri
* **Praguri multiple**: Utilizați culori distincte la praguri de valoare specifice pentru clasificare

{% hint style="info" %}
**Salvarea gradientelor personalizate**: Gradientele personalizate pot fi salvate și reutilizate. Faceți clic pe pictograma de salvare din panoul LUT pentru a păstra schemele de culori personalizate pentru utilizare ulterioară.
{% endhint %}

***

## Flux de lucru interactiv

### Actualizări în timp real

Toate ajustările LUT din zona de testare actualizează imaginea **instantaneu și interactiv**:

* **Comutați stratul** → Imaginea se modifică imediat
* **Selectați gradientul** → Culorile se actualizează instantaneu
* **Reglați intervalul de valori** → Contrastul se modifică în timp real
* **Modificați clasele** → Netezimea gradientului se actualizează imediat
* **Modificați decuparea** → Afișarea fundalului se schimbă instantaneu
* **Editați culorile** → Gradientul personalizat se aplică imediat**Nu este necesar butonul „Aplicați”** - toate modificările sunt live și interactive!

{% hint style="success" %}
**Feedback live**: Feedback-ul vizual instantaneu vă permite să experimentați rapid cu diferite setări până când găsiți vizualizarea optimă pentru nevoile dvs. de analiză.
{% endhint %}

### Flux de lucru de rafinare iterativă

**Flux de lucru tipic de optimizare LUT:**

1.**Selectați stratul index** (de ex., RAW (Reflectanță))
2. **Aplicați indexul** - Alegeți filtrul camerei și formula indexului, trageți cercurile colorate în locația corespunzătoare din formula indexului
3. **Aplicați gradientul LUT** - Începeți cu presetarea Red-Yellow-Green
4. **Verificați valorile pixelilor** - Deplasați cursorul, notați intervalele de valori
5. **Reglați min/max** - Restrângeți intervalul pentru a vă concentra pe vegetație (de exemplu, 0,2 până la 0,9)
6. **Alegeți decuparea** - Încercați „Original Background” pentru context
7. **Rafinați culorile** - Personalizați gradientul dacă este necesar pentru o accentuare specifică
8. **Finalizați setările**- Documentați setările și copiați-le în Setările proiectului pentru procesarea exportului

### Inspectarea valorilor pixelilor

Înțelegerea valorilor reale ale pixelilor este crucială pentru stabilirea unor intervale LUT eficiente:**Cum să inspectați valorile:**

1. Valorile pixelilor se afișează atunci când imaginea are**casetele bifate** fie pentru Index, fie pentru Index și LUT.
2. **Mutați cursorul** peste diferite zone ale imaginii
3. **Observați valorile pixelilor** afișate în legendă pe măsură ce treceți cu cursorul
4. Măriți imaginea pentru a vedea pixelii individuali evidențiați cu o valoare flotantă
5. **Luați notițe** privind intervalele de valori pentru diferite caracteristici:
   * **Vegetație sănătoasă**: de ex., NDVI 0,55-0,85
   * **Vegetație stresată**: de ex., NDVI 0,30-0,50
   * **Sol gol**: de ex., NDVI 0,05-0,25
   * **Apă** (dacă este prezentă): de ex., NDVI -0,05 până la 0,10**Utilizarea valorilor pixelilor pentru a seta intervalele LUT:**După inspectarea valorilor pixelilor, ajustați valorile min/max ale LUT în consecință:**Scenariu de exemplu:*** **Observație**: Valori sol = 0,05-0,25, Stresat = 0,25-0,50, Sănătos = 0,50-0,85
* **Obiectiv**: Vizualizați doar starea de sănătate a plantelor (excludeți solul)
* **Setări LUT**: Min = `0.25`, Max = `0.85`
* **Decupare**: „Fundal original” pentru a vedea solul în culoarea naturală
* **Rezultat**: Gradientul de culoare se aplică doar vegetației, solul apare ca în imaginea originală

{% hint style="info" %}
**Interval dinamic**: Culturi, anotimpuri și stadii de creștere diferite vor avea intervale de valori diferite. Verificați întotdeauna valorile pixelilor din setul dvs. de date specific înainte de a seta intervalele LUT.
{% endhint %}

***

## Indici personalizați (Chloros+)

### Crearea formulelor de indici personalizați

{% hint style="info" %}
**Unde se creează**: Indicii personalizați pot fi configurați în**Setările proiectului** înainte de procesare, precum și în bara laterală a zonei de testare din Vizualizatorul de imagini.
{% endhint %}

**Pentru a crea un indice personalizat:**

1.**Deschideți Setările proiectului** (înainte de procesare) sau bara laterală a zonei de testare din Vizualizatorul de imagini
2. Navigați la **meniul derulant Formula indice**

3. Căutați opțiunea**„Personalizat”** (trebuie să fiți conectat cu licența Chloros+)
4. **Definiți formula** folosind variabilele de bandă:
   * Nume de bandă: `NIR`, `Red`, `Green`, `Blue`, `RedEdge` etc.
   * Operatori: `+`, `-`, `*`, `/`, `^` (exponent)
   * Funcții: `sqrt()`, `abs()` etc. (dacă sunt acceptate)
   * Paranteze: `()` pentru ordinea operațiilor
5. **Denumiți-vă indexul** (de exemplu, „MyIndex” sau „CustomNDVI”)
6. **Salvați configurația**

**Exemple de formule personalizate:**

```

Modified NDVI with offset:
(NIR - Red) / (NIR + Red + 0.5)

Simple ratio:
NIR / Red

Complex multi-band:
(NIR - Red) / (NIR + Red - Blue)

Exponential index:
(NIR / Red) ^ 2
```

{% hint style="warning" %}
**Validarea formulei**: Asigurați-vă că formula dvs. utilizează benzile disponibile în camera dvs. De exemplu, RedEdge este disponibilă numai pe camerele cu un filtru RedEdge.
{% endhint %}

***

## Pași următori

Acum că ați înțeles Index/LUT Sandbox:

* **Aplicați la procesare**: Utilizați setările descoperite în [Setări proiect](../project-settings/project-settings.md)
* **Procesare în lot**: Aplicați indicii optimizați la seturi de date complete
* **Aflați mai multe**: Citiți [Formule de indici multispectrali](../project-settings/multispectral-index-formulas.md)

Documentație conexă:

* [**Straturi de imagine**](image-layers.md) - Gestionarea și vizualizarea straturilor
* [**Deschiderea unei imagini pe ecran complet**](opening-an-image-full-screen.md) - Noțiuni de bază despre vizualizatorul de imagini
* [**Prelucrarea imaginilor (GUI)**](../processing-images-gui/adding-files-to-a-project.md) - Flux de lucru complet de prelucrare
