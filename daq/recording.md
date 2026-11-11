# Înregistrarea și formatul .daq

Un fișier `.daq` reprezintă formatul de înregistrare al senzorului de lumină dChloros

: o **bază de date SQLite** care conține cadre spectrale calibrate provenite de la un senzor DAQ. Înregistrați unul în timpul unei sesiuni de captură, iar procesul de prelucrare a reflectanței poate împărți ulterior fiecare imagine la iradianța descendentă măsurată exact în acel moment.

## Ce conține un fișier .daq

| Proprietate | Valoare |
| --- | --- |
| Container | Baza de date SQLite, un fișier per senzor per înregistrare |
| Nume fișier | Include **ID-ul senzorului**și o**marcă temporală**, de exemplu `daq_data_daq-e-def330_2026_04_13_18h30m00.daq` |
| Spectru pe cadru | 135 de puncte, 340–1010 nm cu pași de 5 nm, plus tristimulul CIE XYZ |
| Unități | Iradianță spectrală calibrată, **W/m²/nm** (pachet de calibrare din fabrică + profilul capacului aplicat) |
| Metadate înregistrate | ID-ul senzorului (cheia pentru preluarea calibrării din fabrică a respectivei unități) și profilul capacului aplicat — a se vedea [Profiluri ale capacului și intervalul calibrat](caps-and-range.md) |

Formatul este identic pentru DAQ-U, DAQ-M, DAQ-E și DAQ-E-S, astfel încât procesarea ulterioară nu ține cont de dispozitivul de transport care a înregistrat datele. O înregistrare DAQ-E-S stochează, de asemenea, înclinarea, rularea și pasul unității lângă fiecare spectru.

**Înregistrările supraviețuiesc unei întreruperi de curent (1.3.0).**O înregistrare în timp real este scrisă cu un jurnal și o sincronizare completă a discului la fiecare confirmare, astfel încât o întrerupere de curent sau un proces oprit duce la pierderea doar a ultimelor 3 secunde, nu a întregului fișier. O înregistrare oprită în timpul scrierii poate lăsa un fișier `<name>.daq-journal` lângă fișierul `.daq`:**păstrați și copiați cele două împreună.** Chloros ul revine la jurnalul anterior la prima citire a fișierului. Un disc ocupat întârzie fișierul, niciodată citirile senzorului.

**Un DAQ-E care încetează să mai transmită este reconectat (1.3.0).** Când fluxul unui DAQ-E se întrerupe (comutatorul său a fost repornit, o conexiune întreruptă), backend-ul îl reconectează cu setările pe care le avea, iar o înregistrare pe acesta continuă în același fișier `.daq`, cu intervalul marcat în fișier.

Înregistrarea calibrată necesită pachetul de calibrare din fabrică al senzorului. Pentru DAQ-U și DAQ-M, backend-ul preia pachetul din cloud-ul MAPIR pe baza ID-ului senzorului (înregistrarea este refuzată dacă nu se poate face acest lucru); unitățile DAQ-E sunt exceptate, deoarece păstrează calibrarea pe dispozitiv.

## Înregistrarea din interfața grafică

Înregistrarea în interfața grafică necesită un **proiect deschis** (în caz contrar, butoanele de înregistrare sunt dezactivate):

* **Înregistrează tot / Oprește tot** — în partea de sus a barei laterale Senzori de lumină; pornește sau oprește simultan o înregistrare `.daq` pe fiecare senzor conectat.
* **Înregistrare / Oprire înregistrare** — pentru fiecare senzor în parte, în fereastra modală de setări (rotița de setări). Un indicator roșu „REC” apare în rândurile de informații în timp real ale senzorului în timpul înregistrării.

Fișierele sunt salvate în `<project>/light_sensor/`, iar când o înregistrare se oprește — fie prin butonul „Oprire”, „Oprire totală” sau prin deconectarea unui senzor de înregistrare — fișierul `.daq` finalizat este **adăugat automat la proiectul deschis**. Acesta apare în lista de fișiere a proiectului fără a fi necesară adăugarea manuală, fiind deja pregătit pentru procesarea reflectanței.

<!-- SCREENSHOT-NEEDED: Light Sensors tab with one DAQ sensor connected and recording: sidebar showing the red "Stop All" state of the Record All button, the sensor row, and the settings modal open with the red "REC" indicator visible in the live info rows. -->

<!-- SCREENSHOT-NEEDED: File Browser / project file list immediately after stopping a DAQ recording, showing the .daq file auto-added to the open project alongside imagery. -->



## Înregistrarea din CLI



CLI înregistrează prin intermediul grupului de senzori al backend-ului (backend-ul trebuie să fie în funcțiune — aceste comenzi sunt clienți HTTP simpli):

```bash
# Connect the sensor into the backend pool
chloros-cli daq pool-connect --eth-host daq-e-def330.local

# Record for 150 seconds, with a human-friendly device label
chloros-cli daq pool-record --sensor-id daq-e-def330 --duration 150 \
    -o ./out --device-name "rooftop-A"

# Or run open-ended and stop explicitly
chloros-cli daq pool-record --sensor-id daq-e-def330            # --duration defaults to 0 = run until --stop
chloros-cli daq pool-record --sensor-id daq-e-def330 --stop
```

Obține valoarea `--sensor-id` din `chloros-cli daq pool-list`. Două valori implicite care merită menționate:

| Opțiune | Implicit |
| --- | --- |
| `--duration` | `0` — înregistrează până la `pool-record --stop` |
| `--output` / `-o` | `~/Documents/DAQ Live View/` pe sistemul de fișiere al **backend-ului**, nu al CLI-ului |

Distincția privind directorul de ieșire contează atunci când CLI-ul vizează un backend de pe o altă mașină: fișierul ajunge acolo unde rulează backend-ul. Dacă backend-ul se află pe același computer, un `-o` relativ (cum ar fi `./out` de mai sus) este rezolvat în raport cu folderul în care ați rulat comanda.

## Înregistrare din Python

`DAQSensorSession` (returnat de `chloros_sdk.connect_daq_sensor()`) expune aceeași înregistrare din pool: `record_start(output_dir=None, device_name=None)` returnează calea fișierului, iar `record_stop()` returnează `{path, rows}`. Consultați [Referința SDK](../reference/sdk-reference.md) pentru API-ul complet al sesiunii. Clasele SDK care accesează direct hardware-ul (numai pentru instalările pe desktop) scriu înregistrările în `~/Documents/DAQ/` în mod implicit; pentru versiunile lansate, calea grupată de mai sus este cea acceptată.

## Utilizarea unui fișier .daq în timpul procesării

Pentru a genera reflectanța din imagini, Chloros are nevoie de iradianța descendentă corelată cu fiecare expunere:

* **Păstrați fișierul `.daq` împreună cu imaginile.**În momentul procesării, fluxul de lucru determină automat**iradianța descendentă corelată cu marcajul temporal** dintr-un fișier `.daq` înregistrat (orice model DAQ) — sau dintr-un fișier `.csv` nativ DAQ-M — găsit alături de imagini. Înregistrările GUI îndeplinesc această cerință automat, deoarece sunt adăugate la proiect în momentul în care se opresc.
* **Calibrarea este preluată la cerere.** Dacă un pachet de calibrare din fabrică pentru fiecare cameră sau fiecare DAQ nu este deja stocat în cache la nivel local, Chloros îl preia automat din cloud-ul MAPIR la prima utilizare (este necesară o singură conexiune la internet; stocat în cache sub `~/.chloros/`).
* **Capturile live își scriu propriul fișier sidecar.** Pentru orice cadru de reflectanță capturat în timp real, valoarea DAQ utilizată efectiv este salvată ca fișier `.daq` în folderul `daq/` al capturilor, astfel încât captura să poată fi reprocesată ulterior fără înregistrarea originală.
* **Înregistrări de la înregistratoarele `chloros_scripts` (1.3.0).**Un fișier `.daq` creat de înregistratoarele publice [`chloros_scripts`](https://github.com/mapircamera/chloros_scripts) este listat ca modelul DAQ care l-a generat, iar limita sa poate fi setată în**Setări proiect → Senzor de lumină DAQ**, la fel ca în cazul oricărei alte înregistrări. Aceste înregistratoare înregistrează valorile brute ale senzorului; Chloros preia calibrarea din fabrică a senzorului prin interfața serială și o aplică.
* **Cui aparține valoarea măsurată (1.3.0).**Lângă fiecare captură `.daq`, un `.daq.cameras.json` indică camera (camerele) ale căror cadre sunt descrise de acea citire, iar procesarea o asociază**numai cu imaginile de la acele camere**. Prin urmare, o cameră autonomă care nu are propriul senzor de lumină nu mai preia valoarea de la o altă cameră — asociați-i un senzor sau înregistrați una. O înregistrare realizată cu DAQ-ul însuși (GUI Record, `daq pool-record`) și orice `.daq` fără acel fișier (inclusiv capturile mai vechi) servesc în continuare fiecare cameră.

## Recuperarea iradianței

Prelucrarea unui proiect exportă, de asemenea, fiecare înregistrare a senzorului de lumină pe care o conține, într-un
`Light Sensor/` lângă produsele imagistice. Acest lucru **nu** necesită imagini: o
înregistrare a senzorului de lumină realizată separat este o captură completă, iar un folder care conține doar fișiere `.daq`
este o intrare validă. Un proiect care conține doar înregistrări ale senzorilor de lumină le listează și
le prelucrează atât din aplicație, cât și din CLI, iar acestea rămân listate după execuție. Execuția
raportează câte produse ale senzorilor de lumină a scris.

| Produs | Ce este |
| --- | --- |
| `<name>_calibrated.daq` | O arhivă care poate fi reprocesată, având aceeași structură ca o înregistrare în timp real, care declară acum pachetul de calibrare care a generat-o. Reimportarea acesteia **nu** o calibrează a doua oară. |
| `<name>_calibrated.csv` | Iradianța spectrală în W/m²/nm pe grila de lungimi de undă proprie a senzorului, un rând per citire, plus coloane fotometrice: puterea totală, lux fotopic și scotopic, PPFD cu defalcarea albastru/verde/roșu și lungimea de undă de vârf. |

Un DAQ-U sau DAQ-M al cărui pachet de calibrare nu poate fi preluat — sunteți offline sau
senzorul respectiv nu are nicio calibrare înregistrată — este **omisi cu un motiv**, nefiind niciodată salvat
ca fișier „calibrat” care conține date brute. Conectați-vă la internet și rulați din nou. Un DAQ-E
are propria sa calibrare, așa că are nevoie de aceasta doar atunci când unitatea nu este conectată și
nimic nu este stocat în cache local.

### DAQ-A: valori brute și de ce acesta este răspunsul corect

**DAQ-A** este anterior sistemului de pachete de calibrare pe serie și nu are niciun pachet de
preluat. Nu este vorba de o omisiune: un DAQ-A este calibrat pe teren în raport cu o
țintă de reflectanță, iar calibrarea bazată pe țintă necesită doar răspunsul *relativ*
al senzorului — ceea ce reprezintă exact valorile brute ale acestuia. Chloros se calibrează astăzi folosind aceste valori.

Așadar, o înregistrare DAQ-A se exportă, dar sub un nume diferit:

```
<project>/
└── Light Sensor/
    ├── <name>_raw.daq
    └── <name>_raw.csv
```

`_raw`, nu `_calibrated` — un nume de fișier diferit, nu un indicator în interiorul fișierului,
deoarece informația trebuie să rămână intactă chiar și atunci când fișierul este trimis prin e-mail doar ca nume simplu. Antetul `.csv`
indică `raw spectral sensor counts (NOT irradiance)` și avertizează că valorile sunt
comparabile **în cadrul** fișierului și nu între senzori. Coloanele care au semnificație
doar pentru iradianța reală — puterea totală, lux, PPFD — sunt lăsate goale, în loc să fie
calculate pe baza numărului de impulsuri.

Înregistrările DAQ-A-SD mai vechi (schema v1.01 / v1.02) înregistrează doar ora de scriere a fișierului, nu o
marcă temporală pentru fiecare citire. Chloros nu va corela imaginile cu acestea — asocierea unui cadru cu un
moment de scriere ar fi greșită, chiar dacă la prima vedere nu ar părea așa — dar exportul le citește corect, iar
fișierul CSV precizează la ce ceas se referă.

Pentru informații complete despre reflectanță — senzor unic cu cameră și senzor dublu (mediu/obiect) — consultați [Fluxuri de lucru pentru reflectanță](reflectance.md).
