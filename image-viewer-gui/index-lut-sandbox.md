# Index/LUT Sandbox

Index/LUT Sandbox este un spațiu de lucru interactiv din cadrul Chloros Image Viewer care vă permite să experimentați calculele indicelui multispectral și vizualizările culorilor în timp real. Acest instrument puternic vă ajută să testați diferite indici, să rafinați intervalele de valori și să creați vizualizări gata de publicare fără a reprocesa întregul set de date.

## Ce este Index/LUT Sandbox?

### Scop

Sandbox oferă:

* **Calcularea indicelui în timp real** - Aplicați instantaneu orice indice de vegetație
* **Ajustarea interactivă a LUT** - Reglați cu precizie gradientele și intervalele de culori
* **Optimizarea fluxului de lucru** - Determinați cele mai bune setări înainte de procesarea în lot

### Sandbox vs. Procesarea proiectului

**Index/LUT Sandbox (interactiv):**

* O singură imagine la un moment dat
* Feedback instantaneu
* Experimental și iterativ
* Fără modificări permanente ale fișierelor
* Perfect pentru explorare și testare

**Procesarea proiectului (în lot):**

* Întregul set de date simultan
* Setări preconfigurate
* Fișiere de ieșire permanente
* Necesită mult timp
* Optimal când setările sunt finalizate

{% hint style=&quot;success&quot; %}
**Cel mai bun flux de lucru**: Utilizați Sandbox pentru a experimenta și a găsi setările optime pentru index și LUT, apoi aplicați aceste setări în timpul procesării proiectului pentru întregul set de date.
{% endhint %}

***

## Lucrul cu Index/LUT Sandbox

### Înțelegerea indexurilor precalculate

În Chloros, indexurile pot fi aplicate în timpul procesării proiectului. Pentru a determina ce setări de index și LUT doriți să aplicați exporturilor, cel mai simplu este să utilizați sandbox-ul vizualizatorului de imagini.

Sandbox-ul vă permite să:

* **Aplicați noi indici și gradientele de culoare (LUT)** pentru a vizualiza datele
* **Reglați setările de vizualizare** în mod interactiv
* **Vizualizați** imaginile indexate deja calculate
* **Inspectați** valorile pixelilor la toate nivelurile de zoom

### Deschiderea Sandbox-ului

Index/LUT Sandbox este accesat în **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> :

1. Faceți clic pe o imagine din grila de imagini a browserului de fișiere, aceasta se deschide în fila **Vizualizator de imagini** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> .
2. Faceți clic pe fila **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> pentru a deschide bara laterală pop-out din stânga, dacă nu este deja deschisă

### Selectarea unei imagini pentru a aplica un index/LUT

Pentru a lucra cu un index în Image Viewer <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> :

1. **Deschideți o imagine** din grila principală de imagini făcând clic pe ea
2. Se va deschide fila **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> se va deschide
3. Faceți clic pe **meniul derulant Layer** (în partea dreaptă sus a vizualizatorului)
4. Selectați stratul din meniul derulant:
   * RAW (Reflectanță)

### Aplicarea unui index la o imagine

Odată ce imaginea este pe ecran complet și bara laterală **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> este deschisă:

1. Bifați caseta Index din partea de sus a barei laterale
2. Alegeți filtrul camerei dvs. din meniul derulant din stânga
3. Alegeți formula de index dorită din meniul derulant din dreapta
4. Trageți cercurile de culoare ale canalului de filtrare în locațiile din formula de index de mai jos
5. Odată ce formula este validă, imaginea se va actualiza și va afișa valorile indexului
6. Mișcați cursorul mouse-ului pentru a vedea valorile din locația cursorului
7. Măriți imaginea pentru a vedea pixelii individuali și valorile asociate acestora.

Fiecare index are un interval de valori și o semnificație specifice:

#### NDVI Exemplu

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

Pentru documentația completă a formulei indexului, consultați [Formule index multispectrale](../project-settings/multispectral-index-formulas.md).

***

## Lucrul cu LUT-uri (tabele de consultare)

### Ce este un LUT?

O **tabelă de consultare (LUT)** mapează valorile numerice ale indicelui la culori pentru vizualizare:

* **Intrare**: Valoarea pixelului indicelui (de exemplu, NDVI 0,65)
* **Ieșire**: culoarea RGB (de exemplu, verde strălucitor)
* **Scop**: Facilitează vizualizarea și interpretarea modelelor**LUT în tonuri de gri vs. LUT color:**

* Tonuri de gri: științific și neutru, afișează datele brute
* LUT color: intuitiv și impactant, evidențiază modelele și diferențele

{% hint style=&quot;success&quot; %}
**Puterea vizualizării**: Aplicarea unui LUT color unei imagini indexate în tonuri de gri facilitează considerabil identificarea modelelor, anomaliilor și zonelor de interes dintr-o singură privire.
{% endhint %}

### Aplicarea unui LUT la o imagine index

Odată ce aveți o imagine index care afișează

1. Faceți clic pe <img src="../.gitbook/assets/image (1) (1).png" alt="" data-size="line"> „+Adăugați LUT”
2. Selectați gradientul de culoare
3. Reglați punctele minime/maxime de decupare
4. Reglați modul de decupare
5. Bifați caseta Index din bara laterală **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> din bara laterală pentru a aplica LUT-ul.

### Alegerea unui gradient de culoare

**Selectarea unui gradient:**

1. În panoul LUT, localizați**bara de gradient colorat**.
2. Treceți mouse-ul peste ea pentru a vizualiza presetările de gradient disponibile.
3. Selectați gradientul dorit.
4. Imaginea **se actualizează imediat** cu culori noi atunci când caseta Index este bifată.

{% hint style=&quot;success&quot; %}
**Cele mai bune practici**: Pentru indici de vegetație precum NDVI, gradientul Red-Galben-Green este cel mai intuitiv, deoarece se aliniază cu asocierile naturale de culori (verde = sănătos, galben = moderat, roșu = stresat).
{% endhint %}

### Reglarea claselor de culori

**Controlul claselor**determină numărul de trepte de culoare discrete care apar în gradientul dvs.:**Opțiuni pentru numărul de clase:*** **2-5 clase**: categorii foarte largi, zone distincte
* **6-10 clase**: echilibrate, bune pentru clasificare
* **11-20 clase**: gradienturi line, aspect continuu
* **20+ clase**: aproape continuu, linie maximă**Cum se ajustează:**

1. În panoul LUT, localizați**pătratele cu mostre de culoare sub bara de gradient**

2. Reglați numărul de clase adăugând cu butonul +
3. Eliminați numărul de clase făcând dublu clic pe o mostră de culoare
4. Gradientul se actualizează **în timp real** pe imagine**Efectul asupra vizualizării:*** **Mai puține clase** (3-5): Creează zone distincte, clasificare simplificată, categorii mai ușor de distins
* **Clasă medie** (6-10): Abordare echilibrată, potrivită pentru majoritatea aplicațiilor
* **Mai multe clase** (15-20): tranziții line, variații detaliate, aspect fotografic**Când se utilizează:*** **Puține clase (3-5)**: diapozitive de prezentare, hărți de clasificare, rapoarte simple
* **Clasele medii (6-10)**: analiză generală, detalii echilibrate, rapoarte standard
* **Multe clase (15-20)**: Analiză științifică, inspecție detaliată, rezultate de calitate publicabilă

### Reglarea fină a intervalelor de valori

**Controalele intervalului de valori**determină care valori ale indexului sunt asociate cu care culori din gradientul dvs.:**Controale interval în panoul LUT:*** **Valoare minimă**: Limita inferioară a scalei de culori
* **Valoare maximă**: Limita superioară a scalei de culori
* **Valori intermediare**: distribuite automat între minim și maxim (pe baza numărului de clase)

#### Reglarea valorilor minime/maxime

**Pentru a regla intervalele de valori:**

1. În panoul LUT, localizați câmpurile de introducere**Valoare minimă**și**Valoare maximă**

2. Faceți clic pe câmpul**Valoare minimă**

3. Introduceți valoarea minimă dorită (de exemplu, `0.2`)
4. Apăsați **Enter** sau faceți clic în afara câmpului
5. Repetați pentru câmpul **Valoare maximă** (de exemplu, `0.9`)
6. Vizualizarea **se actualizează imediat**{% hint style=&quot;info&quot; %}**Scalare automată**: Când aplicați pentru prima dată un LUT, Chloros setează automat minimul/maximul la intervalul de date real din imagine. Puteți apoi restrânge acest interval pentru a vă concentra pe intervale de valori specifice de interes.
{% endhint %}

**Exemplu de ajustări ale intervalului NDVI:*** **Interval complet**: `-1.0` până la `1.0` (afișează toate valorile posibile)
* **Axat pe vegetație**: `0.2` până la `0.9` (exclude solul gol și apa)
* **Numai vegetație sănătoasă**: `0.5` până la `0.9` (evidențiază numai plantele viguroase)
* **Detectarea stresului**: `0.2` până la `0.5` (evidențiază zonele cu probleme)
* **Interval personalizat**: ajustează în funcție de valorile pixelilor observați**De ce să ajustezi intervalele?*** **Crește contrastul** în zona de interes
* **Excludeți valorile irelevante** (de exemplu, corpuri de apă, sol gol)
* **Standardizați vizualizarea** pe mai multe imagini sau date
* **Evidențiați diferențele subtile** într-un interval de valori restrâns

### Decuparea valorilor în afara intervalului

Când valorile pixelilor se situează în afara intervalului minim/maxim definit, puteți controla modul în care sunt afișate utilizând **modurile de decupare**.

#### **Opțiuni disponibile pentru modul de decupare:**

#### 1. Minim și maxim

* Pixelii **sub minim**→ afișare utilizând**prima culoare** din gradient (de exemplu, roșu)
* Pixelii **peste maxim**→ afișare utilizând**ultima culoare** din gradient (de exemplu, verde)
* **Caz de utilizare**: subliniați valorile extreme, afișați intervalul complet de date cu culori saturate la limite
* **Exemplu**: valorile NDVI sub 0,2 apar toate în roșu, valorile peste 0,9 apar toate în verde

#### 2. Fundal transparent

* Pixelii **în afara intervalului**devin**complet transparenți*** Numai pixelii **din interval** afișează gradientul de culoare
* **Caz de utilizare**: suprapunere GIS, izolarea intervalelor de valori specifice, evidențierea numai a zonelor de interes
* **Exemplu**: Afișează numai NDVI 0,4-0,7 în culori, restul fiind transparent

{% hint style=&quot;warning&quot; %}
**Limitare transparență**: Pixelii transparenți vor apărea ca culoarea de fundal în vizualizator. Când sunt exportați în timpul procesării, transparența este păstrată în formatul PNG, dar nu și în JPG.
{% endhint %}

#### 3. Fundal index

* Pixelii **în afara intervalului**sunt afișați în**tonuri de gri** (afișând valorile index brute)
* Pixelii **din interval**afișează**gradient de culoare*** **Caz de utilizare**: evidențiere subtilă, menținerea contextului în timp ce se accentuează zonele de interes
* **Exemplu**: Evidențierea colorată a vegetației stresate (NDVI 0,3-0,5) și afișarea zonelor sănătoase în gri

#### 4. Fundal original

* Pixelii **în afara intervalului**afișează**imaginea multispectrală originală*** Pixelii **în interval**afișează**gradient de culoare*** **Caz de utilizare**: Cel mai intuitiv - combină contextul natural al imaginii cu suprapunerea analitică a culorilor
* **Exemplu**: Vedeți aspectul real al câmpului/culturii cu zonele stresate suprapuse și codificate prin culori

### Alegerea modului de decupare potrivit

| Mod de decupare              | Cel mai potrivit pentru                                   | Stil de vizualizare          |
| -------------------------- | ------------------------------------------ | ---------------------------- |
| **Minim și maxim**    | Afișare completă a datelor, analiză științifică     | Toți pixelii colorați           |
| **Fundal transparent** | Suprapuneri GIS, izolarea intervalelor specifice    | Culoare în interval, gol în afara intervalului |
| **Fundal indexat**       | Accent subtil, menținerea contextului datelor  | Culoare pe interval, gri în afara intervalului  |
| **Fundal original**    | Rapoarte, prezentări, analiză intuitivă | Culoare pe interval, fotografie în afara intervalului |

### Crearea culorilor LUT personalizate

Pentru un control complet asupra vizualizării, puteți crea **gradiente de culori personalizate** prin editarea opririlor individuale de culoare.**Pentru a crea un gradient personalizat:**

1. În panoul LUT, localizați**bara de previzualizare a gradientului**

2. Căutați**pătratele de mostre de culori** sub gradient
3. **Faceți clic pe o oprire de culoare** pentru a o selecta
4. Se deschide un **selector de culori**

5. Alegeți o nouă culoare utilizând:
   * **Roata de culori**: selecție vizuală a culorilor
   * **Cursorii RGB/HSV**: control precis al culorilor
   * **Introducerea codului hexadecimal**: specificarea exactă a culorii (de exemplu, `#FF0000` pentru roșu)
6. Faceți clic în afara selectorului de culori **pentru a aplica noua culoare**

7. Gradientul**se actualizează imediat** pe imagine**Adăugarea sau eliminarea opririlor de culoare:*** **Adăugați o oprire**: faceți clic pe pictograma + pentru a adăuga o nouă mostră la sfârșit
* **Eliminați o oprire**: faceți dublu clic pe pătratul colorat pentru a elimina mostra**Strategii de personalizare:*** **Inversați gradientul**: inversați ordinea culorilor pentru a inversa semnificația (de exemplu, verde = scăzut, roșu = ridicat)
* **Culori de marcă**: potriviți paleta de culori a organizației dvs. pentru rapoarte
* **Potrivit pentru persoanele cu daltonism**: utilizați combinații de portocaliu-albastru sau violet-galben
* **Optimizare pentru imprimare**: alegeți culori care funcționează atât pentru imprimarea color, cât și pentru imprimarea în tonuri de gri
* **Praguri multiple**: utilizați culori distincte la praguri de valori specifice pentru clasificare

{% hint style=&quot;info&quot; %}
**Salvarea gradientelor personalizate**: gradientele personalizate pot fi salvate și reutilizate. Faceți clic pe pictograma de salvare din panoul LUT pentru a păstra schemele de culori personalizate pentru utilizare ulterioară.
{% endhint %}

***

## Flux de lucru interactiv

### Actualizări în timp real

Toate ajustările LUT din sandbox actualizează imaginea **instantaneu și interactiv**:

* **Comutare strat** → Imaginea se modifică imediat
* **Selectați gradientul** → Culorile se actualizează instantaneu
* **Ajustați intervalul de valori** → Contrastul se modifică în timp real
* **Schimbați clasele** → Nivelul de netezime al gradientului se actualizează imediat
* **Modificați decuparea** → Afișarea fundalului se modifică instantaneu
* **Editați culorile** → Gradientul personalizat se aplică imediat**Nu este necesar butonul „Aplicare”** - toate modificările sunt live și interactive!

{% hint style=&quot;success&quot; %}
**Feedback în timp real**: Feedback-ul vizual instantaneu vă permite să experimentați rapid diferite setări până când găsiți vizualizarea optimă pentru nevoile dvs. de analiză.
{% endhint %}

### Flux de lucru iterativ de rafinare

**Flux de lucru tipic de optimizare LUT:**

1.**Selectați stratul index** (de exemplu, RAW (Reflectanță))
2. **Aplicați indexul** - Alegeți filtrul camerei și formula indexului, trageți cercurile colorate în locația corespunzătoare din formula indexului
3. **Aplicați gradientul LUT** - Începeți cu presetarea Red-Yellow-Green
4. **Verificați valorile pixelilor** - Mutați cursorul, notați intervalele de valori
5. **Reglați min/max** - Restrângeți pentru a vă concentra pe vegetație (de exemplu, 0,2 până la 0,9)
6. **Alegeți decuparea** - Încercați „Fundal original” pentru context
7. **Rafinați culorile** - Personalizați gradientul dacă este necesar pentru accentuarea specifică
8. **Finalizați setările**- Documentați setările și copiați-le în Setări proiect pentru procesarea exportului

### Verificarea valorilor pixelilor

Înțelegerea valorilor reale ale pixelilor este esențială pentru setarea intervalelor LUT eficiente:**Cum se verifică valorile:**

1. Valorile pixelilor sunt afișate când imaginea are**bifate** casetele Index sau Index și LUT.
2. **Mutați cursorul** peste diferite zone ale imaginii
3. **Observați valorile pixelilor** afișate în legendă pe măsură ce treceți cu cursorul
4. Măriți pentru a vedea pixelii individuali evidențiați cu o valoare flotantă
5. **Luați notițe** despre intervalele de valori pentru diferite caracteristici:
   * **Vegetație sănătoasă**: de exemplu, NDVI 0,55-0,85
   * **Vegetație stresată**: de exemplu, NDVI 0,30-0,50
   * **Sol gol**: de exemplu, NDVI 0,05-0,25
   * **Apă** (dacă este prezentă): de exemplu, NDVI -0,05 până la 0,10**Utilizarea valorilor pixelilor pentru a seta intervalele LUT:**După inspectarea valorilor pixelilor, ajustați LUT min/max în consecință:**Exemplu de scenariu:*** **Observație**: Valori sol = 0,05-0,25, Stresat = 0,25-0,50, Sănătos = 0,50-0,85
* **Obiectiv**: Vizualizarea numai a stării de sănătate a plantelor (excluzând solul)
* **Setări LUT**: Min = `0.25`, Max = `0.85`
* **Decupare**: „Fundal original” pentru a vedea solul în culori naturale
* **Rezultat**: Gradientul de culoare se aplică numai vegetației, solul apare ca imagine originală

{% hint style=&quot;info&quot; %}
**Interval dinamic**: culturile, anotimpurile și etapele de creștere diferite vor avea intervale de valori diferite. Verificați întotdeauna valorile pixelilor din setul de date specific înainte de a seta intervalele LUT.
{% endhint %}

***

## Indici personalizați (Chloros+)

### Crearea formulelor de indici personalizați

{% hint style=&quot;info&quot; %}
**Unde se creează**: Indicii personalizați pot fi configurați în**Setări proiect** înainte de procesare, precum și în bara laterală a sandbox-ului Image Viewer.
{% endhint %}

**Pentru a crea un indice personalizat:**

1.**Deschideți Setări proiect** (înainte de procesare) sau bara laterală a sandbox-ului Image Viewer
2. Navigați la **meniul derulant Formula index**

3. Căutați opțiunea**„Personalizat”** (trebuie să fiți conectat cu licența Chloros+)
4. **Definiți formula** utilizând variabile de bandă:
   * Nume benzi: `NIR`, `Red`, `Green`, `Blue`, `RedEdge` etc.
   * Operatori: `+`, `-`, `*`, `/`, `^` (exponent)
   * Funcții: `sqrt()`, `abs()` etc. (dacă sunt acceptate)
   * Paranteze: `()` pentru ordinea operațiilor
5. **Denumiți indexul** (de exemplu, „MyIndex” sau „CustomNDVI”)
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

{% hint style=&quot;warning&quot; %}
**Validarea formulei**: Asigurați-vă că formula dvs. utilizează benzile disponibile în camera dvs. De exemplu, RedEdge este disponibil numai pe camerele cu filtru RedEdge.
{% endhint %}

***

## Pași următori

Acum că ați înțeles Index/LUT Sandbox:

* **Aplicați la procesare**: Utilizați setările descoperite în [Setări proiect](../project-settings/project-settings.md)
* **Procesare în lot**: aplicați indici optimizați la seturi de date complete
* **Aflați mai multe**: citiți [Formule de indici multispectrali](../project-settings/multispectral-index-formulas.md)

Documentație conexă:

* [**Straturi de imagine**](image-layers.md) - Gestionarea și vizualizarea straturilor
* [**Deschiderea unei imagini pe ecran complet**](opening-an-image-full-screen.md) - Noțiuni de bază despre vizualizatorul de imagini
* [**Prelucrarea imaginilor (GUI)**](../processing-images-gui/adding-files-to-a-project.md) - Fluxul complet de lucru pentru prelucrare
