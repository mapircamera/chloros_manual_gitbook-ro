# Rețele cu mai multe camere

O **matrice**LATTICE este formată din două sau mai multe camere LATTICE conectate ca o singură unitate sincronizată. O cameră este**master**: aceasta emite un impuls de declanșare GPIO hardware pe o linie de sincronizare partajată (implicit**Line2**), astfel încât fiecare componentă să expună același moment. Chloros adaugă sincronizarea temporală PTP, o previzualizare în timp real (segmente per cameră sau o singură imagine compusă multibandă aliniată) și captură sincronizată — fiecare trecere de captură produce un**grup de cadre** în care toate camerele împărtășesc același timestamp și același ID de cadru (raportat ca `fid:N` la ieșirea de captură).

Matricile reprezintă modul în care camerele mono (M3M) generează indici de vegetație — o cameră contribuie cu o bandă, iar matricea le aliniază într-un stack multibandă. Consultați [Camere mono și indici de vegetație](mono-indices.md).

Există trei moduri echivalente de a conecta o matrice, iar toate rulează același flux „smart-prep”:

| Interfață | Punct de intrare |
| --- | --- |
| Interfață grafică | Fila „Camere” → **Conectare matrice** (buton albastru) |
| CLI | `chloros-cli lattice array-connect --serials SN1,SN2,…` (primul număr de serie = master) |
| Python SDK | `connect_array(serials=[…])` → `ArraySession` (primul număr de serie = master) |

Smart-prep efectuează, în ordine: o verificare a capacității rețelei (ping ICMP DF + sondă GVSP), selectarea nivelului de sincronizare, reducerea automată a dimensiunii cadrului pentru a se potrivi cu lățimea de bandă, activarea PTP, selectarea automată a formatului de pixeli pentru fiecare cameră, inițializarea automată a expunerii pe baza stării salvate a fiecărei camere și configurarea declanșatorului GPIO pe Line2.

{% hint style="info" %}
Camerele trebuie să fie accesibile pe legătură înainte ca oricare dintre aceste operațiuni să funcționeze — consultați [Conectarea camerelor](connecting.md) pentru detectare, adresare și descărcarea calibrării la prima conectare. Pentru configurațiile cu mai multe camere, setările inelului de recepție al plăcii de rețea a gazdei sunt la fel de importante ca viteza legăturii; tabelul complet cu simptome→soluții se găsește în [Referința CLI § Configurarea și reglarea plăcii de rețea gazdă](../reference/cli-reference.md#host-nic-setup--tuning-lattice-arrays).
{% endhint %}

## Dialogul „Conectare matrice”

Fila „Camere” → **Conectare matrice**deschide un asistent în trei pași:**Selectare → Mod de afișare → Setări**.

### Pasul 1 — Selectarea camerei principale și a celor secundare



<!-- SCREENSHOT-NEEDED: Array Connect dialog, Select scene, with 3-4 LATTICE cameras discovered. Table showing Camera / Serial / IP / Master radio / Slave checkbox columns, with the green "GPIO master detected — selections pre-populated" probe banner visible above the table. -->Fereastra de dialog scanează rețeaua imediat ce se deschide („Scanează rețeaua...”), apoi verifică cablajul declanșatorului GPIO („Verifică cablajul GPIO...”). Aveți nevoie de cel puțin **2 camere** pentru a construi o matrice.

Verificarea cablajului precompletează selecția rolurilor atunci când este posibil și afișează unul dintre cele trei bannere:

| Mesaj | Semnificație |
| --- | --- |
| „Maestru GPIO detectat — selecții completate automat” (verde) | Testul a identificat topologia de declanșare; casetele de selectare pentru „maestru” și „slave” sunt deja bifate. |
| „Nu s-a detectat niciun master – verificați cablul GPIO” (portocaliu) | Nicio cameră nu a detectat un impuls de declanșare; verificați cablajul de sincronizare. Puteți alege în continuare rolurile manual. |
| „Fără cablu de sincronizare: {seriale}” (portocaliu) | Camerele listate nu au niciun cablu de sincronizare conectat. |

Tabelul camerelor conține coloanele **Cameră / Serial / IP / Master (radio) / Slave (casetă de selectare)**:

* Alegeți exact **un singur master**și**una sau mai multe camere slave**. Dacă faceți clic din nou pe radio-ul masterului curent, acesta va fi dezactivat.
* O cameră marcată cu **„Fără cablu de sincronizare”** nu poate fi niciodată selectată ca secundară — o cameră secundară fără cablaj de declanșare ar aștepta la nesfârșit pe linia de sincronizare și ar transmite un semnal mort. Conectați acea cameră ca cameră autonomă în schimb.
* Camerele deja conectate în mod autonom *nu* sunt dezactivate: conectarea la matrice eliberează sesiunea autonomă și redeschide camera în cadrul matricei.

**Următorul: Mod afișare →**se activează odată ce au fost alese un master și cel puțin un slave.**Rescanare** reexecută procesul de descoperire și testarea cablajului.

{% hint style="warning" %}
**Anulare** este dezactivată cât timp o scanare sau o verificare este în curs — anularea în timpul verificării poate bloca camera SDK pe firmware-ul camerei LATTICE. Așteptați până când indicatorul de încărcare se oprește.
{% endhint %}

### Pasul 2 — Mod de afișare



<!-- SCREENSHOT-NEEDED: Array Connect dialog, Display Mode scene, showing the two selectable cards ("Separate Cameras" and "Combined Cameras") with Combined selected/highlighted as the default. -->| Mod | Ce obțineți |
| --- | --- |
| **Camere separate** | Câte un panou live pentru fiecare cameră, toate declanșate simultan, astfel încât cadrele să rămână sincronizate. Fiecare cameră își păstrează propria culoare și propriile setări. |
| **Camere combinate** *(implicit)* | Un singur tile care redă compoziția aliniată multibandă NDVI /index. Camerele împărtășesc culoarea matricei. |

Modul de afișare modifică doar prezentarea previzualizării live — comportamentul de captură este același în ambele cazuri.

### Pasul 3 — Setările matricei și rezultatul

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Settings scene, healthy state: left column with ROI / Binning / Pin resolution / Trigger Rate controls, right "Projected Outcome" column showing green "Simultaneous capture" tier, an fps range, the NIC line, the "Sim-emit burst" line, and the "Wire budget" line with a checkmark. -->proiectat

La intrarea în această scenă, Chloros solicită backend-ului o **recomandare**și aplică automat o combinație de ROI + binning care se potrivește cu inelul de recepție al NIC-ului dvs. (preferă binningul în locul decupării ROI, deoarece binningul păstrează întregul câmp vizual). Fiecare modificare pe care o efectuați reexecută analiza în timp real și actualizează panoul**Rezultatul proiectat** din partea dreaptă.

Coloana din stânga — setări:

| Control | Opțiuni | Implicit | Note |
| --- | --- | --- | --- |
| **ROI (Câmp vizual)** | Complet (2048×1536) / Jumătate (1024×768) / Sfert (512×384) | Complet | Decupare senzor: decupare la jumătate/sfert într-o regiune mai mică la pasul nativ al pixelilor. |
| **Binning** | 1× / 2× (sumă 2×2) / 4× (sumă 4×4) | 1× | Binning hardware: 2×2 = câmp vizual complet la un sfert din costul de transmisie; 4×4 = câmp vizual complet la 1/16. Ascuns dacă camerele nu acceptă binning. |
| **Imagine pe cablu** (citire) | — | — | Lățimea × înălțimea post-binning trimisă efectiv pe cablu, rotunjită la multipli de 16 (minim 64). |
| **Rezoluție la pini**| casetă de selectare | dezactivată | Funcția „Chloros” activează automat binning-ul la conectare atunci când rata proiectată scade sub**1,5 fps**. Fixarea menține dimensiunea cadrului aleasă și acceptă rata mai mică — transformând o configurație suprasolicitată într-un refuz categoric al conexiunii, în loc de o reducere automată a ratei. |
| **Rata de declanșare** | 0,5–60 fps, pas 0,1 | necompletat = auto | Rata de declanșare a masterului. Lăsați câmpul gol pentru ca Chloros să o calculeze automat. |
| **Buget de transfer**| 20–2000 MB/s, pas 10 | gol = auto | Cât poate absorbi efectiv gazda, în MB/s —**singura valoare de care depinde întreaga alocare a matricei.** Detectat automat de la adaptorul de rețea. Reduceți-l dacă matricea raportează cadre corupte: valoarea detectată supraestimează capacitatea adaptoarelor USB și a comutatoarelor partajate. Modificarea acesteia reexecută proiecția în timp real. |

Coloana din dreapta — **Rezultat proiectat**:

* **Nivel de sincronizare** — „Captură simultană” (verde), „Captură simultană (emisie eșalonată FTD)” (verde), „Captură eșalonată (deviere de 100 ms)” (portocaliu) sau „Configurație prea mare” (roșu).
* ****Proiecția fps** — afișată ca un interval („slab → puternic”), deoarece rata unei matrice sincronizate este limitată de expunerea celei mai lente camere.
* **Linia NIC** — viteza de legătură și debitul susținut („NIC {mbps} Mbps · susținut {N} MB/s”).
* **Verificare rafale de emisie simultană** — poate inelul de recepție al plăcii de rețea a gazdei să absoarbă o singură rafală simultană de la toate camerele („Rafală de emisie simultană: X MB · inel NIC utilizabil: Y MB ✓/✗”).
* **Verificare buget de bandă** — cererea agregată în regim staționar față de plafonul de bandă fără coliziuni („Buget de bandă: {cerere} MB/s solicitată de {n} camere · plafon {plafon} MB/s ✓/✗ suprasolicitare”).
* **„Numărul maxim de camere pe această bandă: {n} — stabilit de pragul minim de lățime de bandă per cameră, astfel încât gruparea nu îl crește.”** — afișat când vă apropiați de (sau a depăși) limita maximă a numărului de camere.
* **„CADRELE SE VOR PIERDE la aceste setări.”**— avertisment roșu cu motivul indicat de backend, plus o listă de obstacole și**sugestii de remediere** albastre („Pentru a încadra această matrice în rețea” / „Pentru a debloca captarea simultană”).**„Aplică și conectează”** este blocat până când există o proiecție, iar eticheta acestuia îți indică motivul refuzului:

| Etichetă buton | Înțeles | Ce ajută de fapt |
| --- | --- | --- |
| „Se analizează...” | Analiza este încă în curs. | Așteaptă. |
| **„Prea multe camere pentru această rețea”**| Matricea suprasolicită banda (verificarea agregată a eșuat). | Mai puține camere, cadre jumbo de la un capăt la altul sau o placă de rețea mai rapidă.**O zonă de interes (ROI) mai mică NU va ajuta** — vezi mai jos. |
| **„Reduceți ROI pentru a activa”** | Cadrele s-ar pierde la aceste setări (verificarea burst/ring a eșuat). | Reduceți ROI, măriți binning-ul sau remediați inelul de recepție al plăcii de rețea. |



<!-- SCREENSHOT-NEEDED: Array Connect dialog, Settings scene, over-subscribed state: red "Wire budget ... over-subscribed" line, the "Max cameras on this wire" hint, and the Apply button reading "Too many cameras for this network". Reproduce by configuring more cameras than the 1 GbE ceiling (e.g. 7+ cams at 1500 MTU) or with CHLOROS-simulated models via `lattice analyze-array`. -->În timpul conectării, poate apărea un **panou verde de descărcare a calibrării** cu o bară de progres pentru fiecare port serial: la prima conectare a unei camere la un echipament, Chloros descarcă pachetul de calibrare din fabrică de aproximativ 3,8 MB de pe cameră prin GigE (aproximativ 70 de secunde per cameră). Camerele stocate în cache nu afișează niciodată acest panou. Consultați [Conectarea camerelor](connecting.md).

## Lățime de bandă: câte camere încap

Capacitatea de transfer a unei rețele depinde de cablu, nu de Chloros, așa că cifrele de planificare se găsesc în manualul hardware-ului: **[Planificarea lățimii de bandă a rețelei](https://mapir.gitbook.io/lattice-camera/setup/array-bandwidth-planning)**.

Ce face „Chloros” cu aceste date: fereastra de dialog „Conectare” rulează o testare a rețelei, estimează rata de cadre realizabilă și alege un nivel care se potrivește. Dacă matricea suprasolicită cablul, aceasta refuză conectarea în loc să piardă pachete în tăcere — consultați panoul cu rezultatele estimate descris mai sus.

## Când lipsesc cadrele

O cameră poate lipsi dintr-un grup publicat din două motive complet diferite,
iar acestea necesită soluții opuse. Chloros le numără separat, în loc să raporteze un
număr „incomplet” care nu specifică niciunul dintre ele:

| Ce s-a întâmplat | Ce înseamnă | Unde să căutați |
| --- | --- | --- |
| **Corupt**— cadrul a ajuns, dar era defect din punct de vedere structural | Pierdere de pachete GVSP pe traseul de rețea |**Capacitatea de transfer**, inelul de recepție al plăcii de rețea, cadrele jumbo, switch-ul |
| **Nu a ajuns niciodată**— nu a sosit niciun cadru | Camera nu s-a declanșat sau nu a transmis nimic |**Cablul de sincronizare M8**, linia de sincronizare, dacă toate camerele sunt activate |

Împărțirea este reevaluată la fiecare 10 secunde în timp ce matricea transmite. Peste 5 %, aceasta este
înregistrată cu menționarea ambelor valori, iar fiecare buffer corupt este raportat la prima apariție
pentru fiecare cameră, apoi agregat o dată pe minut, astfel încât o sesiune lungă să rămână lizibilă.

**Cadrele corupte cu zero „n-au ajuns” înseamnă că declanșarea și sincronizarea prin cablu sunt perfecte**și fiecare cadru pierdut se află pe traseul rețelei. Soluția este să reduceți**Wire Budget** și
să vă reconectați.

{% hint style="warning" %}
**Reducerea ratei de declanșare nu ajută în cazul cadrelor corupte.** Ritmul de transmisie a pachetelor
camei este stabilit o singură dată, la conectare. Reducerea ratei de declanșare modifică frecvența cu care are loc o rafală,
nu viteza cu care rafala în sine ajunge pe cablu. Pe o configurație măsurată cu 4 camere, o
reducere de 5 ori a ratei de declanșare nu a schimbat nimic, în timp ce scăderea bugetului de bandă de la 240 la
200 MB/s a dus aceeași configurație de la 10,4 % cadre corupte la zero.
{% endhint %}

O matrice în funcțiune nu se poate replanifica singură — deconectați-o și reconectați-o, astfel încât selectorul de timp de conectare
să poată funcționa în funcție de noul buget.

### Adaptorii de rețea USB sunt limitați la 200 MB/s

Un adaptor USB Ethernet afișează rata sa de legătură *Ethernet*, dar ceea ce poate susține efectiv
este limitat de magistrala USB și de driverul său. Un dongle USB 10GbE era creditat
cu un debit de aproximativ 1000 MB/s — o valoare pe care nimeni nu o măsurase vreodată — iar sincronizarea
a patru camere în funcție de această marjă fantomatică a corupt 6–18 % din cadre, în timp ce matricea
raporta în continuare o rată de cadre țintă corespunzătoare. Adaptoarele conectate prin USB sunt acum limitate la
**200 MB/s**. Limita este una absolută, nu procentuală, deoarece restricția este reprezentată de
bus: un adaptor USB 1 GbE atinge aproximativ 80 MB/s și nu este afectat.

Dacă gazda dvs. este cu adevărat mai rapidă decât limita, măriți **Bugetul de bandă** pentru a reflecta acest lucru.

## Sincronizarea temporală PTP

*Sincronizarea* cadrelor provine de la declanșatorul hardware; **PTP** (IEEE 1588 PTPv2) asigură *marcaje temporale* comparabile pe toate dispozitivele. Este activat implicit la conectarea matricei:

* **Backend-ul gazdei Chloros rulează grandmaster-ul PTP**. Camerele LATTICE și senzorii de lumină DAQ-E funcționează ca slave în domeniul 0, astfel încât marcajele temporale ale imaginilor și spectrele DAQ se sincronizează la un singur ceas (~1 ms).
* `--no-ptp` (CLI) îl dezactivează pentru lucrările de laborator — marcajele temporale între camere **nu** sunt atunci comparabile.
* Verificați starea sincronizării cu CLI:

```bash
chloros-cli time-sync status     # grandmaster state, clock identity
chloros-cli time-sync peers      # slaves seen (cameras + DAQ-E sensors)
chloros-cli time-sync cameras    # per-camera PtpStatus / PtpOffsetFromMaster / PtpMeanPathDelay
```

Fila „Camere” în sine nu are un indicator PTP; acolo sunt afișate informațiile de sincronizare pentru fiecare cameră în parte: **Rolul**(Master/Slave),**Linia de sincronizare** și nivelul de capabilități al matricei, toate fiind doar pentru citire. Starea PTP a DAQ-E este afișată în detaliile senzorului din fila „Senzori de lumină”.

## Vizualizarea

<!-- SCREENSHOT-NEEDED: Cameras tab with a connected combined array: sidebar showing the ARRAY row (color badge, array name, "DAQ · on" pill) with indented member camera rows, and the main area showing the combined index composite tile with the LUT-colored NDVI render, top-left array name pill, and top-right fps readout. -->live a matricei

Zona principală de afișare oferă două moduri de afișare (comutabile în bara de sus): **vizualizarea în grilă**(fiecare casetă reprezintă o celulă; trageți pentru a reordona atunci când lacătul grilei este deblocat) și**vizualizarea listă**(matricile pe toată lățimea în partea de sus, o cameră activă dedesubt). Glisorul**Zoom flux** redimensionează casetele; sub o lățime a celulei de 200 px, suprapunerile cu numele/fps se ascund automat.**Modul separat** afișează câte un panou pentru fiecare cameră. Fiecare panou afișează:

* numele camerei (stânga sus),
* o **afişare a fps** (în dreapta sus) — aceasta este *rata reală de captare* a camerei raportată de backend, nu rata de sondare a previzualizării (previzualizarea live este limitată la 30 fps indiferent de rata de captare),
* un punct de stare — verde (transmisie în direct) / portocaliu (încărcare) / roșu (eroare),
* un **indicator de cadru vechi** când nu a sosit niciun cadru nou timp de 2 s — normal timp de ~5 s după orice conectare/deconectare, în timp ce backend-ul reechilibrează bugetul de bandă între camere.**Modul combinat**afișează o singură casetă compusă: backend-ul efectuează debayering, scalare, aliniere, reducerea zgomotului, conversia la radianță pe bandă (plus reflectanța DLS atunci când este asociat un senzor de lumină), evaluează expresia indexului matricei, aplică LUT-ul și transmite rezultatul în format MJPEG. Până la redarea primului cadru aliniat, caseta indică starea sa: „Pregătire matrice…”, „Calibrare aliniere…”, „Așteptare primul cadru…” sau — dacă s-a epuizat bugetul de reîncercări pentru alinierea automată (~30 s) — „Aliniere necesară” cu un buton**Calibrare aliniere**.

Informații utile despre modul combinat:

* Compozitul este sincronizat cu cadrul camerei **master**. Țintirea AE-ROI și măsurarea punctuală pe compozit sunt exacte pentru camera master și aproximative pentru camerele slave; utilizați**Split View** (setări matrice → „Afișează camerele membre”) pentru secțiuni exacte la nivel de pixel pentru fiecare cameră, fără a deschide conexiuni suplimentare la camere.
* **Straturi de afișare**(setări matrice; dezactivat implicit) vă permite să alegeți un strat de prim-plan și unul de fundal — orice cameră membră sau**Index**. Cu prim-plan = Index, pixelii aflați în afara intervalului Min/Max al LUT-ului afișează stratul de fundal.
* **Rezoluția de randare** (implicit 720p) stabilește înălțimea fluxului live *și* dimensiunea de export a compoziției salvate. Imaginile de la fiecare cameră se exportă întotdeauna la rezoluție maximă.
* Alinierea este calculată pentru fiecare sesiune și nu este niciodată păstrată — consultați secțiunea de aliniere din panoul de setări al matricei pentru reziduurile RMS și butonul Recalibrare.

## Captare: monitorizare vs. analiză

Suprafețele de captare ale matricei se împart clar în **nivel de monitorizare**(înregistrează ceea ce vezi) și**nivel de analiză** (înregistrează date brute, calibrează ulterior):

| Flux de lucru | Nivel | Ce se salvează | Interfață grafică | CLI |
| --- | --- | --- | --- | --- |
| **Captură**(imagini statice) | Analiză | Un grup de cadre sincronizate pe fiecare trecere; fișiere per cameră la fiecare nivel de export selectat (brut/debayered/radianță/reflectanță/previzualizare/index) + fișier sidecar `.daq` | Butonul**Captură totală** + Setări de captură | `lattice array-capture` |
| **Înregistrare video index** | Monitorizare | Compoziția index combinată în timp real așa cum este afișată — 8 biți, rezoluție de previzualizare, LUT încorporat; necesită fluxul live deschis | ● Înregistrare video index (matrice combinate) | `lattice array-record` |
| **Seria de imagini raw → creare video**| Analiză | Cadre brute de la senzor la rata maximă de captare + manifest + `.daq`, apoi reconstrucție offline în video calibrat de radianță / reflectanță / indice, sincronizat temporal cu citirile DAQ | ⦿ Înregistrare serie brută →**Creare video** | `lattice array-burst` → `lattice array-build-video` |

Regula generală: dacă pixelii vor furniza *măsurători*, utilizați modul de captură sau de serie (calitate de analiză); dacă doriți doar să *vizualizați sau să demonstrați* ceea ce a înregistrat matricea, înregistrați videoclipul indexat (calitate de monitorizare).

### Setări de captură (GUI)



<!-- SCREENSHOT-NEEDED: Capture Settings pane (gear next to Capture All) with a connected array: capture-mode buttons (Single/Continuous/Interval), the bulk export-type toggle row, the Fastest Capture toggle, and the per-array group card showing the Aligned checkbox and the "Record index video" / "Record raw burst" buttons. -->Roata dințată de lângă **Capture All** deschide panoul Setări de captură (necesită un proiect deschis — capturile sunt salvate în acesta):

* **Mod de captură**:**Single**(o singură trecere) /**Continuous**(consecutiv; limitat de un număr de capturi, implicit 1, sau de o durată, implicit 10 s) /**Interval** (timelapse: N capturi la fiecare interval X pentru un total de Y, implicit 1 la fiecare 5 s timp de 1 minut).
* **Tipuri de export per cameră**: Raw, Debayered, Radiance, Reflectance, Preview, Index — toate opțiunile aplicabile sunt activate implicit. Radiance/Reflectance sunt ascunse pentru camerele cu filtru „RGB”;**Reflectance apare numai atunci când camera are un senzor de lumină DAQ** (propriu sau moștenit de la matrice); Index necesită o expresie de index configurată.
* **Aliniat**(pe matrice, implicit**activat**): deformează exporturile elementelor conform profilului de aliniere al matricei, astfel încât exporturile să fie înregistrate la nivel de pixel. Raw rămâne întotdeauna nedeformat, dar poartă transformarea în metadate.
* **Captură rapidă** (comutator): doar date brute + citirea DAQ atribuită + compozitul cu indice combinat gratuit, omisând calculele de calibrare în momentul capturii pentru o rată maximă — radianța/reflectanța/indicele se reconstruiesc ulterior din fișierul `.daq` salvat.
* Selecțiile rămân valabile împreună cu proiectul. Camerele ascunse sau puse în pauză sunt omise.

CLI-ul echivalent (același punct final de backend, aceeași semantică):

```bash
# One synced group, every applicable export level per camera (the default)
chloros-cli lattice array-capture -o output/

# Interval timelapse: one reflectance pass every 10 s for 5 minutes
chloros-cli lattice array-capture --interval 10 --duration 300 --processing reflectance -o timelapse/

# Fastest grab for a moving rig — raw + .daq now, calibrate later
chloros-cli lattice array-capture --fastest -o flightline/

# 30-second monitoring clip of the combined index view, plus a GIF
chloros-cli lattice array-record --duration 30 --fps 10 --gif -o monitoring/

# 5-second analysis-grade raw burst, then build the combined index video
chloros-cli lattice array-burst --duration 5 --build --products combined:index --fps 10 -o capture/
```

Compresia TIFFă pentru capturi este `deflate` (fără pierderi, implicit) sau `none` — tabelele complete de indicatori, structura folderului de capturi și regulile de reprocesare se găsesc în [Referința CLI](../reference/cli-reference.md#capture-modes-recorders--offline-reprocess).

## Asocierea unui senzor de lumină DAQ

Previzualizările cu reflectanță și iluminare corectate necesită date privind lumina descendentă de la un senzor DAQ (conectat în fila **Senzori de lumină**):

* **Rândul matricei**din bara laterală afișează un buton**„DAQ · pornit/oprit”** — *pornit* atunci când este setat un senzor de lumină la nivel de matrice **sau** orice cameră membră are propriul senzor; informația rapidă afișată indică exact ce senzor alimentează fiecare cameră.
* Alocați la nivel de matrice în setările matricei → **Senzor de lumină ambientală**→ meniul derulant**Senzor de lumină**. Selecția se păstrează odată cu proiectul, se propagă la fiecare cameră membră, iar camerele individuale o pot înlocui în continuare cu propriul senzor.
* Linia de stare de sub aceasta raportează starea în timp real: **Oprit**→ „Se așteaptă primul spectru…” →**„Activ — toate camerele din matrice sunt corectate din punct de vedere al iluminării”** → sau, dacă nu a sosit niciun spectru nou în ultimele 3 s, o notificare de date învechite — ultima citire continuă să fie utilizată (citirile nu expiră niciodată pe calea de captură).

Cu un senzor alocat: tipul de export „Reflectanță” devine disponibil, previzualizările în timp real sunt corectate din punct de vedere al iluminării, expunerea automată predictivă poate utiliza spectrul, iar fiecare captură de reflectanță scrie valoarea DAQ utilizată efectiv ca **`.daq` sidecar** lângă imagine, astfel încât captura să poată fi reprocesată ulterior.

## Opțiuni de „CLI” pentru `.daq`

| Indicator | Implicit | Descriere |
| --- | --- | --- |
| `--serials SN1,SN2,…` | detectare automată a tuturor camerelor LATTICE (necesită ≥2) | **Prima cameră serială este MASTER.** |
| `--line {Line0,Line2,Line3}` | `Line2` | Linie de sincronizare GPIO. |
| `--target-fps F` | auto | Rata de declanșare a Masterului. |
| `--binning {1,2,4}` | auto | Binning hardware. |
| `--force-tier {sim-capture-sim-emit, sim-capture-ftd-stagger, slip-emit-and-capture}` | auto | Suprascriere expertă a selectorului de nivel de sincronizare. |
| `--wire-ceiling-mbps MB_PER_S` | detectat automat | Buget de bandă al gazdei în MB/s — forma „CLI” a câmpului **Buget de bandă**. Reduceți-l dacă matricea raportează cadre corupte. Se salvează odată cu proiectul, astfel încât o reconectare ulterioară îl restabilește. |
| `--no-recommend` | dezactivat | Omite etapa de analiză a rețelei. |
| `--no-ptp` | dezactivat | Dezactivează PTP (marcajele de timp între camere nu vor mai fi comparabile). |

`lattice array-list`, `array-status` și `array-disconnect` gestionează sesiunea persistentă. Referința completă a subcomenzilor, inclusiv alinierea (`align-calibrate` / `align-apply`) și instrumentele de rețea, se găsește în [Referința CLI § chloros-cli lattice](../reference/cli-reference.md#chloros-cli-lattice); echivalentele din SDK (`connect_array`, `ArraySession`, `attach_array`, `analyze_array_network`) se găsesc în [Referința SDK](../reference/sdk-reference.md). Din Python, bugetul de cabluri este `connect_array(..., wire_ceiling_mbps=120)`, iar împărțirea între corupte active și cele care nu au ajuns niciodată se află pe [`/api/camera/array/<id>/capability`](../reference/sdk-reference.md#array-health--which-subsystem-is-losing-frames).
