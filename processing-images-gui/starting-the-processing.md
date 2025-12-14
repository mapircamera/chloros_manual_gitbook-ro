# Începerea procesării

După ce ați importat imaginile, ați marcat țintele de calibrare și ați configurat setările proiectului, sunteți gata să începeți procesarea. Această pagină vă ghidează prin inițierea procesului de procesare Chloros.

## Lista de verificare înainte de procesare

Înainte de a face clic pe butonul Start, verificați dacă totul este pregătit:

* [ ] **Fișiere importate** - Toate imaginile apar în File Browser
* [ ] **Imagini țintă marcate** - Coloana țintă verificată pentru imagini de calibrare
* [ ] **Modele de cameră detectate** - Coloana Model cameră afișează camerele corecte
* [ ] **Setări configurate** - Setările proiectului revizuite și ajustate
* [ ] **Indici selectați** - Indici multispectrali doriti adăugați (dacă este necesar)
* [ ] **Format de export ales** - Format de ieșire adecvat pentru fluxul dvs. de lucru

{% hint style=&quot;info&quot; %}
**Sfat**: Faceți clic pe câteva imagini în File Browser pentru a verifica dacă s-au încărcat corect înainte de procesare.
{% endhint %}

***

## Începerea procesării

### Localizați butonul Start

Butonul Start/Play se află în bara de antet din partea de sus a Chloros:

* Poziție: în centrul ferestrei, în partea de sus
* Pictogramă: **Butonul Play/Start** <img src="../.gitbook/assets/image (2).png" alt="" data-size="line">
* Stare: Butonul este activat (luminos) când este gata de procesare

### Faceți clic pentru a începe

1. Faceți clic pe **butonul Redare/Start** din bara de sus
2. Procesarea începe imediat
3. Butonul devine dezactivat (gri) în timpul procesării
4. Bara de progres se actualizează, afișând starea procesării

{% hint style=&quot;success&quot; %}
**Procesare începută**: Odată ce ați făcut clic, Chloros gestionează automat toate etapele de procesare - detectarea țintei, debayering, calibrare, calculul indexului și exportul.
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

1. **Detectarea țintei** - Scanarea țintelor de calibrare
2. **Procesare** - Aplicarea calibrării și exportarea imaginilor

**Timp de procesare:**

* Mult mai lent decât modul paralel Chloros+
* Potrivit pentru seturi de date mici și medii (&lt; 200 de imagini)

### Modul Chloros+ (procesare paralelă)

**Necesită licență Chloros+**

**Cum funcționează:**

* Procesează simultan mai multe imagini
* Funcționare multi-thread (până la 16 procesoare paralele)
* Utilizează mai multe nuclee CPU
* Accelerare GPU (CUDA) opțională cu plăci grafice NVIDIA

**Bara de progres afișează 4 etape:**

1. **Detectare** - Găsirea țintelor de calibrare
2. **Analizare** - Examinarea metadatelor imaginii și pregătirea pipeline-ului
3. **Calibrare** - Aplicarea corecțiilor și calibrărilor
4. **Exportare** - Salvarea imaginilor și indexurilor procesate

**Interacțiunea cu bara de progres:**

* **Treceți mouse-ul** peste bară pentru a vedea panoul derulant detaliat cu 4 etape
* **Faceți clic** pe bara de progres pentru a bloca panoul derulant în poziție
* **Faceți clic din nou** pentru a debloca și ascunde panoul

**Timp de procesare:**

* Semnificativ mai rapid decât modul gratuit
* Se scalează în funcție de numărul de nuclee CPU
* Accelerarea GPU îmbunătățește și mai mult viteza

{% hint style=&quot;info&quot; %}
**Chloros+ Viteză**: Procesarea paralelă poate fi de 5-10 ori mai rapidă decât modul secvențial pentru seturi de date mari. Un proiect de 500 de imagini care durează 2 ore în modul gratuit poate fi finalizat în 15-20 de minute cu Chloros+.
{% endhint %}

***

## Ce se întâmplă în timpul procesării

### Etapa 1: Detectarea țintei

**Ce face Chloros:**

* Scanează imaginile țintă marcate (sau toate imaginile dacă nu sunt marcate)
* Identifică cele 4 panouri de calibrare din fiecare țintă
* Extrage valorile de reflexie din panourile țintă
* Înregistrează marcajele temporale ale țintelor pentru programarea calibrării

**Durată:** 1-30 secunde (cu ținte marcate), 5-30+ minute (nemarcate)

### Etapa 2: Debayering (conversie RAW)

**Ce face Chloros:**

* Convertește datele RAW în format Bayer în imagini complete RGB
* Aplică un algoritm de demosaicare de înaltă calitate
* Păstrează calitatea maximă a imaginii și detaliile

**Durată:** Variază în funcție de numărul de imagini și viteza procesorului

### Etapa 3: Calibrare

**Ce face Chloros:**

* **Corecție vignette**: Elimină întunecarea lentilelor la margini
* **Calibrarea reflectanței**: Normalizează utilizând valorile țintă ale reflectanței
* Aplică corecții pe toate benzile/canalele
* Utilizează ținta de calibrare adecvată pentru fiecare imagine pe baza timestamp-ului

**Durată:** Majoritatea timpului de procesare

### Etapa 4: Calculul indexului

**Ce face Chloros:**

* Calculează indicii multispectrali configurați (NDVI, NDRE etc.)
* Aplică matematică de bandă imaginilor calibrate
* Generează imagini index pentru fiecare indice selectat

**Durată:** Câteva secunde per imagine

### Etapa 5: Export

**Ce face Chloros:**

* Salvează imaginile calibrate în formatul selectat
* Exportă imagini index cu culori LUT configurate
* Scrie fișiere în subfoldere ale modelului de cameră
* Păstrează numele fișierelor originale cu sufixe

**Durată:** Variază în funcție de formatul de export și dimensiunea fișierului

***

## Comportamentul procesării

### Pipeline de procesare automată

Odată pornit, întregul pipeline rulează automat:

* Nu este necesară interacțiunea utilizatorului
* Toate etapele configurate se execută în ordine
* Actualizări ale progresului afișate în timp real

### Utilizarea computerului în timpul procesării

**Mod liber:**

* Utilizare relativ redusă a procesorului (single-threaded)
* Computerul rămâne receptiv pentru alte sarcini
* Se poate minimiza Chloros și se poate lucra în alte aplicații în siguranță

**Chloros+ Mod paralel:**

* Utilizare ridicată a procesorului (multi-threaded, până la 16 nuclee)
* Cu accelerare GPU: utilizare ridicată a GPU-ului
* Computerul poate fi mai puțin receptiv în timpul procesării
* Evitați pornirea altor sarcini care solicită intens procesorul

{% hint style=&quot;warning&quot; %}
**Sfat de performanță**: Pentru o performanță optimă a Chloros+, închideți alte aplicații și lăsați Chloros să utilizeze toate resursele sistemului.
{% endhint %}

### Procesarea nu poate fi întreruptă

**Limitări importante:**

* Odată începută, procesarea nu poate fi întreruptă
* Puteți anula procesarea, dar progresul va fi pierdut
* Rezultatele parțiale nu sunt salvate
* Dacă anulați, trebuie să reporniți de la început

**Sfat de planificare:** Pentru proiecte foarte mari, luați în considerare procesarea în loturi sau utilizarea CLI pentru un control mai bun.

***

## Monitorizarea procesării

În timp ce procesarea rulează, puteți:

* **Urmărirea barei de progres** - Vedeți procentajul general de finalizare
* **Vizualizarea etapei curente** - Detectare, analiză, calibrare sau export
* **Verificați fila jurnal** - Vedeți mesaje și avertismente detaliate privind procesarea
* **Previzualizați imaginile finalizate** - Unele fișiere exportate pot apărea în timpul procesării

Pentru informații detaliate privind monitorizarea, consultați [Monitorizarea procesării](monitoring-the-processing.md).

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

* S-a constatat că au fost utilizate setări incorecte
* Ați uitat să marcați imaginile țintă
* Au fost importate imagini greșite
* Sistemul funcționează prea lent sau nu răspunde

**După anulare:**

* Verificați și remediați eventualele probleme
* Reglați setările după cum este necesar
* Reporniți procesarea de la început
* Pentru o experiență optimă, închideți complet Chloros și reporniți

{% hint style=&quot;warning&quot; %}
**Fără rezultate parțiale**: Anularea elimină toate progresele. Chloros nu salvează imaginile procesate parțial.
{% endhint %}

***

## Estimări privind timpul de procesare

Timpul real de procesare variază foarte mult în funcție de:

* Numărul de imagini
* Rezoluția imaginii
* Formatul de intrare RAW vs JPG
* Modul de procesare (Free vs Chloros+)
* Viteza procesorului și numărul de nuclee
* Disponibilitatea GPU (numai Chloros+)
* Numărul de indici de calculat
* Complexitatea formatului de export

### Estimări aproximative (Chloros+, imagini de 12 MP, CPU modern)

| Număr de imagini | Mod gratuit | Chloros+ (CPU) | Chloros+ (GPU) |
| ----------- | --------- | -------------- | -------------- |
| 50 imagini   | 15-20 min | 5-8 min        | 3-5 min        |
| 100 imagini  | 30-40 min | 10-15 min      | 5-8 min        |
| 200 imagini  | 1-1,5 ore | 20-30 min      | 10-15 min      |
| 500 imagini  | 2-3 ore   | 45-60 min      | 20-30 min      |
| 1000 imagini | 4-6 ore   | 1,5-2 ore      | 40-60 min      |

{% hint style=&quot;info&quot; %}
**Prima rulare**: Procesarea inițială poate dura mai mult, deoarece Chloros creează cache-uri și profiluri. Procesarea ulterioară a seturilor de date similare va fi mai rapidă.
{% endhint %}

***

## Probleme frecvente la pornire

### Butonul Start dezactivat (gri)

**Cauze posibile:**

* Nu au fost importate imagini
* Backend-ul nu a pornit complet
* Procesarea anterioară încă rulează
* Proiectul nu a fost încărcat complet

**Soluții:**

1. Așteptați inițializarea completă a backend-ului (verificați pictograma din meniul principal)
2. Verificați dacă imaginile sunt importate în File Browser (Browser fișiere)
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
2. **Verificați spațiul disponibil pe disc** - Asigurați-vă că aveți spațiu liber de 2-3 ori mai mare decât dimensiunea setului de date
3. **Închideți aplicațiile inutile** - Eliberați resursele sistemului
4. **Verificați imaginile țintă** - Previzualizați țintele marcate pentru a vă asigura de calitate
5. **Salvați proiectul** - Proiectul se salvează automat, dar este recomandat să îl salvați manual

### În timpul procesării

1. **Evitați intrarea sistemului în stare de repaus** - Dezactivați modurile de economisire a energiei
2. **Păstrați Chloros în prim-plan** - Sau cel puțin vizibil în bara de activități
3. **Monitorizați progresul ocazional** - Verificați dacă există avertismente sau erori
4. **Nu încărcați alte aplicații grele** - În special cu Chloros+ în modul paralel

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
2. **Așteptați finalizarea** - Procesarea se execută automat
3. **Verificați rezultatele** - Consultați [Finalizarea procesării](finishing-the-processing.md)

Pentru informații despre ce trebuie să faceți în timpul procesării, consultați [Monitorizarea procesării](monitoring-the-processing.md).
