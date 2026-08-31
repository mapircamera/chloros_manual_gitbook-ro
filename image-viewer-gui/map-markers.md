# Marcaje pe hartă

Fila „Hartă” afișează imaginile pe o hartă 2D interactivă pe baza coordonatelor GPS ale acestora. Aceasta vă oferă o imagine de ansamblu geografică a unei sesiuni de captură și reprezintă cea mai rapidă modalitate, imediat după import, de a elimina imaginile pe care nu doriți să le prelucrați.

<figure><img src="../.gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

## Accesarea filei „Hartă”

1. Deschideți sau creați un proiect în Chloros
2. Importați imaginile care conțin metadate GPS
3. Faceți clic pe fila **„Hartă”** <img src="../.gitbook/assets/image (3) (1).png" alt="" data-size="line"> din bara laterală din stânga
4. Harta afișează un marcaj la locația GPS a fiecărei imagini

{% hint style="info" %}
**GPS obligatoriu**: pe hartă apar doar imaginile cu coordonate GPS în metadatele EXIF. O imagine fără coordonate rămâne în proiect și se procesează în mod normal — pur și simplu nu are marcaj.
{% endhint %}

***

## Modificarea imaginilor din fila „Harta”

Fila **Harta**<img src="../.gitbook/assets/image (3) (1).png" alt="" data-size="line"> dispune de aceleași butoane de adăugare <img src="../.gitbook/assets/image (3).png" alt="" data-size="line"> <img src="../.gitbook/assets/image (1) (1).png" alt="" data-size="line"> și eliminare <img src="../.gitbook/assets/image (2) (1).png" alt="" data-size="line"> ca și fila [**Browser fișiere**](../processing-images-gui/adding-files-to-a-project.md) <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line">. Aceasta afișează aceeași listă de fișiere de proiect, cu coloane geografice:

| Coloană        | Conținut                                                           |
| ------------- | ------------------------------------------------------------------ |
| **Nume**      | Numele fișierului așa cum a fost înregistrat de aparatul foto                             |
| **Latitudine**  | Grade zecimale, șase zecimale                                |
| **Longitudine** | Grade zecimale, șase zecimale                                |
| **Altitudine**  | Metri, o zecimală — `-` când imaginea nu conține altitudine |

{% hint style="info" %}
Faceți clic pe orice antet de coloană pentru a sorta după acesta; faceți clic din nou pentru a inversa ordinea.
{% endhint %}

{% hint style="warning" %}
**Altitudinea reprezintă înălțimea față de nivelul mării, nu față de sol.** Valoarea provine din eticheta EXIF `GPSAltitude` a imaginii, care se referă la nivelul mediu al mării. Nu este vorba de înălțimea de zbor față de teren, iar Chloros nu va calcula distanța dintre eșantioane la sol pe baza acesteia — deasupra unui câmp situat la 300 m deasupra nivelului mării, o dronă la 100 m AGL înregistrează aproximativ 400 m în această coloană. Utilizați coloana pentru a identifica valorile aberante și pentru a confirma o altitudine de zbor constantă, nu ca măsurătoare AGL.
{% endhint %}

***

## Marcaje de imagine

Fiecare imagine cu date GPS primește un marcaj la coordonatele sale.

### Afișarea markerilor

* Markerii sunt poziționați exact la coordonatele înregistrate pentru fiecare captură
* Markerii aflați la distanță mică unul de altul se pot suprapune vizual atunci când se micșorează imaginea — măriți imaginea pentru a-i separa
* Markerii selectați și evidențiați sunt afișați deasupra celorlalți

### Previzualizare la trecerea cursorului

* **Treceți cursorul** peste orice marcator pentru a afișa o miniatură a acelei imagini împreună cu numele fișierului
* **Faceți clic**pe un marcaj pentru a selecta imaginea și a**fixa** fereastra pop-up deschisă — aceasta rămâne afișată până când faceți clic în altă parte. Cât timp fereastra pop-up este fixată, trecerea cu mouse-ul peste alte marcaje nu o va ascunde
* Aceasta este modalitatea rapidă de a găsi un anumit cadru dintr-o sesiune mare, fără a părăsi harta

<figure><img src="../.gitbook/assets/image (36).png" alt=""><figcaption><p>Fila „Hartă” afișează toate imaginile geotaguite din proiect</p></figcaption></figure>### Super-zoom

{% hint style="success" %}
**SUPER-ZOOM**: când atingeți nivelul maxim de zoom pentru care furnizorul de plăci dispune de imagini, mărirea suplimentară va mări plăcile în loc să se oprească, astfel încât să puteți distinge marcajele care se află aproape unul peste altul.
{% endhint %}

* Funcția super-zoom se activează numai atunci când vă aflați **la** nivelul maxim de zoom al furnizorului pentru acea locație și când încărcarea secțiunilor de hartă s-a finalizat. Sub acest nivel, zoomul funcționează normal
* Intervalul este de **1× până la 32×** peste nivelul maxim al furnizorului
* Un indicator din colț afișează nivelul actual de super-zoom sub formă de procent, iar un buton **×** de lângă acesta vă readuce la zoomul normal cu un singur clic
* Reducerea zoomului se aplică întotdeauna hărții propriu-zise, astfel încât nu puteți rămâne blocat niciodată în modul super-zoom
* Zoomul și panoramarea în timpul super-zoomului transferă decalajul rezultat înapoi către hartă, astfel încât zona descentrată în care v-ați deplasat continuă să solicite secțiuni, în loc să rămână goală
* Marcajele sunt desenate ca elemente vectoriale, nu rasterizate, astfel încât rămân clare la orice nivel de super-zoom

***

## Furnizori de secțiuni de hartă

{% hint style="success" %}
**Selecție automată**: Chloros alege serviciul de secțiuni care oferă cel mai bun nivel de zoom pentru locația imaginilor tale. Poți comuta manual în orice moment.
{% endhint %}

| Furnizor        | Note                                                                                                                                                             |
| --------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Google Maps** | Acoperire globală extinsă; acceptă toate cele patru tipuri de plăci                                                                                                            |
| **Esri ArcGIS**| Imagini aeriene adesea cu rezoluție mai mare în anumite regiuni. Tipul de placă**Terrain** nu este disponibil pentru Esri, iar butonul corespunzător este dezactivat atunci când este selectat Esri |***

## Tipuri de plăci de hartă

Alegeți tipul stratului de hartă folosind butoanele (de la stânga la dreapta):

![](&lt;../.gitbook/assets/image (14).png&gt;)

| Tip                 | Afișează                                                                |
| -------------------- | -------------------------------------------------------------------- |
| **Relief**          | Umbrire în funcție de altitudine cu detalii cartografice (drumuri, etichete). Numai Google       |
| **Hartă**              | Plăci standard de hartă stradală — opțiunea cu cea mai mică lățime de bandă              |
| **Satelit**        | Imagini detaliate din satelit, fără etichete — opțiunea cu cea mai mare lățime de bandă |
| **Hibrid** (implicit) | Imagini din satelit cu drumuri și etichete suprapuse                |

Fila „Harta” se deschide pe modul **Hibrid**. Alegerea dvs. se aplică și la schimbarea furnizorului, acolo unde acesta suportă această opțiune.***

## Navigarea pe hartă

* **Zoom**: rotița de derulare a mouse-ului sau butoanele de zoom de pe hartă
* **Panoramare**: faceți clic și trageți
* **Ecran complet**: butonul de ecran complet extinde harta pe întreaga fereastră***

## Cazuri de utilizare

### Analiza traseului de zbor

* Vizualizați dintr-o privire aria de acoperire a unei sesiuni cu drona
* Identificați lacunele unde s-a omis o trecere
* Confirmați că zborul a urmat traseul planificat

### Analiza măsurătorilor la sol

* Vedeți cum sunt distribuite capturile de la sol
* Localizați cadrele țintă de calibrare în raport cu zona de măsurare
* Decideți unde sunt necesare capturi suplimentare

### Controlul calității

* Identificați imaginile capturate în locuri neașteptate și eliminați-le înainte de procesare
* Sortați după altitudine pentru a identifica un cadru capturat la o înălțime incorectă sau unul în care poziția GPS a fost slabă
* Verificați corespondența dintre locațiile imaginilor și notele de teren

***

## Depanare

### Nu apar marcaje

**Cauze posibile**

* Imaginile nu conțin metadate GPS
* GPS-ul a fost dezactivat pe cameră în timpul capturării
* Datele EXIF au fost eliminate de alt software înainte de import

**Ce trebuie făcut**: asigurați-vă că GPS-ul este activat pe aparatul foto și reimportați fișierele originale. Puteți verifica dacă un anumit fișier are coordonate căutându-l în tabelul de fișiere din fila Hartă — o imagine fără coordonate nu are niciun rând acolo.

### Marcajele sunt amplasate greșit

**Cauze posibile**: o poziționare precară prin satelit în momentul capturării sau abaterea semnalului GPS în timpul sesiunii.**Ce trebuie făcut**: aceasta este o problemă legată de momentul capturării, nu ceva ce Chloros poate corecta ulterior. Pentru lucrări de precizie, utilizați un flux de lucru GPS PPK/RTK — consultați setarea**Aplicare corecții PPK** din [Setări proiect](../project-settings/project-settings.md).

### Harta este goală sau încărcarea secțiunilor se oprește

Furnizorii de segmente sunt servicii online. Dacă segmentele nu mai sunt încărcate, verificați conexiunea la rețea a dispozitivului, apoi încercați să schimbați furnizorul. Dacă ați mărit foarte mult imaginea, apăsați butonul de resetare **×** pentru a reveni la un nivel normal de zoom și permiteți hărții să solicite din nou segmentele.***

## Pagini conexe

* [**Grila de imagini**](image-grid.md) — același set de imagini ca și miniaturile
* [**Deschiderea unei imagini pe ecran complet**](opening-an-image-full-screen.md) — inspectarea detaliată a unei imagini
* [**Adăugarea de fișiere la un proiect**](../processing-images-gui/adding-files-to-a-project.md) — butoanele de adăugare/eliminare a fișierelor comune acestei file
