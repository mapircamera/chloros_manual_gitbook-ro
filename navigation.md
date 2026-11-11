# Interfața grafică: Navigare

Când porniți Chloros pentru prima dată, acesta inițializează modulul de procesare din fundal. Odată ce modulul este gata, apare pictograma meniului principal din colțul din stânga sus <img src=".gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line">, iar filele „Camere” și „Senzori de lumină” devin active în bara laterală din stânga (până atunci, acestea sunt estompate).

<figure><img src=".gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

De la stânga la dreapta, antetul superior conține:

### <img src=".gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line"> Meniul principal

<figure><img src=".gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>Din meniul principal puteți:

* **Proiect nou**— creați un proiect nou. Dacă aveți șabloane de proiect salvate, apare un meniu derulant**Selectare șablon**, astfel încât noul proiect să pornească de la setările unui șablon.
* **Deschidere proiect**— deschideți un proiect existent. Lista include un buton**Deschide dosarul proiectului** care deschide dosarul proiectelor în exploratorul de fișiere.
* **Duplică proiect** — copiază proiectul deschis în prezent sub un nume nou (se sugerează un nume liber, cum ar fi „Proiectul meu (2)”) și deschide copia. _(vizibil după deschiderea unui proiect)_
* **Adăugare fișiere** — adaugă fișiere imagine individuale la proiectul curent _(vizibil după deschiderea unui proiect)_
* **Adăugare folder** — adaugă unul sau mai multe foldere cu imagini la proiectul curent _(vizibil după deschiderea unui proiect)_
* **Pornește procesarea / Oprește procesarea** — pornește sau oprește fluxul de procesare a imaginilor _(activat după adăugarea fișierelor)_
* **Conectează-te la cameră** — accesează [fila Camere](lattice/) pentru a conecta o cameră sau o matrice LATTICE. Funcționează fără a avea un proiect deschis.
* **Conectare la senzorul de lumină** — accesează [fila Senzori de lumină](daq/) pentru a conecta un senzor de lumină DAQ. Funcționează fără a avea un proiect deschis.

{% hint style="info" %}
**Numai pentru Windows**: Interfața grafică Chloros

Desktop este disponibilă pe Windows

. Utilizatorii de Linux

ar trebui să consulte documentația privind [CLI

](CLI.md) și [Python

SDK

](api-python-sdk.md) pentru procesarea fără interfață grafică.
{% endhint %}

### <img src=".gitbook/assets/image (2) (1) (1).png" alt="" data-size="line">

Butonul Redare/Start

Când este activat, butonul de pornire a procesării inițiază fluxul de procesare a imaginilor.

### <img src=".gitbook/assets/image (4).png" alt="" data-size="line">

Bara de progres <img src=".gitbook/assets/image (5).png" alt="" data-size="line">



În modul gratuit Chloros

, care procesează toate fișierele secvențial, bara de progres va afișa 2 etape: Detectarea țintei și Procesarea.

În modul cu licență Chloros

+, care procesează toate fișierele simultan, bara de progres afișează 4 etape: Detectare, Analiză, Calibrare, Export. Dacă treceți cursorul mouse-ului peste bara de progres Chloros

+, se va afișa un panou extins cu cele 4 etape ale barei de progres, astfel încât să puteți urmări procesul. Dacă faceți clic pe bara de progres de sus, panoul derulant va fi blocat; dacă faceți clic din nou, acesta va fi deblocat.

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

## Meniul lateral

Meniul din bara laterală din stânga conține diverse pictograme cu care puteți interacționa, în această ordine de sus în jos:

#### <img src=".gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> [Setări proiect](project-settings/project-settings.md)

Fila „Setări proiect” vă permite să ajustați setările globale ale proiectului și setările de procesare ale acestuia. Reglați aceste setări înainte de a începe procesarea fișierelor.

#### <img src=".gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> Răsfoitor de fișiere

Adăugați fișiere/foldere și eliminați fișiere din proiect. Fișierele duplicate sunt ignorate. Bifați caseta din coloana „țintă” pentru orice imagine țintă, iar procesarea va lua în considerare doar imaginile bifate ca ținte, accelerând considerabil timpul de procesare. Utilizați comutatorul „Imagine/Metadate” pentru a comuta între vizualizarea grilei de miniaturi a imaginii selectate și un tabel detaliat de metadate.

#### <img src=".gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> [Vizualizator de imagini](image-viewer-gui/opening-an-image-full-screen.md)

Când faceți clic pe o imagine în vizualizatorul principal de imagini, aceasta se deschide pe ecran complet în fila Vizualizator de imagini.

#### <img src=".gitbook/assets/image (3) (1).png" alt="" data-size="line"> [Vizualizator de hărți](image-viewer-gui/map-markers.md)

Vizualizați-vă imaginile pe o hartă 2D interactivă pe baza coordonatelor GPS ale acestora. Suportă furnizorii de plăci Google Maps și ESRI, selectând automat cel mai bun serviciu pentru locația dvs. Treceți cu cursorul peste marcaje pentru a vedea previzualizări miniaturale ale imaginilor.

#### <img src=".gitbook/assets/image (17).png" alt="" data-size="line"> [Camere](lattice/)

Conectați și controlați camerele LATTICE în timp real — câte una sau ca matrice sincronizate de mai multe camere. Fila afișează imagini de previzualizare în timp real cu suprapuneri și histograme, setări pentru fiecare cameră și pentru fiecare grup de camere, precum și setări de captură care determină ce camere și ce tipuri de export generează opțiunea „Capture All”. Disponibil odată ce backend-ul este pregătit; consultați [secțiunea LATTICE](lattice/) pentru ghidul complet.

#### <img src=".gitbook/assets/image (23).png" alt="" data-size="line"> [Senzori de lumină](daq/)

Conectați senzorii de lumină DAQ — DAQ-U (USB), DAQ-M (Bluetooth) și DAQ-E (Ethernet) — și vizualizați graficele lor spectrale calibrate în timp real, exprimate în W/m²/nm. De aici puteți înregistra fișiere `.daq` în proiectul deschis, puteți redenumi senzorii, puteți alege profiluri de corecție a capacității și puteți actualiza firmware-ul DAQ-E. Disponibil odată ce backend-ul este pregătit; consultați [secțiunea DAQ](daq/) pentru ghidul complet.

#### <img src=".gitbook/assets/icon_log.JPG" alt="" data-size="line"> Jurnal de depanare

Verificați jurnalul pentru mesaje de depanare atunci când apar probleme. Copiați/descărcați jurnalul și trimiteți-l către [Asistența MAPIR](https://www.mapir.camera/community/contact) pentru asistență.

Noutate în versiunea 1.3.0: pictograma de eroare din partea de sus a filei jurnalului (**Raportați o eroare**) trimite către MAPIR un raport cu descrierea dvs., până la 10 imagini, jurnalele Chloros și detaliile sistemului. Trebuie să fiți autentificat pentru a trimite un raport. Consultați [Raportarea unei erori din fila jurnal](processing-images-gui/monitoring-the-processing.md#reporting-a-bug-from-the-log-tab).

#### <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> [Autentificare utilizator](chloros+-login.md)

Bara laterală de autentificare a utilizatorului vă permite să vă autentificați în contul dvs. Chloros+ pentru a debloca funcții avansate. De asemenea, puteți vizualiza versiunea curentă a aplicației, precum și seta limba textului afișat în interfața grafică Chloros și în CLI.
