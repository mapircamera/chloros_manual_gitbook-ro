# Senzori de lumină DAQ

> **Căutați informații despre hardware?**Senzorii în sine — modelele, montarea, capacele, porturile, alimentarea și aplicația SCANNER — sunt descrise în**[manualul de utilizare DAQ](https://mapir.gitbook.io/daq)**. Acest capitol tratează utilizarea acestora începând cu Chloros.

Senzorii de lumină **DAQ** din MAPIR măsoară lumina ambientală sub formă de spectre calibrate radiometric. În Chloros, aceștia îndeplinesc două roluri:

* **Un instrument spectral autonom** — diagrame spectrale în timp real, date colorimetrice și înregistrări `.daq`, toate accesibile din [fila Senzori de lumină](gui.md), [CLI](cli-quick-start.md) sau din Python SDK.
* **O sursă de iradianță descendentă pentru reflectanță** — în timpul procesării, Chloros interpolează valorile `.daq` pentru fiecare capturăși utilizează lumina descendentă măsurată pentru a converti radianța camerei în reflectanță (`--reflectance-source daq`), fără a fi necesar un panou în scenă pentru benzile calibrate.

<!-- SCREENSHOT-NEEDED: product photo of the DAQ-U, DAQ-M, and DAQ-E units side by side, each with its Sunshine cosine-corrector cap fitted (request from hardware team — no repo asset exists) -->

***

## Trei modele, un singur format de date

| Model | Transport | Descoperire |
| --- | --- | --- |
| **DAQ-U** | USB (serial) | scanare prin port serial |
| **DAQ-M** | Bluetooth Low Energy | scanare BLE după nume |
| **DAQ-E** | Ethernet (IPv4, alimentat prin PoE) | mDNS `_daq-e._tcp` (nume de gazdă `daq-e-<id>.local`) |

Toate cele trei utilizează același protocol de comunicație și furnizează date identice:

* Un **spectru de 135 de puncte de la 340 la 1010 nm, cu pași de 5 nm**, plus valorile tristimulus CIE XYZ, în fiecare cadru.
* **Iradianța spectrală calibrată radiometric în W/m²/nm** — pachetul de calibrare din fabrică al fiecărei unități (împreună cu profilul său activ de corecție a capacului) este aplicat înainte ca datele să ajungă la dumneavoastră.
* Același **format de înregistrare `.daq`** (un fișier SQLite). Procesarea ulterioară este identică, indiferent de modul de transport care a generat fișierul.

Stivele de transport (serial USB, BLE, mDNS/zeroconf) sunt incluse în backend-ul Chloros — nu este necesară instalarea nimicului pentru a comunica cu oricare dintre cele trei modele din interfața grafică sau prin intermediul comenzilor `pool-*` ale CLI.

***

## Interval calibrat: 340–1010 nm raportat, ~374–974 nm calibrat

Senzorul raportează grila completă de 340–1010 nm, dar câștigul radiometric trasabil conform NIST acoperă aproximativ **374–974 nm**. Chloros refuză împărțirea la reflectanța absolută pentru orice bandă a camerei cu mai puțin de jumătate din ponderea sa spectrală în interiorul acelui interval calibrat; banda omisă este raportată cu motivul de omitere `dls-uncalibrated-band-<nm>`.

Dintre codurile de produs (SKU) ale filtrelor LATTICE disponibile pe piață, doar **F988** este afectat:

Reflectanța F988 este calibrată folosind un panou de reflectanță din scenă: banda se află în afara intervalului calibrat al senzorului de lumină DAQ, astfel încât Chloros aplică cea mai recentă captură a panoului și o menține între observările panoului.

Dacă o captură F988 este procesată având la dispoziție doar date DAQ, Chloros respinge reflectanța bazată pe DAQ pentru banda respectivă cu motivul de omitere `dls-uncalibrated-band-988` — [fluxul de lucru cu panoul de reflectanță](../calibration-targets.md) este calea acceptată pentru F988.

***

## ID-urile senzorilor

Fiecare DAQ raportează un ID de senzor stabil. Forma acestuia diferă în funcție de model:

| Model | Forma ID-ului | Exemplu |
| --- | --- | --- |
| DAQ-U | 5 octeți, cu cratime | `CB-7C-A8-2E-5F` |
| DAQ-M | 5 octeți, cu cratime | `CB-74-02-30-6B` |
| DAQ-E | `daq-e-<6 hex digits>` | `daq-e-def330` |

ID-ul senzorului este:

* înscris în fiecare fișier `.daq` pe care îl înregistrează,
* cheia pe care Chloros o utilizează pentru a prelua pachetul de calibrare din fabrică al respectivei unități,
* valoarea pe care o transmiteți către `--sensor-id` în comenzile CLI și `pool-*`, și
* pentru DAQ-E, de asemenea, numele de gazdă mDNS al acestuia (`daq-e-def330.local`) — valoarea acceptată de `--eth-host`.

***

## Calibrarea din fabrică și cloud-ul

Fiecare unitate DAQ este calibrată individual din fabrică cu un lanț radiometric trasabil conform NIST, iar Chloros încarcă pachetul de calibrare al fiecărei unități, asociat cu ID-ul senzorului. Raportul de calibrare pentru fiecare unitate (PDF) poate fi descărcat din setările senzorului din [fila Senzori de lumină](gui.md).

{% hint style="warning" %}
**DAQ-U și DAQ-M necesită acces la cloud pentru calibrare.**Niciunul dintre aceste modele nu stochează date pe dispozitiv: pachetele lor de calibrare din fabrică se află în cloud-ul MAPIR și sunt preluate pe baza ID-ului senzorului (apoi stocate în cache local). Chloros are nevoie de o conexiune la internet pentru a furniza date calibrate în W/m²/nm de la un DAQ-U sau DAQ-M.**DAQ-E este excepția** — acesta stochează calibrarea pe dispozitiv.

<!-- PRE-PUBLISH-CHECK: LAUNCH item 3 (DAQ-M end-to-end connect smoke) was still unverified as of 2026-08-16 — re-confirm the DAQ-M cloud-calibration flow on the release build before publishing this page. -->

{% endhint %}***

## Unde sunt stocate înregistrările

| Suprafață | Destinație implicită `.daq` |
| --- | --- |
| Interfața grafică — fila Senzori de lumină | `<project folder>/light_sensor/` (înregistrările finalizate sunt adăugate automat la proiect) |
| CLI — `daq pool-record` | `~/Documents/DAQ Live View/` pe mașina care rulează backend-ul |

Fiecare nume de fișier `.daq` include ID-ul senzorului și o marcă temporală.

***

## În acest capitol

* [**Fila DAQ din Chloros**](gui.md) — ghid complet al interfeței grafice: conectarea fiecărui model, setări pentru fiecare senzor, diagrame de spectru, date colorimetrice în timp real, reflectanță cu senzor dublu și înregistrare.
* [**CLI Ghid de pornire rapidă (pool-\*)**](cli-quick-start.md) — controlarea senzorilor DAQ din `chloros-cli daq pool-*`, calea acceptată din linia de comandă.
* [**Profiluri de limite și interval calibrat**](caps-and-range.md) — ce limite există pentru fiecare model, cum se declară acestea și intervalul spectral calibrat în detaliu.
* [**Înregistrarea și formatul .daq**](recording.md) — formatul SQLite `.daq` și fluxurile de lucru de înregistrare.
* [**Rețeaua DAQ-E și sincronizarea temporală**](ethernet-ptp.md) — modurile de transport DAQ-E și sincronizarea temporală PTP.
* [**Fluxuri de lucru pentru reflectanță**](reflectance.md) — utilizarea datelor de radiație descendentă din DAQ pentru a genera reflectanța.
* Pentru documentația completă la nivel de indicator, consultați [Referința CLI](../reference/cli-reference.md) (secțiunea `chloros-cli daq`) și [Referința SDK](../reference/sdk-reference.md) (`chloros_sdk.connect_daq_sensor()`), ambele redactate astfel încât să poată fi utilizate direct de asistenții AI.
