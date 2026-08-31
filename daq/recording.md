# Înregistrarea și formatul .daq

Un fișier `.daq` reprezintă formatul de înregistrare al senzorului de lumină Chloros: o **bază de date SQLite** conținând cadre spectrale calibrate provenite de la un singur senzor DAQ. Înregistrați unul în timpul unei sesiuni de captură, iar procesul de prelucrare a reflectanței poate împărți ulterior fiecare imagine la iradianța descendentă măsurată exact în acel moment.

## Ce conține un fișier .daq

| Proprietate | Valoare |
| --- | --- |
| Container | Baza de date SQLite, un fișier per senzor per înregistrare |
| Nume fișier | Include **ID-ul senzorului**și o**marcă temporală**, de exemplu `daq_data_daq-e-def330_2026_04_13_18h30m00.daq` |
| Spectru pe cadru | 135 de puncte, 340–1010 nm cu pași de 5 nm, plus tristimulul CIE XYZ |
| Unități | Iradianță spectrală calibrată, **W/m²/nm** (pachet de calibrare din fabrică + profilul capacului aplicat) |
| Metadate înregistrate | ID-ul senzorului (cheia pentru preluarea calibrării din fabrică a respectivei unități) și profilul capacului în vigoare — a se vedea [Profiluri ale capacului și intervalul calibrat](caps-and-range.md) |

Formatul este identic pentru DAQ-U, DAQ-M și DAQ-E, astfel încât procesarea ulterioară nu ține cont de dispozitivul de transport care a înregistrat datele.

Înregistrarea calibrată necesită pachetul de calibrare din fabrică al senzorului. Pentru DAQ-U și DAQ-M, backend-ul preia pachetul din cloud-ul MAPIR pe baza ID-ului senzorului (înregistrarea este refuzată dacă acest lucru nu este posibil); unitățile DAQ-E sunt exceptate, deoarece păstrează calibrarea pe dispozitiv.

## Înregistrarea din interfața grafică

Înregistrarea în interfața grafică (GUI) necesită un **proiect deschis** (în caz contrar, butoanele de înregistrare sunt dezactivate):

* **Înregistrează tot / Oprește tot** — în partea de sus a barei laterale „Senzori de lumină”; pornește sau oprește simultan o înregistrare `.daq` pe fiecare senzor conectat.
* **Înregistrare / Oprire înregistrare** — pentru fiecare senzor în parte, în fereastra de setări cu rotița de angrenaj. Un indicator roșu „REC” apare în rândurile cu informații în timp real ale senzorului în timpul înregistrării.

Fișierele sunt salvate în `<project>/light_sensor/`, iar atunci când o înregistrare se oprește — fie prin butonul Oprire, Oprire totală, fie prin deconectarea unui senzor de înregistrare — fișierul `.daq` finalizat este **adăugat automat la proiectul deschis**. Acesta apare în lista de fișiere a proiectului fără a fi necesară adăugarea manuală, fiind deja pregătit pentru procesarea reflectanței.

<!-- SCREENSHOT-NEEDED: Light Sensors tab with one DAQ sensor connected and recording: sidebar showing the red "Stop All" state of the Record All button, the sensor row, and the settings modal open with the red "REC" indicator visible in the live info rows. -->

<!-- SCREENSHOT-NEEDED: File Browser / project file list immediately after stopping a DAQ recording, showing the .daq file auto-added to the open project alongside imagery. -->

## Înregistrarea de la CLI

CLI înregistrează prin intermediul grupului de senzori al backend-ului (backend-ul trebuie să fie în funcțiune — aceste comenzi sunt clienți HTTP de tip „thin”):

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

| Opțiune | Implicită |
| --- | --- |
| `--duration` | `0` — înregistrează până la `pool-record --stop` |
| `--output` / `-o` | `~/Documents/DAQ Live View/` pe sistemul de fișiere al **backend-ului**, nu pe cel al CLI |

Distincția privind directorul de ieșire este importantă atunci când CLI vizează un backend de pe o altă mașină: fișierul ajunge acolo unde rulează backend-ul.

## Înregistrare de la Python

`DAQSensorSession` (returnat de `chloros_sdk.connect_daq_sensor()`) expune aceeași înregistrare din pool: `record_start(output_dir=None, device_name=None)` returnează calea fișierului, iar `record_stop()` returnează `{path, rows}`. Consultați [Referința SDK](../reference/sdk-reference.md) pentru sesiunea completă API. Clasele de hardware direct ale SDK (numai pentru instalările pe desktop) scriu înregistrările în `~/Documents/DAQ/` în mod implicit; pentru versiunile lansate, calea comună de mai sus este ruta acceptată.

## Utilizarea unui fișier .daq în timpul procesării

Pentru a genera reflectanța din imagini, Chloros are nevoie de iradianța descendentă corelată cu fiecare expunere:

* **Păstrați fișierul `.daq` împreună cu imaginile.**În momentul procesării, fluxul de lucru determină automat**iradianța descendentă corelată cu marca temporală** dintr-un fișier `.daq` înregistrat (orice model DAQ) — sau dintr-un fișier `.csv` nativ DAQ-M — găsit alături de imagini. Înregistrările GUI îndeplinesc această cerință automat, deoarece sunt adăugate la proiect în momentul în care se opresc.
* **Calibrarea este preluată la cerere.** Dacă un pachet de calibrare din fabrică pentru fiecare cameră sau fiecare DAQ nu este deja stocat în cache local, Chloros îl preia automat din cloud-ul MAPIR la prima utilizare (este necesară o singură conexiune la internet; se memorează în cache sub `~/.chloros/`).
* **Capturile în timp real își scriu propriul fișier sidecar.** Pentru orice cadru de reflectanță capturat în timp real, valoarea citită de DAQ utilizată efectiv este salvată ca un fișier sidecar `.daq` alături de imagini, astfel încât captura să poată fi reprocesată ulterior fără înregistrarea originală.

## Recuperarea iradianței

Prelucrarea unui proiect exportă, de asemenea, fiecare înregistrare a senzorului de lumină pe care o conține, într-un
folder `Light Sensor/` situat lângă produsele imagistice. Acest lucru **nu** necesită imagini: un
senzor de lumină zburat independent reprezintă o captură completă, iar un folder care conține doar fișiere `.daq`
este o intrare validă. Rularea raportează câte produse ale senzorului de lumină a scris.

| Produs | Ce este |
| --- | --- |
| `<name>_calibrated.daq` | O arhivă reprocesabilă în același format ca o înregistrare în timp real, care declară acum pachetul de calibrare care a generat-o. Reimportarea acesteia **nu** o calibrează a doua oară. |
| `<name>_calibrated.csv` | Iradianța spectrală în W/m²/nm pe grila de lungimi de undă proprie a senzorului, un rând pentru fiecare citire, plus coloane fotometrice: puterea totală, lux fotopic și scotopic, PPFD cu defalcarea albastru/verde/roșu și lungimea de undă de vârf. |

Un DAQ-U sau DAQ-M al cărui pachet de calibrare nu poate fi preluat — sunteți offline sau
senzorul respectiv nu are nicio calibrare înregistrată — este **omisi cu un motiv**, nefiind niciodată salvat
ca fișier „calibrat” care conține date brute. Conectați-vă la internet și rulați din nou. Un DAQ-E
are propria sa calibrare, așa că are nevoie de aceasta doar când unitatea nu este conectată și
nu este nimic stocat în cache la nivel local.

### DAQ-A: valori brute și de ce acesta este răspunsul corect

**DAQ-A** este anterior sistemului de pachete de calibrare pe serie și nu are niciun pachet de
preluat. Nu este vorba de o omisiune: un DAQ-A este calibrat pe teren folosind o
țintă de reflectanță, iar calibrarea bazată pe țintă necesită doar răspunsul *relativ*
al senzorului — care este exact ceea ce reprezintă numărările brute. Chloros se calibrează astăzi folosindu-le.

Așadar, o înregistrare DAQ-A se exportă, dar sub un alt nume:

```
<project>/
└── Light Sensor/
    ├── <name>_raw.daq
    └── <name>_raw.csv
```

`_raw`, nu `_calibrated` — un nume de fișier diferit, nu un indicator în interiorul fișierului,
deoarece informația trebuie să se păstreze chiar și atunci când fișierul este trimis prin e-mail doar cu numele său simplu. Antetul `.csv`
indică `raw spectral sensor counts (NOT irradiance)` și avertizează că valorile sunt
comparabile **în cadrul** fișierului și nu între senzori. Coloanele care au semnificație
doar pentru iradianța reală — puterea totală, lux, PPFD — sunt lăsate goale, în loc să fie
calculate pe baza numărului de impulsuri.

Înregistrările DAQ-A-SD mai vechi (schema v1.01 / v1.02) înregistrează doar ora de scriere a fișierului, nu un
timestamp pentru fiecare citire. Chloros nu va corela imaginile cu acestea — asocierea unui cadru cu un
moment de scriere ar fi greșită, fără să pară vreodată greșită — dar exportul le citește corect, iar
CSV precizează pe ce ceas se bazează.

Pentru informații complete despre reflectanță — senzor unic cu cameră și senzor dublu (mediu/obiect) — consultați [Fluxuri de lucru privind reflectanța](reflectance.md).
