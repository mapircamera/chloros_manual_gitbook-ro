# Începerea procesării

După ce ați importat imaginile, ați marcat țintele de calibrare și ați configurat setările proiectului, sunteți gata să începeți procesarea. Această pagină vă ghidează în inițierea fluxului de procesare Chloros.

## Lista de verificare pentru preprocesare

Înainte de a face clic pe butonul Start, verificați dacă totul este pregătit:

* [ ] **Fișiere importate** – Toate imaginile apar în File Browser
* [ ] **Imagini țintă marcate** – Coloana „Target” a fost verificată pentru imaginile de calibrare (sau a fost importată o înregistrare `.daq` pentru LATTICE)
* [ ] **Modele de camere detectate** – Coloana „Model cameră” afișează camerele corecte
* [ ] **Setări configurate** – Setările proiectului au fost verificate și ajustate
* [ ] **Indici selectați** – Indicii multispectrali doriti au fost adăugați (dacă este necesar)
* [ ] **Format de export ales** - Format de ieșire adecvat pentru fluxul dvs. de lucru

{% hint style="info" %}
**Sfat**: Faceți clic pe câteva imagini din Browserul de fișiere pentru a verifica dacă s-au încărcat corect înainte de procesare.
{% endhint %}

***

## Pornirea procesării

### Localizați butonul „Start”

Butonul „Start/Play” se află în bara de antet superioară a ferestrei Chloros:

* Poziție: în centrul părții superioare a ferestrei
* Pictogramă: **Butonul Redare/Start** <img src="../.gitbook/assets/image (2) (1) (1).png" alt="" data-size="line">
* Stare: Butonul este activat (luminos) când este gata de procesare

### Faceți clic pentru a începe

1. Faceți clic pe **butonul Redare/Start** din bara de sus
2. Procesarea începe imediat
3. Butonul devine un buton **Oprire** în timpul procesării
4. Bara de progres se actualizează, afișând starea procesării

{% hint style="success" %}
**Procesarea a început**: Odată ce ați făcut clic, Chloros gestionează automat toate etapele de procesare – detectarea țintelor, debayering, calibrare, calculul indexului și exportul. Acesta detectează automat dacă proiectul dvs. este de tip Survey3, LATTICE sau o combinație a acestora și aplică fluxul de procesare corespunzător fiecărei camere.
{% endhint %}

***

## Înțelegerea modurilor de procesare

Chloros funcționează în două moduri diferite de procesare, în funcție de licența dvs.:

### Mod gratuit (procesare secvențială)

**Disponibil pentru toți utilizatorii**

**Cum funcționează:**

* Procesează imaginile una câte una, secvențial
* Funcționare cu un singur thread
* Consum redus de memorie

**Bara de progres afișează 2 etape:**

1.**Detectarea țintelor** - Scanarea țintelor de calibrare
2. **Prelucrare** - Aplicarea calibrării și exportarea imaginilor**Timp de procesare:**

* Mult mai lent decât modul paralel Chloros+
* Potrivit pentru seturi de date mici și medii (&lt; 200 de imagini)

### Modul Chloros+ (procesare paralelă)

**Necesită licență Chloros+**

**Cum funcționează:**

* Procesează simultan mai multe imagini folosind un [flux de procesare cu 4 fire](../processing-architecture/processing-pipeline.md)
* [Adaptarea dinamică a calculului](../processing-architecture/dynamic-compute-adaptation.md) selectează automat strategia optimă pentru hardware-ul dvs. la pornirea programului
* Accelerare GPU (CUDA) cu plăci grafice NVIDIA (desktop și Jetson)
* **Numărul de procesori de lucru se adaptează la hardware**: strategiile GPU rulează**1-4 procesori de lucru simultan** (scalate în funcție de VRAM — un Jetson cu memorie redusă rulează 1, o placă grafică de desktop de 12 GB+ rulează până la 4); sistemele care utilizează doar CPU rulează un procesor de lucru pe fiecare nucleu fizic, minus unul**Bara de progres afișează 4 etape** (corespunzătoare celor 4 fire de execuție din pipeline):

1. **Detectare** (Firul 1) – Identificarea țintelor de calibrare
2. **Analiză** (Firul 2) – Examinarea metadatelor imaginii și calcularea calibrării
3. **Calibrare** (Fir 3) – Debayering, corectarea vignetării, calibrare, calcularea indexului
4. **Export** (Fir 4) – Salvarea imaginilor procesate și a indexurilor**Interacțiunea cu bara de progres:*** **Treceți cu mouse-ul** peste bară pentru a vedea panoul derulant detaliat cu 4 etape
* **Faceți clic** pe bara de progres pentru a fixa panoul derulant în poziție
* **Faceți clic din nou** pentru a debloca și a ascunde panoul**Timp de procesare:**

* Semnificativ mai rapid decât modul gratuit
* Accelerarea GPU îmbunătățește și mai mult viteza

{% hint style="info" %}
**Chloros+ Viteză**: Procesarea paralelă poate fi de 5-10 ori mai rapidă decât modul secvențial pentru seturi mari de date. Un proiect de 500 de imagini care durează 2 ore în modul gratuit se poate finaliza în 15-20 de minute cu Chloros+.
{% endhint %}

***

## Ce se întâmplă în timpul procesării

### Etapa 1: Detectarea țintelor

**Ce face Chloros:**

* Scanează imaginile pe care le-ați bifat în coloana „Țintă” (toate imaginile, dacă nu este bifată niciuna)
* Identifică panourile de calibrare din fiecare țintă
* Extrage valorile de reflectanță din panourile țintă
* Înregistrează marcajele temporale ale țintelor pentru programarea calibrării

**Durată:** 1–30 de secunde (cu ținte marcate), 5–30+ minute (nemarcate)

### Etapa 2: Debayering (conversie RAW)

**Ce face Chloros:**

* Convertește datele RAW în format Bayer în imagini complete cu 3 canale (modulele mono LATTICE rămân monobandă — pentru acestea, procesul de debayering este omis, cu o notă în jurnal)
* Aplică algoritmul de demosaic selectat
* Păstrează calitatea maximă a imaginii și detaliile

**Durată:** Variază în funcție de numărul de imagini și de viteza procesorului (CPU) sau a plăcii grafice (GPU)

### Etapa 3: Calibrare

**Ce face Chloros:*** **Corecția vignetării**: Elimină întunecarea produsă de obiectiv la margini
* **Calibrarea reflectanței**: Normalizează folosind valorile țintă ale reflectanței și/sau datele de radiație descendentă din DAQ
* Aplică corecții pe toate benzile/canalele
* Utilizează referința de calibrare adecvată pentru fiecare imagine în funcție de marcajul temporal

**Durată:** Majoritatea timpului de procesare

### Etapa 4: Calculul indicilor

**Ce face Chloros:**

* Calculează indicii multispectrali configurați (NDVI, NDRE etc.)
* Aplică operații matematice pe benzi imaginilor calibrate
* Generează imagini-index pentru fiecare indice selectat

**Durată:** Câteva secunde pe imagine

### Etapa 5: Export

**Ce face Chloros:**

* Salvează imaginile procesate în formatul selectat
* **LATTICE fan-out**: fiecare cadru LATTICE brut este exportat ca fiecare produs activat într-o singură etapă — debayering, previzualizare, radianță (întotdeauna float32), reflectanță
* Scrie fișierele în arborele de ieșire al proiectului: `<project>/<camera>/<format>/<Product>_Images/`
* **Păstrează numele fișierului sursă** — folderul identifică produsul, nu se adaugă niciun sufix**Durată:** Variază în funcție de formatul de export și de dimensiunea fișierului***

## Comportamentul procesării

### Fluxul de procesare automat

Odată pornit, întregul flux se execută automat:

* Nu este necesară interacțiunea utilizatorului
* Toate etapele configurate se execută în ordine
* Actualizări ale progresului afișate în timp real
* Fișierele exportate sunt salvate pe disc pe măsură ce sunt finalizate — puteți deschide rezultatele finalizate în timp ce procesul continuă

### Utilizarea computerului în timpul procesării

**Mod liber:**

* Utilizare relativ redusă a procesorului (un singur thread)
* Calculatorul rămâne receptiv pentru alte sarcini
* Puteți minimiza fereastra Chloros fără probleme și puteți lucra în alte aplicații

**Chloros+ Mod paralel:**

* Utilizare ridicată a procesorului (CPU) în cadrul grupului de procesare al strategiei
* Cu accelerare GPU: utilizare ridicată a GPU-ului
* Calculatorul poate fi mai puțin receptiv în timpul procesării
* Evitați să porniți alte sarcini care solicită intens procesorul (CPU)

{% hint style="warning" %}
**Sfat de performanță**: Pentru o performanță optimă a Chloros+, închideți celelalte aplicații și permiteți Chloros să utilizeze toate resursele sistemului.
{% endhint %}

### Procesarea nu poate fi pusă în pauză (dar oprirea se face complet)

* Odată pornită, procesarea nu poate fi pusă în pauză și reluată ulterior
* Dând clic pe **Stop**, rularea se oprește complet încă de la primul clic
* Produsele deja exportate înainte de oprire rămân pe disc
* O rulare oprită raportează cu exactitate ce a finalizat (consultați liniile `[RUN-SUMMARY]` din jurnal)
* O nouă execuție pornește fluxul de lucru de la început

**Sfat de planificare:** Pentru proiecte foarte mari, luați în considerare procesarea în loturi sau utilizarea CLI pentru un control mai bun.***

## Monitorizarea procesării

În timp ce procesarea se desfășoară, puteți:

* **Urmăriți bara de progres** – Vedeți procentajul general de finalizare
* **Vizualizați etapa curentă** – Detectare, Analiză, Calibrare sau Export
* **Verificați fila „Jurnal”** — Vedeți mesaje detaliate de procesare și avertismente
* **Previzualizați imaginile finalizate** — Fișierele de export apar pe disc în timpul procesării

Pentru informații detaliate despre monitorizare, consultați [Monitorizarea procesării](monitoring-the-processing.md).

***

## Oprirea procesării

Dacă trebuie să opriți procesarea:

### Cum se oprește

1. Localizați **butonul Oprire** (înlocuiește butonul Start în timpul procesării)
2. Faceți clic o dată pe el — bara afișează **„Se oprește...”** în timp ce imaginea în curs de procesare se finalizează
3. Procesul se încheie într-o stare definitivă de oprire, iar jurnalul afișează un raport detaliat `[RUN-SUMMARY]` cu privire la ceea ce a fost finalizat

### Când să opriți procesarea

**Motive valabile pentru oprire:**

* Ați constatat că au fost utilizate setări incorecte
* Ați uitat să marcați imaginile țintă
* Au fost importate imagini greșite
* Sistemul rulează prea lent sau nu răspunde

**După oprire:**

* Produsele exportate înainte de oprire rămân pe disc
* Verificați și remediați eventualele probleme, ajustați setările după cum este necesar
* Reporniți procesarea — procesul începe de la început

***

## Estimări privind durata procesării

Timpul real de procesare variază semnificativ în funcție de:

* Numărul de imagini
* Rezoluția imaginilor
* Formatul de intrare RAW sau JPG
* Modul de procesare (Free sau Chloros+)
* Viteza procesorului și numărul de nuclee
* Disponibilitatea GPU-ului (numai pentru Chloros+)
* Numărul de indici de calculat
* Numărul de produse de export activate (LATTICE)

### Estimări aproximative (Chloros+, imagini de 12 MP, procesor modern)

| Număr de imagini | Mod gratuit | Chloros+ (CPU) | Chloros+ (GPU) |
| ----------- | --------- | -------------- | -------------- |
| 50 de imagini   | 15-20 min | 5-8 min        | 3-5 min        |
| 100 de imagini  | 30-40 min | 10-15 min      | 5-8 min        |
| 200 de imagini  | 1-1,5 ore | 20-30 min      | 10-15 min      |
| 500 de imagini  | 2-3 ore   | 45-60 min      | 20-30 min      |
| 1000 de imagini | 4-6 ore   | 1,5-2 ore      | 40-60 min      |

{% hint style="info" %}
**Prima rulare**: Procesarea inițială poate dura mai mult, deoarece Chloros creează cache-uri și profiluri. Procesarea ulterioară a seturilor de date similare va fi mai rapidă.
{% endhint %}

***

## Probleme frecvente la pornire

### Butonul de pornire este dezactivat (estompat)

**Cauze posibile:**

* Nu au fost importate imagini
* Backend-ul nu a pornit complet
* Procesarea anterioară încă rulează
* Proiectul nu a fost încărcat complet

**Soluții:**

1. Așteptați inițializarea completă a backend-ului (verificați pictograma din meniul principal)
2. Verificați dacă imaginile sunt importate în File Browser
3. Reporniți Chloros dacă butonul rămâne dezactivat
4. Verificați jurnalul de depanare pentru mesaje de eroare

### Procesarea începe, apoi eșuează imediat

**Cauze posibile:**

* Nu există imagini valide în proiect
* Fișiere imagine corupte
* Spațiu insuficient pe disc
* Memorie insuficientă (RAM)

**Soluții:**

1. Verificați jurnalul de depanare <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> pentru mesaje de eroare
2. Verificați spațiul disponibil pe disc
3. Încercați să procesați un subset mai mic de imagini
4. Verificați dacă imaginile nu sunt corupte

### Procesarea se finalizează, dar nu se scriu imagini

O execuție care a solicitat produse imagistice, dar nu a scris niciuna, este tratată ca un **eșec, nu ca un succes** — Chloros raportează acest lucru în mod clar:

* Jurnalul GUI afișează indicii `[RUN-SUMMARY]` care indică cauza probabilă — nu s-au importat imagini, nu s-a detectat nicio țintă sau toate produsele solicitate au fost omise ca fiind inaplicabile (de exemplu, solicitarea de radianță/reflectanță de la camere care utilizează doar RGB)
* Echivalentul CLI (`chloros-cli process`) afișează `Processing finished but wrote no image products.` și **iese cu un cod de ieșire diferit de zero**, astfel încât scripturile să îl poată detecta
* O execuție deliberată numai cu metadate (toate produsele de export dezactivate, fără indici) este totuși considerată reușită

Consultați [Referința CLI](../reference/cli-reference.md#a-run-that-writes-no-images-fails) pentru semantica completă.

### Avertisment „Nu s-au detectat ținte”

**Cauze posibile:**

* Ați uitat să marcați imaginile țintă
* Imaginile țintă nu conțin ținte vizibile
* Setările de detectare a țintelor sunt prea stricte

**Soluții:**

1. Consultați [Alegerea imaginilor țintă](choosing-target-images.md)
2. Marcați imaginile corespunzătoare în coloana „Țintă”
3. Verificați dacă țintele sunt vizibile în imaginile marcate
4. Reglați setările de detectare a țintelor, dacă este necesar

***

## Sfaturi pentru o procesare reușită

### Înainte de a începe

1. **Testați mai întâi cu un subset mic** – Prelucrați 10-20 de imagini pentru a verifica setările
2. **Verificați spațiul disponibil pe disc** – Asigurați-vă că aveți liber un spațiu de 2-3 ori mai mare decât dimensiunea setului de date (mai mult dacă sunt activate toate produsele LATTICE)
3. **Închideți aplicațiile inutile** – Eliberați resursele sistemului
4. **Verificați imaginile țintă** – Previzualizați țintele marcate pentru a vă asigura de calitate
5. **Salvați proiectul** – Proiectul se salvează automat, dar este recomandat să îl salvați manual

### În timpul procesării

1. **Evitați trecerea sistemului în modul de repaus** – Dezactivați modurile de economisire a energiei
2. **Mențineți Chloros în prim-plan** — Sau cel puțin vizibil în bara de activități
3. **Monitorizați ocazional progresul** — Verificați dacă există avertismente sau erori
4. **Nu încărcați alte aplicații grele** – În special în modul paralel Chloros+

### Accelerare GPU Chloros+

Dacă utilizați accelerarea GPU NVIDIA:

1. Actualizați driverele NVIDIA la cea mai recentă versiune
2. Asigurați-vă că GPU-ul are 4 GB+ VRAM (7 GB+ pentru debayering simultan cu suport pentru texturi)
3. Închideți aplicațiile care solicită intens GPU-ul (jocuri, editare video)
4. Monitorizați temperatura GPU-ului (asigurați o răcire adecvată)

***

## Pași următori

Odată ce procesarea a început:

1. **Monitorizați progresul** – Consultați [Monitorizarea procesării](monitoring-the-processing.md)
2. **Așteptați finalizarea** – Procesarea se desfășoară automat
3. **Verificați rezultatele** – Consultați [Finalizarea procesării](finishing-the-processing.md)

Pentru informații despre ce trebuie să faceți în timpul procesării, consultați [Monitorizarea procesării](monitoring-the-processing.md).
