# Matrice cu mai multe camere

O **matrice**LATTICE este formată din două sau mai multe camere LATTICE conectate ca o singură unitate sincronizată. Una dintre camere este**master**: aceasta emite un impuls de declanșare GPIO la nivel de hardware pe o linie de sincronizare comună (implicit**Line2**), astfel încât fiecare componentă să înregistreze același moment. Chloros adaugă sincronizarea temporală PTP, o previzualizare în timp real (segmente pentru fiecare cameră sau o singură imagine compusă multibandă aliniată) și captură sincronizată — fiecare rundă de captură produce un **grup de cadre** în care toate camerele împărtășesc același timestamp și același ID de cadru (raportat ca `fid:N` la ieșirea capturii).

Matricile reprezintă modul în care camerele mono (M3M) generează indici de vegetație — o cameră contribuie cu o bandă, iar matricea le aliniază într-un teanc multibandă. Consultați [Camere mono și indici de vegetație](mono-indices.md).

Există trei moduri echivalente de a conecta o matrice, iar toate rulează același flux „smart-prep”:

| Suprafață | Punct de intrare |
| --- | --- |
| GUI | Fila „Cameras” → **Connect Array** (buton albastru) |
| CLI | `chloros-cli lattice array-connect --serials SN1,SN2,…` (primul număr de serie = master) |
| Python SDK | `connect_array(serials=[…])` → `ArraySession` (primul număr de serie = master) |

Smart-prep efectuează: o verificare a capacității rețelei (ping ICMP DF + sondă GVSP), selectarea nivelului de sincronizare, o proiecție a ratei de cadre la dimensiunea cadrului solicitată, decizia privind transportul comprimat pe 12 biți (vezi [Transport comprimat pe 12 biți](#packed-12-bit-transport)), activarea PTP, selectarea automată a formatului de pixeli pentru fiecare cameră, inițializarea automată a expunerii pe baza stării salvate a fiecărei camere și configurarea declanșatorului GPIO pe Linia 2.

**Senzor complet în mod implicit (nou în 1.3.0).** Connect păstrează dimensiunea cadrului pe care ați solicitat-o — senzorul complet de 2048×1536, fără binning și fără decupare, cu excepția cazului în care alegeți altfel — și permite ca rata de cadre să scadă la nivelul oferit de gazdă și de rețea. Nu efectuează niciodată binning sau decupare din proprie inițiativă (versiunea 1.2.0 activa automat binningul sub 1,5 fps). Binningul sau o zonă de interes (ROI) mai mică depind de alegerea dvs.: în fereastra de dialog „Connect”, alegeți una dintre opțiuni, iar secțiunea „Projected Outcome” afișează rata de cadre rezultată; `array-connect` afișează opțiunile mai rapide împreună cu ratele estimate.

{% hint style="info" %}
Camerele trebuie să fie accesibile pe legătură înainte ca orice dintre aceste funcții să funcționeze — consultați [Conectarea camerelor](connecting.md) pentru detectare, adresare și descărcarea calibrării la prima conectare. Pentru configurațiile cu mai multe camere, setările inelului de recepție al plăcii de rețea a gazdei sunt la fel de importante ca viteza legăturii; tabelul complet cu simptome→soluții se găsește în [Referință CLI § Configurarea și reglarea plăcii de rețea a gazdei](../reference/cli-reference.md#host-nic-setup--tuning-lattice-arrays).
{% endhint %}

## Fereastra de dialog „Conectare matrice”

Fila „Camere” → **Conectare matrice**deschide un asistent în trei pași:**Selectare → Mod de afișare → Setări**.

### Pasul 1 — Selectați camera principală și camerele secundare

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Select scene, with 3-4 LATTICE cameras discovered (an FRGB among them). Table showing Camera / Serial / IP / Master radio / Slave checkbox columns in camera order (M3C first, FRGB at the top), with the FRGB pre-selected as master and the green "Sync cable found on N cameras. Any of them can be master; the RGB camera (else the first) is pre-selected." probe banner visible above the table. -->



Fereastra de dialog scanează rețeaua imediat ce se deschide („Scanează rețeaua...”), apoi verifică cablurile de sincronizare („Verifică cablurile de sincronizare...”) pentru a identifica camerele conectate la linia de sincronizare M8. Aveți nevoie de cel puțin **2 camere** pentru a configura o matrice.

Cablurile de sincronizare M8 se întâlnesc la o singură joncțiune, astfel încât **orice cameră conectată prin cablu poate fi master**. Testul predefineste rolurile — camera RGB (`-FRGB`) ca master atunci când este conectata prin cablu, altfel prima camera conectata prin cablu, iar celelalte camere conectate prin cablu ca slave — si afiseaza unul dintre cele trei mesaje:

| Mesaj | Semnificatie |
| --- | --- |
| „Cablu de sincronizare detectat la {n} camere. Oricare dintre ele poate fi master; camera RGB (altfel prima) este preselectată.” (verde) | Sonda a detectat camerele pe linia de sincronizare; casetele de selectare pentru master radio și slave sunt deja bifate. |
| „Nu s-a detectat nicio cameră principală — verificați cablul GPIO” (portocaliu) | Nicio cameră nu a detectat un impuls de declanșare; verificați cablajul de sincronizare. Rolurile sunt încă preselectate (camera RGB, altfel prima cameră din listă, ca principală) și le puteți modifica. |
| „Fără cablu de sincronizare: {seriale}” (portocaliu) | Camerele listate nu au niciun cablu de sincronizare conectat. |

Tabelul camerelor are coloanele **Cameră / Serial / IP / Master (radio) / Slave (casetă de selectare)**. Rândurile sunt listate în**ordinea camerelor** — camerele M3C (Bayer) înaintea celor M3M (mono); printre M3C, FRGB, FRGN, FOCN, apoi FNGB; M3M după lungimea de undă a filtrului, începând cu cea mai mică (F450 înaintea lui F808). Lista de camere din bara laterală și dispunerea implicită a grilei folosesc aceeași ordine.

* Alegeți exact **un master**și**unul sau mai mulți slave**. Dacă faceți clic din nou pe butonul radio al masterului curent, acesta se dezactivează. Alegerea unui alt master se face cu un singur clic: noul master părăsește grupul de slave, iar masterul anterior devine slave (cu excepția cazului în care nu are cablu de sincronizare).
* O cameră marcată cu **„Fără cablu de sincronizare”** nu poate fi niciodată selectată ca subordonată — o subordonată fără cablaj de declanșare ar aștepta la nesfârșit pe linia de sincronizare și ar transmite un semnal mort. Conectați acea cameră ca cameră autonomă în schimb.
* Camerele deja conectate în mod autonom *nu* sunt dezactivate: conectarea la matrice eliberează sesiunea autonomă și redeschide camera în cadrul matricei.

**Următorul: Mod de afișare →**se activează odată ce au fost alese o cameră principală și cel puțin o cameră secundară.**Rescanare** reexecută procesul de detectare și verificarea cablajului.

{% hint style="warning" %}
**Anulare** este dezactivată în timp ce o scanare sau o verificare este în curs — anularea în timpul verificării poate bloca SDK-ul camerei pe firmware-ul camerei LATTICE. Așteptați ca indicatorul de încărcare să se oprească.
{% endhint %}

### Pasul 2 — Mod de afișare



<!-- SCREENSHOT-NEEDED: Array Connect dialog, Display Mode scene, showing the two selectable cards ("Separate Cameras" and "Combined Cameras") with Combined selected/highlighted as the default. -->

| Mod | Ce obțineți |
| --- | --- |
| **Camere separate** | Câte un panou live pentru fiecare cameră, toate declanșate simultan, astfel încât cadrele să rămână sincronizate. Fiecare cameră își păstrează propria culoare și propriile setări. |
| **Camere combinate** *(implicit)* | Un singur panou care redă compoziția aliniată multibandă NDVI/index. Camerele împărtășesc culoarea matricei. |

Modul de afișare modifică doar prezentarea previzualizării live — comportamentul de captură este același în ambele cazuri.

### Pasul 3 — Setările matricei și rezultatul

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Settings scene, healthy state: left column with ROI / Binning / Trigger Rate / Wire Budget controls (1.3.0 has no "Pin resolution" checkbox), right "Projected Outcome" column showing green "Simultaneous capture" tier, an fps range, the NIC line, the "Sim-emit burst" line, and the "Wire budget" line with a checkmark. -->

proiectat

La intrarea în această scenă, Chloros solicită backend-ului o **recomandare**și o aplică:**senzorul complet — fără binning, fără decupare ROI — ori de câte ori rata de cadre o permite**, indiferent de rata oferită de gazdă și de conexiune. Doar atunci când nicio rată nu poate suporta senzorul complet, recomandarea micșorează cadrul, iar în acest caz preferă binningul în locul decupării ROI, deoarece binningul păstrează câmpul vizual complet. De aici, faceți un compromis între rezoluție și rata de cadre. Fiecare modificare pe care o faceți reexecută analiza în timp real și actualizează panoul**Rezultatul proiectat** din partea dreaptă.

Coloana din stânga — setări:

| Control | Opțiuni | Implicit | Note |
| --- | --- | --- | --- |
| **ROI (Câmp vizual)** | Complet (2048×1536) / Jumătate (1024×768) / Sfert (512×384) | Complet | Decupare senzor: decupare la jumătate/sfert într-o regiune mai mică la pasul nativ al pixelilor. |
| **Binning** | 1× / 2× (sumă 2×2) / 4× (sumă 4×4) | 1× | Binning hardware: 2×2 = câmp vizual complet la un sfert din costul de transmisie; 4×4 = câmp vizual complet la 1/16. Ascuns dacă camerele nu suportă binning. |
| **Imagine pe cablu** (citire) | — | — | Lățimea × înălțimea post-binning trimise efectiv pe cablu, rotunjite la multipli de 16 (minimum 64). |
| **Rată de declanșare** | 0,5–60 fps, pas 0,1 | gol = auto | Rata de declanșare a masterului. Lăsați câmpul gol pentru ca Chloros să o calculeze automat. |
| **Buget de bandă**| 20–2000 MB/s, pas de 10 | gol = auto | Cât poate absorbi efectiv gazda, în MB/s —**singura valoare de care depinde întreaga alocare a matricei.** Detectată automat de la adaptorul de rețea. Reduceți-o dacă matricea raportează cadre corupte: valoarea detectată supraestimează capacitatea adaptoarelor USB și a comutatoarelor partajate. Modificarea acesteia reexecută proiecția în timp real. Câmpul se deschide cu bugetul cu care a fost conectată ultima matrice (salvat odată cu proiectul), astfel încât un buget pe care l-ați corectat o dată este păstrat. |

Caseta de selectare **Rezoluția pinilor** din versiunea 1.2.0 a dispărut: funcția „Conectare” păstrează întotdeauna dimensiunea cadrului pe care o alegeți și reduce în schimb rata cadrelor.

Columna din dreapta — **Rezultat proiectat**:

* **Nivel de sincronizare** — „Captură simultană” (verde), „Captură simultană (emisie eșalonată FTD)” (verde), „Captură eșalonată (decalaj de 100 ms)” (chihlimbar) sau „Configurație prea mare” (roșu). Chloros alege nivelul: captură simultană ori de câte ori cablul poate fi partajat la o rată la care camerele pot fi sincronizate (rata de cadre scade pe măsură ce cota fiecărei camere se micșorează) și captură eșalonată numai când acest lucru nu este posibil — astfel încât o matrice se conectează la orice rețea, la rata de cadre permisă de rețea.
* **Proiecția fps** — rata de cadre la care se preconizează că va funcționa matricea. În cazul capturii simultane, este vorba de o singură valoare: rata este stabilită mai întâi, iar expunerea automată umple intervalul de timp pe care aceasta îl oferă, apoi crește amplificarea, astfel încât o scenă slab iluminată să nu o reducă (vezi [Cum gestionează expunerea automată expunerea și amplificarea](camera-settings.md)). În cazul capturii eșalonate, aceasta este afișată ca un interval („slab iluminat → puternic iluminat”), deoarece expunerea fiecărei camere prelungește eșalonarea într-o scenă slab iluminată.
* **Linie NIC** — viteza de legătură și debitul susținut („NIC {mbps} Mbps · susținut {N} MB/s”).
* **Verificare rafale emise simultan** — poate inelul de recepție al plăcii de rețea a gazdei să absoarbă o singură rafală simultană de la toate camerele („Rafale emise simultan: X MB · inel NIC utilizabil: Y MB ✓/✗”).
* **Verificare buget de bandă** — cererea agregată în regim staționar față de limita maximă a benzii fără coliziuni („Buget de bandă: {cerere} MB/s solicitată de {n} camere · limită maximă {limită maximă} MB/s ✓/✗ depășită”).
* **„Număr maxim de camere pe această rețea: {n} — stabilit de pragul minim de lățime de bandă per cameră, astfel încât gruparea nu îl crește.”** — afișat când vă apropiați de (sau depășiți) limita maximă a numărului de camere.
* **„CADRELE SE VOR PIERDE la aceste setări.”**— avertisment roșu cu motivul indicat de backend, plus o listă de factori care blochează funcționarea și**sugestii de remediere** albastre („Pentru a încadra această matrice în rețea” / „Pentru a debloca captarea simultană”).**Aplicare și conectare** este blocată până când există o proiecție, iar eticheta sa îți indică motivul refuzului:

| Eticheta butonului | Înțeles | Ce ajută de fapt |
| --- | --- | --- |
| „Se analizează...” | Analiza este încă în curs. | Așteptați. |
| **„Prea multe camere pentru această rețea”**| Rețeaua nu poate suporta matricea nici măcar cu o singură cameră la un moment dat (verificarea agregată a eșuat): sunt mult mai multe camere decât poate gestiona, sau alte camere care transmit pe aceeași rețea. | Mai puține camere, cadre jumbo de la un capăt la altul sau o placă de rețea mai rapidă.**O zonă de interes (ROI) mai mică NU va ajuta** — vezi mai jos. |
| **„Reduceți ROI pentru a activa”** | Cadrele s-ar pierde la aceste setări (verificarea burst/ring a eșuat). | Reduceți ROI, măriți binning-ul sau remediați inelul de recepție al plăcii de rețea. |

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Settings scene, over-subscribed state: red "Wire budget ... over-subscribed" line, the "Max cameras on this wire" hint, and the Apply button reading "Too many cameras for this network". Reproduce by configuring more cameras than the 1 GbE ceiling (e.g. 7+ cams at 1500 MTU) or with CHLOROS-simulated models via `lattice analyze-array`. -->



În timpul conectării, poate apărea un **panou verde de descărcare a calibrării** cu o bară de progres pentru fiecare port serial: la prima conectare a unei camere la un computer, Chloros descarcă pachetul de calibrare din fabrică de ~3,8 MB de la cameră prin GigE (aproximativ 70 de secunde per cameră). Camerele stocate în cache nu afișează niciodată acest panou; cu mai mult de șase camere, rândurile acestuia se derulează. Consultați [Conectarea camerelor](connecting.md).

### Transport comprimat pe 12 biți

Formatele pe 12 biți (`BayerRG12`, `Mono12`) sunt transmise în mod normal sub formă de cuvinte de 16 biți. **Transportul**comprimat**trimite aceiași 12 biți în 1,5 octeți pe pixel și îi decomprimă pe gazdă, astfel încât fiecare cadru necesită cu 25 % mai puțin timp pentru a fi transmis — la senzor la capacitate maximă, aproximativ o treime mai multe cadre pe secundă.**Datele salvate sunt identice.** Conexiunea de tip „array” decide automat acest lucru (`auto`): comprimă doar atunci când acest lucru este util și gazda dispune de putere de procesare suficientă pentru decomprimare. Interfața grafică (GUI) nu are nicio opțiune pentru aceasta; din interfața de linie de comandă (CLI) utilizați `array-connect --packed {auto,on,off}` (implicit `auto`). Formatele de 8 biți și 16 biți nu sunt niciodată comprimate.

## Lățime de bandă: câte camere încap

Capacitatea de transport a unui array depinde de conexiunea fizică, nu de Chloros, așa că cifrele de planificare se găsesc în manualul hardware-ului: **[Planificarea lățimii de bandă a array-ului](https://mapir.gitbook.io/lattice-camera/setup/array-bandwidth-planning)**.

Ce face Chloros cu acestea: fereastra de dialog de conectare execută o testare a rețelei, estimează rata de cadre realizabilă și alege un nivel adecvat. Un array se conectează la orice rețea: pe o legătură lentă, fiecare cameră este adaptată la cota sa de lățime de bandă, iar rata de cadre scade, în loc ca array-ul să fie refuzat sau cadrele sale să fie reduse. Doar o matrice pe care conexiunea nu o poate suporta nici măcar pentru o singură cameră la un moment dat este refuzată (**Prea multe camere pentru această rețea**), în loc să se piardă pachete în tăcere — consultați panoul cu rezultatele proiectate descris mai sus.

## Când lipsesc cadrele

O cameră poate lipsi dintr-un grup publicat din două motive complet diferite,
iar acestea necesită soluții opuse. Chloros le numără separat, în loc să raporteze un
număr „incomplet” care nu specifică niciunul dintre ele:

| Ce s-a întâmplat | Ce înseamnă | Unde să căutați |
| --- | --- | --- |
| **Corupt**— cadrul a ajuns, dar era defect din punct de vedere structural | Pierderea de pachete GVSP pe traseul rețelei |**Bugetul de bandă**, inelul de recepție al plăcii de rețea, cadrele jumbo, comutatorul |
| **Nu a ajuns niciodată**— nu a sosit niciun cadru | Camera nu s-a declanșat sau nu a transmis nimic |**Cablul de sincronizare M8**, linia de sincronizare, dacă fiecare membru este activat |

Împărțirea este reevaluată la fiecare 10 secunde în timp ce matricea transmite. Peste 5 %, aceasta este
înregistrată cu menționarea ambelor valori, iar fiecare buffer corupt este raportat la prima apariție
pentru fiecare cameră, apoi agregat o dată pe minut, astfel încât o sesiune lungă să rămână lizibilă.

**Cadrele corupte cu zero „n-au ajuns” înseamnă că declanșarea și sincronizarea prin cablu sunt perfecte**și fiecare cadru pierdut se află pe traseul rețelei. Soluția este să reduceți**Wire Budget** și
să vă reconectați.

{% hint style="warning" %}
**Reducerea ratei de declanșare nu ajută în cazul cadrelor corupte.** Ritmul de transmisie a pachetelor
camerelor este stabilit o singură dată, la conectare. Reducerea ratei de declanșare modifică frecvența cu care are loc o rafală,
nu viteza cu care rafala însăși ajunge pe cablu. Pe o configurație măsurată cu 4 camere, o
reducere de 5 ori a ratei de declanșare nu a schimbat nimic, în timp ce scăderea bugetului de bandă de la 240 la
200 MB/s a dus aceeași configurație de la 10,4 % cadre corupte la zero.
{% endhint %}

O matrice în funcțiune nu se poate replanifica singură — deconectați-o și reconectați-o, astfel încât programul de conectare
să poată funcționa în conformitate cu noul buget.

### Adaptorii de rețea USB sunt limitați

Un adaptor Ethernet USB anunță rata sa de legătură *Ethernet*, dar ceea ce poate susține efectiv
este limitat de magistrala USB și de driverul său. Un dongle USB 10GbE era considerat
a avea un debit de aproximativ 1000 MB/s — o valoare pe care nimeni nu o măsurase vreodată — iar sincronizarea
a patru camere în funcție de această marjă fantomatică a corupt 6–18 % din cadre, în timp ce matricea
raporta în continuare o rată de cadre țintă corespunzătoare. Adaptoarele conectate prin USB sunt acum limitate la
**200 MB/s** pe Windows și NVIDIA Jetson, și **400 MB/s** pe Linux x86_64. Limita este una absolută, nu procentuală, deoarece restricția este reprezentată de
bus: un adaptor USB 1 GbE atinge aproximativ 80 MB/s și nu este afectat.

Dacă gazda dvs. este cu adevărat mai rapidă decât limita, măriți **Wire Budget** pentru a reflecta acest lucru.

## Sincronizarea temporală PTP

*Sincronizarea* cadrelor provine de la declanșatorul hardware; **PTP** (IEEE 1588 PTPv2) asigură *marcaje temporale* comparabile pe toate dispozitivele. Este activată implicit la conectarea matricei:

* **Backend-ul gazdă Chloros rulează grandmaster-ul PTP**. Camerele LATTICE și senzorii de lumină DAQ-E funcționează ca slave în domeniul 0, astfel încât marcajele de timp ale imaginilor și spectrele DAQ se sincronizează pe un singur ceas (~1 ms).
* `--no-ptp` (CLI) îl dezactivează pentru lucrările de laborator — în acest caz, marcajele de timp între camere **nu** sunt comparabile.
* Verificați starea sincronizării cu ajutorul CLI:

```bash
chloros-cli time-sync status     # grandmaster state, clock identity, health (OK, or why its packets are not reaching the network)
chloros-cli time-sync peers      # slaves seen (cameras + DAQ-E sensors)
chloros-cli time-sync cameras    # per-camera PtpStatus / PtpOffsetFromMaster / PtpMeanPathDelay
```

Fila „Cameras” (Camere) nu are un indicator PTP propriu; informațiile de sincronizare pentru fiecare cameră afișate acolo sunt **Role**(Rol) (Master/Slave),**Sync Line** (Linie de sincronizare) și nivelul de capabilități al matricei, toate fiind doar pentru citire. Starea PTP a DAQ-E este afișată în detaliile senzorului din fila „Light Sensors” (Senzori de lumină).

## Vizualizarea

<!-- SCREENSHOT-NEEDED: Cameras tab with a connected combined array: sidebar showing the ARRAY row (color badge, array name, "DAQ · on" pill) with indented member camera rows in camera order (M3C before M3M, monos by wavelength), and the main area showing the combined index composite tile with the LUT-colored NDVI render, top-left array name pill, and top-right fps readout. -->

în timp real a matricei Zona principală de afișare oferă două moduri de prezentare (comutabile din bara de sus): **vizualizarea în grilă**(fiecare casetă reprezintă o celulă; se pot reordona prin glisare când lacătul grilei este deblocat) și**vizualizarea listă**(rânduri pe toată lățimea: o matrice**combinată**reprezintă un singur rând — plăcuța sa compozită — în timp ce o matrice**separată**oferă**fiecărui membru propriul rând**, cu propriul flux live și propriul tabel de ținte de calibrare; sub matrice, o singură cameră independentă — cea activă). Glisorul**Zoom flux** redimensionează casetele; sub o lățime a celulei de 200 px, suprapunerile cu numele/fps se ascund automat. Camerele componente apar în bara laterală și în dispunerea implicită a grilei în ordinea camerelor (M3C înaintea M3M, FRGB mai întâi, camerele monocrome în funcție de lungimea de undă); deblocarea grilei și glisarea au prioritate asupra ordinii grilei.

<!-- SCREENSHOT-NEEDED: list view of a connected SEPARATE array (1.3.0): one full-width row per member camera, each with its live feed and its own Calibration Target table below it. -->



**Modul separat** afișează o secțiune per cameră. Fiecare secțiune afișează:

* numele camerei (stânga sus),
* o **afişare a fps** (dreapta sus) — aceasta este *rata reală de captare* a camerei raportată de backend, nu rata de interogare a previzualizării (previzualizarea live este limitată la 30 fps indiferent de rata de captare),
* un punct de stare — verde (transmisie) / portocaliu (încărcare) / roșu (eroare),
* un **indicator de cadru învechit** când nu a sosit niciun cadru nou timp de 2 s — normal timp de ~5 s după orice conectare/deconectare, în timp ce backend-ul reechilibrează bugetul de bandă între camere.**Modul combinat**afișează o singură casetă compozită: backend-ul efectuează debayering, scalare, aliniere, reducerea zgomotului, conversia la radianță pe bandă (plus reflectanța DLS atunci când este asociat un senzor de lumină), evaluează expresia indexului matricei, aplică LUT-ul și transmite rezultatul în format MJPEG. Până la redarea primului cadru aliniat, caseta explică starea sa: „Pregătire matrice…”, „Calibrare aliniere…”, „Așteptarea primului cadru…”, sau — dacă încercările automate de aliniere ale casetei (aproximativ un minut) se epuizează — „Aliniere necesară” cu un buton**Calibrare aliniere**.

Afișajul fps al plăcii matricei raportează, de asemenea, starea fluxului:**„Aplicarea setărilor…”**în timp ce o modificare a setărilor (format, binning, ROI) repornește camerele,**„Blocat”**când matricea este activă, dar a încetat să mai transmită grupuri de cadre, și**„Oprit”**când a încetat transmisia — în acest caz, reconectați matricea. Când separarea spectrală a trebuit să limiteze o bandă afișată în imaginea compusă pe o mare parte din pixelii săi (25 % sau mai mult), caseta afișează**„Date de bandă cu încredere scăzută · N%”**: valorile indexului calculate pe baza acelei benzi sunt suspecte (de obicei, o bandă a primit prea puțină lumină).

Informații utile privind modul combinat:

* Imaginea compozită este aliniată la cadrul camerei **master**. Țintirea AE-ROI și măsurarea punctuală pe imaginea compozită sunt exacte pentru camera master și aproximative pentru camerele slave; utilizați**Split View** (setări matrice → „Afișează camerele membre”) pentru casete exacte la nivel de pixel pentru fiecare cameră, fără a deschide conexiuni suplimentare pentru camere.
* **Straturi de afișare** (setări matrice; dezactivat implicit) vă permite să alegeți un strat de prim-plan și unul de fundal — orice cameră membră sau**Index**. Cu prim-planul setat la Index, pixelii aflați în afara intervalului minim/maxim al LUT-ului afișează stratul de fundal.
* **Rezoluția de randare** (implicit 720p) stabilește înălțimea fluxului live *și* dimensiunea de export a imaginii compozite salvate. Imaginile pentru fiecare cameră se exportă întotdeauna la rezoluție maximă.
* Alinierea este calculată pentru fiecare sesiune și nu este niciodată păstrată. Chloros aliniază **fiecare matrice automat după conectare, în ambele moduri de afișare**, odată ce expunerea automată a camerelor membre s-a stabilizat (continuă să încerce până la 5 minute). Secțiunea**Aliniere**din panoul de setări al matricei afișează reziduurile RMS și butonul**Recalibrare aliniere** — inclusiv pentru matrice separate, care nu au o secțiune compozită.

## Captarea: monitorizare vs. analiză

Suprafețele de captură ale matricei se împart clar în **nivel de monitorizare**(înregistrează ceea ce vezi) și**nivel de analiză** (înregistrează date brute, calibrează ulterior):

| Flux de lucru | Nivel | Ce salvează | GUI | CLI |
| --- | --- | --- | --- | --- |
| **Captură**(imagini statice) | Analiză | Un grup de cadre sincronizate pe trecere; fișiere per cameră la fiecare nivel de export selectat (brut/debayered/radianță/reflectanță/previzualizare/index) + fișier sidecar `.daq` |**Captură totală** + Setări de captură | `lattice array-capture` |
| **Înregistrare video index** | Monitorizare | Compoziția index combinat în timp real așa cum este afișată — 8 biți, rezoluție de previzualizare, LUT încorporat. Din CLI, fără nicio fereastră GUI deschisă, backend-ul redă NDVI-ul matricei (sau `--expr`) odată ce matricea este aliniată; o înregistrare care nu scrie niciun cadru eșuează | ● Înregistrare video index (matrice combinate) | `lattice array-record` |
| **Seria brută → creare video**| Analiză | Cadre brute ale senzorului la rata maximă de captare + manifest + `.daq`, apoi reconstrucție offline într-un videoclip calibrat de radianță / reflectanță / indice, sincronizat temporal cu citirile DAQ | ⦿ Înregistrare serie brută →**Creare video** | `lattice array-burst` → `lattice array-build-video` |

Regula generală: dacă pixelii vor furniza *măsurători*, utilizați captură sau serie (calitate de analiză); dacă aveți nevoie doar să *vizualizați sau să demonstrați* ce a înregistrat matricea, înregistrați videoclipul indexat (calitate de monitorizare).

### Setări de captură (GUI)

<!-- SCREENSHOT-NEEDED: Capture Settings pane (gear next to Capture All) with a connected array: capture-mode buttons (Single/Continuous/Interval), the bulk export-type toggle row, the Fastest Capture toggle, and the per-array group card showing the Aligned checkbox and the "Record index video" / "Record raw burst" buttons. -->



Roata dințată de lângă **Capture All** deschide panoul Setări de captură (necesită un proiect deschis — capturile sunt salvate în acesta):

* **Mod de captură**:**Single**(o singură trecere) /**Continuous**(consecutiv; limitat de un număr de capturi, implicit 1, sau de o durată, implicit 10 s) /**Interval** (timelapse: N capturi la fiecare interval X pentru un total de Y, implicit 1 la fiecare 5 s timp de 1 minut).
* **Tipuri de export pe cameră**: Raw, Debayered, Radiance, Reflectance, Preview, Index — toate opțiunile aplicabile sunt activate implicit. Radiance/Reflectance sunt ascunse pentru camerele cu filtru RGB; **Reflectance apare doar când camera are un senzor de lumină DAQ** (propriu sau moștenit de la matrice); Index necesită o expresie de index configurată.
* **Aliniat**(pe matrice, implicit**activat**): aliniază exporturile elementelor la profilul de aliniere al matricei, astfel încât exporturile să fie înregistrate la nivel de pixel. Fișierele Raw rămân întotdeauna nealiniate, dar conțin transformarea în metadate. Fișierele membre ale matricei sunt salvate în orientarea nativă a fiecărei camere (orice setare de oglindire/rotație este înregistrată în fișier, nu aplicată).
* **Captură rapidă** (comutator): numai format raw + citirea DAQ atribuită + compoziția cu indice combinat liber, omisând calculele de calibrare în momentul capturii pentru o viteză maximă — reconstruiți radianța/reflectanța/indicele ulterior din fișierul `.daq` salvat.
* Selecțiile rămân valabile pentru proiect. Camerele ascunse sau puse în pauză sunt omise.
* **Continuu** pe matrice rulează în backend-ul Chloros (nou în 1.3.0): când fiecare cameră din listă aparține unei matrice, backend-ul rulează singur bucla de captură, scriind un grup în timp ce îl alege pe următorul, fără nicio solicitare per captură. O unitate plină oprește rularea („Oprit: unitate plină. S-au salvat N, F a eșuat”). Consultați [Setări și moduri de captură](capture.md#capture-modes) pentru nota Microsoft Defender privind ratele ridicate de captură.

CLI-ul echivalent (același punct final al backend-ului, aceeași semantică):

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

Compresia TIFF pentru capturi este `deflate` (fără pierderi, implicită) sau `none` — tabelele complete de indicatori, structura folderului de capturi și regulile de reprocesare se găsesc în [Referința CLI](../reference/cli-reference.md#capture-modes-recorders--offline-reprocess).

## Asocierea unui senzor de lumină DAQ

Previzualizările cu reflectanță și iluminare corectate necesită date privind lumina descendentă de la un senzor DAQ (conectat în fila **Senzori de lumină**):

* **Rândul matricei**din bara laterală afișează un**buton „DAQ · pornit/oprit”** — *pornit* atunci când este configurat un senzor de lumină la nivel de matrice **sau** orice cameră din matrice are propriul senzor; informația de instrument indică exact ce senzor alimentează fiecare cameră.
* Alocați setarea la nivel de matrice în setările matricei → **Senzor de lumină ambientală**→ meniul derulant**Senzor de lumină**. Selecția se păstrează odată cu proiectul, se propagă către fiecare cameră din matrice, iar camerele individuale o pot înlocui în continuare cu propriul senzor.
* Linia de stare de sub aceasta raportează starea în timp real: **Oprit**→ „Se așteaptă primul spectru…” →**„Activ — toate camerele din matrice au iluminarea corectată”** → sau, dacă nu a sosit niciun spectru nou în ultimele 3 secunde, o notificare de date învechite — ultima citire continuă să fie utilizată (citirile nu expiră niciodată pe calea de captură).

Cu un senzor alocat: tipul de export „Reflectanță” devine disponibil, previzualizările în timp real sunt corectate din punct de vedere al iluminării, expunerea automată predictivă poate utiliza spectrul, iar fiecare captură de reflectanță scrie citirea DAQ utilizată efectiv ca **fișier `.daq`** în folderul capturilor `daq/`, astfel încât captură să poată fi reprocesată ulterior. Alături de acesta, un fișier `.daq.cameras.json` identifică camerele ale căror cadre sunt descrise de acea citire; la reprocesare, Chloros asociază citirea numai cu imaginile provenite de la acele camere.

## Opțiuni CLI

**pentru `array-connect`**

| Indicator | Implicit | Descriere |
| --- | --- | --- |
| `--serials SN1,SN2,…` | detectare automată a tuturor camerelor LATTICE (necesită ≥2) | **Primul număr de serie este MASTER.** |
| `--line {Line0,Line2,Line3}` | `Line2` | Linie de sincronizare GPIO. |
| `--target-fps F` | auto | Rata de declanșare a Masterului. |
| `--binning {1,2,4}` | auto | Binning hardware. Setarea „auto” înseamnă senzor complet (fără binning) ori de câte ori rata de cadre o permite; setarea „connect never” nu aplică binning doar pentru a menține rata ridicată. |
| `--packed {auto,on,off}` | `auto` | [Transport comprimat pe 12 biți](#packed-12-bit-transport): `auto` comprimă doar atunci când este util și gazda poate suporta decomprimarea. |
| `--force-tier {sim-capture-sim-emit, sim-capture-ftd-stagger, slip-emit-and-capture}` | auto | Suprascriere avansată a selectorului de nivel de sincronizare. |
| `--wire-ceiling-mbps MB_PER_S` | detectat automat | Bugetul de bandă al gazdei în MB/s — forma CLI a câmpului **Wire Budget**. Reduceți-l dacă matricea raportează cadre corupte. Se salvează odată cu proiectul, astfel încât o reconectare ulterioară îl restabilește. |
| `--no-recommend` | dezactivat | Omiteți etapa de analiză a rețelei. |
| `--no-ptp` | dezactivat | Dezactivați PTP (marcajele temporale între camere nu vor mai fi comparabile). |

`lattice array-list`, `array-status` și `array-disconnect` gestionează sesiunea persistentă. Referința completă a subcomenzilor, inclusiv alinierea (`align-calibrate` / `align-apply`) și instrumentele de rețea, se găsește în [Referința CLI § chloros-cli lattice](../reference/cli-reference.md#chloros-cli-lattice); echivalentele din SDK (`connect_array`, `ArraySession`, `attach_array`, `analyze_array_network`) se găsesc în [Referința SDK](../reference/sdk-reference.md). Din Python, bugetul de transmisie este `connect_array(..., wire_ceiling_mbps=120)`, iar separarea între datele corupte în timp real și cele care nu au ajuns niciodată se găsește în [`/api/camera/array/<id>/capability`](../reference/sdk-reference.md#array-health--which-subsystem-is-losing-frames).
