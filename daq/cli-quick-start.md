# CLI Ghid de pornire rapidă (pool-*)

Unitatea `chloros-cli` livrată controlează senzorii DAQ prin intermediul familiei de comenzi **`daq pool-*`** — clienți HTTP de tip „thin” care controlează senzorul prin intermediul grupului de senzori persistent al backend-ului Chloros. Backend-ul deține controlul asupra transportului, astfel încât interfața grafică (GUI), scripturile CLI și SDK împart toate un singur identificator activ, în loc să se concureze pentru port. Tot ce are nevoie un client este accesibil prin `pool-*`: conectare, transmisie în flux, înregistrare de fișiere `.daq` calibrate și schimbarea profilurilor de capac.

`pool-*` este, de asemenea, **singura** interfață DAQ din versiunile lansate. `chloros-cli daq --help` listează subcomenzile `pool-*`, iar apelarea unei subcomenzi DAQ de acces direct la hardware într-o versiune lansată duce la ieșirea din program cu o eroare explicită care menționează pachetul lipsă și vă redirecționează către `pool-*` — nimic nu eșuează în tăcere. (Comenzile de acces direct la hardware se execută numai dintr-o versiune MAPIR a codului sursă; nici `pip install chloros-sdk` nu le pune la dispoziție.)

***

## Cerințe preliminare

* **Backend-ul Chloros trebuie să fie în execuție** — comenzile `pool-*` sunt clienți HTTP, nu drivere hardware. Pe Windows, porniți aplicația desktop Chloros (aceasta lansează backend-ul). Pe Linux/Jetson fără ecran, activați serviciul: `sudo systemctl enable --now chloros-backend.service`.
* **Autentificare Chloros+ (nivel cu plată)**: rulați mai întâi `chloros-cli login`. Aplicarea se face pe partea de server — fără autentificare, comenzile eșuează cu `401 AUTH_REQUIRED`; în planul gratuit (Iron), acestea eșuează cu `403 PLAN_UPGRADE_REQUIRED`.
* Comenzile vizează `http://127.0.0.1:5000` în mod implicit; familia `daq pool-*` respectă variabila de mediu `CHLOROS_BACKEND_URL` dacă backend-ul dvs. rulează în altă parte.

***

## O sesiune de cinci minute

```bash
# 1. Connect a sensor into the backend pool (pick the line matching your model)
chloros-cli daq pool-connect                                  # smart-detect any DAQ
chloros-cli daq pool-connect --port COM3                      # DAQ-U on a specific COM port
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF          # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-def330.local    # DAQ-E by hostname (reliable)

# 2. List the pool — this shows the sensor_id used by every command below
chloros-cli daq pool-list

# 3. Read the most recent calibrated spectrum frame (add --json for scripting)
chloros-cli daq pool-latest --sensor-id daq-e-def330 --json

# 4. Record a calibrated .daq file for 60 seconds
chloros-cli daq pool-record --sensor-id daq-e-def330 --duration 60 \
  --device-name "field-A"

# 5. Release the sensor when done
chloros-cli daq pool-disconnect --sensor-id daq-e-def330
```

***

## `pool-connect` — deschide un senzor din grup

| Variantă | Semnificație |
| --- | --- |
| `daq pool-connect` | Detectare inteligentă: găsește orice DAQ de pe această mașină. |
| `daq pool-connect --port PORT` | DAQ-U pe un port serial specific (de ex. `COM3`, `/dev/ttyUSB0`). |
| `daq pool-connect --ble` | DAQ-M prin BLE, adresă MAC scanată automat. |
| `daq pool-connect --mac MAC` | DAQ-M la un MAC BLE cunoscut (implică `--ble`). |
| `daq pool-connect --eth-host HOST` | DAQ-E la un nume de gazdă sau o adresă IP cunoscută — **calea fiabilă**. |
| `daq pool-connect --eth` | DAQ-E cu detectare automată (mDNS, cu recurgere la ARP). A se vedea avertismentul de mai jos. |

Opțiuni de configurare, toate opționale:

| Opțiune | Semnificație |
| --- | --- |
| `--integration-time MS` / `-t MS` | Timpul de integrare manual în milisecunde. |
| `--frame-avg N` / `-f N` | Numărul mediu de cadre pe spectru raportat. |
| `--no-ae` | Dezactivează expunerea automată (AE este activată implicit). |
| `--no-stream` | Conectare fără pornirea fluxului (se reia mai târziu cu `pool-stream --start`). |
| `--cap-id CAP` | Profil de corecție a plafonului; valoarea implicită a backend-ului este `sunshine_cosine`. Vezi [`pool-set-cap`](#pool-set-cap-declare-the-fitted-cap). |

{% hint style="warning" %}
**Avertisment privind descoperirea automată a `--eth`.** Pe o gazdă cu mai multe interfețe de rețea (mai mult de o interfață de rețea activă), primul `pool-connect --eth` după pornire poate rămâne gol chiar și dacă senzorul este în stare bună — procesul de descoperire poate omite interfața senzorului cât timp cache-ul ARP este gol. Dacă `--eth` nu găsește nimic, reîncercați sau omiteți complet procesul de detectare folosind `--eth-host <ip-or-hostname>`, care este calea fiabilă pe mașinile cu mai multe interfețe de rețea. Numele de gazdă al DAQ-E este `daq-e-<id>.local` (de exemplu, `daq-e-def330.local`); funcționează și adresa IP simplă a acestuia.
{% endhint %}

## `pool-list` — vedeți ce este conectat

Afișează fiecare senzor din grupul din fundal, inclusiv `sensor_id`, de care au nevoie toate celelalte comenzi:

| Model | Format `sensor_id` | Exemplu |
| --- | --- | --- |
| DAQ-U / DAQ-M | 5 octeți cu cratime | `CB-7C-A8-2E-5F` |
| DAQ-E | `daq-e-<6 hex digits>` | `daq-e-def330` |

## `pool-latest` — citirea cadrelor de spectru

```bash
chloros-cli daq pool-latest --sensor-id daq-e-def330 --recent 10 --json
```

Returnează cel mai recent cadru sau cele mai recente cadre `--recent N`; `--json` generează o ieșire lizibilă de către mașini pentru scripturi. Cadrele reprezintă iradianța spectrală (W/m²/nm) calibrată radiometric pe grila de 135 de puncte, cu intervalul 340–1010 nm, cu profilul capacului senzorului deja aplicat. Pentru valori cantitative ale iradianței, se calculează media a cel puțin 15 secunde de cadre — aceasta este o caracteristică a instrumentului, nu un defect.

## `pool-stream` — întrerupere sau reluare a transmiterii în flux

```bash
chloros-cli daq pool-stream --sensor-id daq-e-def330 --stop    # pause
chloros-cli daq pool-stream --sensor-id daq-e-def330 --start   # resume
```

## `pool-record` — înregistrați un fișier `.daq`

```bash
chloros-cli daq pool-record --sensor-id daq-e-def330 --duration 150 \
  --output ~/Documents/spectra --device-name "rooftop-A"
chloros-cli daq pool-record --sensor-id daq-e-def330 --stop
```

| Indicator | Implicit | Semnificație |
| --- | --- | --- |
| `--duration SEC` / `-d SEC` | `0` | Durata înregistrării în secunde; `0` înseamnă rulare până la emiterea comenzii `--stop`. |
| `--output DIR` / `-o DIR` | `~/Documents/DAQ Live View/` | Directorul de ieșire, rezolvat **pe mașina care rulează backend-ul**. |
| `--device-name NAME` | — | Etichetă stocată împreună cu înregistrarea. |
| `--stop` | — | Oprește o înregistrare în curs de desfășurare. |

{% hint style="info" %}
Înregistrarea are loc în backend, astfel încât fișierul `.daq` ajunge în sistemul de fișiere al **mașinii backend** — în mod implicit în `~/Documents/DAQ Live View/` acolo, nu neapărat acolo unde ați rulat CLI. Numele fișierelor includ ID-ul senzorului și o marcă temporală.
{% endhint %}

## `pool-set-cap` — declararea capacului montat

```bash
chloros-cli daq pool-set-cap --sensor-id daq-e-def330 --cap-id sunshine_cosine
```

ID-ul capacului selectează profilul de corecție măsurat din fabrică aplicat fiecărui spectru și **trebuie să corespundă cu capacul montat fizic pe senzor** — nici senzorul, nici software-ul nu pot detecta singure capacul, iar selecția este înregistrată în fiecare fișier `.daq`. Valoarea implicită peste tot este `sunshine_cosine` (fiecare DAQ este livrat cu capacul de corecție cosinus „Sunshine” instalat, cu o atenuare de ~12× conform proiectului — o schimbare nedeclarată a capacului corectează eronat spectrele cu aproximativ acest factor).

| `--cap-id` | Disponibil pe |
| --- | --- |
| `sunshine_cosine` (implicit) | DAQ-U, DAQ-M, DAQ-E |
| `fov_15`, `fov_45`, `fov_90` | DAQ-U, DAQ-E |
| `fov_30`, `fov_60` | numai DAQ-U |
| `none` | numai DAQ-E — a se vedea nota |

Un ID de capac care nu face parte din setul senzorului este respins la conectare, generând o eroare clară. `none` (DAQ-E) înseamnă că capacul a fost îndepărtat fizic — se aplică în continuare un profil geometric din fabrică pentru difuzorul de sticlă încastrat al DAQ-E, deci nu este o operație inactivă, iar un DAQ-E fără capac reprezintă o configurație de banc de testare, nu o configurație de teren acceptată. (Un DAQ-U fără capac este cu adevărat „gol” și nu necesită deloc profil de corecție; DAQ-M se utilizează împreună cu capacul său Sunshine.)

## `pool-disconnect` — eliberarea senzorilor

```bash
chloros-cli daq pool-disconnect --sensor-id daq-e-def330   # one sensor
chloros-cli daq pool-disconnect --all                      # everything in the pool
```

***

## Rezumatul comenzilor

| Comandă | Scop |
| --- | --- |
| `daq pool-connect [--port P \| --ble \| --mac M \| --eth \| --eth-host H] [-t MS] [-f N] [--no-ae] [--no-stream] [--cap-id CAP]` | Deschide un senzor din grupul de senzori din backend. |
| `daq pool-list` | Afișează fiecare senzor din grup împreună cu `sensor_id`. |
| `daq pool-latest --sensor-id ID [--recent N] [--json]` | Cele mai recente N cadre de spectru calibrate. |
| `daq pool-stream --sensor-id ID [--start \| --stop]` | Reluare / pauză a transmiterii în flux. |
| `daq pool-record --sensor-id ID [-d SEC] [-o DIR] [--device-name NAME] [--stop]` | Pornire / oprire a unei înregistrări `.daq` (pe partea backend). |
| `daq pool-set-cap --sensor-id ID --cap-id CAP` | Schimbă profilul de corecție a capacității în timpul rulării. |
| `daq pool-disconnect --sensor-id ID [--all]` | Eliberează un senzor sau pe toți. |

***

## Depanarea primei conectări a DAQ-E

1. DAQ-E nu are LED de stare — verificați alimentarea prin intermediul indicatorului PoE/link de pe comutator sau de pe portul injectorului și așteptați câteva secunde după pornire pentru ca acesta să se inițializeze și să se conecteze la rețea.
2. Calculatorul backend trebuie să se afle în **același domeniu de difuzare** cu senzorul — mDNS nu traversează routerele.
3. Pe Windows, acceptați solicitarea firewall-ului Defender la prima rulare (mDNS UDP 5353, date DAQ-E UDP 5002, PTP UDP 319/320).
4. Încă nu primiți nimic de la `--eth`? Utilizați `--eth-host` cu numele de gazdă al unității (`daq-e-<id>.local`) sau cu adresa IP — aceasta este calea fiabilă, în special în cazul gazdelor cu mai multe adrese IP.

***{% hint style="info" %}**Sfat pentru asistenții AI.** Fiecare pagină a acestui manual este furnizată în format Markdown brut — adăugați `.md` la slug-ul cu litere mici al paginii, URL (această pagină: `https://mapir.gitbook.io/chloros/daq/cli-quick-start.md`); indexul lizibil de către mașini este `https://mapir.gitbook.io/chloros/llms.txt`. Pentru documentația completă la nivel de opțiuni a `chloros-cli daq` și a oricărei alte familii de comenzi, accesați [Referința CLI](../reference/cli-reference.md) (`https://mapir.gitbook.io/chloros/reference/cli-reference.md`); calea către Python este `chloros_sdk.connect_daq_sensor()` în [Referința SDK](../reference/sdk-reference.md).
{% endhint %}
