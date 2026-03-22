# Începerea procesării

După ce ați importat imaginile, ați marcat țintele de calibrare și ați configurat setările proiectului, sunteți gata să începeți procesarea. Această pagină vă ghidează prin inițierea fluxului de procesare Chloros.

## Lista de verificare pentru preprocesare

Înainte de a face clic pe butonul Start, verificați dacă totul este pregătit:

* [ ] **Fișiere importate** - Toate imaginile apar în File Browser
* [ ] **Imagini țintă marcate** - Coloana Target este bifată pentru imaginile de calibrare
* [ ] **Modele de cameră detectate** - Coloana Camera Model afișează camerele corecte
* [ ] **Setări configurate** - Setările proiectului au fost verificate și ajustate
* [ ] **Indici selectați** - Indicii multispectrali doriți adăugați (dacă este necesar)
* [ ] **Format de export ales** - Format de ieșire adecvat pentru fluxul dvs. de lucru

{% hint style="info" %}
**Sfat**: Faceți clic pe câteva imagini din File Browser pentru a verifica dacă s-au încărcat corect înainte de procesare.
{% endhint %}

***

## Pornirea procesării

### Localizați butonul Start

Butonul Start/Redare se află în bara de antet superioară a Chloros:

* Poziție: Centrul de sus al ferestrei
* Pictogramă: **Butonul Redare/Start** <img src="../.gitbook/assets/image (2) (1).png" alt="" data-size="line">
* Stare: butonul este activat (luminos) când este gata de procesare

### Faceți clic pentru a începe

1. Faceți clic pe **butonul Redare/Start** din bara de sus
2. Procesarea începe imediat
3. Butonul devine dezactivat (gri) în timpul procesării
4. Bara de progres se actualizează, afișând starea procesării

{% hint style="success" %}
**Procesare începută**: Odată ce ați făcut clic, Chloros gestionează automat toate etapele de procesare - detectarea țintei, debayering, calibrare, calcularea indexului și export.
{% endhint %}

***

## Înțelegerea modurilor de procesare

Chloros funcționează în două moduri de procesare diferite, în funcție de licența dvs.:

### Mod gratuit (procesare secvențială)

**Disponibil pentru toți utilizatorii**

**Cum funcționează:**

* Procesează imaginile una câte una, secvențial
* Funcționare cu un singur thread
* Utilizare redusă a memoriei

**Bara de progres afișează 2 etape:**

1.**Detectare țintă** - Scanare pentru ținte de calibrare
2. **Procesare** - Aplicarea calibrării și exportarea imaginilor**Timp de procesare:**

* Mult mai lent decât modul paralel Chloros+
* Potrivit pentru seturi de date mici și medii (&lt; 200 de imagini)

### Modul Chloros+ (procesare paralelă)

**Necesită licență Chloros+**

**Cum funcționează:**

* Procesează simultan mai multe imagini folosind un [flux de procesare cu 4 thread-uri](../processing-architecture/processing-pipeline.md)
* [Adaptarea dinamică a calculului](../processing-architecture/dynamic-compute-adaptation.md) selectează automat strategia optimă pentru hardware-ul dvs.
* Accelerare GPU (CUDA) cu plăci grafice NVIDIA (desktop și Jetson)
* Scalabil de la un Jetson Nano (1 worker) la un desktop cu GPU de 12 GB+ (3-4 workers)

**Bara de progres afișează 4 etape** (corespunzătoare celor 4 fire de execuție ale pipeline-ului):

1. **Detectare** (Fir 1) - Identificarea țintelor de calibrare
2. **Analiză** (Fir 2) - Examinarea metadatelor imaginii și calcularea calibrării
3. **Calibrare** (Fir 3) - Debayering GPU, corectarea vignetării, calcularea indexului
4. **Export** (Fir 4) - Salvarea imaginilor procesate și a indexurilor**Interacțiunea cu bara de progres:*** **Treceți cu mouse-ul** peste bară pentru a vedea panoul detaliat cu 4 etape
* **Faceți clic** pe bara de progres pentru a fixa panoul derulant în poziție
* **Faceți clic din nou** pentru a debloca și a ascunde panoul**Timp de procesare:**

* Semnificativ mai rapid decât modul gratuit
* Se scalează în funcție de numărul de nuclee CPU
* Accelerarea GPU îmbunătățește și mai mult viteza

{% hint style="info" %}
**Chloros+ Viteză**: Procesarea paralelă poate fi de 5-10 ori mai rapidă decât modul secvențial pentru seturi mari de date. Un proiect de 500 de imagini care durează 2 ore în modul gratuit se poate finaliza în 15-20 de minute cu Chloros+.
{% endhint %}

***

## Ce se întâmplă în timpul procesării

### Etapa 1: Detectarea țintelor

**Ce face Chloros:**

* Scanează imaginile țintă marcate (sau toate imaginile dacă nu este marcată niciuna)
* Identifică cele 4 panouri de calibrare din fiecare țintă
* Extrage valorile de reflectanță din panourile țintă
* Înregistrează marcajele de timp ale țintelor pentru programarea calibrării

**Durată:** 1-30 de secunde (cu ținte marcate), 5-30+ minute (nemarcate)

### Etapa 2: Debayering (conversie RAW)

**Ce face Chloros:**

* Convertește datele RAW în format Bayer în imagini complete RGB
* Aplică un algoritm de demosaic de înaltă calitate
* Păstrează calitatea maximă a imaginii și detaliile

**Durată:** Variază în funcție de numărul de imagini și de viteza procesorului

### Etapa 3: Calibrare

**Ce face Chloros:*** **Corecție vignetă**: Elimină întunecarea lentilelor la margini
* **Calibrare reflectanță**: Normalizează folosind valorile țintă de reflectanță
* Aplică corecții pe toate benzile/canalele
* Folosește ținta de calibrare adecvată pentru fiecare imagine pe baza marcajului temporal

**Durată:** Majoritatea timpului de procesare

### Etapa 4: Calculul indicilor

**Ce face Chloros:**

* Calculează indicii multispectrali configurați (NDVI, NDRE etc.)
* Aplică operații matematice pe benzi imaginilor calibrate
* Generează imagini index pentru fiecare index selectat

**Durată:** Câteva secunde per imagine

### Etapa 5: Export

**Ce face Chloros:**

* Salvează imaginile calibrate în formatul selectat
* Exportează imaginile index cu culorile LUT configurate
* Scrie fișierele în subfolderele modelului de cameră
* Păstrează numele de fișiere originale cu sufixe

**Durată:** Variază în funcție de formatul de export și dimensiunea fișierului***

## Comportamentul procesării

### Fluxul de procesare automat

Odată pornit, întregul flux se execută automat:

* Nu este necesară interacțiunea utilizatorului
* Toate etapele configurate se execută în ordine
* Actualizări ale progresului afișate în timp real

### Utilizarea computerului în timpul procesării

**Modul liber:**

* Utilizare relativ redusă a procesorului (un singur thread)
* Calculatorul rămâne receptiv pentru alte sarcini
* Se poate minimiza Chloros în siguranță și se poate lucra în alte aplicații

**Chloros+ Mod paralel:**

* Utilizare ridicată a procesorului (multi-threaded, până la 16 nuclee)
* Cu accelerare GPU: Utilizare ridicată a GPU-ului
* Computerul poate fi mai puțin receptiv în timpul procesării
* Evitați să porniți alte sarcini care solicită intens procesorul

{% hint style="warning" %}
**Sfat de performanță**: Pentru o performanță optimă, închideți alte aplicații și permiteți programului să utilizeze toate resursele sistemului.
{% endhint %}

### Procesarea nu poate fi întreruptă

**Limitări importante:**

* Odată începută, procesarea nu poate fi întreruptă
* Puteți anula procesarea, dar progresul se pierde
* Rezultatele parțiale nu sunt salvate
* Trebuie să reporniți de la început dacă ați anulat

**Sfat de planificare:** Pentru proiecte foarte mari, luați în considerare procesarea în loturi sau utilizarea CLI pentru un control mai bun.***

## Monitorizarea procesării

În timp ce procesarea rulează, puteți:

* **Urmăriți bara de progres** - Vedeți procentajul general de finalizare
* **Vizualizați etapa curentă** - Detectare, Analiză, Calibrare sau Export
* **Verificați fila jurnal** - Vedeți mesaje detaliate de procesare și avertismente
* **Previzualizați imaginile finalizate** - Unele fișiere de export pot apărea în timpul procesării

Pentru informații detaliate despre monitorizare, consultați [Monitorizarea procesării](monitoring-the-processing.md).

***

## Anularea procesării

Dacă trebuie să opriți procesarea:

### Cum se anulează

1. Localizați **butonul Oprire/Anulare** (înlocuiește butonul Start în timpul procesării)
2. Faceți clic pe butonul Oprire
3. Procesarea se oprește imediat
4. Rezultatele parțiale sunt eliminate

### Când să anulați

**Motive valabile pentru anulare:**

* V-ați dat seama că au fost utilizate setări incorecte
* Ați uitat să marcați imaginile țintă
* Au fost importate imagini greșite
* Sistemul rulează prea lent sau nu răspunde

**După anulare:**

* Verificați și remediați eventualele probleme
* Reglați setările după cum este necesar
* Reporniți procesarea de la început
* Pentru o experiență optimă, închideți complet Chloros și reporniți

{% hint style="warning" %}
**Fără rezultate parțiale**: Anularea anulează tot progresul. Chloros nu salvează imaginile procesate parțial.
{% endhint %}

***

## Estimări privind timpul de procesare

Timpul real de procesare variază foarte mult în funcție de:

* Numărul de imagini
* Rezoluția imaginii
* Formatul de intrare RAW vs JPG
* Modul de procesare (Free vs Chloros+)
* Viteza procesorului și numărul de nuclee
* Disponibilitatea GPU-ului (numai Chloros+)
* Numărul de indici de calculat
* Complexitatea formatului de export

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

### Butonul Start dezactivat (gri)

**Cauze posibile:**

* Nu s-au importat imagini
* Backend-ul nu a pornit complet
* Procesarea anterioară încă rulează
* Proiectul nu s-a încărcat complet

**Soluții:**

1. Așteptați ca backend-ul să se inițializeze complet (verificați pictograma din meniul principal)
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

### Avertisment „Nu s-au detectat ținte”

**Cauze posibile:**

* Ați uitat să marcați imaginile țintă
* Imaginile țintă nu conțin ținte vizibile
* Setările de detectare a țintelor sunt prea stricte

**Soluții:**

1. Consultați [Alegerea imaginilor țintă](choosing-target-images.md)
2. Marcați imaginile corespunzătoare în coloana Țintă
3. Verificați dacă țintele sunt vizibile în imaginile marcate
4. Reglați setările de detectare a țintelor, dacă este necesar

***

## Sfaturi pentru o procesare reușită

### Înainte de a începe

1. **Testați mai întâi cu un subset mic** - Procesați 10-20 de imagini pentru a verifica setările
2. **Verificați spațiul disponibil pe disc** - Asigurați-vă că aveți de 2-3 ori mai mult spațiu liber decât dimensiunea setului de date
3. **Închideți aplicațiile inutile** - Eliberați resursele sistemului
4. **Verificați imaginile țintă** - Previzualizați țintele marcate pentru a vă asigura de calitate
5. **Salvați proiectul** - Proiectul se salvează automat, dar este recomandat să salvați manual

### În timpul procesării

1. **Evitați trecerea sistemului în modul de repaus** - Dezactivați modurile de economisire a energiei
2. **Păstrați Chloros în prim-plan** - Sau cel puțin vizibil în bara de activități
3. **Monitorizați progresul ocazional** - Verificați dacă există avertismente sau erori
4. **Nu încărcați alte aplicații grele** - În special cu modul paralel Chloros+

### Chloros+ Accelerare GPU

Dacă utilizați accelerarea GPU NVIDIA:

1. Actualizați driverele NVIDIA la cea mai recentă versiune
2. Asigurați-vă că GPU-ul are 4 GB+ VRAM
3. Închideți aplicațiile care solicită intens GPU-ul (jocuri, editare video)
4. Monitorizați temperatura GPU-ului (asigurați-vă că răcirea este adecvată)

***

## Pași următori

Odată ce procesarea a început:

1. **Monitorizați progresul** - Consultați [Monitorizarea procesării](monitoring-the-processing.md)
2. **Așteptați finalizarea** - Procesarea se desfășoară automat
3. **Verificați rezultatele** - Consultați [Finalizarea procesării](finishing-the-processing.md)

Pentru informații despre ce trebuie să faceți în timpul procesării, consultați [Monitorizarea procesării](monitoring-the-processing.md).
