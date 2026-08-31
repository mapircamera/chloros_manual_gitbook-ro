# Profiluri ale capacelor și intervalul calibrat

> Capacele în sine — ce capac se livrează cu ce senzor, modul de montare al acestora și comportamentul lor optic — sunt documentate în **[manualul de utilizare DAQ](https://mapir.gitbook.io/daq)**. Această pagină tratează *declararea* capacului montat către Chloros, ceea ce asigură corectitudinea corecției.

Calibrarea radiometrică din fabrică a fiecărui senzor de lumină DAQ descrie senzorul *gol*. Capacul fizic montat peste difuzor modifică lumina pe care o colectează senzorul, astfel încât Chloros aplică un **profil de corecție a capacului** măsurat din fabrică peste pachetul de calibrare. Declararea capacului corect face parte din procesul de obținere a datelor calibrate — această pagină prezintă ce capace există pentru fiecare model, cum se declară acestea și care este de fapt domeniul spectral calibrat al senzorului.

## Disponibilitatea capacelor în funcție de model

| Profilul capacului (`cap_id`) | Capac fizic | DAQ-U | DAQ-M | DAQ-E |
| --- | --- | --- | --- | --- |
| `sunshine_cosine` | Capac de corecție cosinus pentru lumina solară (**implicit pe fiecare model**) | Da | Da | Da |
| `fov_15` / `fov_45` / `fov_90` | Conuri de restrângere a câmpului vizual (15° / 45° / 90°) | Da | — | Da |
| `fov_30` / `fov_60` | Conuri de limitare a câmpului vizual (30° / 60°) | Da | — | — |
| `none` | Fără capac montat | — | — | Da |

Note specifice modelului:

* **DAQ-M are un singur profil de capac: `sunshine_cosine`.** „Bare-plus-Sunshine-cap” este denumirea sa comercială, iar un DAQ-M fără capac nu necesită niciun profil geometric.
* **Un DAQ-U fără capac este cu adevărat „bare”** — nu necesită deloc un profil geometric, motiv pentru care nu există un profil `none` pentru acesta.
* **`none` pe un DAQ-E NU este o operație inactivă (no-op).** Difuzorul încastrat și acoperit cu sticlă al modelului DAQ-E are o corecție geometrică reală proprie, astfel încât „fără capac” este în sine un profil măsurat pe acest model.
* Un **DAQ-E fără capac nu poate măsura lumina directă a soarelui la nicio altitudine** — capacul Sunshine reprezintă configurația de teren. Nu planificați lucrări în aer liber cu un DAQ-E fără capac.

În setările per senzor din interfața grafică (pictograma roată din fila Senzori de lumină), meniul derulant **Cap** oferă, de asemenea, opțiunea „Niciunul (senzor fără capac)” pe modelele DAQ-U și DAQ-M — pe aceste două modele, „fără capac” înseamnă pur și simplu că nu se aplică nicio corecție de capac, conform notelor de mai sus. Selectați această opțiune numai atunci când capacul este îndepărtat fizic.

## Declararea capacului — și de ce este importantă

**Codul declarat `cap_id` trebuie să corespundă cu capacul care se află fizic pe senzor.** Nici senzorul, nici software-ul nu pot detecta capacul montat. Declararea determină două lucruri:

1. **Corecția în timp real** aplicată fiecărui spectru.
2. **Codul capacului înregistrat în fiecare înregistrare `.daq`**, pe care se bazează procesarea ulterioară a reflectanței.

Capacul Sunshine atenuează cu aproximativ **12× conform proiectării**, astfel încât înregistrarea cu un capac declarat greșit duce la o scalare eronată a spectrelor cu aproximativ acest factor. Declarați imediat schimbările de capac.

### Setarea capacului

GUI: fila Senzori de lumină → pictograma roată dințată din rândul senzorului → meniul derulant **Capac**. Valoarea implicită pentru fiecare model este `sunshine_cosine` (toți senzorii DAQ sunt livrați cu corectorul cosinus instalat), iar selecția se păstrează odată cu proiectul.

<!-- SCREENSHOT-NEEDED: DAQ tab per-sensor settings modal (gear icon) scrolled to the Cap dropdown, open to show the per-model choices with "Sunshine (cosine corrector)" selected. Use a connected DAQ-E so the Hostname/Firmware/PTP rows are also visible above it. -->

CLI (backend-ul trebuie să fie în funcțiune):

```bash
# Declare at connect time
chloros-cli daq pool-connect --eth-host daq-e-def330.local --cap-id sunshine_cosine

# Swap at runtime (after physically changing the cap)
chloros-cli daq pool-set-cap --sensor-id daq-e-def330 --cap-id fov_45
```

CLI acceptă sintactic întreaga listă `cap_id` (`{none, fov_15, fov_30, fov_45, fov_60, fov_90, sunshine_cosine}`); fiecare profil este validat în raport cu modelul senzorului la conectare, astfel încât un ID de capac indisponibil (de exemplu, un ID exclusiv E pe un DAQ-U) generează o eroare clară, în loc să se corecteze incorect. Valoarea implicită a backend-ului, atunci când nu se transmite nimic, este `sunshine_cosine`.

Python SDK notă: `cap_id` **nu** este un buton SDK — `connect_daq_sensor()` / `DAQSensorSession` nu expun niciun parametru de capac. Selectați limita prin comenzile CLI de mai sus sau din meniul derulant al interfeței grafice; consultați [Referința SDK](../reference/sdk-reference.md).

Avansat: profilurile sunt incluse în instalarea Chloros la `daq/cap_profiles/<u|m|e>/<cap_id>.json` și pot fi suprascrise de fiecare utilizator la `~/.chloros/daq_cap_profiles/<u|m|e>/<cap_id>.json`.

Separat de limitele de expunere, senzorii care nu au fost niciodată recalibrați primesc automat o mică ajustare a offsetului de întuneric derivată din flota de vehicule — fără a fi necesară nicio acțiune din partea utilizatorului.

## Performanța limitei de expunere la soare (configurația pentru exterior)

Cifre pe care vă puteți baza pentru a elabora proceduri:

| Proprietate | Valoare |
| --- | --- |
| Câmp vizual | 180° emisferic |
| Eroare de răspuns cosinus | ≤ ±4 % până la 60° incidență; ≤ ±4,5 % până la 70° |
| Limită pentru soare jos | Nu se recomandă sub ~15° elevație solară |
| Atenuare | ~12× (conform proiectului) |
| Repetabilitate la remontarea capacului | ≈ 1,5 % |
| Iradianța cantitativă | Media citirilor efectuate pe **≥ 15 s** (caracteristică a instrumentului, nu un defect) |

Pentru orice valoare cantitativă a iradianței — inclusiv referințele de reflectanță — utilizați media citirilor efectuate pe o perioadă de cel puțin 15 secunde, în loc de o singură imagine.

## Domeniu spectral calibrat

| Proprietate | Valoare |
| --- | --- |
| Eșantionare spectrală | 340–1010 nm în pași de 5 nm (135 de puncte) |
| Interval calibrat radiometric | **~374–974 nm** (impus de software) |

Senzorul raportează grila completă de 340–1010 nm, dar câștigul radiometric trasabil la NIST acoperă intervalul ~374–974 nm. Chloros **refuză împărțirea în funcție de reflectanța absolută** pentru orice bandă a camerei cu mai puțin de jumătate din ponderea sa spectrală în interiorul acestui interval, raportând motivul de omitere `dls-uncalibrated-band-<nm>` în loc să genereze un produs necalibrat. Dintre codurile de produs (SKU) ale camerelor disponibile în comerț, doar filtrul F988 se află în afara acestui interval; acesta utilizează în schimb fluxul de lucru cu panoul de reflectanță — consultați [Fluxuri de lucru cu reflectanță](reflectance.md).

Pentru modelele de senzori, transporturi și ID-urile senzorilor, consultați [Prezentarea generală DAQ](README.md). Pentru modul în care se consumă ștampila de capac în timpul procesării, consultați [Înregistrarea și formatul .daq](recording.md).
