# Profiluri ale capacelor și domeniu calibrat

> Capacele în sine — care se livrează cu ce senzor, modul în care se montează și comportamentul lor optic — sunt documentate în **[manualul de utilizare DAQ](https://mapir.gitbook.io/daq)**. Această pagină tratează *declararea* capacului montat către Chloros
, ceea ce asigură corectitudinea corecției.

Calibrarea radiometrică din fabrică a fiecărui senzor de lumină DAQ descrie senzorul *gol*. Capacul fizic montat peste difuzor modifică lumina colectată de senzor, astfel încât Chloros
aplică un **profil de corecție a capacului** măsurat din fabrică peste pachetul de calibrare. Declararea capacului corect face parte din procesul de obținere a datelor calibrate — această pagină prezintă ce capace există pentru fiecare model, cum se declară acestea și care este de fapt domeniul spectral calibrat al senzorului.

## Disponibilitatea capacelor în funcție de model

| Profilul capacului (`cap_id`) | Capac fizic | DAQ-U | DAQ-M | DAQ-E |
| --- | --- | --- | --- | --- |
| `sunshine_cosine` | Capac de corecție cosinus pentru lumina solară (**implicit pe fiecare model**) | Da | Da | Da |
| `fov_15` / `fov_45` / `fov_90` | Conuri de restricționare a câmpului vizual (15° / 45° / 90°) | Da | — | Da |
| `fov_30` / `fov_60` | Conuri de limitare a câmpului vizual (30° / 60°) | Da | — | — |
| `none` | Fără capac montat | — | — | Da |

Note specifice modelului:

* **DAQ-M are un singur profil de capac: `sunshine_cosine`.** „Bare-plus-Sunshine-cap” este denumirea produsului, iar un DAQ-M fără capac nu necesită profil geometric.
* **Un DAQ-U fără capac este cu adevărat „bare”** — nu necesită deloc profil geometric, motiv pentru care nu există profilul `none` pentru acesta.
* **`none` pe un DAQ-E NU este o operație inactivă.** Difuzorul încastrat și acoperit cu sticlă al modelului DAQ-E are o corecție geometrică reală proprie, astfel încât „fără capac” este în sine un profil măsurat pe acest model.
* Un **DAQ-E fără capac nu poate măsura lumina directă a soarelui la nicio înălțime** — capacul Sunshine reprezintă configurația de teren. Nu planificați lucrări în aer liber cu un DAQ-E fără capac.

În setările per senzor din interfața grafică (pictograma roată din fila „Light Sensors”), meniul derulant **Cap** oferă, de asemenea, opțiunea „None (bare sensor)” pe modelele DAQ-U și DAQ-M — pe aceste două modele, „fără capac” înseamnă pur și simplu că nu se aplică nicio corecție de capac, conform notelor de mai sus. Selectați această opțiune numai atunci când capacul este îndepărtat fizic.

## Declararea capacului — și de ce este importantă

**Codul declarat `cap_id` trebuie să corespundă cu capacul care se află fizic pe senzor.** Nici senzorul, nici software-ul nu pot detecta capacul montat. Declararea determină două lucruri:

1. **Corecția în timp real** aplicată fiecărui spectru.
2. **Codul capacului înregistrat în fiecare înregistrare `.daq`**, pe care se bazează procesarea ulterioară a reflectanței.

Capacul Sunshine atenuează cu aproximativ **12× conform proiectării**, astfel încât înregistrarea cu un capac declarat greșit duce la o scalare eronată a spectrelor cu aproximativ acest factor. Declarați imediat schimbările de capac.

### Setarea capacului

GUI: fila Senzori de lumină → pictograma roată dințată din rândul senzorului → meniul derulant **Capac**. Valoarea implicită pentru fiecare model este `sunshine_cosine`, o presupunere (modelele DAQ-U, DAQ-M și DAQ-E sunt livrate fără capac, iar o înregistrare indică faptul că s-a presupus prezența capacului până când alegeți unul), iar selecția se păstrează pentru proiect.



CLI<!-- SCREENSHOT-NEEDED: DAQ tab per-sensor settings modal (gear icon) scrolled to the Cap dropdown, open to show the per-model choices with "Sunshine (cosine corrector)" selected. Use a connected DAQ-E so the Hostname/Firmware/PTP rows are also visible above it. -->
(backend-ul trebuie să fie în funcțiune):

```bash
# Declare at connect time
chloros-cli daq pool-connect --eth-host daq-e-def330.local --cap-id sunshine_cosine

# Swap at runtime (after physically changing the cap)
chloros-cli daq pool-set-cap --sensor-id daq-e-def330 --cap-id fov_45
```

CLI
acceptă sintactic lista completă `cap_id` (`{none, fov_15, fov_30, fov_45, fov_60, fov_90, sunshine_cosine}`); fiecare profil este validat în raport cu modelul senzorului la conectare, astfel încât un ID de capac indisponibil (de exemplu, un ID specific doar pentru E pe un DAQ-U) generează o eroare clară, în loc să se aplice o corecție eronată. Valoarea implicită a backend-ului, atunci când nu se transmite nimic, este `sunshine_cosine`.

Python
SDK
: denumiți capul la momentul conectării cu `connect_daq_sensor(cap_id="sunshine_cosine")` sau schimbați-l în timpul rulării cu `DAQSensorSession.set_cap("fov_45")`; consultați [Referința SDK
](../reference/sdk-reference.md#daq-sensor-sessions).

Nivel avansat: profilurile sunt incluse în instalarea Chloros
la `daq/cap_profiles/<u|m|e>/<cap_id>.json` și pot fi suprascrise de fiecare utilizator la `~/.chloros/daq_cap_profiles/<u|m|e>/<cap_id>.json`.

Indiferent de limite, senzorii care nu au fost niciodată recalibrați primesc automat o mică ajustare a offsetului de întuneric derivată din flota de dispozitive — fără nicio acțiune din partea utilizatorului.

## DAQ-E-S: difuzor fix și înclinare

**DAQ-E-S** este un DAQ-E cu două diferențe importante pentru Chloros
. Toate celelalte funcții (descoperire, conectare, PTP, înregistrare, setări ale dispozitivului) funcționează exact ca la un DAQ-E.

* **Difuzorul său este permanent, deci nu necesită nicio corecție de capac.** Difuzorul se afla pe unitate atunci când a fost măsurată calibrarea din fabrică, astfel încât calibrarea indică deja iradianța reală descendentă. Chloros
nu îi aplică niciun profil de plafon, indiferent de solicitări: un plafon denumit pentru acesta este ignorat cu un mesaj, înregistrările sale indică faptul că plafonul provine din model, iar avertismentul privind plafonul presupus nu se declanșează niciodată pentru acesta. În Setări proiect → Senzor de lumină DAQ, rândul său nu afișează nicio limită, iar suprascrierea limitei la nivel de proiect nu îl afectează. Nu aplicați manual o limită datelor sale: corecția Sunshine aplicată ar indica o valoare de aproximativ 11 ori mai mare.
* **Este echipat cu un accelerometru pe 3 axe.** Răspunsul cosinus al difuzorului este valabil doar cât timp acesta este orizontal, astfel încât un senzor înclinat raportează o iradianță eronată fără alte simptome. Fiecare citire DAQ-E-S conține, prin urmare, înclinarea, rularea și tangajul său, iar un fișier `.daq` înregistrat de la unul dintre acestea le stochează alături de fiecare spectru (`imu_tilt_deg`, `imu_roll_deg`, `imu_pitch_deg`, …), astfel încât să puteți distinge un nor de o montură lovită.

Pentru fiecare câmp de atitudine și modul în care este raportată o eroare în fluxul de atitudine, consultați [DAQ-E-S în Referința CLI
](../reference/cli-reference.md#daq-e-s--fixed-diffuser-and-tilt-sensor).

## Performanța capacului de protecție solară (configurația pentru exterior)

Valori pe care vă puteți baza pentru a elabora proceduri:

| Proprietate | Valoare |
| --- | --- |
| Câmp vizual | 180° emisferic |
| Eroare de răspuns cosinus | ≤ ±4 % până la un unghi de incidență de 60°; ≤ ±4,5 % până la 70° |
| Limită pentru soare jos | Nu se recomandă sub ~15° elevație solară |
| Atenuare | ~12× (conform proiectului) |
| Repetabilitatea remontării capacului | ≈ 1,5 % |
| Iradianța cantitativă | Media a **≥ 15 s** de citiri (caracteristică a instrumentului, nu un defect) |

Pentru orice valoare cantitativă a iradianței — inclusiv referințele de reflectanță — utilizați o medie a cel puțin 15 secunde de citiri, în loc de un singur cadru.

## Interval spectral calibrat

| Proprietate | Valoare |
| --- | --- |
| Eșantionare spectrală | 340–1010 nm în pași de 5 nm (135 de puncte) |
| Interval calibrat radiometric | **~374–974 nm** (impus de software) |

Senzorul raportează întreaga grilă de 340–1010 nm, dar câștigul radiometric trasabil la NIST acoperă intervalul ~374–974 nm. Chloros
**refuză divizarea pe baza reflectanței absolute** pentru orice bandă a camerei cu mai puțin de jumătate din ponderea sa spectrală în interiorul acestui interval, raportând motivul de omitere `dls-uncalibrated-band-<nm>` în loc să genereze un produs necalibrat. Dintre codurile de produs (SKU) ale camerelor disponibile pe piață, doar filtrul F988 se află în afara acestui interval; acesta utilizează în schimb fluxul de lucru cu panoul de reflectanță — consultați [Fluxuri de lucru cu reflectanță](reflectance.md).

Pentru modelele de senzori, transporturi și ID-urile senzorilor, consultați [Prezentarea generală DAQ](README.md). Pentru modul în care se consumă ștampila de capac în timpul procesării, consultați [Înregistrarea și formatul .daq](recording.md).
